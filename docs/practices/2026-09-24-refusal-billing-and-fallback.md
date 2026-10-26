---
title: 거절 청구와 fallback, HTTP 200 으로 오는 거절에 요금이 붙기 시작함
source: https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback
author: Anthropic
published: 2026-09-24
collected: 2026-09-27
tags: [claude, anthropic, llm-cost, prompt-caching, routing, guardrail]
---

출처: [Refusals and fallback, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback) · [Fallback credit](https://platform.claude.com/docs/en/build-with-claude/fallback-credit) · [Stop reasons and fallback](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons) · [API release notes](https://platform.claude.com/docs/en/release-notes/api) (2026-09-27 확인). 프론트매터의 `published` 는 청구 변경이 올라온 릴리스 노트 항목 날짜임. 문서 페이지 자체에는 발행일이 없음

## 요약

Claude Fable 5.1, Fable 5, Opus 5.5, Opus 5 는 안전 분류기가 요청을 거절할 수 있고, 거절은 에러가 아니라 **`stop_reason: "refusal"` 이 박힌 HTTP 200** 으로 옴. 여기서 **2026-09-24 에 과금이 바뀌었음.** 출력이 하나도 나오지 않은 거절도 `stop_details.category` 가 **`bio`, `frontier_llm`, `reasoning_extraction`** 이면 그 모델의 정가로 청구됨. `cyber`, `general_harms`, 그리고 category 가 `null` 인 거절은 여전히 청구되지 않음. 스트리밍 중간에 나는 거절은 이전부터 청구되고 있었음.

이 저장소 관점에서 걸리는 지점이 둘임. 첫째, 거절은 200 이라서 **에러율이나 5xx 로 만든 계기판에는 한 줄도 안 잡힘.** 그런데 이제 돈이 나감. 둘째, 재시도를 다른 모델로 보내면 **캐시가 모델별이라 prefix 를 새로 write** 해야 하는데, 이걸 메우려고 Anthropic 이 붙여 둔 장치가 `fallback_credit_token` 이고 **유효기간이 5분**임. 즉 거절 처리를 "사용자에게 실패를 보여주고 끝"으로 두면 청구서와 캐시 양쪽에서 손해를 봄.

## 거절 응답의 모양

거절은 `content` 가 빈 배열인 정상 응답임. `stop_details` 가 붙고, 다른 stop reason 에서는 `stop_details` 가 항상 `null` 임.

- **`category`**: 분류기가 걸린 정책 영역. 명명된 범주에 매핑되지 않으면 `null` 이고, 이 `null` 은 임시값이 아니라 정상적인 영구 값임
- **`explanation`**: 사람이 읽는 설명. 문서가 **문구가 안정적이지 않으니 파싱하지 말고 표시만 하라**고 적음
- **`recommended_model`**: `fallbacks` 를 보낸 요청에만 들어옴. API 가 fallback 시도를 건너뛴 경우(예: fallback 모델이 rate limit) 직접 재시도할 모델 이름이고, 그 외에는 `null`. 보장이 아니라 힌트임

분기 기준도 문서가 못을 박음. `content` 나 `stop_details` 안쪽 필드가 아니라 **`stop_reason` 또는 `stop_details.type`** 으로 판단할 것. `category` 와 `explanation` 은 `null` 이 될 수 있어서 그걸 조건으로 쓰면 거절을 놓침.

| `category` | 무엇인가 | 출력 전 거절도 청구되나 |
|---|---|---|
| `"cyber"` | 멀웨어·익스플로잇 개발 같은 사이버 위해. **정상적인 보안 작업도 걸릴 수 있다**고 문서가 적음 | 아니오 |
| `"bio"` | 위험한 실험 기법 같은 생물학적 위해. 유익한 생명과학 작업도 걸릴 수 있음 | 예 |
| `"frontier_llm"` | 경쟁 AI 모델 개발 지원. Anthropic 상업 약관에서 제한하는 영역이고 정상적인 ML 작업도 걸릴 수 있음 | 예 |
| `"reasoning_extraction"` | 모델의 내부 추론을 **응답 텍스트로 재현하라**고 요구하는 요청 | 예 |
| `"general_harms"` | 위 네 범주 밖의 사용정책 영역 | 아니오 |

## 2026-09-24 에 바뀐 것

릴리스 노트가 적은 근거는 둘임. 대규모로 안전장치를 우회하려는 시도를 억제하기 위해서고, 위 세 범주는 **Anthropic 이 오탐(false positive) 물량을 낮게 측정한 범주**라는 것임. 문서에는 "as of September 2026" 이라는 단서가 붙음.

> "To disrupt attempts to circumvent Anthropic's safeguards at scale, a refusal that arrives before any output is billed when its `stop_details.category` is `"bio"`, `"frontier_llm"`, or `"reasoning_extraction"`."

부수 조건들이 실무에서 더 중요함.

- 청구는 **실행한 모델의 정가**로 붙음. 거절 전용 할인 같은 것은 없음
- 청구되든 안 되든 **`content` 는 비어 있고 `usage` 에는 토큰 수가 잡힘.** 즉 `usage` 를 그대로 합산하는 코드는 청구되지 않는 거절까지 비용으로 세고, 반대로 `content` 가 비었다고 건너뛰는 코드는 청구되는 거절을 놓침
- 거절이든 아니든 **rate limit 에는 차감됨**
- 전 플랫폼 공통임. Claude API, Amazon Bedrock, Claude Platform on AWS, Google Cloud, Microsoft Foundry
- mid-stream 거절은 input 토큰과 **이미 스트리밍된 output 까지** 정가로 청구됨. 이건 이번 변경 이전과 같음

**`reasoning_extraction` 이 앱 개발자에게 제일 현실적인 항목임.** 나머지 둘은 도메인이 뚜렷한데, 이 범주는 프롬프트 문구 하나로 만들어짐. "생각한 과정을 단계별로 적어라" 류의 지시를 시스템 프롬프트에 박아 둔 기능이 있으면, 그게 거절이 되고 이제 청구까지 됨. 문서가 제시하는 대안은 텍스트로 받지 말고 **adaptive thinking 으로 구조화된 형태로 받으라**는 것임. [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)가 "프롬프트에서 무엇을 지울지"를 토큰 절감으로 다뤘는데, 이제 같은 종류의 문구가 **거절 + 청구**라는 다른 비용도 만듦.

## fallback 경로 세 가지

거절된 요청을 다른 모델로 재시도하는 방법이 셋이고, 문서가 상황별로 갈라 둠.

| 상황 | 경로 | 대가 |
|---|---|---|
| Claude API, 가장 단순하게 | 서버측 `fallbacks` 파라미터 (베타) | 요청 한 번, 응답 한 번. 베타 헤더 `server-side-fallback-2026-07-01` 필요 |
| 플랫폼 상관없이, Anthropic SDK 사용 | SDK 의 refusal-fallback 미들웨어 | 클라이언트에 한 번 설정. `BetaFallbackState` 를 대화 단위로 공유해야 함 |
| raw HTTP 또는 자체 재시도 로직 | 직접 재시도 + fallback credit | 전부 내 손. 크레딧을 안 쓰면 캐시 write 를 새로 물어야 함 |

앞의 둘은 **fallback credit 을 자동으로 적용함.** 크레딧 문서를 읽어야 하는 쪽은 세 번째뿐임.

서버측 경로의 제약이 꽤 구체적임.

- `fallbacks: "default"` 로 두면 거절 범주에 맞춰 Anthropic 이 권하는 모델로 서버가 라우팅함. 범주에 권장 fallback 이 없으면 **거절이 그대로 유지됨**
- 직접 최대 **3개**까지 모델을 지정할 수도 있음. 순서대로 시도하고, 각 항목은 요청 모델의 **허용 타깃**이어야 함. 허용 목록은 베타 헤더를 켜면 Models API 의 `allowed_fallback_models` 로 공개됨
- **Message Batches API 에서는 지원하지 않음.** `fallbacks` 가 들어간 배치 항목은 에러 결과로 돌아옴. Bedrock, Google Cloud, Microsoft Foundry 에서도 못 씀. 그쪽은 SDK 미들웨어로 가야 함
- **분류기 거절만 fallback 을 트리거함.** rate limit, overload, 서버 에러는 그대로 나에게 옴
- 베타 헤더 날짜가 정확히 `2026-07-01`(`"default"` 와 명시 목록 둘 다) 또는 `2026-06-01`(명시 목록만)이어야 함. 다른 값이면 `fallbacks` 가 400 으로 거부됨

여기서 제일 쉽게 놓치는 항목 하나. **fallback 모델이 rate limit 이거나 overload 면 fallback 시도 자체가 일어나지 않고 앞선 거절이 반환됨.** 문서 표현대로 "부하가 걸리면 fallback 이 거절로 퇴화함". fallback 모델의 rate limit 을 예상 거절 물량에 맞춰 잡아 두지 않으면, 정작 필요한 순간에 안 도는 안전장치가 됨.

## 청구 계기판이 또 한 번 틀어지는 지점

[서버측 compaction](2026-09-20-claude-api-compaction.md)에서 정리한 함정이 여기서 그대로 반복됨. **최상위 `usage` 는 반환된 메시지를 만든 시도만 설명하고**, 시도별 실제 청구 기록은 `usage.iterations` 배열임.

> "The `usage.iterations` array is the per-attempt record of what you're billed."

- 거절한 모델은 일반 `message` 항목으로, 응답을 서비스한 모델은 **`fallback_message` 항목**으로 들어감
- 사슬의 모든 모델이 거절하면 마지막 모델의 거절이 응답이 되고, 앞선 홉마다 `message` 항목이 남음
- **서로 다른 모델의 토큰이 한 필드로 합산되는 일은 없음.** 그래서 모델별 단가를 붙이려면 `iterations` 를 돌아야 함
- 출력 없이 거절한 시도는 청구되는 범주일 때만 청구되지만, **토큰 수는 어느 쪽이든 그 항목에 기록됨**

[대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md)의 레버 3 이 "가시성 대시보드부터"였는데, 거절은 200 이고 fallback 은 `iterations` 안에 있으니 **에러율 기준 모니터링과 최상위 `usage` 기준 비용 집계는 둘 다 이 지출을 못 봄.** 문서도 같은 처방을 적어 둠. 거절마다 이벤트 하나, fallback 이 서비스한 응답마다 이벤트 하나를 내고 **두 카운트의 차이에 알람을 걸 것.**

## fallback credit

프롬프트 캐시는 모델별임. 그래서 거절 후 다른 모델로 재시도하면 이미 캐싱된 prefix 를 새 모델 캐시에 **처음부터 write** 해야 하고, write 는 read 보다 비쌈. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 다룬 산술이 그대로 걸리는 자리임. fallback credit 은 그 차액을 없애서 **처음부터 새 모델에 있었던 것처럼 청구**하게 만듦.

흐름은 네 단계임.

1. 거절될 수 있는 요청에 `anthropic-beta: fallback-credit-2026-07-01` 을 붙임 (`server-side-fallback-2026-07-01` 도 같은 필드를 열어 주고, 이전 `fallback-credit-2026-06-01` 도 계속 받음)
2. 거절 응답의 `stop_details` 에서 **`fallback_credit_token`** 과 **`fallback_has_prefill_claim`** 둘을 읽음. 크레딧이 없으면 둘 다 `null`
3. 거절된 요청 본문에서 시작해 `model` 만 fallback 모델로 바꾸고 최상위 `fallback_credit_token` 을 더함. `fallback_has_prefill_claim` 이 `true` 면 거절 응답의 `content` 를 담은 **assistant 메시지 하나를 뒤에 덧붙임**. 그러면 재시도가 중단 지점부터 이어서 쓰고, **이미 끝난 서버 툴 호출은 다시 실행되지 않음**
   - `content` 를 **글자 그대로 복사하면 안 되는 경우가 둘** 있음. 마지막 블록이 `text` 면 **뒤쪽 공백을 떼야** 하고, 짝이 되는 `tool_result` 가 없는 **클라이언트측 `tool_use` 블록은 빼야** 함. 이걸 빠뜨리면 멀쩡한 요청이 400 으로 떨어져 아래 재시도 사다리를 괜히 타게 됨. 서버 툴이 이미 돈 뒤였다면 사다리 끝에서 이어쓰기를 포기하는 순간 **크레딧을 잃거나 그 툴을 다시 돌려 또 과금**됨
4. 재시도에도 **같은 베타 헤더**를 보냄. 헤더 없이는 토큰이 리딤되지 않음

리딤 조건이 까다로운 편임. 프롬프트 모양을 만드는 필드는 전부 정확히 일치해야 함.

| 규칙 | 필드 |
|---|---|
| 정확히 일치해야 함 | `system`, `messages`, `tools`, `tool_choice`, `thinking`, `cache_control`, 그리고 쓰는 경우 `output_config`, `mcp_servers`, `context_management`, `container` |
| 재시도에서 바꿔도 됨 | `model`, `max_tokens`, `stop_sequences`, `temperature`, `top_p`, `top_k`, `stream`, `metadata`, `service_tier` |

여기에 함정이 둘 붙음.

- **베타 헤더도 일치해야 함.** 한쪽에만 있는 헤더 때문에 본문이 같아도 매치가 깨질 수 있고, 그때 나오는 400 메시지가 본문 불일치와 **같은 문구**라서 원인을 오해하기 쉬움. `server-side-fallback-*` 과 `fallback-credit-*` 두 계열만 예외임
- **크레딧을 쓸 때는 앞 턴의 `thinking`·`redacted_thinking` 블록을 지우면 안 됨.** 크레딧 없이 재시도할 때는 지우는 게 보통인데, 크레딧 리딤은 본문 일치를 요구하므로 반대가 됨. 서버가 그 블록들을 알아서 처리함

크레딧이 실제로 적용됐는지는 재시도의 `usage` 로 봄. 같은 요청을 토큰 없이 보냈을 때와 비교해 **`cache_creation_input_tokens` 가 줄고 `cache_read_input_tokens` 가 같은 양 늘어남.** 변동이 0 이면 토큰은 인정됐지만 리프라이스할 게 없었던 것(예: 재시도 모델 캐시가 이미 따뜻했음).

토큰 자체의 성질도 알아 둘 것.

- **거절 5분 뒤 만료됨.** 그 뒤에는 토큰 없이 재시도
- 거절을 받은 **조직과 워크스페이스**에서만 리딤됨. Bedrock 과 Google Cloud 는 워크스페이스가 없어 플랫폼의 호출자 신원에 묶임
- **stateless 임.** 서버가 토큰에 대해 아무것도 저장하지 않고, 조회하거나 무효화하는 엔드포인트도 없음
- **Message Batches 의 거절은 크레딧 토큰을 발급하지 않음.** 배치 결과의 `stop_details` 에는 `fallback_credit_token` 이 아예 없음

재시도가 400 으로 거부되면 사다리를 한 칸씩 내려감. 이어쓰기 형태가 거부되면 본문 그대로 + 토큰, 그것도 거부되면 토큰을 포기하고 재시도. 단 `"redemption temporarily unavailable"` 메시지는 **일시적 거부라서 사다리를 내려가면 안 되고**, 5분 창 안에서 같은 토큰으로 다시 보내는 것이 맞음.

여기에 돈이 걸린 예외가 하나 있음. **거절된 요청이 서버 툴을 이미 실행했다면 토큰 없는 재시도는 그 툴을 다시 돌리고 다시 청구함.** 문서는 그 경우 조용히 재시도하지 말고 400 을 호출자에게 올리라고 적음. 자동 재시도 코드를 짤 때 이 분기를 빼면 비용이 눈에 안 보이는 곳에서 두 배가 됨.

## sticky routing

한 번 fallback 이 일어난 대화는, 이후 `fallbacks` 를 포함한 요청이 **원래 모델을 건너뛰고 그 fallback 모델로 바로 감.** 매 턴 예측 가능한 거절을 사서 물지 않게 하는 장치임.

- 유지 기간은 **약 1시간**이고 조직 범위임
- 저장되는 것은 **대화 prefix 의 콘텐츠 해시 + 서비스한 모델**이고, 메시지 내용 자체는 저장되지 않음
- **best-effort 임.** 원래 모델이 언제든 다시 시도될 수 있다는 것을 코드가 감당해야 함
- 스트리밍에서는 스트림이 열리기 전에 결정되므로 `message_start` 의 `model` 이 이미 fallback 모델 ID 임. 이 턴에는 `fallback` 콘텐츠 블록이 없으니 `usage.iterations` 의 `fallback_message` 항목으로 식별해야 함

## 앱에 붙일 때 실제로 걸리는 것

iOS 앱이 자체 백엔드를 거쳐 Claude 를 부르는 구성을 전제로, 이 문서에서 코드로 옮겨야 하는 항목만 모으면 이럼.

- **거절은 성공 응답이라 에러 핸들링 경로에 안 걸림.** `URLSession` 이든 서버 프록시든, 성공 파서 안에서 `stop_reason` 을 봐야 함. 에러 분기에 거절 처리를 붙여 두면 영원히 안 불림
- **부분 출력을 버리는 것은 fallback 없이 거절로 끝난 경우뿐임.** 거절로 끝났으면 흘러나온 텍스트는 불완전으로 취급하라는 것이 문서 지시고, 스트리밍 UI 는 이미 그려 놓은 글자를 치우는 동작을 따로 만들어야 함. 반대로 **같은 스트림에서 fallback 이 성공하면 이미 그린 글자를 치우면 안 됨.** 문서가 "nothing you have already received is invalidated" 라고 못박고, fallback 모델은 그 부분 출력을 **이어서** 씀. 지우면 완성된 답의 앞부분이 사라짐
- **스트리밍과 비스트리밍의 결과가 다름.** 비스트리밍에서 mid-output 거절이 나면 API 가 부분 출력을 버리고 fallback 모델이 처음부터 답함. 스트리밍에서는 같은 스트림에서 이어지고 **부분 출력의 `text` 블록만** fallback 모델에 컨텍스트로 넘어감. 같은 기능을 두 방식으로 굴리면 동작이 갈리므로 하나로 고정하는 편이 나음
- **다음 턴에 돌려보낼 때 블록을 골라야 함.** `fallback` 블록은 **위치까지 그대로** 유지해야 하고(API 가 그 위치로 주변 thinking 블록을 검증함), 마지막 `fallback` 블록보다 앞에 있는 `thinking`·`redacted_thinking`·`connector_text` 와 클라이언트측 `tool_use` 는 떨어내야 함
- **재시도 예산은 요청 단위로 잡을 것.** 한 턴에서 거절이 여러 번 날 수 있음(에이전트와 그 서브에이전트). 그리고 **`fallbacks` 는 툴 실행 안에서 부르는 모델 호출로 전파되지 않음.** 서브에이전트 호출에는 따로 붙여야 함
- **fallback 설정을 요청의 속성으로 둘 것.** 공유 플래그나 캐시된 설정값으로 두면 어긋나서, 보호가 가장 필요한 재시도 경로에서 조용히 빠짐

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 캐시가 모델별이고 write 가 read 보다 비싸다는 전제. fallback credit 은 모델을 갈아탈 때 그 write 차액을 없애는 장치임 |
| [서버측 compaction](2026-09-20-claude-api-compaction.md) | `usage.iterations` 를 안 더하면 과소 보고된다는 같은 함정. 그 배열에 이제 거절 시도와 fallback 시도까지 들어옴 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 거절 분류기가 붙은 모델 중 하나. 그 모델의 thinking 블록은 다른 fallback 모델로 넘어가지 않아 API 가 떨어뜨림 |
| [대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md) | 레버 3 의 가시성 대시보드. 거절은 200 이라 에러율 기반 계기판에 안 잡히면서 청구는 됨 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | 프롬프트에서 무엇을 지울지. `reasoning_extraction` 은 프롬프트 문구가 만들어 내는 거절이고 이제 청구 대상임 |

## 짚어야 할 것

- **오탐률 수치가 없음.** 문서는 세 범주를 "low volumes of false positives" 로만 적고 숫자를 주지 않음. 이번 변경으로 청구가 얼마나 늘어나는지, 거절 자체가 얼마나 자주 나는지에 대한 값도 **원문에서 찾을 수 없었음**
- **청구되는 범주는 바뀔 수 있다고 문서가 직접 적음.** 위 표를 코드에 하드코딩해서 "이 범주는 무료" 같은 분기를 만들면 안 됨. 문서의 Billed 열이 단일 출처임
- **`cyber` 는 청구되지 않지만 빈도가 낮다는 뜻은 아님.** 정상적인 보안 작업도 이 범주에 걸릴 수 있다고 문서가 적음. 보안 관련 기능을 붙였으면 거절 자체는 자주 볼 수 있고, 다만 출력 전 거절이면 돈은 안 나감
- **서버측 기본 라우팅이 어느 모델을 고르는지 공개되지 않음.** 문서는 라우팅이 서버측이고 Models API 에 모델별로 공개되지 않는다고 적음. 무엇이 서비스했는지는 응답의 최상위 `model` 필드와 `usage.iterations` 의 `fallback_message` 항목으로만 확인함. 문서 예시에 나오는 모델은 Claude Opus 4.8 임
- **허용 타깃 목록을 직접 확인하지 못했음.** 문서가 명시한 것은 Fable 5.1 과 Fable 5 의 타깃이 `claude-opus-4-8` 과 `claude-opus-5` 라는 것까지임. 다른 모델의 목록은 Models API 를 호출해야 하고 이 문서에서는 하지 않았음
- **`fallback_has_prefill_claim` 이 없을 수 있음.** Bedrock, Google Cloud, Microsoft Foundry 는 이 필드 지원이 롤아웃 중이라 부재할 수 있고, 그때는 `false` 로 취급하지 말고 이어쓰기 형태를 먼저 시도한 뒤 거부 처리에 맡기라는 것이 문서 지시임
- **한 조합에서는 크레딧을 어떤 형태로도 리딤할 수 없음.** `output_config.format` 이나 툴 사용을 강제하는 `tool_choice` 를 쓴 요청에서 **서버 툴 실행 뒤에** 거절이 나면, 이어쓰기도 본문 그대로도 막힘. 이때는 토큰을 버려야 하고 그 재시도는 툴을 다시 청구함
- 같은 릴리스 노트 날짜에 올라온 다른 항목(Compliance API 쪽 변경)은 이 문서 범위 밖이고 확인하지 않았음

## 유효기간

**2026-09-27 확인 기준**임. 거절 응답의 모양과 청구 규칙은 문서 본문에 박힌 값이지만, **서버측 fallback(`server-side-fallback-2026-07-01`)과 fallback credit(`fallback-credit-2026-07-01`)은 둘 다 베타**라 파라미터와 필드 이름이 움직일 수 있음. 다시 볼 때 확인할 것 셋. **청구되는 범주 목록이 바뀌었는지**(문서가 바뀔 수 있다고 예고한 부분이고 표의 Billed 열이 단일 출처임), **`fallbacks` 가 베타를 벗어나고 Bedrock·Google Cloud·Microsoft Foundry 로 확대됐는지**, 그리고 **크레딧 토큰의 5분 창과 sticky routing 의 1시간이 그대로인지**. 앞의 둘은 [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서 먼저 잡힘.
