---
title: Foundation Models 가용성 대조, 베타 딱지는 떨어졌고 PCC 조건은 셋이 됨
source: https://developer.apple.com/documentation/updates/foundationmodels
author: Apple
collected: 2026-09-28
tags: [apple, ios, foundation-models, on-device-llm, swift, watchos, context-window]
---

출처: [Foundation Models updates](https://developer.apple.com/documentation/updates/foundationmodels) · [Private Cloud Compute (개발자 페이지)](https://developer.apple.com/private-cloud-compute/) · [`com.apple.developer.private-cloud-compute`](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.developer.private-cloud-compute) · 심볼 페이지 [`LanguageModel`](https://developer.apple.com/documentation/foundationmodels/languagemodel) · [`PrivateCloudComputeLanguageModel`](https://developer.apple.com/documentation/foundationmodels/privatecloudcomputelanguagemodel) · [`SystemLanguageModel`](https://developer.apple.com/documentation/foundationmodels/systemlanguagemodel) · [`Attachment`](https://developer.apple.com/documentation/foundationmodels/attachment) · [`DataAttachmentRepresentable`](https://developer.apple.com/documentation/foundationmodels/dataattachmentrepresentable) (전부 **2026-09-28 확인**)

## 요약

이 저장소의 Apple 문서 둘이 답을 못 내고 남겨둔 항목이 하나 있음. [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md)이 "PCC·Core AI·dynamic profiles·attachments는 전부 Beta 표시"라고 적었고, [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)는 What's New 문구까지만 대조하고 **프레임워크 문서의 가용성 표시는 대조하지 못했다**고 밝힘. 그것을 심볼 페이지의 가용성 주석으로 직접 확인한 것이 이 문서임.

결과는 셋으로 갈림. 첫째, **iOS 27.0 표면은 베타가 아님.** `LanguageModel`, `PrivateCloudComputeLanguageModel`, `DynamicProfile`, `Attachment`, `LanguageModelCapabilities` 가 모두 `27.0` 도입에 베타 표시 없음. 둘째, **베타 딱지는 사라진 게 아니라 27.2 베타로 옮겨갔음.** 지금 베타인 것은 데이터 첨부 경로 하나(`DataAttachmentRepresentable`, `DataEntryRepresentable`, `supportsDataAttachmentType(_:)`)이고 도입 버전이 **27.2**임. 셋째, **PCC 무료 구간의 조건이 둘이 아니라 셋임.** Small Business Program 가입과 다운로드 200만 미만에 **엔타이틀먼트 할당**이 더 붙고, 조건을 벗어나면 **6개월** 안에 대체 수단으로 옮겨야 함.

덤으로 확인된 것 둘. **watchOS 는 세션은 되는데 온디바이스 시스템 모델이 가용성 목록에 없음.** 그리고 온디바이스 모델은 **OS 업데이트로 교체됨**을 문서가 명시하고 버전 목록을 셋으로 나눠 둠.

## 가용성 주석으로 본 27.0 표면

심볼 페이지의 플랫폼 배열에서 읽은 값임. 표시가 없는 칸은 그 플랫폼이 배열에 없다는 뜻임.

| 심볼 | 도입 | 베타 | watchOS |
|---|---|---|---|
| `LanguageModel` (프로토콜) | 27.0 | 아님 | 27.0 |
| `PrivateCloudComputeLanguageModel` | 27.0 | 아님 | 27.0 |
| `LanguageModelSession.DynamicProfile` | 27.0 | 아님 | 27.0 |
| `LanguageModelCapabilities` | 27.0 | 아님 | 27.0 |
| `Attachment` | 27.0 | 아님 | 27.0 |
| `LanguageModelSession` | 26.0 | 아님 | **27.0** |
| `SystemLanguageModel` | 26.0 | 아님 | **배열에 없음** |
| `DataAttachmentRepresentable` | **27.2** | **베타** | 27.2 |
| `DataEntryRepresentable` | **27.2** | **베타** | 27.2 |
| `LanguageModel.supportsDataAttachmentType(_:)` | **27.2** | **베타** | 27.2 |

실무 결론은 단순함. **`#available(iOS 27, *)` 분기 안에서 PCC 와 dynamic profiles 를 쓰는 코드는 이제 베타 API 에 의존하지 않음.** 반대로 임의 타입을 첨부로 넘기는 코드를 쓰려면 27.2 SDK 가 필요하고, 그건 아직 베타임.

## 베타 딱지가 27.2 로 옮겨간 것

`Attachment` 자체는 27.0 정식이고 이미지 첨부 이니셜라이저(`init(_:orientation:)`, `init(imageURL:orientation:)`)가 그 안에 있음. 27.2 베타에서 새로 생긴 것은 **이미지가 아닌 임의 데이터를 첨부로 넘기는 경로**임.

- `Attachment.init(_:)` 가 `Content` 가 `DataAttachmentRepresentable` 을 따를 때 쓸 수 있는 형태로 추가됨
- `DataAttachmentRepresentable` 은 `transcriptRepresentation`(타입 `Transcript.DataAttachment`)과 그것으로부터 되살리는 `init(_:) throws` 둘을 요구함. 즉 **첨부가 트랜스크립트에 실려 왕복하는 것**이 설계 전제임
- 모델이 그 타입을 받는지는 `supportsDataAttachmentType(_:)`, `supportsDataEntryType(_:)` 로 **런타임에 물어봐야 함.** `LanguageModel` 프로토콜 요구사항에 들어갔고 `async throws -> Bool` 임

여기서 볼 것은 기능이 아니라 **형태**임. 첨부 타입 지원 여부가 컴파일 타임 상수가 아니라 모델에 던지는 질의로 들어왔음. `LanguageModel` 프로토콜이 서드파티 백엔드까지 받는 구조라 그럴 수밖에 없는데, 그 대가로 첨부를 쓰는 코드마다 능력 확인 분기가 하나 늘어남. [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md)이 적은 `capabilities.contains(.reasoning)` 확인과 같은 계열의 비용임.

## PCC 무료 구간의 조건은 셋

개발자 페이지가 자격 요건을 이렇게 적음.

> "Access to PCC is available to developers who meet the following criteria:
> * Are enrolled in the App Store Small Business Program.
> * Have fewer than 2 million first-time app downloads from any of their apps on the App Store.
> * Have the Private Cloud Compute entitlement assigned to their account."

이 저장소가 지금까지 인용해 온 것은 앞의 둘이었음. [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md)과 [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)가 인용한 What's New 문구에 세 번째가 없기 때문임. 엔타이틀먼트 자체는 [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md)이 `com.apple.developer.private-cloud-compute` 로 이미 적어뒀는데, 그것이 **개발 단계의 승인 절차가 아니라 무료 구간 자격 요건의 한 항목**이라는 것이 이번에 확인된 형태임. 엔타이틀먼트 문서도 managed entitlement 라서 접근 요청이 필요하다고 적고 개발자 페이지로 넘김.

일정에 직접 걸리는 항목 둘이 같은 페이지에 있음.

> "If any app subsequently exceeds the 2 million first-time downloads threshold, or the developer is no longer enrolled in the App Store Small Business Program, the developer will be notified and must migrate to an alternative solution within 6 months."

**6개월**임. 앱이 잘되면 PCC 를 걷어내고 다른 추론 경로를 붙이는 작업이 반년 안에 생긴다는 뜻이고, 그 시점에 비용 구조가 0 에서 유료로 바뀜. PCC 를 전제로 기능을 설계할 때 **대체 백엔드를 실제로 붙일 수 있는 구조인지**가 선택이 아니라 요건이 됨. `LanguageModel` 프로토콜로 모델을 갈아끼울 수 있게 만들어 두는 것이 여기서 값을 함.

> "Where Apple Intelligence is available, eligible developers can use PCC in their apps distributed on the App Store, and test PCC features via TestFlight or ad hoc distribution. Installs during testing are not counted as first-time app downloads."

**TestFlight 와 ad hoc 설치는 200만 카운트에 안 들어감.** 베타 테스터를 늘려서 한도가 앞당겨지는 일은 없음.

한도 쪽 API 는 `PrivateCloudComputeLanguageModel.QuotaUsage` 로 정리돼 있고 `isLimitReached`, `limitIncreaseSuggestion`, `status`, `resetDate` 를 가짐. 문서가 **쿼터와 가용성은 서로 직교한다**고 명시함. 한도를 다 써도 모델은 여전히 available 로 보고됨. 즉 `availability` 만 보고 분기하는 코드는 한도 초과를 못 잡음.

## watchOS 는 세션만 오고 온디바이스 모델은 안 옴

`LanguageModelSession` 의 플랫폼 배열에 watchOS 가 **27.0** 으로 들어 있음. `SystemLanguageModel` 의 배열에는 watchOS 가 **없음**(iOS·iPadOS·Mac Catalyst·macOS·visionOS 26.0 만). `PrivateCloudComputeLanguageModel` 과 `LanguageModel` 은 watchOS 27.0 에 있음.

읽히는 그림은 이럼. **watchOS 에서 Foundation Models 세션을 열 수는 있지만 그 세션에 물릴 모델은 온디바이스 시스템 모델이 아님.** PCC 나 직접 구현한 `LanguageModel` 이 대상이 됨. 워치에서 온디바이스로 도는 것을 전제로 기능을 잡아뒀다면 여기서 설계가 갈림. 다만 Apple 이 이것을 제약으로 서술한 문장은 찾지 못했고, 위는 가용성 주석에서 읽은 것임.

## 온디바이스 모델은 OS 업데이트로 교체됨

`SystemLanguageModel` 문서가 이렇게 적음.

> "Apple periodically updates `SystemLanguageModel` in routine OS updates to improve the on-device model's abilities and performance."

그리고 모델 버전 목록을 셋으로 나눠 둠. **26.0 부터 26.3**, **26.4**, **27.0**. updates 페이지는 한 걸음 더 나감.

> "Because the model changes when a person updates to iOS 27, iPadOS 27, macOS 27, and visionOS 27, test your prompts with the new model to verify your app's behavior."

이것이 실무에 주는 것은 경고가 아니라 **작업 항목**임. 프롬프트가 OS 마이너 업데이트에 묶인 의존성을 갖는다는 뜻이고, 회귀를 잡으려면 프롬프트 출력에 대한 고정 케이스가 있어야 함. [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md)이 말한 "게이트는 확신이 아니라 틀렸을 때의 통증으로 연다"가 여기 그대로 걸림. 온디바이스 모델을 쓰는 기능은 **내가 아무것도 안 바꿔도 아래가 바뀌는** 쪽이라 통증 쪽으로 기울어 있음.

컨텍스트 창 쪽은 이번에도 못 풀었음. `contextSize` 는 `SystemLanguageModel` 과 `PrivateCloudComputeLanguageModel` 양쪽에 `Int` 프로퍼티로 있고 설명은 "The maximum context size in tokens supported by the model." 인데, **토큰 수가 문서에 박혀 있지 않음.** [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md)의 4K / 32K 는 베타 문서 기준 값 그대로 남아 있고, 모델이 OS 업데이트로 교체된다는 위 문장과 겹쳐 보면 **런타임에 `contextSize` 를 읽는 편이 안전하다**는 그 문서의 결론이 더 강해짐.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md) | 그 문서가 Beta 로 적어둔 네 항목의 현재 표시를 대조한 것. 4K / 32K 는 여전히 미확인 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 그 문서가 "프레임워크 문서로 다시 확인해야 함"으로 남긴 항목 |
| [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md) | PCC 무료 구간을 조건 둘로 요약한 지점. 세 번째 조건이 여기서 붙음 |
| [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md) | 모델이 OS 업데이트로 바뀌는 의존성을 무엇으로 막을지 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md) | 6개월 안에 대체 백엔드로 옮기는 경우 서드파티 전송 공개 요건이 새로 생김 |

