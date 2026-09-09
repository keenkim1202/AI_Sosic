---
title: Claude Fable 5.1, 캐시 읽기가 4분의 1이 되고 히스토리 편집이 에러가 됨
source: https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1
author: Anthropic
published: 2026-09-01
collected: 2026-09-09
tags: [claude, fable-5-1, prompt-caching, effort, thinking, coding-agent, llm-cost]
---

출처: [What's new in Claude Fable 5.1](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1) · [Claude Platform 릴리스 노트 2026-09-01](https://platform.claude.com/docs/en/release-notes/api) · [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) · [Models overview](https://platform.claude.com/docs/en/models/overview) · [Introducing Claude Fable 5.1 and Claude Mythos 5.1](https://www.anthropic.com/claude-fable-and-mythos-5-1) (2026-09-09 확인)

## 요약

2026-09-01에 나온 모델인데, 이 저장소 관점에서 값어치 하는 건 벤치마크가 아니라 **캐시와 프리픽스 규율**임. 입력·출력 단가는 Fable 5와 같은데 **캐시 읽기만 기본 입력가의 0.025배**로 내려갔음. 다른 Claude 모델은 전부 0.1배임. 쓰기는 그대로라서, **프리픽스를 깨는 행위의 상대 비용이 12.5배에서 50배로 벌어짐.** 두 번째가 더 중요한데, thinking 블록이 자기 앞의 히스토리에 묶여서 **앞 턴을 고치면 캐시가 깨지는 게 아니라 400 에러가 남.** [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 "이러면 캐시가 깨진다"고 정리해둔 목록이 이제 상당 부분 "이러면 요청이 거부된다"로 올라온 것임. 대신 캐시를 깨지 않고 effort와 턴별 지시를 바꾸는 beta 표면 둘이 같이 들어왔음.

## 기본 정보

| | |
|---|---|
| 모델 ID | `claude-fable-5-1` |
| 컨텍스트 | **1M 토큰** (기본값이자 최대값), 전 구간 표준 단가 |
| 최대 출력 | 128K 토큰 |
| thinking | adaptive **상시 ON**. `disabled`나 `budget_tokens`는 400 |
| 기본 effort | `high` |
| 가격 | 입력 **$10** / 출력 **$50** per MTok. Fable 5와 동일 |
| 캐시 읽기 | **$0.25 / MTok**, 기본 입력가의 **0.025배** |
| 캐시 쓰기 | 5분 $12.50, 1시간 $20. **변동 없음** |
| 최소 캐시 길이 | 512 토큰. 변동 없음 |
| 토크나이저 | Fable 5와 동일 (Opus 4.7에 도입된 것) |
| 은퇴 | 2027-09-01보다 이르지 않음 |
| 데이터 보존 | **30일 필수.** 별도 승인 없이는 zero data retention 불가 |

문서가 스스로 배치를 이렇게 정리함. 대부분의 작업은 Opus 5로 시작하고, **긴 호흡의 에이전틱 작업이거나 Opus 5를 높은 effort로 돌려도 자기 eval을 못 넘길 때** Fable 5.1을 쓰라는 것. 기본값 자리를 가져가는 모델이 아님.

같은 날 나온 Claude Mythos 5.1은 능력이 같고 안전장치 구성만 다른 제한 접근 모델임. 승인된 조직만 받으므로 이 저장소 범위 밖이고, 아래 내용은 전부 Fable 5.1 기준임.

## 캐시 읽기 0.025배가 실제로 바꾸는 것

가격표에서 바뀐 칸은 하나뿐임.

| 항목 | Fable 5 | Fable 5.1 |
|---|---|---|
| 기본 입력 | $10 / MTok | $10 / MTok |
| 5분 캐시 쓰기 | $12.50 / MTok (1.25배) | $12.50 / MTok (1.25배) |
| 1시간 캐시 쓰기 | $20 / MTok (2배) | $20 / MTok (2배) |
| **캐시 읽기** | $1 / MTok (0.1배) | **$0.25 / MTok (0.025배)** |
| 출력 | $50 / MTok | $50 / MTok |

여기서 나오는 결론은 "싸졌다"가 아님. **쓰기와 읽기의 비율이 벌어졌다**는 것임.

```
Fable 5   : 1.25배 쓰기 / 0.1배   읽기 = 12.5
Fable 5.1 : 1.25배 쓰기 / 0.025배 읽기 = 50
```

캐시가 살아 있는 상태로 프리픽스를 재사용하는 것과, 미스가 나서 프리픽스를 다시 올리는 것 사이의 격차가 **4배로 커졌음.** 즉 히트율이 같아도 캐시를 깨는 실수 하나의 상대적 손해가 예전보다 크다는 뜻임. 절대 청구액은 내려가는데 **캐시 규율의 중요도는 오히려 올라가는** 구조고, Anthropic이 밝힌 "일반 워크로드 약 25% 절감, 고에이전틱 작업은 최대 약 45%"라는 수치도 캐시 히트를 전제로 한 것임.

### 이어붙일까 자를까를 다시 계산

[Claude Cowork](../practices/2026-08-10-claude-cowork.md)에서 "후속 메시지 대신 대화 재시작"이 언제 이득인지 캐시를 넣고 따져둔 표가 있음. 같은 전제(프리픽스 5k, 서픽스 25k, 한 턴 더 주고받음)를 Fable 5.1 배수로 다시 넣으면 이렇게 됨.

| 선택 | Fable 5 (읽기 0.1배) | Fable 5.1 (읽기 0.025배) |
|---|---|---|
| 이어붙이기 | 30k × 0.1 = **3k** | 30k × 0.025 = **0.75k** |
| truncate, 되돌릴 지점에 캐시 있음 | 5k × 0.1 = **0.5k** | 5k × 0.025 = **0.125k** |
| truncate, 그 지점에 캐시 없음 | 5k × 1.25 = **6.25k** | 5k × 1.25 = **6.25k** |

세 번째 줄만 그대로임. 그래서 판단 기준은 그 문서의 결론과 같되 **더 날카로워짐.** 되돌릴 지점에 캐시가 살아 있으면 자르는 게 여전히 싸고, 없으면 이어붙이는 게 낫다는 방향은 유지되는데, 캐시 없는 지점으로 되돌리는 선택의 손해가 이제 이어붙이기 대비 **8배가 넘음**(6.25k 대 0.75k). 예전엔 약 2배였음. 그리고 서픽스가 길수록 이어붙이기 쪽이 더 유리해지므로, **긴 세션을 습관적으로 잘라내던 절약 요령은 이 모델에서 재검토 대상임.**

## 깨지는 것에서 에러가 되는 것으로

이 문서에서 가장 조심해서 볼 부분임. thinking 블록마다 어느 모델이 만들었는지와 **자기 앞의 히스토리가 무엇이었는지**가 기록됨. 그 앞이 바뀌면 다음 요청이 거부됨.

> "Modifying anything before a Claude Fable 5.1 thinking block (the `system` prompt, the `tools`, or an earlier message) results in an error on the next request, or in the block being dropped if you opt into that."

에러 메시지는 `The block is bound to a different conversation`이고 400임.

| 뒤의 thinking 블록을 **무효화**하는 것 | **유지**되는 것 |
|---|---|
| 앞 턴을 편집·재배열·삭제하면서 뒤 턴은 남기기 | 맨 앞부터 순서대로 thinking 블록 묶음을 제거 |
| 요청마다 앞 턴에 텍스트를 주입했다가 다음 요청에서 빼기 | 서버측 context editing이나 compaction으로 히스토리 정리 |
| 같은 대화 중에 top-level `system`이나 `tools` 배열을 재구성 | `cache_control` 마커 이동 |
| 이미지·문서 URL이 나중 요청에 다른 바이트를 반환 (URL이 아니라 바이트 기준이라 서명된 회전 URL은 무관) | 요청 사이에 `effort` 변경 |

맨 앞이 아닌 위치에서 thinking 블록 하나를 빼면 그 뒤 전부가 무효가 됨.

**이 표를 캐시 미스 원인 8가지와 나란히 놓으면 겹치는 게 보임.** [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 정리한 목록 중 4번(compaction·수동 히스토리 재작성), 5번(툴 변경), 6번(동적 시스템 프롬프트), 7번(오래된 메시지를 수정하는 익스텐션)이 여기 그대로 다시 나옴. 원리가 같기 때문임. **둘 다 프리픽스 불변성에 걸린 것**이고, 캐시 쪽에서는 조용한 과금으로, thinking 쪽에서는 요청 거부로 나타남. 원문도 같은 문단에서 append-only로 다루라고 하면서 "These patterns also keep the prompt cache warm"이라고 붙임.

적용 조건에 단서가 둘 있음. 이 검사는 **2026-08-31 이후에 만들어진 계정에만 강제**되고, 그 전 계정은 요청이 `thinking.block_binding.prefix_mismatch_behavior`를 설정할 때만 작동함. 그리고 Claude Code, claude.ai, Claude Managed Agents, Claude Agent SDK는 프리픽스를 대신 지켜준다고 원문이 명시함. 즉 **직접 `messages` 배열을 조립하는 코드만 이 문제를 만남.**

거부 대신 블록을 버리고 진행하려면 `thinking-binding-controls-2026-08-01` 헤더와 함께 `prefix_mismatch_behavior: "drop_block"`을 보냄. 버려진 블록은 응답의 `input_transformations`에 `reason: "prefix_binding_mismatch"`로 보고됨. 원문이 권하는 진단 절차가 이거임. **한 세션을 `drop_block`으로 돌려보고 `input_transformations`에 뭐가 찍히는지 보면 자기 통합이 히스토리를 고치고 있는지 알 수 있음.**

## 캐시를 깨지 않고 바꾸는 두 표면

위 제약의 반대편으로 들어온 것들임. 둘 다 beta 헤더가 필요함.

**대화 중간 effort 변경** (`mid-conversation-output-config-2026-07-01`). `role: "system"` 메시지에 `output_config.effort`를 실어 넣으면 다음 사용자 턴부터 새 레벨이 적용되는데, **프롬프트 캐시를 무효화하지 않음.** 어려운 단계에서 올리고 반복 작업에서 내리라는 게 문서 권고임. Fable 5.1, Mythos 5.1, Opus 5에서 지원됨.

```json
{"role": "system", "content": [], "output_config": {"effort": "low"}}
```

이건 이 저장소의 기존 서술을 **갱신하는 항목임.** [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 히트율 점검 5번이 "툴 변경, 그리고 reasoning level(effort) 변경. 후자도 같은 효과 냄"이었고, [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)의 관련 문서 줄도 "실서비스에서 effort를 동적으로 바꾸면 캐시 비용을 물게 됨"이었음. 이 beta 표면을 쓰는 경로에서는 **그 대가가 없어짐.** 다만 beta이고 모델이 한정되니 일반 규칙으로 옮기면 안 됨.

**턴 한정 시스템 메시지** (`mid-conversation-system-clear-at-2026-08-21`). `role: "system"` 메시지에 `clear_at: "next_user_message"`를 붙이면 그 턴에만 시스템 프롬프트 권한으로 작동하고, 이후 사용자 메시지가 생기면 렌더링을 멈춤. **메시지는 `messages`에 그대로 남아 계속 되돌려 보내므로 앞쪽이 바뀌지 않고**, 꺼진 뒤에는 입력 토큰을 먹지 않음.

```json
{
  "role": "system",
  "clear_at": "next_user_message",
  "content": "Results have landed in your inbox. Check it before running more code."
}
```

용도가 정확히 그 문제임. 툴 루프에서 매 턴 리마인더를 히스토리에 주입했다가 다음 요청에 빼는 패턴을 대체하는 것. **캐시 미스 원인 6번(동적 시스템 프롬프트)과 7번(오래된 메시지를 수정하는 익스텐션)에 대해 프로토콜 차원의 답이 처음 나온 셈임.** 타임스탬프나 상태 줄을 넣고 싶었던 자리가 여기임.

## 깨지는 변경: 강제 툴 호출이 사라짐

`tool_choice`의 `{"type": "any"}`와 `{"type": "tool", "name": "..."}`가 400 `invalid_request_error`를 반환함. `auto`(기본)와 `none`은 그대로임. 토큰 카운팅 엔드포인트에도 같은 검증이 걸림.

이유가 설득력 있음. **thinking이 상시 ON인데 강제 툴 호출은 그 단계를 건너뛰게 되고, 그러면 모델이 작업 내용을 툴 인자 안에 써버려서 인자 품질이 떨어짐.** 대체 경로는 셋임.

- 스키마를 지키려면 `tool_choice: auto`를 두고 strict tool use에 `strict: true`
- 또는 스키마를 structured outputs로 옮김
- 텍스트로 답하지 말고 툴을 부르게 하려면 프롬프트에 조건을 적음 ("Use the `get_weather` tool to answer")

Fable 5로 짜둔 코드에서 깨지는 건 이거랑 위의 히스토리 편집, 그리고 이전 모델이 Fable 5.1의 thinking 블록을 못 읽는다는 것 셋임. 마지막 항목은 **모델을 갈아타는 라우터나 폴백을 쓰면 조용히 블록이 버려진다**는 뜻이라, 라우팅 계층을 두고 있으면 확인할 지점임.

## 코드를 안 바꿔도 달라지는 동작

문서가 "prompting fix가 각각 있다"며 나열한 것 중 비용에 직접 걸리는 것들.

| 달라진 것 | 무엇을 물게 되나 |
|---|---|
| **병렬 툴 호출이 더 들쭉날쭉함.** Fable 5가 묶어 보내던 것을 턴당 하나씩 보낼 수 있음 | 답 품질은 안 떨어지는데 턴 수, 왕복, 벽시계 시간이 늘어남. 커스텀 에이전트 루프에서 특히 |
| **툴 실행 중 진행 보고가 줄어듦.** 높은 effort에서 더 그럼 | 긴 턴이 사용자에게 조용해 보임. `thinking.display`를 `"updates"`(beta)로 두면 진행 보고만 텍스트로 받음 |
| **`low` effort에서 기억으로 답하는 빈도가 늘어남.** 검색·조회 툴을 덜 부름 | 최신 정보가 필요한 턴은 effort를 올려야 함. 위의 대화 중간 변경이 여기 쓰임 |
| **작은 수정에도 파일 전체를 다시 씀** | 결과는 대개 같은데 출력 토큰과 시간이 더 듦 |

[대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md)의 레버 4가 "툴 출력 장황함 감사"와 "덜 수다스러운 하네스"였는데, 여기서는 방향이 반대로 걸림. **모델이 덜 말해서 생기는 비용**(턴 수 증가, 전체 파일 재작성)이라 하네스를 조용하게 만드는 것으로는 안 잡히고 프롬프트로 되돌려야 함.

그 밖에 채팅에서 굵게·헤더·목록을 덜 쓰고, 문서를 요약할 때 원문 구절을 인용 표시 없이 옮기는 경향이 있다고 문서가 적음. 후자는 이 저장소처럼 요약을 쌓는 용도에서 직접 걸리는 항목임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 히트율 점검 8가지 중 넷이 thinking 블록 무효화 목록과 같은 것임. 5번(effort 변경)은 beta 표면으로 갱신됨 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 이어붙이기 대 truncate 계산표를 0.025배로 다시 그려야 함 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | 빼야 할 것과 넣어야 할 것의 목록이 모델별로 갈림. 여기는 내레이션을 오히려 **요청해야** 하는 쪽 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 레버 1의 "새 모델이 나왔다고 갈아타지 않는다". 이 모델도 기본값 자리가 아니라 조건부 선택지임 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 툴 목록을 결정적 순서로 내보내라는 권고. `tools` 배열 재구성이 이제 에러 사유이기도 함 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | Xcode 26.3이 얹은 Claude Agent SDK가 프리픽스를 대신 지켜주는 하네스 목록에 들어 있음 |

## 짚어야 할 것

- **벤치마크 수치는 이 문서에 옮기지 않았음.** 뉴스룸 페이지에 Terminal-Bench, OSWorld, HLE 등의 자체 측정치가 있는데 전부 제조사 자체 발표이고, 이 저장소가 쓸 판단(캐시·프리픽스·비용)에 영향을 주지 않음. 필요하면 원문에서 직접 볼 것
- **"25% 절감, 고에이전틱 최대 45%"는 Anthropic 추정치임.** 산출 근거와 워크로드 정의가 공개되지 않았고, 캐시 히트율에 전적으로 의존하는 숫자임. 자기 트레이스로 재보지 않고 인용하면 안 됨
- **위 두 beta 표면은 beta임.** 헤더 이름과 필드가 바뀔 수 있고, 턴 한정 시스템 메시지는 Fable 5.1 계열에서만 확인됨
- **캐시 관련 계산은 전부 공개 단가로 한 산술임.** 실제 청구는 최소 캐시 길이 512 토큰, TTL, 게이트웨이 정책에 따라 달라짐. 재판매 게이트웨이를 끼면 0.025배가 그대로 전달되지 않을 수 있음
- **직접 호출해보지 않았음.** 이 문서는 공식 문서, 릴리스 노트, 가격표, 뉴스룸을 대조해 정리한 것임. `input_transformations`가 실제로 무엇을 찍는지는 확인하지 못했음
- 제한 접근 모델(Mythos 5.1)과 엔터프라이즈 쪽 발표는 이 저장소 범위 밖이라 다루지 않았음
- 원문 문서들에 이 저장소를 겨냥한 지시문이나 주입 시도는 없었음. 명령문은 전부 개발자용 마이그레이션 안내였고 데이터로만 취급했음

## 유효기간

**2026-09-09 확인 기준**임. 가격표와 모델 스펙은 [Pricing](https://platform.claude.com/docs/en/about-claude/pricing)과 [Models overview](https://platform.claude.com/docs/en/models/overview)가 상시 갱신되므로 인용 전에 대조할 것. 특히 **0.025배 캐시 읽기가 이 세대에만 붙은 예외인지 다음 모델로 이어지는지**가 이 문서의 결론을 좌우함. beta 헤더 둘은 정식 승격되거나 이름이 바뀔 수 있으니 릴리스 노트를 따라가는 편이 나음.
