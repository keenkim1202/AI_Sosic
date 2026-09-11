---
title: Claude Fable 5.1, 캐시 읽기가 4분의 1로 내려가고 히스토리 편집이 에러가 됨
source: https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1
author: Anthropic
published: 2026-09-01
collected: 2026-09-11
tags: [claude, fable-5-1, prompt-caching, effort, thinking, migration]
---

출처: [What's new in Claude Fable 5.1](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1) · [Effort](https://platform.claude.com/docs/en/build-with-claude/effort) · [Claude Platform 릴리스 노트](https://platform.claude.com/docs/en/release-notes/overview) (2026-09-11 확인)

## 요약

Anthropic이 2026-09-01에 **Claude Fable 5.1**(`claude-fable-5-1`)과 Project Glasswing 참가자용 **Mythos 5.1**을 냈음. 능력 향상보다 이 저장소에 중요한 건 **프롬프트 캐시 규칙이 통째로 다시 그어졌다는 것**임. 캐시 읽기가 기본 input의 **0.025배**로 내려갔고(다른 Claude 모델은 0.1배), **effort를 대화 중간에 바꿔도 캐시가 살아남는** 경로가 beta로 생겼고, 한 턴만 사는 시스템 메시지가 생겨서 매 턴 주입했다 지우는 리마인더가 더는 캐시를 깨지 않음. 대신 반대 방향으로 못이 하나 박혔음. **히스토리를 편집하면 400 에러가 남.** [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)에서 "append 지향 트랜스크립트가 유리하다"고 정리해둔 것이 이제 취향이나 비용 문제가 아니라 API가 강제하는 규칙임.

## 기본 정보

| | |
|---|---|
| 모델 ID | `claude-fable-5-1`, `claude-mythos-5-1` |
| 발표 | 2026-09-01 |
| 컨텍스트 | **1M 토큰**이 기본값이자 최대값. 전 구간 표준 단가 |
| 최대 출력 | 128k 토큰 |
| thinking | adaptive 상시 ON. `enabled`와 `disabled` 둘 다 400 |
| 토크나이저 | Fable 5와 동일(Opus 4.7에서 도입). 4.7 이전 모델 대비 같은 텍스트가 **약 30% 더 많은 토큰** |
| 최소 캐시 길이 | 512 토큰 (변화 없음) |
| 문서가 권하는 자리 | **"대부분의 워크로드는 Opus 5로 시작"**. Fable 5.1은 고난도 추론과 장시간 에이전틱 작업용 |

마지막 줄이 중요함. 번호가 커졌다고 상위 호환이 아님. 원문이 대는 이동 기준은 **Opus 5를 높은 effort로 돌려도 자기 eval이 모자랄 때**임.

## 캐시 읽기가 0.025배로 내려감

원문 표임.

| 기본 input | 5분 캐시 쓰기 | 1시간 캐시 쓰기 | 캐시 읽기 | 출력 |
|---|---|---|---|---|
| $10 / MTok | $12.50 / MTok | $20 / MTok | **$0.25 / MTok** | $50 / MTok |

> "Cache reads (hits and refreshes) cost 0.025 times the base input price on these models, compared with 0.1 on other Claude models."

배수로 옮기면 읽기 **0.025배**, 5분 쓰기 **1.25배**, 1시간 쓰기 **2배**임. 쓰기는 그대로고 읽기만 4분의 1이 됨.

그래서 [Claude Cowork](../practices/2026-08-10-claude-cowork.md)에 적어둔 "이어붙일까 truncate할까" 계산을 다시 돌려야 함. ⚠️ 아래 표는 **문서에 적힌 배수로 이 저장소가 직접 계산한 것**이고 원문에 있는 수치가 아님. 조건은 그쪽과 같게 prefix 5k, suffix 25k, 앞으로 한 턴으로 잡았음.

| 선택 | 읽기 0.1배 (기존 모델) | 읽기 0.025배 (Fable 5.1) |
|---|---|---|
| 이어붙이기 | 30k × 0.1 = 3k | 30k × 0.025 = **0.75k** |
| truncate, 되돌릴 지점 캐시 히트 | 5k × 0.1 = 0.5k | 5k × 0.025 = **0.125k** |
| truncate, 그 지점 캐시 없음 | 5k × 1.25 = 6.25k | 5k × 1.25 = **6.25k** |

결론의 방향은 안 바뀜. 캐시가 살아 있으면 truncate가 싸다는 건 그대로임. 바뀐 건 **판돈의 기울기**임. 이어붙이기의 비용이 0에 가까워지는데 재작성 비용은 그대로라, **캐시 없는 지점으로 되돌리는 실수의 상대적 대가가 훨씬 커짐**. 기존 모델에서 2배 남짓이던 격차가 8배 넘게 벌어짐. 긴 세션에서 append-only는 이제 안전할 뿐 아니라 경제적으로도 더 확실한 기본값임.

## effort를 대화 중간에 바꿔도 캐시가 살아남음

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 히트율 체크리스트 5번이 "툴 변경, 그리고 **reasoning level(effort) 변경**"이었음. 그 항목이 이제 모델과 방식에 따라 갈림.

| 방식 | 캐시 |
|---|---|
| 다음 요청에서 top-level `output_config.effort`를 바꿈 | 프롬프트 렌더링이 바뀌므로 **캐시가 처음부터 다시 시작** |
| `role: "system"` 메시지에 `output_config.effort`만 실어 중간에 끼움 (beta) | 앞쪽이 안 바뀌므로 **캐시 prefix가 계속 맞음** |

두 번째가 per-message effort임. beta 헤더 `mid-conversation-output-config-2026-07-01`이 필요하고 **Fable 5.1, Mythos 5.1, Opus 5**에서 됨. Fable 5는 지원하지 않고 400을 돌려줌.

```json
{"role": "system", "content": [], "output_config": {"effort": "low"}}
```

`content`가 비어 있어서 mid-conversation 시스템 메시지의 배치 제약을 안 받음. `messages` 어디에든 넣을 수 있고, 새 레벨은 **다음 `user` 턴부터** 적용돼 다시 바꿀 때까지 유지됨.

문서가 top-level 변경보다 이쪽을 권하는 이유를 하나 더 댐. 캐시만이 아니라 **조종도 더 잘 먹힘**. 앞선 답변들이 예전 레벨에서 쓰였기 때문에 모델이 그쪽에 맞춰 일관되려는 경향이 있음.

**Opus 5에도 적용된다는 게 이 저장소 입장에서 특히 큼.** [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)에 붙여둔 "실서비스에서 effort를 동적으로 바꾸면 캐시 비용을 물게 됨"은 이제 **top-level로 바꿀 때만** 맞는 말임.

## 턴 한 번만 사는 시스템 메시지

같은 체크리스트 6번이 "동적 시스템 프롬프트. 타임스탬프, 랜덤값, 변하는 프로젝트 컨텍스트"였음. 여기에도 대응이 생김.

`role: "system"` 메시지에 `clear_at: "next_user_message"`를 달면 그 텍스트가 **이번 턴에만** 시스템 프롬프트 권한으로 렌더링되고, 뒤에 `user` 메시지가 생기는 순간 렌더링을 멈춤. 메시지 자체는 `messages`에 남아 계속 그대로 돌려보냄.

```json
{
  "role": "system",
  "clear_at": "next_user_message",
  "content": "Results have landed in your inbox. Check it before running more code."
}
```

핵심은 **앞쪽이 하나도 안 바뀐다**는 것임. 캐시가 계속 맞고, 뒤의 thinking 블록도 유효하게 남고, 렌더링이 끝난 메시지는 input 토큰을 먹지 않음. beta 헤더는 `mid-conversation-system-clear-at-2026-08-21`.

툴 루프에서 "이 툴 출력은 사용자에게 안 보임" 같은 리마인더를 매 턴 히스토리에 주입했다가 다음 요청에서 빼는 패턴이 흔한데, 그게 정확히 캐시를 깨는 행위였음. 이제 대체 경로가 생긴 것임.

## 히스토리 편집이 400이 됨

breaking change 쪽이고, 위 세 항목의 반대 급부로 읽어야 함.

Fable 5.1의 thinking 블록보다 **앞에** 있는 것을 고치면(시스템 프롬프트, `tools`, 앞선 메시지) 다음 요청이 에러가 남. 메시지는 `The block is bound to a different conversation`.

적용 범위에 단서가 둘 붙음. **Mythos 5.1은 이 검사를 하지 않음.** 그리고 강제 적용 대상은 **2026-08-31 이후 만들어진 계정**이고, 그 이전 계정은 불일치를 기록만 해두다가 요청이 `thinking.block_binding.prefix_mismatch_behavior`를 지정할 때만 작동함.

뒤의 모든 thinking 블록을 무효화하는 패턴.

- 뒤 턴을 남긴 채 앞 턴을 편집·재정렬·삭제
- 요청마다 앞 턴에 텍스트(리마인더, 상태 줄)를 주입했다가 다음 요청에서 빼기
- 같은 대화 안에서 top-level `system`이나 `tools` 배열을 다시 만들기
- 같은 URL이 나중 요청에 다른 바이트를 주는 이미지·문서. 검사가 URL이 아니라 **바이트**를 보므로 같은 파일을 가리키는 회전 서명 URL은 괜찮음

반대로 뒤 블록이 살아남는 것.

- thinking 블록을 **맨 앞부터 연속으로** 걷어내기. 중간에서 빼면 그 뒤가 전부 무효
- 서버측 context editing이나 compaction으로 히스토리 줄이기
- `cache_control` 마커 옮기기
- 요청 사이에 `effort` 바꾸기

Claude Code, claude.ai, Claude Managed Agents, Claude Agent SDK는 이 prefix를 알아서 지켜줌. 직접 `messages` 배열을 만드는 코드만 해당함.

이게 중요한 이유는 [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 "가지치기의 역설" 절이 **돈 얘기였는데 이제 에러 얘기가 됐다**는 것임. 중간을 지우면 비싸진다가 아니라, 중간을 지우면 요청이 거절됨. 그리고 같은 문서가 취했던 태도, 즉 **compaction은 캐시 실패가 아니라 의도된 리셋**이라는 관점이 그대로 정답이 됨. 서버측 compaction은 편집으로 안 치기 때문임.

## forced tool use가 사라짐

`tool_choice`의 `{"type": "any"}`와 `{"type": "tool", "name": "..."}`가 400을 돌려줌. `auto`(기본)와 `none`은 그대로임. token counting 엔드포인트에도 같은 검증이 걸림.

이유가 설명돼 있음. thinking이 상시 ON인데 강제 툴 호출은 그 단계를 건너뛰게 되고, 그러면 모델이 **풀이를 툴 인자 안에 써 넣어서 인자 품질이 떨어짐**. 스키마를 지키게 하려면 `auto`를 유지한 채 strict tool use를 켜거나 structured outputs로 옮기고, 툴을 부르게 하려면 프롬프트로 언제 쓰는지 적으라는 게 문서 권고임.

## 코드를 안 고쳐도 달라지는 것들

원문이 "Changed from Claude Fable 5"로 묶은 것 중 에이전트 루프에 바로 걸리는 것들임.

| 변화 | 실무에서 보이는 모습 |
|---|---|
| **병렬 툴 호출이 들쭉날쭉해짐** | Fable 5가 묶어 던지던 것을 턴당 하나씩 부를 수 있음. 답 품질은 안 떨어지는데 턴 수, 토큰, 벽시계 시간이 늚 |
| **진행 상황 텍스트가 줄어듦** | 특히 높은 effort에서 말수가 줄어 긴 턴이 조용해 보임 |
| **`low` effort에서 검색을 덜 부름** | 최저 레벨에서 기억으로 답하는 빈도가 올라감. 신선한 정보가 필요한 턴은 effort를 올릴 것 |
| **작은 수정에도 파일을 통째로 다시 씀** | 결과는 대개 같은데 출력 토큰과 시간이 더 듦 |
| 산문이 조밀해지고 채팅에서 서식을 덜 씀 | 예전 모델용 anti-formatting 규칙이 필요한 구조까지 눌러버릴 수 있음 |
| 요약할 때 원문 구절을 인용 표시 없이 옮기는 빈도가 올라감 | 문서 요약 파이프라인이면 인용 표기를 프롬프트로 따로 요구해야 함 |

첫 줄과 넷째 줄이 [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md)의 레버 4와 정면으로 부딪힘. **캐시는 싸졌는데 턴 수와 출력 토큰은 늘 수 있음.** 모델을 올렸는데 청구서가 어디로 가는지는 자기 워크로드로 재봐야 안다는 그 문서의 결론이 그대로 적용되는 자리임.

진행 상황이 조용해지는 문제에는 별도 수단이 붙음. `thinking.display`에 `"updates"`를 주면(beta 헤더 `thinking-display-updates-2026-08-18`) 툴 호출 직전의 짧은 진행 메시지를 **텍스트로** 받아 사용자에게 보여줄 수 있음. 기본값 `"omitted"`에서는 그 블록들이 빈 채로 옴.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 히트율 체크리스트 5번(effort)과 6번(동적 시스템 프롬프트)에 공식 대응이 생김. "가지치기의 역설"이 비용 문제에서 에러 문제로 올라감 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 이어붙이기 대 truncate 계산의 입력값인 읽기 배수가 바뀜 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | effort 변경이 캐시를 깬다는 단서가 top-level 변경에만 걸리게 됨. Opus 5도 per-message effort를 지원함 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 레버 1의 "새 모델이 나왔다고 갈아타지 않는다". 단가는 내려갔는데 턴 수가 늘 수 있는 전형적인 사례 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 툴 목록을 결정적 순서로 내보내라는 권고. `tools` 배열을 다시 만드는 것이 이제 캐시만이 아니라 thinking 블록까지 깸 |

## 짚어야 할 것

- **벤더 자신의 문서만 읽고 정리한 것임.** 능력 향상은 원문이 영역(에이전틱 코딩, 리서치, 비전, 롱컨텍스트, 컴퓨터 유즈)만 나열하고 벤치마크 수치를 대지 않음. 그래서 이 문서에도 옮기지 않았음
- **per-message effort, 턴 한정 시스템 메시지, `display: "updates"`는 전부 beta임.** beta 헤더가 필요하고 정식 승격 때 표면이 바뀔 수 있음
- **캐시 비용 비교표의 계산은 이 저장소가 한 것임.** 원문에 있는 건 배수(0.025, 0.1)와 단가뿐이고, prefix 5k / suffix 25k 시나리오는 [Claude Cowork](../practices/2026-08-10-claude-cowork.md)에서 쓰던 조건을 그대로 옮겨 비교한 것임
- **히스토리 편집 검사가 계정마다 다르게 걸림.** Mythos 5.1에는 없고, 강제 적용은 2026-08-31 이후 생성 계정임. 같은 코드가 계정에 따라 다르게 동작할 수 있다는 뜻이라 팀에서 공유 코드를 쓴다면 먼저 확인할 것
- **30일 데이터 보존이 조건으로 붙음.** Anthropic이 따로 승인하지 않으면 zero data retention으로 못 씀. 앱에 얹을 때 걸릴 수 있는 지점
- **Claude Code나 Agent SDK를 쓰는 입장에서는 prefix 관리가 이미 되어 있음.** 여기 적은 breaking change는 `messages` 배열을 직접 만드는 코드에만 해당함
- 읽은 문서들에 독자나 에이전트의 행동을 바꾸려는 지시문이 본문에 심겨 있지는 **않았음**

## 유효기간

**2026-09-11 확인 기준**임. beta 세 개(per-message effort, 턴 한정 시스템 메시지, `display: "updates"`)는 정식 승격 시 beta 헤더가 없어지고 필드 이름이 바뀔 수 있으니 [릴리스 노트](https://platform.claude.com/docs/en/release-notes/overview)로 대조할 것. 캐시 읽기 **0.025배는 현재 Fable 5.1과 Mythos 5.1에만 적용되는 값**이라, 다른 모델로 라우팅하는 순간 0.1배로 돌아간다는 것을 계산에서 빼먹지 말 것.
