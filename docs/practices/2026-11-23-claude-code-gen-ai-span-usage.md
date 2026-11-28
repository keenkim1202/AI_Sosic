---
title: Claude Code 스팬의 gen_ai.usage.*, 같은 스팬에 캐시를 넣는 쪽과 빼는 쪽이 같이 서 있음
source: https://github.com/anthropics/claude-code/releases/tag/v2.1.331
author: Anthropic
published: 2026-11-23
collected: 2026-11-28
tags: [claude-code, opentelemetry, observability, llm-cost, prompt-caching, telemetry]
---

출처: [Claude Code v2.1.331 릴리스 노트](https://github.com/anthropics/claude-code/releases/tag/v2.1.331) (`published_at` 기준 **2026-11-23**, `gh api` 로 받음) · [Monitoring, Claude Code Docs](https://code.claude.com/docs/en/monitoring-usage) (**2026-11-28 확인**. 페이지가 자바스크립트로 렌더링되므로 같은 경로의 `.md` 로 본문을 받았음)

## 요약

**2026-11-23 v2.1.331 이 `claude_code.llm_request` 스팬에 `gen_ai.usage.*`, `gen_ai.response.model`, `gen_ai.operation.name` 을 붙였음.** 릴리스 노트가 한 줄을 덧붙여 뒀는데 그 줄이 이 변경의 전부임. **`gen_ai.usage.input_tokens` 는 프롬프트 캐시 토큰을 포함함.** 그런데 같은 스팬의 기존 속성 `input_tokens` 는 문서가 **캐시 읽기와 쓰기를 뺀 값**이라고 못 박아 둔 것임. 이름이 거의 같고 의미가 반대인 속성이 **한 스팬에 둘** 서게 됨.

더 걸리는 쪽은 문서임. 2026-11-28 기준 Monitoring 문서는 **"Claude Code doesn't set `gen_ai.usage.*` attributes"** 라고 적은 채이고, 그래서 `input_tokens` + `cache_read_tokens` + `cache_creation_tokens` 를 **직접 더하라는 레시피**를 싣고 있음. 릴리스 노트를 따라 속성이 실제로 나온다면, 그 레시피를 그대로 짠 쿼리에 새 속성을 같이 더하는 순간 **캐시 몫이 두 번** 들어감. 어느 쪽이 지금 동작인지는 실행해 보지 않고는 가릴 수 없었음.

이 저장소의 [Claude API OpenTelemetry 내보내기](./2026-10-14-claude-api-opentelemetry.md)가 **"Claude Code·Agent SDK·Cowork 는 이 문서의 범위가 아님"** 으로 선을 그어 둔 자리임. 그쪽 Messages API 스팬에서 이미 **"캐시 속성은 더하지 않음, `gen_ai.usage.input_tokens` 가 이미 캐시 읽기와 쓰기를 포함함"** 이 규칙이었으므로, 두 표면의 `gen_ai.usage.input_tokens` 는 이제 같은 뜻으로 맞춰진 셈임. **엇갈리는 것은 Claude Code 가 원래 쓰던 자기 이름들임.**

## 같은 스팬에 두 체계가 겹침

`claude_code.llm_request` 의 토큰 속성을 의미로 갈라 적으면 이럼.

| 속성 | 캐시 포함 여부 | 출처 |
|---|---|---|
| `input_tokens` | **제외.** 문서가 "Excludes tokens read from or written to the prompt cache" 로 적음 | Monitoring 문서의 스팬 속성 표 |
| `cache_read_tokens` | 캐시에서 읽은 토큰 | 같음 |
| `cache_creation_tokens` | 캐시에 쓴 토큰. 5분·1시간 수명을 **합쳐서** 셈 | 같음 |
| `gen_ai.usage.input_tokens` | **포함.** 릴리스 노트가 "includes prompt cache tokens" 로 적음 | v2.1.331 릴리스 노트 |

읽는 법은 하나임. **접두사가 `gen_ai.` 인 것과 아닌 것을 섞어서 더하면 안 됨.** 한쪽을 골라야 하고, 고른 쪽의 캐시 규칙이 반대라는 것을 알고 골라야 함. 대시보드 쿼리가 `sum(input_tokens)` 로 짜여 있었다면 속성 이름이 안 겹치니 깨지지는 않음. 깨지는 것은 **새 속성이 보여서 "표준 이름이 생겼으니 그걸 쓰자"로 옮기는 경우**임. 그때 캐시가 조용히 합계로 들어오고, 캐시 읽기 단가가 입력 단가와 다르므로 **비용 환산이 통째로 틀림.**

`gen_ai.response.model` 과 `gen_ai.operation.name` 은 성질이 다름. 전자는 기존 `model`·`gen_ai.request.model` 과 달리 **응답이 실제로 어느 모델에서 왔는지**를 가리키는 이름이고, 후자는 OpenTelemetry GenAI 규약의 연산 종류 칸임. 다만 **두 속성의 값 범위와 요청 쪽 모델과 갈라지는 조건은 문서에서 확인할 수 없었음.** 릴리스 노트 한 줄에 이름만 있음.

## 문서가 아직 반대로 적혀 있음

Monitoring 문서에는 `Map input tokens to OpenTelemetry GenAI semantic conventions` 절이 따로 있음. 2026-11-28 기준 내용이 이럼.

> "Claude Code doesn't set `gen_ai.usage.*` attributes."

그리고 규약에 맞는 총합을 **직접 만들라고** 지시함.

- 스팬이나 이벤트에서: `input_tokens` + `cache_read_tokens` + `cache_creation_tokens`
- `claude_code.token.usage` 메트릭에서: `"input"` + `"cacheRead"` + `"cacheCreation"` 타입

캐시 쪽 이름 매핑도 같은 절에 있음. `cache_read_tokens` 가 `gen_ai.usage.cache_read.input_tokens` 로, `cache_creation_tokens` 가 `gen_ai.usage.cache_write.input_tokens` 로 감. 규약 구판은 쓰기 쪽을 `gen_ai.usage.cache_creation.input_tokens` 로 부르므로 **백엔드가 기대하는 이름을 쓰라**고 적혀 있음. 즉 캐시 쓰기 속성은 이름이 두 가지로 돌아다니는 상태임.

같은 날짜의 스팬 속성 표에도 `gen_ai.usage.*` 도 `gen_ai.response.model` 도 `gen_ai.operation.name` 도 올라와 있지 않음. 표에 있는 `gen_ai.` 속성은 넷뿐임: `gen_ai.system`, `gen_ai.request.model`, `gen_ai.response.id`, `gen_ai.response.finish_reasons`. 재시도는 속성이 아니라 `gen_ai.request.attempt` **스팬 이벤트**로 따로 들어감.

**판단은 이럼. 지금 이 속성을 쿼리에 박으면 안 됨.** 릴리스 노트와 문서가 서로 반대인 상태에서는 어느 쪽이 유효한지 외부에서 가릴 방법이 없고, 둘 중 하나는 곧 고쳐짐. 켜 둔 수집기에서 **속성이 실제로 오는지 눈으로 먼저 확인**하는 것이 유일하게 안전한 순서임.

## 캐시를 세는 자리가 셋으로 갈려 있음

이 변경이 왜 성가신지는 Claude Code 가 토큰을 세 신호로 내보내기 때문임. 셋의 캐시 취급이 전부 다름.

| 신호 | 캐시가 들어오는 방식 |
|---|---|
| `claude_code.llm_request` 스팬 | `input_tokens` 는 제외, `cache_read_tokens`·`cache_creation_tokens` 가 따로. 여기에 캐시 포함본 `gen_ai.usage.input_tokens` 가 더 붙음 |
| `claude_code.api_request` 이벤트 | 같은 네 필드에 **`cache_creation_5m_tokens`·`cache_creation_1h_tokens`** 가 더 있음. **v2.1.300 이상** 필요이고, API 응답이 안 주면 `cache_creation_tokens` 가 0 이 아니어도 둘은 0 으로 나옴 |
| `claude_code.token.usage` 메트릭 | `type` 이 `input`·`output`·`cacheRead`·`cacheCreation` 넷. **`cacheCreation` 은 5분과 1시간을 합쳐서 셈** |

TTL 별로 갈라 보려면 **이벤트만** 쓸 수 있다는 뜻임. 스팬도 메트릭도 5분·1시간을 합쳐서 줌. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 세어 둔 "히트율 낮을 때 확인할 8가지" 중 TTL 선택을 보는 항목이 여기에 걸림. 그리고 `cache_creation_5m_tokens` 가 0 인 것이 **"5분 캐시를 안 썼다"가 아니라 "API 가 내역을 안 줬다"일 수 있다**는 조건이 문서에 박혀 있으므로, 0 을 신호로 읽는 쿼리는 틀림. [Cache diagnostics](./2026-09-23-prompt-cache-diagnostics.md) 쪽이 **원인**을 지목하는 경로이고, 이쪽은 **양**을 세는 경로임. 둘을 섞으면 안 됨.

## 비용은 토큰 합산으로 만들 자리가 아님

`claude_code.cost.usage` 가 **USD 단위**로 따로 있음. API 요청마다 올라가고 `model` 로 쪼갤 수 있음. 토큰을 세어 단가를 곱하는 코드를 쓰지 않아도 되는 자리임. 모델을 섞어 쓰는 환경에서 단가 테이블을 코드 안에 또 하나 만드는 문제를 피하려면 이쪽이 맞음.

대신 이 메트릭에도 버전 경계가 있음. 문서가 적어 둔 것이 둘임.

- mod 가 `$.model.complete`·`$.model.classify` 로 내는 요청(`query_source` 가 `plugin_model_complete`·`plugin_model_classify`)은 **v2.1.324 부터** 집계에 들어감. 그래서 **올리고 나면 총액이 한 단 올라감**
- `agent.name`·`skill.name`·`plugin.name`·`mcp_server.name`·`mcp_tool.name` 은 기본으로 `custom`·`third-party` 로 가려짐. 실제 이름을 받으려면 `OTEL_LOG_TOOL_DETAILS=1` 이 필요하고, **v2.1.273 이전**에는 그 변수를 켜도 비용·토큰 카운터와 `api_request`·`api_error`·`api_refusal` 이벤트가 가려진 값을 내보냈음

읽어야 할 것은 이것임. **버전을 올리면 지출 그래프에 계단이 생기는데 그것이 쓰임새가 늘어난 것이 아님.** 집계 범위가 늘어난 것임. [대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md)의 레버 3(가시성·트립와이어·예산)에서 임계값으로 경고를 띄우는 설계라면, 업그레이드 날짜를 그래프에 표시해 두지 않으면 그 계단이 트립와이어를 때림.

## 켜는 조건과 스키마의 안정 구간

스팬 자체가 아직 **베타**임. 켜려면 셋이 필요함.

- `CLAUDE_CODE_ENABLE_TELEMETRY=1`: 텔레메트리 전체 스위치. 이것 없으면 아무것도 안 나감
- `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`: 스팬 추적. `ENABLE_ENHANCED_TELEMETRY_BETA` 도 받음
- `OTEL_TRACES_EXPORTER`: 보낼 곳

그리고 **`ENABLE_BETA_TRACING_DETAILED` 는 다른 것임.** 문서가 "디버깅에만 쓰고 프로덕션에 켜지 말 것"이라고 경고를 박아 뒀음. 내보내는 양이 많고, 그 모드에서만 나오는 콘텐츠 속성(`system_prompt_preview`, `tool_input`, `response.model_output` 등)은 **안정 스팬 스키마 밖**이라고 명시함. 켜려면 `BETA_TRACING_ENDPOINT` 를 같이 줘야 하고 그 쌍이 **로그와 트레이스의 목적지를 바꿈.** 대화형 CLI 세션은 조직이 allowlist 에 올라가 있어야 하고, Agent SDK 와 `-p` 세션은 그 조건이 없음.

이 구분이 이번 변경에 바로 걸림. **`gen_ai.usage.*` 는 콘텐츠 속성이 아니므로 안정 스키마 쪽에 들어가는 이름임.** 그런데 그 안정 스키마를 서술하는 문서가 아직 "설정하지 않는다"로 적혀 있음. 베타에서 속성이 먼저 나가고 문서가 따라오는 순서로 읽히는데, 그 사이 구간에서는 **문서가 쿼리의 근거가 되지 못함.**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude API OpenTelemetry 내보내기](./2026-10-14-claude-api-opentelemetry.md) | 그 문서가 범위 밖으로 선을 그은 Claude Code 쪽임. Messages API 스팬에서는 `gen_ai.usage.input_tokens` 가 캐시를 포함하는 쪽이었고, 두 표면의 의미가 이제 맞춰짐 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | TTL 별 쓰기량을 보려면 이벤트만 쓸 수 있음. 스팬과 메트릭은 5분·1시간을 합침 |
| [Cache diagnostics](./2026-09-23-prompt-cache-diagnostics.md) | 캐시의 원인 대 양. 미스 지점을 지목하는 경로와 토큰을 세는 경로가 다름 |
| [대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md) | 레버 3 의 트립와이어가 업그레이드로 생긴 집계 계단에 걸리는 자리 |
| [Usage and Cost Analytics API](./2026-10-29-usage-cost-analytics-api.md) | 과금 숫자를 맞추는 쪽. 스팬 집계는 추이와 원인 분석용임 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 캐시 읽기 배수가 모델마다 갈림. 캐시 포함 합계를 입력 단가로 곱하면 틀리는 이유 |
| [Claude Code 의 effort 해석 순서](./2026-10-27-claude-code-effort-resolution.md) | 스팬의 `effort` 속성이 해석 결과를 요청 단위로 보여 줌. 그 문서의 순서를 실측으로 대조할 자리 |

