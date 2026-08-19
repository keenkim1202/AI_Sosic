---
title: Prompt Caching In Agents, 코딩 에이전트의 프롬프트 캐시
source: https://earendil.com/posts/prompt-caching/
author: Earendil Engineering
published: 2026-07-22
collected: 2026-08-10
tags: [prompt-caching, kv-cache, coding-agent, llm-cost, inference]
---

출처: [Prompt Caching In Agents, EARENDIL](https://earendil.com/posts/prompt-caching/)

## 요약

코딩 에이전트는 매 턴 이전 컨텍스트를 거의 통째로 다시 보냄. 그래서 프롬프트 캐시가 비용이랑 지연시간을 다 쥐고 있는데, 이 캐시가 토큰 prefix 단위라 앞쪽 토큰 하나만 바뀌어도 뒤가 전부 무효가 됨. 캐시를 구현 디테일로 취급하면 안 되는 이유가 이거임.

## KV 캐시가 뭔지

트랜스포머는 입력 읽고 attention state 계산하는 prefill, 토큰 하나씩 만드는 decode 두 단계로 돔. 각 토큰이 만든 key/value를 들고 있으면 다음 토큰이 재계산 없이 그걸 참조함. 이게 KV 캐시. 프롬프트 캐싱은 이 상태 수명을 한 번의 생성 이후까지 늘리는 것.

```
request 1: [system][tools][user][assistant][tool result][user]
           <------------------ prefill ------------------>

request 2: [system][tools][user][assistant][tool result][user][new]
           <-------------- reusable prefix -------------><---->
                                                          새 작업만
```

의미가 같아도 토크나이즈 결과가 다르면 캐시 공유 안 됨. 크기는 의외로 작아서, 여러 기법 쓰면 긴 대화도 수 GB 수준까지 줄어듦.

## 캐시를 어디에 두느냐

| | 세션 어피니티 | 분산 캐시 |
|---|---|---|
| 동작 | 계산한 GPU 근처에 두고 같은 워커로 라우팅 | KV 블록을 다른 메모리 티어나 워커 간 공유 |
| 장점 | 추가 인프라 거의 안 들고 빠름 | 스케줄링 유연하고 복구 쉬움 |
| 단점 | 워커 과부하·재시작·eviction에 취약 | 블록 이동·인덱싱·보관 자체가 시스템 문제 |

## 툴 로드아웃이 캐시를 깨는 이유

여기가 실무에서 제일 중요함. 툴 정의는 대화보다 앞에 오고, 내부적으로 시스템 프롬프트에 접혀 들어감. 그래서 툴을 추가하든 제거하든 스키마를 바꾸든, 심지어 **순서만 바꿔도** 불일치 지점이 프롬프트 맨 앞으로 튐.

```
turn 1: [system][read][write][bash][conversation.........]
turn 2: [system][read][write][bash][deploy][conversation.]
                                   ↑ 여기부터 뒤 전부 무효
```

"필요할 때만 툴 로드한다"는 최적화가 여기서 역효과 남. 스키마 몇백 토큰 아끼려다 수만 토큰짜리 대화를 통째로 재처리하게 됨. MCP나 플러그인 카탈로그 쓰면 자주 밟는 지뢰.

해법은 additive tool loading. 툴 목록에 끼워 넣는 대신 트랜스크립트 중간의 tool result 지점에서 활성화시켜서 기존 prefix를 보존함. Anthropic은 deferred definitions랑 `tool_reference`로, OpenAI는 tool-search 아이템으로 지원.

단어 그대로 **"추가"일 때만** 통함. 제거, 교체, 재정렬은 그대로 캐시를 깸. 타임스탬프 주입하거나 시스템 프롬프트 매번 재구성하는 익스텐션도 똑같이 파괴적임.

## 5분 TTL이라는 함정

Anthropic 기본 캐시 수명이 5분임. 일반적인 코딩 활동보다 짧음. 사용자는 계속 작업 중이라고 느끼는데, 프로바이더 입장에선 그냥 고립된 요청의 나열임.

```
model request → run tests for 7 minutes → model request
                (캐시 트래픽 없음)
```

긴 빌드, 테스트, 점심, 미팅, diff 검토만으로 캐시가 만료됨. 프롬프트는 똑같은데 만료된 prefix가 정상 input 가격으로 다시 처리되고, 캐시 재작성 비용도 붙을 수 있음. Claude Code는 자체 구독 사용자한테 1시간으로 늘려 쓰는데, API 토큰 가격 내는 입장에선 수지가 안 맞는 경우가 많음.

## 미스가 얼마나 비싼가

과금은 uncached input, cache write, cache read 셋으로 나뉨. read는 할인, write는 프리미엄 붙는 게 일반적.

10만 토큰 세션으로 보면, 히트했을 땐 대부분이 낮은 read 가격으로 처리되고 새 내용만 정상 가격이 붙음. 미스하면 10만 토큰 전체를 정상 input 가격으로 재처리하고 캐시 재작성 비용까지 붙음. 캐시 만료 후 `continue` 한 마디가 놀랍도록 비싼 이유가 이거임.

인센티브가 어긋나는 지점도 있음. 사용자랑 GPU 사업자는 둘 다 히트를 원함. 비용 줄고 처리량 늘어나니까. 반면 uncached 요율로 매출 얻는 게이트웨이나 리셀러한테는 미스가 더 큰 청구서가 됨. 프로바이더가 일부러 방해한다는 얘긴 아니지만, 캐시 성능이 관측 가능해야 하는 이유는 됨.

## 가지치기의 역설

중간 내용을 지우면 그 지점부터 prefix가 바뀌고, 살아남은 대화 전체를 다시 처리해야 함. 재작성 즉시 비용이 절약분을 넘는 경우가 많음.

```
1회 재작성 비용 ≈ 편집 후 남은 토큰 × (uncached 가격 - cache-read 가격)
턴당 미래 절약  ≈ 제거한 토큰 × cache-read 가격
```

회계 문제만도 아님. 오래된 tool result에는 모델이 이후 결정을 내릴 때 근거로 쓴 정보가 들어 있음. 그래서 원문 저자들은 안정적이고 append 지향적인 트랜스크립트를 선호하고, compaction은 캐시 실패가 아니라 **의도된 리셋**으로 취급함.

반대 경우도 있음. 캐시 할인이 없거나 높은 히트율이 애초에 불가능한 프로바이더라면 pruning이 나을 수 있음.

## 히트율 낮을 때 확인할 8가지

1. 유휴. 명령 실행이나 리뷰, 대화 중단이 보관 윈도우를 넘김
2. 모델·프로바이더 전환. KV 상태는 모델 고유라 안 옮겨짐
3. 브랜치 이동. rewind나 fork는 세션 ID가 같아도 토큰 시퀀스를 바꿈
4. compaction이나 수동 히스토리 재작성
5. 툴 변경, 그리고 **reasoning level(effort) 변경**. 후자도 같은 효과 냄
6. 동적 시스템 프롬프트. 타임스탬프, 랜덤값, 변하는 프로젝트 컨텍스트
7. 오래된 메시지나 payload를 수정하는 익스텐션
8. 프로바이더 라우팅과 eviction. 프롬프트가 동일해도 KV 블록이 그곳에 없으면 미스

## 실무 메모

- MCP 서버를 동적으로 붙였다 뗐다 하지 말 것. 세션 시작 시 고정이 대개 이득임
- 시스템 프롬프트에 현재 시각이나 랜덤 ID 넣지 말 것. 매 턴 전부 날아감
- 긴 빌드·테스트 전후로는 캐시 만료를 의식할 것
- 비용이 이상하면 히트율부터 봄. 원인 대부분이 위 8가지 안에 있음
- **prefix 일치라는 원리는 넓게 적용됨.** 다만 TTL, 세션 어피니티, `cache_control`, additive tool loading, 과금 방식은 에이전트·모델·게이트웨이마다 다름. 원문은 Pi 기준이니 자기 하네스에서 다시 확인할 것

## 관련 문서

- [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md): effort 바꾸면 캐시 깨지는 문제와 연결됨
