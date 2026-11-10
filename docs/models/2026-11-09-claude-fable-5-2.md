---
title: Claude Fable 5.2, 히스토리 편집 400 이 모든 계정으로 확대되고 입출력 단가는 내려감
source: https://www.anthropic.com/claude-fable-and-mythos-5-2
author: Anthropic
published: 2026-11-09
collected: 2026-11-10
tags: [claude, model-release, thinking, effort, prompt-caching, llm-cost, coding-agent]
---

출처: [Introducing Claude Fable 5.2 and Claude Mythos 5.2](https://www.anthropic.com/claude-fable-and-mythos-5-2) (2026-11-09) · [What's new in Claude Fable 5.2](https://platform.claude.com/docs/en/models/fable-5-2/whats-new-fable-5-2) · [모델 페이지](https://platform.claude.com/docs/en/models/fable-5-2/overview) · [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api) (2026-11-09 항목) · [Effort](https://platform.claude.com/docs/en/build-with-claude/effort) · [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations) (2026-11-10 확인)

## 요약

2026-11-09 릴리스. 벤치마크보다 먼저 봐야 하는 것은 이 저장소가 **재확인 과제로 남겨 둔 질문의 답**임. [thinking 블록의 세 가지 바인딩](../practices/2026-10-12-preserved-thinking-bindings.md)이 "prefix 검사가 전체 계정으로 확대됐는지"를 물어뒀고 답은 "아직 아님"이었음. Fable 5.2 와 Mythos 5.2 에서는 **계정 생성일과 무관하게 전체 계정에 강제**됨. 즉 2026-08-31 이전에 만든 계정에서 Fable 5.1 로 멀쩡히 돌던 히스토리 편집 코드가, 모델 ID 한 줄만 갈면 400 으로 떨어짐. 둘째, 가격이 내려감. input **$10 → $8**, output **$50 → $40** 이고 캐시 읽기는 **$0.20/MTok**. 셋째, thinking 블록의 모델 바인딩이 더 좁아져서 **Fable 5.2 의 블록은 Mythos 5.2 외에 어느 모델도 못 읽음.** 공식 폴백 대상인 Opus 4.8·Opus 5 도 못 읽으니 서버측 폴백 자체가 추론을 버리는 경로임. 그래도 이 모델은 기본값이 아님. 문서가 **"대부분의 워크로드는 최신 Opus 로 시작하라"**를 그대로 유지함.

## 기본 정보

| | |
|---|---|
| 모델 ID | `claude-fable-5-2` (Bedrock 은 `anthropic.claude-fable-5-2`) |
| 릴리스 | 2026-11-09. 은퇴는 **2027-11-09 이전은 아님** |
| 컨텍스트 / 최대 출력 | 1M 토큰(기본값이자 최대값) / 128K 토큰 |
| 가격 | input **$8** / output **$40** / 5m cache write $10 / 1h cache write $16 / **cache read $0.20** (MTok) |
| 배치 / fast mode | 배치 $4 / $20 · fast mode **$16 / $80** |
| thinking | adaptive **always on**. 끌 수 없음 |
| 기본 effort | `high`. `low`~`max` 다섯 단계 전부 지원 |
| 토크나이저 | **Opus 4.7 과 함께 들어온 것** |
| 지식 컷오프 | 2026-06 (training data cutoff 도 같음) |
| 입출력 | 텍스트·이미지 → 텍스트 |
| 데이터 보존 | **30일 고정.** 별도 승인 없이는 zero data retention 으로 못 씀. Covered Model |
| Mythos 5.2 | 같은 능력·같은 가격. Cyber Verification Program 같은 검증 프로그램을 통과한 조직만 |

## 히스토리 편집 검사, 계정 조건이 사라짐

[Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md) 문서가 적어둔 단서가 이것이었음. prefix 검사는 **2026-08-31 이후 생성된 계정에만 강제**되고, 그 이전 계정은 요청이 `prefix_mismatch_behavior` 를 지정할 때만 동작함. 그래서 옛 계정에서는 앞 턴을 편집해도 에러가 안 났음. Fable 5.2 는 그 조건을 떼었음.

> "On Claude Fable 5.2 and Claude Mythos 5.2, a request that sends back an invalid thinking block and leaves `prefix_mismatch_behavior` unset gets a 400 error, whatever the account's age."

문서가 이 함의를 직접 적음. **스위치 전에는 에러 없이 돌던 코드가 스위치 후에 깨짐.**

> "So code on Claude Fable 5.1 that edits its history can run without errors on an older account before the switch and fail after it."

검사에 걸리면 400 이고 메시지는 `The block is bound to a different conversation` 임. Fable 5.1 과 같은 문언이고, 완화책도 같음. `thinking-binding-controls-2026-08-01` 베타 헤더에 `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` 을 얹으면 걸린 블록을 버리고 진행하며 `input_transformations` 에 보고됨. 문서는 이것을 **편집을 고치기 전의 임시 조치(stopgap)**로 못 박고, 블록을 버리면 출력 토큰이 늘어난다는 쪽으로 링크를 걺. 그 비용 폭은 이 저장소의 [thinking 블록의 세 가지 바인딩](../practices/2026-10-12-preserved-thinking-bindings.md)에 이미 수치로 있음(**2.5%에서 67.1%**).

안 걸리는 조건도 문서가 명시함. Claude Code, claude.ai, Claude Managed Agents, Claude Agent SDK 가 요청을 만들어 주면 아무것도 안 바뀜. 직접 `messages` 를 만드는 코드도 다음 넷이 전부 참이면 안전함.

- `system` 과 `tools` 를 대화 전체에서 그대로 유지
- `messages` 에 **추가만** 함. 앞 메시지를 고치거나 순서를 바꾸거나 지우지 않음
- assistant 턴을 **API 가 돌려준 그대로** 블록 하나도 빼지 않고 같은 순서로 되돌려 보냄
- Preserved thinking 의 점검 목록에 걸리는 행위를 하지 않음

실무적으로는 iOS 앱에서 흔한 패턴 둘이 정확히 여기 걸림. 요청마다 앞 턴에 상태 줄이나 리마인더를 끼워 넣다 빼는 것, 그리고 로컬에 저장한 대화를 다시 조립해 보낼 때 블록 일부를 떨어뜨리는 것. 후자는 `Codable` 모델이 thinking 블록을 안 담고 있으면 **저장과 복원만으로** 성립함.

## thinking 블록을 읽는 모델이 더 좁아짐

Fable 5.2·Mythos 5.2 는 서로의 블록과 Opus 5.5, Sonnet 5.5, Haiku 5.5, Fable 5.1, Mythos 5.1, Opus 5, Fable 5, Mythos 5, 그리고 그 이전 Opus·Sonnet·Haiku 블록을 읽음. 반대 방향이 문제임.

> "No other model reads their blocks."

| 이동 방향 | 결과 |
|---|---|
| 다른 모델 → Fable 5.2 · Mythos 5.2 | 추론이 유지됨 (Claude API 와 Google Cloud(Google managed) 기준) |
| Fable 5.2 · Mythos 5.2 → 다른 모델 | 그 턴들의 추론이 사라짐. **Opus 5.5·Fable 5.1 포함** |
| Fable 5.2 ↔ Mythos 5.2 | 유지됨 |

버릴 때 **에러가 아니라 200** 이고 버린 블록은 과금되지 않음. 헤더를 안 붙이면 버렸다는 사실조차 응답에 없음. 이 저장소가 [Claude Sonnet 5.5](./2026-09-28-claude-sonnet-5-5.md)와 바인딩 문서에서 "조용히 나빠지는 실패"로 정리한 모양 그대로임.

여기에 폴백이 겹치는 자리가 새로 생김. Fable 5.2 의 **허용된 서버측 폴백 대상은 Opus 4.8 과 Opus 5** 인데, 둘 다 Fable 5.2 의 블록을 못 읽음. 즉 거절이 나서 폴백이 걸리면 그 대화의 추론은 버려진 상태로 답이 나옴. [거절 청구와 fallback](../practices/2026-09-24-refusal-billing-and-fallback.md)이 정리한 비용 축에 **품질 축이 하나 더 붙은 것**임. `reasoning_extraction` 처럼 권장 폴백이 없는 범주는 거절이 그대로 남는 것도 그대로임.

계정 바인딩도 이 모델에 걸림. Fable 5.2·Mythos 5.2 가 만든 블록은 **만든 계정이나 연결된 계정에서만** 유효하고, 다른 계정이 보내면 버려지고 요청은 성공함. 보고 사유는 `organization_binding_mismatch` 임. 바인딩 문서가 "계정 바인딩은 Sonnet 5.5·Haiku 5.5 블록만 해당"으로 적어둔 범위가 **이번에 Fable 계열까지 넓어진 것**임.

## 코드를 고쳐야 하는 나머지 셋

| 변경 | 내용 |
|---|---|
| **computer use 툴 교체** | `computer_toolset_20260801` 툴셋만 받음. 예전 `computer_20251124` 툴을 선언하면 **400 `invalid_request_error`**. 예외는 Amazon Bedrock 하나이고 거기서는 예전 툴이 그대로 동작함 |
| **advisor 툴 짝 맞추기** | Fable 5.2·Mythos 5.2 executor 는 **advisor 도 Fable 5.2 나 Mythos 5.2** 여야 함. Fable 5.1 을 advisor 로 두면 400. 반대로 모든 executor 가 Fable 5.2 를 advisor 로 받음 |
| **강제 툴 호출 없음** | `tool_choice` 가 `{"type": "any"}` 나 `{"type": "tool", ...}` 이면 400. `thinking: {"type": "enabled"}` + `budget_tokens` 도, `{"type": "disabled"}` 도 400 |

advisor 쪽에 함정이 하나 더 있음. advisor 의 `model` 을 바꾸는 것은 **`tools` 편집**이라서 위의 prefix 검사에 그대로 걸림. 그래서 문서가 "새 대화에서 바꿔라"로 적음. 돌아가는 대화의 advisor 만 갈아끼우면 다음 요청이 400 으로 떨어짐. **`drop_block` 을 켜 두지 않았다면 배포 직후에 터지는 모양임.**

강제 툴 호출의 대안은 Fable 5.1 과 같음. `tool_choice: {"type": "auto"}` 를 유지하고 strict tool use 의 `strict: true` 를 쓰거나 JSON outputs 를 쓰는 것. 다만 단서가 붙음. **customer-managed encryption keys 를 쓰는 조직에서는 Fable 5.2·Mythos 5.2 에 structured outputs 가 제공되지 않음.** strict tool use 와 JSON outputs 둘 다 해당하고, 그 환경에서는 `strict` 를 빼고 툴 입력을 코드에서 검증해야 함. 앱에서 JSON 을 받아 디코드하는 쪽이라면 **스키마 보장이 조직 설정에 따라 사라지는 자리**라 기억해 둘 만함.

## effort 와 가격, 이 저장소의 결론 두 개가 갱신됨

**첫째, 대화 중간 effort 변경.** Fable 5.1 문서에 "지원 모델은 Fable 5.1·Mythos 5.1·Opus 5 셋뿐이고 Claude API 한정"이라고 적어뒀음. 2026-11-10 기준 Effort 문서는 per-message `output_config` 로 캐시를 깨지 않고 effort 를 바꾸는 경로를 **Fable 5.2·Mythos 5.2·Fable 5.1·Opus 5.5·Sonnet 5.5·Haiku 5.5** 에 대해 적음(여전히 베타이고 플랫폼이 지원하는 곳에서만). Sonnet 5.5 는 `between_tools`, Haiku 5.5 는 `disabled` 와 같이 쓰면 중간 변경이 400 이라는 단서가 각각 붙음. **"effort 를 바꾸면 캐시가 깨진다"를 일반 규칙으로 들고 있으면 이제 거의 모든 최신 모델에서 틀린 쪽**이 됨. [Claude Code 의 effort 해석 순서](../practices/2026-10-27-claude-code-effort-resolution.md)에서 `ultrathink` 가 한 턴만 올리는 경로를 두고 "캐시를 두 번 깨는 셈"이라고 적어둔 유보도 이 목록과 함께 다시 봐야 함.

**둘째, 캐시 읽기 배수의 지형.** Fable 5.1 문서는 "다른 Claude 모델은 전부 0.1배인데 Fable 5.1·Mythos 5.1 만 0.025배"로 적었음. 지금 모델 페이지 각주는 셋으로 갈림. **0.1배가 기본, Fable 5.2·Mythos 5.2·Fable 5.1·Mythos 5.1 이 0.025배, Opus 5.5 와 Sonnet 5.5 가 0.05배**임. 단가로 보면 이렇게 됨. 아래 배수와 기본 단가는 모델 페이지 각주와 가격표에서 가져온 값이고, 캐시 읽기 단가 환산은 그 둘을 곱한 계산임.

| 모델 | input | cache read 배수 | cache read 단가 |
|---|---|---|---|
| Fable 5.2 | $8 | 0.025배 | $0.20 |
| Fable 5.1 | $10 | 0.025배 | $0.25 |
| Opus 5.5 | $4 | 0.05배 | $0.20 |
| Sonnet 5.5 | $2 | 0.05배 | $0.10 |

읽히는 결론이 둘임. Fable 5.2 는 **입출력은 여전히 Opus 5.5 의 정확히 2배**($8/$40 대 $4/$20)인데, **캐시 읽기 단가는 Opus 5.5 와 같아짐**($0.20). 긴 prefix 를 오래 재사용하는 에이전트 루프에서는 두 모델의 단가 차이가 입출력에만 남는다는 뜻임. 반대로 캐시 히트율이 낮은 워크로드면 2배 차이를 그대로 맞음.

effort 자체는 `high` 가 기본이고 문서 권고가 분명함. **"대부분의 능력 향상은 `high` 에서 이미 나오고, 그 위는 이득이 과제에 따라 들쭉날쭉함."** `max` 는 모델의 천장을 재거나 시간과 토큰이 상관없는 드문 과제에만 두라고 적고, `xhigh` 보다 느리고 비싼데 이득은 작을 때가 많다고 덧붙임. Fable 5.1 에서 쓰던 레벨을 그대로 옮기지 말고 eval 로 다시 쓸어 보라는 것도 같음.

## 코드를 안 고쳐도 드러나는 동작 변화

문서가 "behavior differences" 로 따로 모아둔 것들임. 품질 문제라기보다 **프롬프트를 손봐야 하는 자리**임.

- **"깊게 생각하지 말고 바로 답하라" 류의 지시가 `medium` 이상에서 통하지 않음.** Fable 5.1 은 대개 따랐는데 Anthropic 자체 테스트에서 Fable 5.2 는 `medium` 이상이면 그래도 생각했고, `high` 이상에서는 거의 매 응답 전에 생각함. 쉬운 턴이 느리거나 비싸면 **그 턴의 effort 를 내리는 것**이 처방임
- **긴 과제에서 턴이 일을 끝내기 전에 종료될 수 있음.** 남은 일 목록, 계속할지 묻는 제안, 흔한 장애물에서의 중단 형태이고 `low`·`medium` 에서 더 자주 나옴
- **코드 소유권을 프롬프트가 말하지 않으면 보수적으로 움직임.** 작은 코딩 과제 테스트에서, 누가 코드를 소유하는지 안 적으면 기존 구조에 덧붙이고 근처 문제는 고치지 않고 보고하는 쪽으로 기울었음
- **툴 호출이 길게 이어지는 동안 사용자에게 보이는 텍스트가 거의 없을 수 있음.** 첫 툴 호출을 예고 없이 하기도 함. 진행 표시를 UI 로 만들어야 하는 쪽에 바로 걸림
- **서브에이전트 위임 정도가 하네스와 과제에 따라 갈림.** 위임을 권하는 프롬프트가 깔린 제품에서는 적극적으로 위임하고, **서브에이전트가 세션 비용의 큰 몫을 차지할 수 있음**
- **스타일 규칙을 아주 충실히 따름.** 문장 길이에 하드 리밋을 걸면 산문이 뚝뚝 끊김

마지막 둘은 이 저장소의 [최소 하네스 대 오케스트레이터](../practices/2026-09-30-minimal-harness-mle-agents.md)와 붙여 읽을 자리임. 위임을 켜면 성능이 떨어지고 비용은 오르는 쪽으로 측정된 논문이 있는데, 모델 문서가 "위임 정도는 당신 하네스가 정한다"고 적은 것이니 **기본값을 하네스 쪽에서 눌러 두는 선택지**가 생김.

## 벤치마크

전부 **Anthropic 자체 발표**이고, 각주가 "따로 적지 않으면 각 모델의 최고 effort 기준"이라고 밝힘. Fable 5.2·Fable 5.1·Opus 5.5 점수는 세이프가드를 켠 상태임. 방향만 볼 것.

| 벤치마크 | Fable 5.2 | Fable 5.1 | Opus 5.5 |
|---|---|---|---|
| FrontierCode 2.0 (에이전틱 코딩) | 49.1% (`xhigh`) | 32.4% (`xhigh`) | 45.5% |
| FrontierSWE v2 (에이전틱 코딩) | 69.9% | 56.3% | 62.3% |
| Terminal-Bench-Science 0.1 | 74.6% | 52.6% | 58.7% |
| OSWorld 2.1 (offline, partial) | 89.3% | 83.5% | 87.2% |
| Humanity's Last Exam (툴 사용) | 74.2% | 65.6% | 67.7% |
| Chartography (툴 없음) | 73.4% | 44.8% | 64.4% |
| Remote Labor Index | 41.3% (`high`) | 17.9% | 21.3% |

Terminal-Bench-Science 의 Fable 5.1 값 52.6% 는 이 저장소의 [Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md)에 적힌 수치와 같음. 두 발표 사이에 기준이 안 바뀐 항목이라는 뜻임.

발표가 같이 내는 **비용당 성능 주장**은 벤치마크 점수보다 실무에 가까움. 다만 "cost per task" 의 산출 방식은 발표에 없어서 재현할 수 없음.

- FrontierCode 를 `high` 로 돌리면 **45.3%** 로 Opus 5.5 최고점(45.5%)과 거의 같고 **과제당 비용은 약 절반**
- WANDR 을 `medium` 으로 돌리면 Opus 5.5 최고점과 같은 점수에 **비용은 약 40%**
- Humanity's Last Exam 을 `low` 로 돌리면 Opus 5.5 최고점과 같은 점수에 **비용은 10분의 1 미만**
- 속도는 Fable 5.1·Mythos 5.1 대비 **최대 30% 빠름**, fast mode 는 **최대 2.5배**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md) | 그 문서가 재확인 과제로 남긴 둘의 답. 히스토리 편집 검사는 **전체 계정으로 확대됐고**, 대화 중 effort 변경은 **여전히 베타지만 모델이 늘었음** |
| [thinking 블록의 세 가지 바인딩](../practices/2026-10-12-preserved-thinking-bindings.md) | 세 바인딩이 한 모델에서 가장 세게 걸리는 사례. 모델 바인딩은 더 좁아지고, prefix 는 전체 계정, 계정 바인딩은 Fable 계열까지 확대 |
| [거절 청구와 fallback](../practices/2026-09-24-refusal-billing-and-fallback.md) | 폴백 대상(Opus 4.8·Opus 5)이 이 모델의 thinking 블록을 못 읽음. 폴백의 대가에 **추론 상실**이 추가됨 |
| [Fast mode 정식 공개](../practices/2026-10-15-fast-mode-opus-5-5-ga.md) | fast mode 단가가 또 정확히 **2배**($16/$80). "빨라지는 건 OTPS 뿐"이라는 그 문서의 분해가 그대로 적용되는지는 이번에 확인하지 않았음 |
| [Claude 모델 은퇴 일정](./2026-09-30-claude-model-retirement-schedule.md) | 은퇴 하한이 **2027-11-09** 로 올라옴. Fable 5.1 은 2027-09-01 로 그대로이고 폐기 공지는 없음 |
| [Claude Haiku 5.5](./2026-10-07-claude-haiku-5-5.md) | 같은 **Opus 4.7 토크나이저**임. 그 문서가 적은 "약 30% 더 센다"가 1M 창의 토큰 예산 산술에 그대로 들어감 |
| [API 버전 2026-10-21](../practices/2026-10-21-claude-api-version-migration.md) | `temperature`·`top_p`·`top_k` 는 그 버전에서 아예 제거된 파라미터임. 이 모델에서도 omit 이 정답 |
| [툴 정의를 메시지 안에 넣기](../practices/2026-09-22-inline-tool-definitions.md) | `tools` 를 안 고치고 툴을 바꾸는 경로(`inline-tools-2026-09-15`)가 prefix 검사를 통과하는 공식 수단으로 문서에 올라옴 |
| [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md) | on-demand compaction(`compact-2026-09-04`)도 같은 이유로 권장됨. 히스토리를 직접 자르는 대신 서버에 맡기는 쪽 |
| [Claude Code 의 effort 해석 순서](../practices/2026-10-27-claude-code-effort-resolution.md) | 기본 effort 가 `high` 라서 Claude Code 기본값과 API 기본값이 갈리는 모델 목록에 영향. 그리고 Claude Code 는 prefix 를 알아서 지켜줌 |
| [최소 하네스 대 오케스트레이터](../practices/2026-09-30-minimal-harness-mle-agents.md) | 위임을 켜면 성능이 내려간 측정과, "위임 정도는 하네스가 정한다"는 모델 문서 서술이 같은 자리를 가리킴 |

