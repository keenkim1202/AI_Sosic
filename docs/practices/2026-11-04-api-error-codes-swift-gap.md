---
title: API 에러 코드 125개, 그런데 Swift 경로는 type 과 message 둘만 받음
source: https://platform.claude.com/docs/en/api/errors
author: Anthropic
published: 2026-11-04
collected: 2026-11-25
tags: [claude, anthropic, swift, ios, foundation-models, migration]
---

출처: [Claude API errors](https://platform.claude.com/docs/en/api/errors) · [Handle errors](https://platform.claude.com/docs/en/api/handle-errors) · [`ErrorDetails` (API 레퍼런스 Shared)](https://platform.claude.com/docs/en/api/$shared) · [API 릴리스 노트 2026-11-04](https://platform.claude.com/docs/en/release-notes/api) · [SDKs, CLI, and libraries](https://platform.claude.com/docs/en/cli-sdks-libraries/overview) · [Apple Foundation Models 라이브러리](https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/apple-foundation-models) (전부 **2026-11-25 확인**) · Swift 쪽은 [`ClaudeForFoundationModels`](https://github.com/anthropics/ClaudeForFoundationModels) 의 `main` 소스를 `gh api` 로 직접 받아 읽었음(마지막 푸시 **2026-10-07**, 최신 릴리스 **0.2.3**, 2026-11-25 확인)

## 요약

**2026-11-04 에 공식 SDK 에 에러 코드가 들어갔음.** 응답 본문을 파싱하지 않고도 에러의 한 가지 원인만 골라 잡을 수 있게 된 변경이고, 각 코드가 **자기 이름의 예외 클래스**를 가짐(Go 는 상수). 코드는 `error.details.error_code` 에 들어오고, 레퍼런스의 `ErrorDetails` 합집합에 지금 **125개**가 올라와 있음. 그중 **20개는 `error_code` 외에 기계가 읽을 필드를 같이 들고 옴.**

이 변경의 요점은 새 기능이 아니라 **`type` 하나로 뭉쳐 있던 원인이 갈라졌다**는 것임. 429 `rate_limit_error` 가 대표 사례로, 레이트 리밋과 **월 지출 상한 초과(`enforced_spend_limit_reached`)** 가 같은 `type` 으로 왔음. 전자는 기다리면 풀리고 후자는 **접근이 복구될 때까지 계속 실패**하므로, 둘을 못 가르면 재시도 루프가 의미 없이 돌게 됨.

그런데 **이 저장소 주인이 쓰는 경로에는 이 변경이 안 닿음.** 공식 클라이언트 SDK 는 Python·TypeScript·C#·Go·Java·PHP·Ruby **7종이고 Swift 가 없음.** iOS 에서 쓰는 [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md) 는 범용 Messages API 클라이언트가 아니고, 소스를 열어 보면 에러 본문에서 **`type` 과 `message` 만 디코드하고 `details` 를 버림.** 즉 Swift 앱에서는 위의 429 두 종류가 `LanguageModelError.rateLimited` 하나로 합쳐져 도착함.

## `error_code` 가 `type` 과 무엇이 다른가

에러 본문의 모양 자체는 그대로임. 최상위 `error` 에 `type` 과 `message` 가 항상 있고 응답에 `request_id` 가 붙음. 여기에 **일부 에러만** `error.details.error_code` 를 더 가짐.

```json
{
  "type": "error",
  "error": {
    "type": "rate_limit_error",
    "message": "You have reached your API usage limits: ...",
    "details": { "error_code": "enforced_spend_limit_reached" }
  },
  "request_id": "req_018EeWyXxfu5pfWkrYcMdjWG"
}
```

갈라지는 방향이 이럼. `type` 은 **넓은 범주**이고 `error_code` 는 **그 안의 한 가지 원인**임. 문서가 못 박아 둔 운용 규칙 셋.

- **`message` 를 코드로 매칭하지 말 것.** 레퍼런스가 `message` 필드 설명에 "Its wording can change, so don't match on it" 이라고 적어 둠
- **코드 집합은 늘어남.** 코드가 없거나 모르는 코드면 `type` 이나 HTTP 상태로 처리하는 게 맞음
- **같은 429 를 돌려주는 다른 API 는 코드를 안 붙임.** 문서가 예로 든 것이 Claude Managed Agents 임. 거기서는 `message` 밖에 구분 수단이 없고, 그런데 `message` 는 매칭하면 안 됨. 즉 **그쪽은 가를 방법이 없다는 뜻**임

## 좁은 분기를 넓은 분기보다 앞에 둘 것

SDK 에서 코드별 클래스는 상태 코드 클래스의 **서브클래스**임. Python 기준으로 `RateLimitError` 가 `EnforcedSpendLimitReachedError` 까지 잡음. 429 분기를 먼저 쓰면 지출 상한도 거기로 빨려 들어감.

```python
except anthropic.EnforcedSpendLimitReachedError as e:
    # 코드 하나: enforced_spend_limit_reached
    print("Monthly spend cap reached:", e.api_message)
except anthropic.RateLimitError as e:
    # 그 밖의 모든 429
    print("Rate limited. Request ID:", e.request_id)
except anthropic.APIConnectionError:
    # 응답이 없는 실패. 네트워크 끊김, 타임아웃
    print("Connection failed")
except anthropic.APIStatusError as e:
    # 그 밖의 모든 에러 응답
    print(e.status_code, e.type, e.api_message, e.error_code, e.details, e.request_id)
```

에러가 코드에 닿는 시점은 **SDK 자체 재시도가 끝난 뒤**임. 기본값이 **최대 2회 재시도(총 3회 시도)**이고, 재시도 여부는 응답의 `x-should-retry` 헤더가 있으면 그것을 따르고 없으면 연결 에러·타임아웃·408·409·429·500 이상을 재시도함. 대기 시간은 `retry-after` 가 있으면 그만큼, 없으면 **0.5초에서 8초 사이 지수 백오프**임.

여기서 지출 상한이 걸리는 이유가 분명해짐. 기본 설정은 `x-should-retry` 가 없는 429 를 **자동으로 재시도**하고, 지출 상한 429 는 **접근이 복구될 때까지 계속 같은 429** 를 돌려줌. 문서가 "limit your retries after a 429" 로 적은 자리가 이것임.

스트림은 또 다름. 스트림이 시작된 뒤에는 이미 200 이 나갔으므로 에러가 **error 이벤트**로 옴. SDK 는 재시도하지 않고, 그 에러의 HTTP 상태는 **스트림의 200 이거나 비어 있음.** 그래서 **상태별 분기도, 코드별 클래스 분기도 스트림 에러를 못 잡음.** `error.type` 을 직접 보거나, 코드가 붙어 있으면 `error_code` 필드를 읽어 문자열로 비교해야 함.

## 코드가 데이터를 들고 오는 경우

125개 중 **20개**가 `error_code` 말고 다른 필드를 같이 실음. 이게 이 변경에서 실제로 코드를 줄여 주는 부분임. 앱 쪽에 걸리는 것들.

| 코드 | 같이 오는 필드 | 쓰임 |
|---|---|---|
| `image_dimensions_too_large` | `max_side_length_pixels`, `paths` | 넘긴 상한값과 **넘긴 이미지의 위치 전부**(`messages.0.content.1` 형태, `tool_result` 안의 중첩까지) |
| `thread_unsupported_request` | `unsupported` | thread 와 같이 못 쓰는 필드 이름 |
| `organization_on_hold` | `appeal_url` | 이의 제기 경로 |
| `workspace_policy_violation`, `approval_required` 등 | `field` | 문제가 된 필드 |

첫 줄이 이 저장소의 미해결 항목 하나를 푼 자리임. [요청 크기 상한 128MB](./2026-10-28-request-size-and-image-limits.md)가 **"many-image 임계값의 실제 픽셀 상한이 문서에 없고 에러 메시지로만 알 수 있다"**로 남겨 뒀는데, 이제 그 상한이 `max_side_length_pixels` 로 **구조화된 필드**로 옴. 레퍼런스의 필드 설명이 "the largest width or height, in pixels, that an image **on this request** may have" 임. 요청별로 달라지는 값, 즉 많은 이미지를 넣었을 때 좁아지는 그 상한을 가리킴. 문서에 숫자가 박힌 것은 아니지만, **런타임에 읽어서 그 값으로 리사이즈하고 재시도하는 코드를 쓸 수 있게 됨.** `paths` 가 어느 블록인지까지 알려주므로 전체를 다시 줄일 필요도 없음.

다만 이 코드가 가리키는 것이 하드 상한(8,000px)인지 many-image 로 좁아진 상한인지는 **레퍼런스 문언만으로 구분되지 않음.** "the image size limit" 으로만 적혀 있음.

## 이 저장소가 쫓던 기능들의 에러 코드

125개 대부분은 조직 관리·스토리지·플러그인 쪽이라 이 저장소 범위 밖임. 쓸모 있는 것만 추리면 **이 저장소가 이미 문서로 정리해 둔 기능마다 코드가 하나씩 붙어 있음.**

| 코드 | 레퍼런스 설명 요지 | 관련 문서 |
|---|---|---|
| `enforced_spend_limit_reached` | 조직이 API 티어의 월 사용 임계값을 넘음 | [Usage and Cost Analytics API](./2026-10-29-usage-cost-analytics-api.md) |
| `thread_fingerprint_mismatch` | thread 의 턴 사이에서 고정돼야 하는 필드가 저장된 값과 다름 | [Message threads](https://platform.claude.com/docs/en/build-with-claude/message-threads) (원문 문서) |
| `thread_already_continued` | `previous_message_id` 가 가리키는 응답은 **한 번만** 이어갈 수 있음 | 위와 같음 |
| `compaction_signature_invalid` | `compaction` 블록의 `signature` 검증 실패 | [서버측 compaction](./2026-09-20-claude-api-compaction.md) |
| `compaction_unavailable` | 일시적 서버 사유. 재시도하라고 명시 | 위와 같음 |
| `user_profile_required` | `billing_policy.bill_to` 가 `"user_profile"` 인데 프로필을 안 넘김 | [user profiles](./2026-10-20-user-profiles-end-customer-attribution.md) |
| `linked_organization_spend_limit_reached` | 연결된 조직이 자기 지출 한도에 닿았거나 크레딧이 없음. **어느 쪽인지 응답이 말하지 않음** | 위와 같음 |
| `tool_reference_unresolved` | 툴 변경이나 MCP 목록이 그 위치에 정의되지 않은 툴을 참조 | [인라인 툴 정의](./2026-09-22-inline-tool-definitions.md) |
| `available_tools_limit_exceeded` | 메시지 뒤 사용 가능한 툴이 개수나 크기 상한을 넘음 | 위와 같음 |
| `request_body_too_large_after_decoding` | 압축 본문이 **압축을 푼 뒤** 상한을 넘음 | [요청 크기 상한 128MB](./2026-10-28-request-size-and-image-limits.md) |
| `request_body_timeout` | 본문이 제때 도착하지 못해 **아무것도 실행되지 않음** | 위와 같음 |

마지막 둘이 모바일에서 바로 걸리는 종류임. 요청 크기 문서가 128MB 라는 숫자만 다뤘는데, 실제로는 **압축 전후 크기와 업로드 속도**가 각각 별도 거절 사유로 올라와 있음. 느린 셀룰러에서 사진 여러 장을 올리면 상한을 안 넘고도 `request_body_timeout` 으로 떨어짐.

## Swift 경로에서는 코드가 사라짐

공식 클라이언트 SDK 목록에 **Swift 가 없음.** Python·TypeScript·C#·Go·Java·PHP·Ruby 7종이고, Apple 쪽은 "Libraries and integrations" 로 따로 분류돼 있음. 그 분류의 설명이 정확함. **Messages API 를 그대로 노출하는 클라이언트가 아니라 다른 프레임워크의 API 표면으로 Claude 를 꺼내는 패키지**임. 라이브러리 문서도 "**not** a general-purpose Messages API client" 라고 박아 둠.

문서에 적힌 에러 처리는 이 정도임. 컨텍스트 초과는 `.contextSizeExceeded`, 429 는 `.rateLimited`, 타임아웃은 `.timeout` 으로 매핑되고 대응이 없는 것은 `ClaudeError` 로 옴. 코드 얘기는 없음. 그래서 소스를 열었음.

`Sources/ClaudeAPI/APIError.swift` 의 디코더가 결정적임.

```swift
private enum CodingKeys: String, CodingKey { case type, message }
```

**`details` 가 `CodingKeys` 에 없음.** 즉 `error_code` 는 Swift 타입에 도달하기 전에 버려짐. `Kind` 는 `invalid_request_error`·`authentication_error`·`permission_error`·`not_found_error`·`request_too_large`·`rate_limit_error`·`api_error`·`overloaded_error` 여덟 개와 미래 호환용 `.other` 임. 즉 **이 패키지가 가진 분해능은 2026-11-04 이전의 `type` 단위 그대로임.**

`Sources/ClaudeForFoundationModels/ErrorMapper.swift` 가 그 `Kind` 로 분기함.

| `APIError.kind` | 앱에 도착하는 것 |
|---|---|
| `.rateLimit`, `.overloaded` | `LanguageModelError.rateLimited(resetDate: nil)` |
| `.requestTooLarge` | `LanguageModelError.contextSizeExceeded(contextSize: 0, tokenCount: 0)` |
| `.invalidRequest` 중 메시지에 `context` 가 들어간 것 | `.contextSizeExceeded(0, 0)` |
| `.authentication` | App Attest 면 `ClaudeError.attestationFailed`, 아니면 `.missingCredential` |
| `.permission`, `.api`, `.invalidRequest`, `.notFound`, `.other` | `APIError` 자체 |

여기서 실무에 바로 걸리는 것 셋.

**첫째, 지출 상한 429 와 보통 429 가 구분 불가임.** 둘 다 `resetDate: nil` 인 `.rateLimited` 로 옴. 주석이 그 선택을 밝혀 뒀음. API 가 복구 시점을 말해 주지 않으니 날짜를 **지어내지 않고** 호출자가 백오프를 정하게 둔다는 것. 판단으로는 맞지만, 결과적으로 앱은 **"기다리면 풀리는 것"과 "결제 상태가 바뀔 때까지 안 풀리는 것"을 같은 UI로 다루게 됨.** 라이브러리 문서가 권하는 패턴(`.rateLimited` 를 잡아 그 턴만 `SystemLanguageModel` 로 떨어뜨리기)은 두 경우 모두에 맞는 대응이라 그 자체로는 안전한데, **백오프 재시도로 처리하면 지출 상한에서 의미 없이 돌게 됨.**

**둘째, 매퍼가 `message` 문자열을 매칭함.** `.invalidRequest where error.message.lowercased().contains("context")` 가 그 줄임. API 레퍼런스가 같은 필드에 대해 "wording can change, so don't match on it" 이라고 적은 바로 그것을 하고 있음. 문구가 바뀌면 컨텍스트 초과가 조용히 `.contextSizeExceeded` 가 아니라 날 `APIError` 로 떨어짐. `error_code` 가 있으면 필요 없는 매칭이라, 이 줄은 **`details` 를 버린 대가**로 읽는 게 맞음.

**셋째, 매퍼가 돌려주는 `APIError` 를 앱이 분해할 수 없음.** `.permission`·`.api`·`.notFound` 등은 `APIError` 를 그대로 올리는데, 그 타입 선언이 `public` 이 아니라 **`package struct APIError`** 임. 패키지 밖에서는 `catch let e as APIError` 가 안 됨. `LocalizedError` 는 따르므로 `localizedDescription` 으로 `"\(kind.rawValue): \(message)"` 문자열은 읽히는데, 그러면 다시 **문자열 매칭**으로 돌아감.

`contextSizeExceeded(contextSize: 0, tokenCount: 0)` 도 같은 계열임. 주석이 솔직하게 적어 뒀음. API 가 창 크기도 프롬프트 토큰 수도 안 돌려주니 0 을 넣는다는 것. 프레임워크 타입의 필드를 채우려면 값이 필요한데 출처가 없는 경우고, **앱이 그 숫자를 믿고 분기하면 안 되는 자리**임.

## 그래서 iOS 에서 무엇을 하면 되나

셋 중 하나임.

1. **프록시를 쓰는 중이면 코드 분기를 프록시에 둠.** `.proxied` 구성은 자기 백엔드가 Messages API 응답을 그대로 받으므로, 그쪽이 7종 SDK 중 하나를 쓰면 코드별 클래스를 그대로 쓸 수 있음. 앱에는 이미 해석된 결과만 내려보내면 됨. 지출 상한처럼 **재시도가 무의미한 경우를 앱이 알아야 하는 자리**가 이것으로 해결됨
2. **App Attest 로 백엔드 없이 가는 중이면 지금은 방법이 없음.** 요청이 앱에서 Claude API 로 직접 가고 응답 본문은 패키지가 디코드하므로, `error_code` 를 볼 지점이 앱 코드에 없음. 재시도 정책을 보수적으로(횟수를 적게, 429 뒤에는 사람이 한 번 개입하도록) 두는 쪽이 맞음
3. **URLSession 으로 Messages API 를 직접 호출하는 코드가 있으면 거기서는 그냥 읽으면 됨.** `error.details.error_code` 는 그냥 JSON 필드임. SDK 가 주는 것은 타입 안전성이지 접근 경로가 아님

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md) | 그 문서가 "에러는 `LanguageModelError` 로 매핑되는 것만 매핑됨"으로 한 줄 적은 지점. 여기서 소스를 열어 **무엇이 디코드 단계에서 버려지는지**까지 내려감 |
| [요청 크기 상한 128MB](./2026-10-28-request-size-and-image-limits.md) | many-image 픽셀 상한이 `max_side_length_pixels` 로 구조화돼 나오게 됨. 그 문서의 미해결 항목 하나가 런타임 쪽에서 풀림 |
| [API 버전 2026-10-21](./2026-10-21-claude-api-version-migration.md) | 에러 코드는 버전 변경이 아니라 **SDK 메이저 안에서의 추가**임. `type` 값과 `details` 내용은 버전 정책상 확장될 수 있는 쪽으로 명시돼 있음 |
| [거절 응답과 청구](./2026-09-24-refusal-billing-and-fallback.md) | 거절은 200 으로 오는 정상 응답이고 여기 것은 에러 응답임. 같은 "원하는 답이 안 온 경우"지만 처리 경로가 다름 |
| [user profiles 와 최종 고객 귀속](./2026-10-20-user-profiles-end-customer-attribution.md) | `user_profile_*` 와 `linked_organization_*` 코드가 그 문서의 설정 실수를 가리키는 자리 |
| [서버측 compaction](./2026-09-20-claude-api-compaction.md) | `compaction_*` 코드 9개가 그 문서가 다룬 블록 왕복의 실패 모양임 |

## 짚어야 할 것

- **125 라는 수는 레퍼런스의 `ErrorDetails` 합집합을 센 것임**(2026-11-25). 문서가 "집합은 늘어난다"고 명시했으므로 이 숫자는 셈한 날짜와 함께만 의미가 있음. 세는 방법은 `$shared` 페이지에서 `error_code: "..."` 를 유일값으로 세면 됨
- **직접 호출해 본 것이 아니라 문서와 소스만 읽었음.** 실제 429 응답 본문이나 `max_side_length_pixels` 의 실제 값은 확인하지 못했음
- **`enforced_spend_limit_reached` 가 App Attest 경로에서도 오는지는 확인할 수 없었음.** 레퍼런스가 "Messages API 가 돌려준다"로 적고 App Attest 토큰은 Messages API 호출만 인가하므로 올 수 있는 모양이지만, 문서가 그 조합을 따로 적지는 않았음
- **Swift 패키지 쪽은 `main` 브랜치 소스임.** 마지막 푸시가 최신 릴리스 0.2.3 과 같은 시점(2026-10-07)이라 릴리스 내용과 같다고 봤지만, 태그를 체크아웃해 대조한 것은 아님
- **`package` 접근 수준 때문에 앱이 `APIError` 를 못 잡는다는 것은 선언에서 읽은 것임.** 실제로 컴파일해 확인하지 않았음
- 매퍼가 `message` 를 매칭하는 것은 **이 패키지가 `details` 를 안 받는 구조의 결과로 읽은 것**이고, 패키지 저자들이 그 이유를 적어 둔 것은 아님. 다른 이유가 있을 수 있음
- 이 문서가 비교한 것은 **문서에 적힌 표면**임. 7종 SDK 가 실제로 125개 클래스를 다 내보내는지는 확인하지 않았음. 문서 자신이 "클래스가 다른 네임스페이스에 있을 수 있으니 소스를 검색하라"고 적어 둠

## 유효기간

**2026-11-25 확인 기준이고 기능 자체는 2026-11-04 항목임.** 코드가 늘어난다는 것이 문서에 박혀 있으므로 **위의 표를 갱신하려 들지 말고** `$shared` 레퍼런스에서 새로 받는 편이 맞음. 다시 볼 때 확인할 것 셋. **`ClaudeForFoundationModels` 의 `APIError` 가 `details` 를 디코드하게 됐는지**(`CodingKeys` 에 `details` 가 추가되는지가 단일 신호임), **공식 클라이언트 SDK 목록에 Swift 가 들어왔는지**, 그리고 **`image_dimensions_too_large` 의 설명이 하드 상한과 many-image 상한을 구분하게 됐는지.** 첫째가 가장 값어치 있는 변경이고, 그때 [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md)와 이 문서를 같이 고쳐야 함. 패키지가 베타를 벗어났는지도 같이 볼 것.
