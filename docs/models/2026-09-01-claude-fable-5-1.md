---
title: Claude Fable 5.1, 캐시 읽기가 1/4이 되고 트랜스크립트 편집이 에러가 됨
source: https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1
author: Anthropic
published: 2026-09-01
collected: 2026-09-08
tags: [claude, prompt-caching, effort, thinking, coding-agent, llm-cost]
---

출처: [What's new in Claude Fable 5.1](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1) · [모델 개요](https://platform.claude.com/docs/en/models/fable-5-1/overview) · [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) · [발표문](https://www.anthropic.com/claude-fable-and-mythos-5-1) (문서 전부 2026-09-08 확인)

## 요약

2026-09-01 릴리스. 능력 향상보다 **에이전트 하네스를 쓰는 쪽에 실제로 영향을 주는 변경 셋**이 중요함. 첫째, **캐시 읽기가 기본 input의 0.025배**로 내려감. 다른 모델은 전부 0.1배인데 이 모델만 예외이고, 입력·출력·캐시 쓰기 가격은 Fable 5와 동일함. 둘째, **트랜스크립트 앞부분을 편집하면 캐시가 식는 정도가 아니라 다음 요청이 400으로 거절됨.** 이 저장소가 "append 지향적으로 유지하라"고 여러 번 적어둔 권고가 프로토콜 수준의 강제로 올라온 것임. 셋째, **effort를 대화 중간에 바꿔도 프롬프트 캐시가 무효화되지 않음**(베타). 공식 권장 기본값은 여전히 **Opus 5**이고, Fable 5.1은 "긴 호흡의 에이전틱 작업" 또는 "Opus 5를 높은 effort로 돌려도 eval이 모자랄 때" 쓰라고 문서가 못 박음.

## 기본 정보

| | |
|---|---|
| 모델 ID | `claude-fable-5-1` |
| 릴리스 | 2026-09-01 |
| 컨텍스트 / 최대 출력 | 1M 토큰 / 128K 토큰 |
| 가격 | input **$10** / output **$50** per MTok |
| 캐시 | 5분 쓰기 $12.50, 1시간 쓰기 $20, **읽기 $0.25** per MTok |
| thinking | adaptive **상시 ON**. 끌 수 없음 |
| 기본 effort | `high` |
| 신뢰 지식 컷오프 | 2026-06 |
| 은퇴 시점 | 2027-09-01보다 이르지 않음 |
| 최소 캐시 길이 | 512 토큰 (Fable 5와 동일) |

같은 능력을 가진 **Claude Mythos 5.1**(`claude-mythos-5-1`)이 따로 있는데 초대 기반이라 이 문서에서는 다루지 않음.

## 캐시 읽기 0.025배, 이 모델만의 예외

Pricing 문서의 각주가 명시적임.

> "Cache hits and refreshes on Claude Fable 5.1 and Claude Mythos 5.1 are priced at 0.025x the base input price. All other models use the standard 0.1x multiplier."

숫자로 놓으면 이럼. 캐시 쓰기 배수는 안 바뀌었다는 게 요점임.

| | Fable 5 | Fable 5.1 | Opus 5 |
|---|---|---|---|
| 기본 input | $10 | $10 | $5 |
| 5분 캐시 쓰기 | $12.50 (1.25배) | $12.50 (1.25배) | $6.25 (1.25배) |
| 1시간 캐시 쓰기 | $20 (2배) | $20 (2배) | $10 (2배) |
| 캐시 읽기 | $1 (0.1배) | **$0.25 (0.025배)** | $0.50 (0.1배) |

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 정리한 구조 그대로 읽으면 의미가 분명함. 그 글의 요지는 **히트는 싸고 미스는 비싸다**였는데, 이 모델은 히트 쪽만 4분의 1로 내리고 미스 쪽(재작성)은 그대로 뒀음. 즉 **캐시가 살아 있는 경로와 깨진 경로의 격차가 벌어졌음.** 캐시를 깨는 습관의 벌점이 커진 것이지 캐시 관리가 덜 중요해진 게 아님.

⚠️ **아래 표는 원문에 없는 계산임.** [Claude Cowork](../practices/2026-08-10-claude-cowork.md)에 있는 "이어붙이기 대 truncate" 표에 공식 배수만 갈아끼운 것이고, 조건도 그쪽과 같음(prefix 5k, suffix 25k, 한 턴 추가).

| 선택 | 0.1배 모델 | Fable 5.1 (0.025배) |
|---|---|---|
| 이어붙이기 | 30k × 0.1 = 3k | 30k × 0.025 = **0.75k** |
| truncate, 되돌릴 지점 캐시 히트 | 5k × 0.1 = 0.5k | 5k × 0.025 = **0.125k** |
| truncate, 그 지점 캐시 없음 | 5k × 1.25 = 6.25k | 5k × 1.25 = **6.25k** |

판단 기준 자체는 안 바뀜. **되돌릴 지점에 캐시가 살아 있으면 truncate가 싸고, 없으면 이어붙이는 게 낫다.** 다만 세 번째 줄만 값이 그대로라, 캐시 없는 지점으로 되돌리는 선택의 상대적 손해가 커짐.

## 트랜스크립트 편집이 권고가 아니라 에러가 됨

이 문서에서 제일 중요한 부분임. Fable 5의 thinking 블록은 **자기 앞의 prefix에 묶임.** 시스템 프롬프트, `tools` 배열, 또는 더 앞선 메시지를 고치면 다음 요청이 400으로 떨어짐.

> "Modifying anything before a Claude Fable 5.1 thinking block (the `system` prompt, the `tools`, or an earlier message) results in an error on the next request, or in the block being dropped if you opt into that."

에러 메시지는 `The block is bound to a different conversation`이고, 검사가 **강제되는 대상은 2026-08-31 이후에 만들어진 계정**임. 그 전에 만든 계정은 불일치를 기록만 하고 `thinking.block_binding.prefix_mismatch_behavior`를 요청에 실을 때만 동작함. Mythos 5.1은 이 검사를 돌리지 않음.

문서가 나열한 **깨는 패턴**과 **안전한 패턴**이 갈리는 지점이 실무에서 바로 쓰임.

| 뒤 thinking 블록을 전부 무효화하는 것 | 유효하게 남는 것 |
|---|---|
| 앞 턴을 편집·재정렬·삭제하고 뒤를 남김 | thinking 블록을 **맨 앞부터 연속으로** 걷어내기 |
| 앞 턴에 리마인더나 상태 줄을 주입했다가 다음 요청에서 빼기 | 서버측 context editing이나 compaction으로 히스토리를 줄이기 |
| 같은 대화 안에서 `system`이나 `tools` 배열을 다시 만들기 | `cache_control` 마커를 옮기기 |
| 이미지·문서 URL이 나중 요청에 다른 바이트를 돌려주기 | 요청 사이에 `effort`를 바꾸기 |

Claude Code, claude.ai, Claude Managed Agents, Claude Agent SDK는 이 prefix를 알아서 지켜줌. 직접 `messages` 배열을 만드는 코드가 위험함.

이 저장소 관점에서 이 변경이 값어치 하는 이유는 이것임. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 "가지치기의 역설"은 히스토리를 재작성하면 **돈이 더 든다**는 경제 논증이었고, [Claude Cowork](../practices/2026-08-10-claude-cowork.md)의 truncate 계산도 같은 층위였음. 이제 같은 행위가 **비싼 선택이 아니라 거절되는 요청**이 됨. 원문이 붙인 처방도 정확히 그 방향임. 대화를 append-only로 다루고, 지시는 mid-conversation system message로 넣고, 툴 변경은 mid-conversation tool change로 하라는 것. 문서가 "These patterns also keep the prompt cache warm"이라고 덧붙임.

## effort를 대화 중간에 바꿔도 캐시가 안 깨짐

베타 헤더 `mid-conversation-output-config-2026-07-01`로 켬. `role: "system"` 메시지에 `output_config`만 실어 보내면 **다음 user 턴부터** 새 effort가 적용됨.

> "On Claude Fable 5.1 you can change the effort level mid-conversation without invalidating the prompt cache."

Fable 5.1, Mythos 5.1, 그리고 **Opus 5**가 Claude API에서 지원함.

이 저장소에 적힌 것과 어긋나는 지점이라 따로 적어둠. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 히트율 점검 8가지 중 5번이 "툴 변경, 그리고 reasoning level(effort) 변경"을 캐시 무효화 원인으로 들었고, [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)의 관련 문서 줄도 "실서비스에서 effort를 동적으로 바꾸면 캐시 비용을 물게 됨"이라고 적었음. **이 경로에 한해서는 더 이상 그렇지 않음.** 다만 베타 헤더 없이 최상위 `output_config`만 요청마다 바꾸는 옛 방식이 캐시에 어떤 영향을 주는지는 문서가 말하지 않아 **확인할 수 없었음.**

같이 들어온 것이 **턴 한정 시스템 메시지**임(베타 헤더 `mid-conversation-system-clear-at-2026-08-21`). `clear_at: "next_user_message"`를 붙이면 그 턴에만 시스템 프롬프트 권한을 갖고, 이후에는 렌더링되지 않으면서 `messages` 안에는 그대로 남음. 그래서 앞이 안 바뀌고 캐시도 계속 맞음. 지운 메시지는 input 토큰을 먹지 않음. **"매 턴 리마인더를 주입했다 지우는" 하네스 패턴의 공식 대체재**임.

## 강제 툴 호출이 사라짐

`tool_choice`의 `{"type": "any"}`와 `{"type": "tool", "name": "..."}`가 400 `invalid_request_error`를 돌려줌. `auto`(기본)와 `none`은 그대로임.

이유를 문서가 밝힘. thinking이 상시 ON인데 강제 툴 호출은 그걸 건너뛰게 되고, 그러면 모델이 **툴 인자 안에 사고 과정을 써 넣어서 인자 품질이 떨어짐.** 대체 경로는 셋임. 스키마를 지키려면 `auto` + strict tool use, 또는 structured outputs. 툴을 반드시 부르게 하려면 프롬프트로 언제 쓰는지 말할 것.

## 코드 변경 없이 달라지는 동작

Fable 5 대비 기본 행동이 바뀐 것들임. 문서가 항목마다 프롬프팅 처방을 따로 연결해 뒀음.

- **병렬 툴 호출이 들쭉날쭉해짐.** Fable 5가 묶어 부르던 자리에서 턴당 하나만 부를 수 있음. 답 품질은 안 떨어지지만 턴 수, 토큰, 벽시계 시간이 늘어남. 커스텀 코딩 에이전트와 bash·에디터 하네스에서 드러남
- **긴 툴 실행 중 진행 상황을 덜 말함.** 특히 높은 effort에서. 진행 업데이트는 `thinking.display: "updates"`(베타 헤더 `thinking-display-updates-2026-08-18`)로 텍스트로 받을 수 있고, 추론 자체는 계속 숨겨짐
- **`low` effort에서 검색을 덜 하고 기억으로 답함.** 최신 정보가 필요한 턴은 effort를 올릴 것
- **파일을 조금 고칠 때 통째로 다시 쓰는 경향.** 결과는 대개 같지만 출력 토큰과 시간이 더 듦
- 그 밖에 산문 밀도가 높아지고, 채팅에서 볼드·헤더·목록을 덜 쓰고, 요약할 때 원문 구절을 인용 표시 없이 옮기는 경향이 있음

첫 두 항목이 [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)와 정반대 방향이라는 게 눈에 걸림. 그쪽은 Opus 5가 **너무 많이 떠들어서** 내레이션을 억제하라는 표를 담고 있는데, 이쪽은 **덜 말해서** 내레이션을 명시적으로 요구하라고 함. 모델별 프롬프팅 지침을 그대로 옮기면 안 되는 이유가 이것임.

## 그 밖에

- **생성 텍스트에 통계적 워터마크가 박힘.** 토큰이나 숨은 문자를 추가하지 않고 의미·품질에 영향이 없다고 명시함. 코드 실행 툴이 만든 이미지·영상·음성 파일은 Files API로 받을 때 C2PA Content Credentials가 붙음
- 거절은 HTTP 200에 `stop_reason: "refusal"`로 오고, 허용된 폴백 대상은 **Opus 4.8과 Opus 5**임. 모델을 바꾸느라 날아간 프롬프트 캐시 비용은 fallback credit으로 환불됨
- thinking 블록은 **한 방향으로만** 넘어감. Fable 5.1은 이전 모델의 thinking을 읽지만 그 반대는 안 됨. 라우터나 폴백이 대화 중간에 모델을 바꾸면 읽지 못하는 블록은 조용히 버려지고, 버려진 블록은 과금되지 않음
- 토크나이저는 Fable 5와 같음. Opus 4.7 이전 모델 대비 같은 텍스트가 약 30% 더 많은 토큰이 됨
- 30일 데이터 보존이 걸리고 별도 승인 없이는 zero data retention으로 쓸 수 없음

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 캐시 읽기 배수와 가지치기의 역설. effort 변경이 캐시를 깬다는 8가지 항목 중 5번이 이 모델에서 갱신됨 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 이어붙일지 truncate할지의 계산. 배수가 바뀌어 세 경우의 격차가 벌어짐 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | 내레이션과 검증 지시의 방향이 정반대임. 모델별 지침을 옮겨 쓰면 안 되는 사례 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 레버 1, 새 모델이 나왔다고 갈아타지 말고 자기 워크로드로 재보라는 원칙이 그대로 적용됨 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 툴 목록을 결정적 순서로 내보내라는 권고. 여기서는 `tools` 배열 재구성이 아예 에러가 됨 |
| [Harness · Loop · Graph](../practices/2026-07-28-loop-graph-harness.md) | 트랜스크립트 편집 금지는 harness 계층의 영속성 설계 제약으로 내려옴 |

## 짚어야 할 것

- **능력 향상 수치는 Anthropic 자체 평가임.** 발표문이 벤치마크 표를 싣고 있으나 이 문서에는 옮기지 않았음. 인용하려면 발표문을 직접 볼 것
- **공식 권장은 여전히 Opus 5임.** 문서가 "For most workloads, start with Claude Opus 5"라고 적고, Fable 5.1은 긴 호흡의 에이전틱 작업이나 Opus 5로 eval이 모자랄 때 쓰라고 함. input 기준으로 Opus 5의 두 배 가격이고 지연시간도 더 느린 쪽으로 표시돼 있음
- **캐시 읽기 할인은 이 모델 한정임.** Opus 5나 Sonnet 5로 돌아가면 0.1배로 돌아감. 라우팅으로 모델을 섞어 쓰는 하네스라면 모델마다 캐시 경제가 다르다는 걸 계산에 넣어야 함
- **베타 헤더 셋에 의존하는 기능이 많음.** per-message effort, 턴 한정 시스템 메시지, 진행 업데이트가 전부 베타임. 정식화되면서 헤더 이름과 필드가 바뀔 수 있음
- **직접 돌려보지 않았음.** 이 문서는 공식 문서 세 곳과 발표문을 읽고 정리한 것임. 특히 prefix 바인딩 검사가 실제 하네스에서 어떻게 걸리는지는 확인하지 못했음
- 원문 어디에도 읽는 쪽을 향한 지시문이나 명령문은 없었음

## 유효기간

**2026-09-08 확인 기준**임. 가격표와 모델 스펙은 바뀔 수 있고, 베타 기능 셋은 정식화 시점에 표면이 달라짐. 다시 볼 때는 [Pricing](https://platform.claude.com/docs/en/about-claude/pricing)의 캐시 각주가 아직 이 모델만 예외로 두는지, 그리고 [What's new](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1)의 베타 헤더 셋이 정식으로 올라갔는지부터 대조하면 됨.