## 짚어야 할 것

- **이 문서가 확인한 것은 가용성 주석과 자격 요건 문언까지임.** 베타 표시가 떨어졌다는 것이 동작이 바뀌지 않는다는 보장은 아님. 27.0 에서 시그니처가 고정됐다는 것까지로 읽어야 함
- **watchOS 에서 온디바이스 모델을 못 쓴다는 것은 가용성 배열에서 읽은 추론임.** Apple 이 그것을 제약으로 적은 문장은 찾지 못했음
- `GenerationOptions.ToolCallingMode` 는 updates 페이지가 iOS 27 신규 API 로 적었는데 **심볼 페이지 경로를 찾지 못해 케이스와 가용성을 확인할 수 없었음**
- Core AI 와 오픈소스 프로바이더 패키지 쪽은 이 문서 범위 밖임. updates 페이지의 링크 목록만 확인했고 저장소는 열지 않았음
- 27.2 는 **베타임.** `DataAttachmentRepresentable` 의 도입 버전과 요구 멤버는 정식에서 바뀔 수 있음
- 가용성 주석은 심볼 페이지의 플랫폼 배열에서 읽었음. 배열에 플랫폼이 없는 것과 명시적으로 unavailable 로 표시된 것을 문서가 화면에서 구분해 주지 않으므로, `SystemLanguageModel` 의 watchOS 항목은 **없음**으로만 적었음

## 유효기간

**2026-09-28 확인 기준**임. 베타 표시는 릴리스마다 움직이므로 다시 볼 때는 [Foundation Models updates](https://developer.apple.com/documentation/updates/foundationmodels) 에 27.2 절이 생겼는지부터 볼 것. 확인할 것 둘. **`DataAttachmentRepresentable` 계열이 27.2 정식에서 베타를 벗어났는지**, 그리고 **PCC 자격 요건 세 항목의 문언이 그대로인지.** 후자는 무료 구간 자체의 조건이라 바뀌면 기능 범위를 다시 잡아야 하는 종류의 변경임. 컨텍스트 토큰 수가 문서에 박히는지도 같이 볼 값어치가 있음.
