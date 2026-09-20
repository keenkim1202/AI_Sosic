---
title: 서버측 compaction, 히스토리를 줄이는 일이 API 기능이 됨
source: https://platform.claude.com/docs/en/build-with-claude/compaction
author: Anthropic
collected: 2026-09-20
tags: [claude, compaction, prompt-caching, context-engineering, agent-memory, llm-cost]
---

출처: [Compaction, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/compaction) · [Context editing](https://platform.claude.com/docs/en/build-with-claude/context-editing) · [Claude Platform release notes](https://platform.claude.com/docs/en/release-notes/overview) (2026-09-20 확인)

## 요약

긴 대화를 줄이는 일을 클라이언트가 하지 않고 서버가 하는 경로임. 이 저장소에 걸리는 지점이 셋임. 첫째, [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 "가지치기의 역설"로 정리한 재작성 비용이 **요약 하나 길이로 상한이 잡힘**. 시스템 프롬프트 끝에 캐시 중단점을 두면 compaction이 일어나도 그쪽 캐시는 살아 있고 새로 써야 하는 건 요약뿐임. 둘째, **`usage.input_tokens`가 compaction 몫을 포함하지 않음.** 총액을 보려면 `usage.iterations` 배열을 전부 더해야 하고, 안 고치면 비용 대시보드가 조용히 과소 보고함. 셋째, 릴리스 노트 기준 **2026-09-14에 on-demand 모드가 붙었음**(베타 헤더 `compact-2026-09-04`). 요약 요청을 대화 턴과 분리해서 **백그라운드로 돌리고** 최근 턴은 원문 그대로 남김.

그리고 이건 이제 선택지가 줄어든 자리임. [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)에서 앞 턴을 직접 편집하면 400이 나는데, **서버측 compaction은 그 검사를 통과하는 경로로 문서가 명시한 쪽**임.

## 두 가지 모드

| 모드 | 베타 헤더 | 언제 도는가 |
|---|---|---|
| threshold (기본) | `compact-2026-01-12` | input 토큰이 임계값에 닿으면 응답 도중에 |
| on-demand | `compact-2026-09-04` | 내가 요청할 때. 대화 턴과 별도 요청 |

**threshold 모드**는 `context_management.edits`에 `{"type": "compact_20260112"}`를 넣어 켬. 동작 순서는 임계값 감지, 요약 생성, `compaction` 블록 생성, 그 컨텍스트로 응답 계속임. 다음 요청부터 API가 **`compaction` 블록보다 앞에 있는 콘텐츠 블록을 전부 떨어뜨림**. 그래서 응답을 통째로 `messages`에 붙이는 것이 가장 단순한 처리고, 문서도 그렇게 적음.

> "You must pass the `compaction` block back to the API on subsequent requests to continue the conversation with the shortened prompt."

파라미터는 넷임.

| 파라미터 | 기본값 | 내용 |
|---|---|---|
| `trigger` | `input_tokens` **150,000** | 지원하는 타입은 `input_tokens` 하나. 값은 **최소 50,000** |
| `pause_after_compaction` | `false` | `true`면 요약만 만들고 `stop_reason: "compaction"`으로 멈춤 |
| `instructions` | `null` | 요약 프롬프트. **기본 프롬프트를 보완하는 게 아니라 통째로 교체함** |
| `type` | 없음(필수) | `"compact_20260112"` |

`pause_after_compaction`이 실무에서 제일 쓸모 있음. 요약이 나온 뒤 **블록을 더 끼워 넣고** 이어갈 수 있어서, 최근 몇 턴은 요약하지 말고 원문으로 다시 붙이는 식의 정책을 클라이언트가 정할 수 있음.

`instructions`는 반대로 함정임. 커스텀을 주면 기본 프롬프트가 사라지는데, 기본 프롬프트에는 요약을 `<summary></summary>`로 감싸라는 지시가 들어 있음. 포맷까지 내가 다시 지정해야 한다는 뜻임.

**on-demand 모드**는 요약을 대화에서 떼어냄. 릴리스 노트가 적은 순서가 이럼. 최상위 `compaction` 파라미터를 보내면 **서명된 `compaction` 블록**이 오고, 이후 요청에서 그 블록을 원래 메시지들 자리에 먼저 실어 보냄. 문서 개요가 이렇게 적음.

> "That request is separate from your conversation turns and returns only the summary, so it can run in the background."

[Memory Engineer](2026-08-02-memory-engineer.md)의 Nvidia 렌즈가 내린 처방이 정확히 이것임. 구축은 길게 읽고 짧게 쓰는 prefill 덩어리라 **지연시간에 민감한 경로에서 떼어놓으라**는 것. 그 글이 백그라운드 인덱싱 잡으로 다루라고 한 작업을 API가 모드로 만들어 준 셈임.

## 캐시와 어떻게 만나는가

이 문서를 쓰는 이유가 여기임. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 산술은 이랬음. 중간을 지우면 그 지점부터 prefix가 바뀌고 **살아남은 대화 전체**를 다시 처리해야 해서, 재작성 비용이 절약분을 넘는 경우가 많다는 것.

문서가 주는 배치는 그 "전체"를 쪼갬.

- `compaction` 블록에 `cache_control`을 붙임
- **시스템 프롬프트 끝에 캐시 중단점을 따로 둠**

이렇게 하면 compaction이 일어나도 시스템 프롬프트 쪽 캐시는 유효하게 남아 읽기로 처리되고, 새로 써야 하는 건 요약 블록뿐임. 즉 재작성 비용이 대화 길이가 아니라 **요약 길이**에 걸림. 가지치기의 역설이 사라지는 게 아니라 **상한이 생기는 것**으로 읽어야 함.

대신 공짜가 아님. compaction은 요약을 만들기 위한 **별도의 샘플링 단계**를 돌리고, 그 단계의 input은 줄이기 전의 대화 전체임. 아래 과금 항목이 그 얘기임.

## 과금 관측이 조용히 틀어지는 지점

`usage`의 모양이 바뀜. 문서가 직접 경고함.

> "The top-level `input_tokens` and `output_tokens` do not include compaction iteration usage."

```json
"usage": {
  "input_tokens": 23000,
  "output_tokens": 1000,
  "iterations": [
    { "type": "compaction", "input_tokens": 180000, "output_tokens": 3500 },
    { "type": "message",    "input_tokens": 23000,  "output_tokens": 1000 }
  ]
}
```

문서가 든 예시에서 최상위 `input_tokens`는 **23,000**인데 실제로 청구되는 건 compaction 쪽 **180,000**까지 더한 값임. 총액을 보려면 `iterations`를 전부 더해야 함.

[대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md)의 레버 3이 "가시성 대시보드부터"였는데, 그 대시보드가 `usage.input_tokens`를 읽고 있으면 **compaction을 켜는 순간 조용히 과소 보고**로 바뀜. 지출이 줄어든 것처럼 보이는 계기판은 아무것도 없는 것보다 나쁨. 켤 때 같이 고쳐야 하는 코드가 이것임.

토큰 카운팅 쪽 단서도 하나 있음. `/v1/messages/count_tokens`는 **이미 있는 `compaction` 블록은 반영하지만 새 compaction을 유발하지는 않음.** 줄이기 전 값은 `context_management.original_input_tokens`로 따로 봄.

## 무엇이 사라지는가

[Memory Engineer](2026-08-02-memory-engineer.md)가 Memento 연구에서 뽑은 문장이 여기 그대로 걸림. 지워진 추론은 완전히 사라지지 않고 모델 안에 그림자가 남지만, **노트만으로 컨텍스트를 재구성하면 정확도를 잃는다**는 것. 서버측 compaction은 그 재구성을 API가 대신 해 주는 것이지, 손실이 없다는 뜻이 아님.

Fable 5.1과 Mythos 5.1에 붙는 단서가 둘 있음.

- **`compaction` 블록 앞의 thinking 블록은 넘어오지 않음**
- 커스텀 `instructions`를 준 요청은 **보이는 대화만 요약 입력으로 씀.** 앞선 thinking 블록은 요약기의 입력에 들어가지 않음

두 번째가 덜 알려진 쪽임. 요약 프롬프트를 내 손으로 쓰는 순간 요약기가 보는 재료가 줄어듦. 기본 프롬프트를 바꿀 이유가 정말 있는지 먼저 재는 편이 나음.

릴리스 노트는 on-demand 에서 **요약 뒤에 남긴 턴의 thinking 은 유효하게 유지된다**고 적음. 위 항목과 어긋나 보이지만 **어긋나지 않음.** 갈리는 기준이 모드가 아니라 `compaction` 블록을 기준으로 한 **위치**임. 블록보다 앞이면 넘어오지 않고, 요약 뒤에 원문으로 남긴 턴이면 유효함.

## 언제 안 쓰는 게 맞나

[Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) 문서가 이미 답을 하나 갖고 있음. 그 모델은 캐시 읽기가 기본 input 가격의 **0.025배**라 긴 prefix를 그냥 들고 가는 게 싸짐. Anthropic 자신이 이렇게 적음.

> "Because cache reads are now cheaper, compacting early to save cost may no longer be the right cost-intelligence tradeoff on Claude Fable 5.1, so experiment with later compaction points."

즉 **비용만 보고 임계값을 낮게 잡는 것은 모델에 따라 손해**임. 기본값이 150,000인 것도 그 방향임. 줄일 이유가 비용이 아니라 품질(대화가 길어지면 응답이 나빠짐)이면 얘기가 다르고, 문서가 내세우는 근거도 그쪽에 가까움.

정리하면 셋으로 갈림.

1. **컨텍스트 창을 넘길 게 확실한 장시간 작업**: 켜는 게 맞음. 대안이 없어서가 아니라, 아래 갈라 쓰기 절에서 보듯 문서가 이 경우의 1차 전략으로 지목한 쪽이기 때문임. 베타라 못 쓰거나 플랫폼·모델이 안 받으면 클라이언트측 compaction(`tool_runner`)과 context editing 이 남음
2. **비용만이 목적이고 모델의 캐시 읽기가 싼 경우**: 임계값을 늦게 잡고 재 보는 것부터
3. **툴 결과가 대부분인 에이전트 루프**: compaction이 아니라 context editing 쪽이 맞을 수 있음

## context editing과 갈라 쓰기

같은 `context_management` 아래 있지만 다른 물건임. 헤더도 `context-management-2025-06-27`로 따로임.

| 전략 | 무엇을 지우나 | 캐시에 미치는 영향 |
|---|---|---|
| `clear_tool_uses_20250919` | 오래된 tool result(옵션으로 tool input까지). 기본 트리거 100,000, 기본 `keep` 3개 | 지우는 지점에서 **prefix 무효화**. `clear_at_least`로 재사용 구간을 벌어 줌 |
| `clear_thinking_20251015` | thinking 블록. `keep`에 숫자나 `"all"` | **유지하면 캐시가 보존되고**, 지우면 그 지점에서 무효화 |
| 서버측 compaction | 요약 앞의 모든 블록 | 요약 블록만 새로 write. 시스템 프롬프트는 중단점을 따로 두면 살아남음 |

문서의 권고 자체는 분명함. 장시간 대화와 에이전틱 워크플로의 **1차 전략은 서버측 compaction**이고, context editing은 무엇을 지울지 더 세밀하게 잡아야 할 때 쓰는 것임. SDK의 클라이언트측 compaction(`tool_runner`)도 남아 있지만 서버측을 권함.

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 히트율 점검 8가지 중 4번이 "compaction이나 수동 히스토리 재작성"이었는데, 이제 그 줄을 둘로 갈라야 함. **수동 재작성은 Fable 5.1에서 에러가 되고, compaction은 캐시 배치가 딸린 지원 경로임.**

## 지원 범위

- 모델: `claude-fable-5-1`, `claude-mythos-5-1`, `claude-fable-5`, `claude-mythos-5`, `claude-mythos-preview`, `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7`, `claude-opus-4-6`, `claude-sonnet-5`, `claude-sonnet-4-6`
- 플랫폼: Claude API, Claude Platform on AWS, Amazon Bedrock, Google Cloud, Microsoft Foundry (전부 베타)
- 스트리밍: `compaction` 블록은 중간 스트리밍 없이 **완성된 요약이 델타 하나로** 옴. 진행률을 보여주려면 이 블록에 기대면 안 됨

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | "가지치기의 역설"의 재작성 비용. 캐시 중단점 배치로 상한이 생기는 것이 이 문서의 추가분 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 앞 턴 편집이 400이 되는 검사. 서버측 compaction이 통과 경로로 명시된 쪽이고, 캐시 읽기 0.025배가 임계값 판단을 바꿈 |
| [Memory Engineer](2026-08-02-memory-engineer.md) | 의도적 망각과 구축을 백그라운드로 떼어놓는 처방. on-demand 모드가 그 형태임 |
| [대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md) | 레버 4의 "compaction을 더 자주". 레버 3의 가시성 대시보드가 `usage.iterations`로 깨짐 |
| [Claude Cowork](2026-08-10-claude-cowork.md) | "대화를 새로 시작하는 게 언제 손해인가"를 캐시 배수로 따진 표. 서버가 대신 해 주는 선택지가 하나 늘어남 |

## 짚어야 할 것

- **on-demand 절의 본문을 확보하지 못했음.** 문서 페이지를 받으면 해당 절이 잘려 나옴. 릴리스 노트 항목이 흐름까지는 적어 둠. 최상위 `compaction` 파라미터를 보내면 보낸 메시지들을 요약한 **서명된 `compaction` 블록**이 오고, 이후 요청에서 그 블록을 그 메시지들 자리에 먼저 보냄. **그래도 파라미터와 서명 블록의 필드 이름, 블록을 고치거나 순서를 바꿨을 때의 동작은 확인하지 못했음.** 붙이기 전에 API 레퍼런스로 직접 볼 것
- **thinking 블록 서술 둘은 모순이 아니라 위치 얘기임.** 페이지는 `compaction` 블록 **앞**의 thinking이 안 넘어온다고 적고, 릴리스 노트는 요약 뒤에 **남긴 턴**의 thinking이 유효하다고 적음. 블록을 기준으로 앞이냐 뒤냐가 갈리는 것이지 모드가 갈리는 것이 아님. 다만 **커스텀 `instructions`를 주면 또 달라짐.** 그때는 보이는 대화만 요약 입력이 되고 앞선 thinking은 요약기에 들어가지 않음
- **요약 품질에 대한 수치는 어디에도 없음.** 이 기능이 정확도를 얼마나 깎는지 Anthropic이 낸 측정값을 찾지 못했음. [Memory Engineer](2026-08-02-memory-engineer.md)가 인용한 15포인트는 Memento 논문 쪽 수치지 이 기능의 수치가 아님
- **베타임.** 헤더 이름에 날짜가 박혀 있고 모드가 늘어나는 중이라, 정식 승격 때 파라미터 이름이 바뀔 여지가 있음
- 가격 배수는 이 문서에서 다시 계산하지 않았음. 모델마다 캐시 읽기 배수가 다르므로 임계값 판단은 자기 모델 기준으로 재야 함

## 유효기간

**2026-09-20 확인 기준**임. threshold 모드는 `compact-2026-01-12`, on-demand 모드는 `compact-2026-09-04`로 베타 헤더가 둘이고 후자가 2026-09-14 항목이라 아직 움직이는 중임. 다시 볼 때는 [릴리스 노트](https://platform.claude.com/docs/en/release-notes/overview)에서 compaction 항목이 더 붙었는지부터 보고, 확인할 것 둘. **on-demand가 베타를 벗어났는지**, 그리고 **`usage.iterations` 집계가 기본 동작으로 바뀌었는지**. 후자는 비용 계측 코드를 다시 건드려야 하는 변경임.