## 짚어야 할 것

- **이 모델은 기본값이 아님.** 문서 첫 문단이 "대부분의 워크로드는 최신 Opus 로 시작하고, Opus 를 높은 effort 로 돌린 eval 이 요구를 못 맞출 때 Fable 5.2 로 옮기라"고 적음. 입출력 단가는 Opus 5.5 의 2배임
- **가격 인하를 "싸졌다"로 읽으면 안 됨.** 내려간 것은 Fable 5.1 대비이고, 같은 날짜 기준으로 Opus 5.5·Sonnet 5.5 보다는 여전히 비쌈
- **벤치마크는 자체 발표 수치임.** 최고 effort 기준이 섞여 있고(일부 항목만 effort 가 표기됨), 상세 방법론은 시스템 카드에 있다고만 적혀 있음. **시스템 카드 본문은 이번에 읽지 않았음**
- **비용당 성능 주장의 산출 방식이 공개돼 있지 않음.** "과제당 비용 절반", "10분의 1 미만"의 분모가 무엇인지(토큰 수인지 벽시계 시간까지 포함인지) 발표에 없어 **확인할 수 없었음**
- **fast mode 의 캐시 거동을 이 모델에 대해 확인하지 않았음.** Opus 5.5 에서는 fast 와 standard 가 system·messages 캐시를 공유하지 않았는데, Fable 5.2 에서도 같은지는 이번에 보지 않았음
- **Claude Code 나 구독 플랜에서 이 모델이 어떻게 노출되는지 확인하지 않았음.** 위 내용은 전부 Claude API 기준임
- **Foundation Models 프레임워크 경유 경로에서 이 모델을 고를 수 있는지 확인하지 않았음.** [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md)가 그 경로에 베타 헤더가 없다고 적어뒀으니, 쓸 수 있다 해도 `drop_block` 같은 완화책은 그쪽에서 못 쓴다고 봐야 함
- 원문 발표 페이지와 공식 문서 어디에도 **지시문·명령문 형태의 삽입은 없었음.** 전부 제품 설명과 표임

## 유효기간

**2026-11-10 확인 기준**임. 릴리스 다음 날이라 베타 표시와 플랫폼 가용성이 가장 빠르게 움직이는 시점임. 다시 볼 때 확인할 것 셋. **대화 중 effort 변경이 베타를 벗어났는지**, **`computer_20251124` 의 Bedrock 예외가 닫혔는지**, 그리고 **CMEK 조직의 structured outputs 제한이 풀렸는지**. 가격과 캐시 배수는 [모델 페이지](https://platform.claude.com/docs/en/models/fable-5-2/overview)의 Pricing 표와 각주로 대조하는 편이 빠르고, 은퇴 하한은 [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)의 상태 표에서 보면 됨.