## 짚어야 할 것

- **1차 출처 둘이 서로 반대로 말함.** 릴리스 노트는 속성을 추가했다고 적고, 같은 날짜에 받은 공식 문서는 설정하지 않는다고 적음. 이 문서는 **둘이 어긋나 있다는 사실까지만** 확인한 것이고, 실제로 어느 속성이 스팬에 실려 나오는지는 수집기를 켜서 눈으로 봐야 함. 확인하지 못했음
- **`gen_ai.operation.name` 의 값이 무엇인지 확인할 수 없었음.** 릴리스 노트에 이름만 있고 문서 표에 항목이 없음. OpenTelemetry GenAI 규약 쪽 값 목록으로 추정하는 것은 이 문서에서 하지 않음
- **`gen_ai.response.model` 이 `model` 과 갈라지는 조건도 확인할 수 없었음.** fallback 이나 모델 승격이 끼었을 때 두 값이 달라지는지가 실무에서 의미 있는 질문인데, 1차 출처에 서술이 없음
- **스팬은 과금 기록이 아님.** 이것은 Messages API 쪽과 같은 결론이고 Claude Code 쪽도 다르지 않음. 지출 숫자는 `cost.usage` 나 Usage API 로 맞추고, 스팬은 원인 분석에 씀
- 스팬 추적이 **베타**이므로 속성 이름과 스키마가 더 움직임. 이번 변경 자체가 그 증거임

## 유효기간

**2026-11-28 확인 기준이고 Claude Code v2.1.331(2026-11-23) 시점임.** 다시 볼 때 확인할 것 셋. **Monitoring 문서의 `Map input tokens to OpenTelemetry GenAI semantic conventions` 절이 고쳐졌는지**(지금 "doesn't set" 으로 적힌 문장이 유지되는지 바뀌는지가 어느 쪽이 맞았는지를 알려 줌), **스팬 속성 표에 `gen_ai.usage.*`·`gen_ai.response.model`·`gen_ai.operation.name` 세 행이 올라왔는지**, 그리고 **스팬 추적이 베타를 벗어났는지.** 속성 목록은 이 문서의 표를 고치지 말고 [Monitoring 문서](https://code.claude.com/docs/en/monitoring-usage)를 `.md` 로 다시 받아 대조할 것. 릴리스 쪽은 `gh api repos/anthropics/claude-code/releases` 에서 `gen_ai` 를 grep 하는 편이 빠름.
