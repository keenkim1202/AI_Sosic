---
title: FaceTime Live Assistance API, 서드파티 자막·통역을 통화에 꽂는 확장점과 계약서가 그어둔 선
source: https://developer.apple.com/support/terms/apple-developer-program-license-agreement/
author: Apple
published: 2026-10-13
collected: 2026-10-14
tags: [apple, ios, app-store, compliance, privacy, swift]
---

출처: [Apple Developer Program License Agreement](https://developer.apple.com/support/terms/apple-developer-program-license-agreement/) (3.3.8(J)·3.3.11 전문 확인, 2026-10-14 기준) · [개발자 뉴스](https://developer.apple.com/news/)의 "Updated Apple Developer Program License Agreement now available" (2026-10-13 항목) · [LiveAssistanceExtension](https://developer.apple.com/documentation/livecommunicationkit/liveassistanceextension) · [LiveAssistance](https://developer.apple.com/documentation/livecommunicationkit/liveassistance) · [LiveCommunicationKit](https://developer.apple.com/documentation/livecommunicationkit)

## 요약

Apple이 **2026-10-13** 개발자 프로그램 라이선스 계약을 개정하면서 **3.3.8(J) Live Assistance API** 조항을 신설함. 같은 이름의 프레임워크 심볼이 `LiveCommunicationKit`에 **27.2 베타**로 올라와 있고, 둘을 붙여 읽으면 **남의 FaceTime 통화에 내 서비스를 참가자로 꽂는 확장점**이 생긴 것임. 자막이나 통역 기능을 앱으로 만들던 쪽에는 시스템 경로가 처음 열린 셈임.

다만 **계약서가 용도를 접근성으로 못 박음.** 정의 조항은 "FaceTime 통화에 연결하는 API"로 넓게 써 두고, 사용 조항에서 **접근성 서비스 제공자**가 **최종 사용자의 초대로** 들어가는 경우로 좁힘. 범용 AI 비서를 통화에 넣는 설계는 이 문언으로는 안 됨.

구조에서 제일 중요한 사실은 **확장이 받는 것이 통화 오디오가 아니라는 것**임. `LiveAssistanceRequest`가 가진 건 **`id`(UUID)와 `url`(FaceTime이 만든 URL) 둘뿐**이고, 통화에 실제로 참가하는 건 그 URL을 받아 든 **서비스 제공자**임. 즉 온디바이스 모델을 꽂는 자리가 아니고, 통화 내용이 내 서버로 나가는 구조임. 그래서 이 기능을 만들면 [App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md)의 **5.1.2(i)** 공개·동의 요건이 그대로 따라옴.

덤으로 같은 계약서에서 **3.3.11 AI·머신러닝 조항 전문**을 확인했음. 그 문서가 "공지 요약만 봤고 계약서 전문은 못 읽었음"으로 남겨둔 자리임. 아래에 문언을 그대로 옮겨 둠.

## 2026-10-13 개정에서 바뀐 것

공지가 나열한 항목은 여덟 개임. 이 문서에 걸리는 것만 표로 추림.

| 조항 | 공지의 서술 |
|---|---|
| **3.2** | "Updated security and related requirements for using Apple's software and services" |
| 정의, **3.3.8(J)** | "Specified terms for use of the Live Assistance API" |
| 정의, **3.3.8(K)** | "Specified terms for use of the App Replacement Entitlement" |

나머지는 Apple HomeView의 iOS 앱 위젯과 CarPlay(정의·6.4 계열), Group Purchases와 Volume Purchasing의 다중 좌석 구독(Schedule 2·3), 그리고 세무 정보 보고임. 이 저장소 주제에서 벗어나므로 다루지 않음. 번역본은 **한 달 안에** 올라온다고 공지가 적음.

⚠️ **3.2의 "보안 요건 갱신"이 무엇인지는 확인하지 못했음.** 공지 문장이 요약뿐이고 개정 전후를 비교할 수 있는 이력 문서를 찾지 못함. AI 기능과 직접 관련이 있는지도 알 수 없음.

## 계약서가 허용한 범위

정의 조항은 이렇게 적음.

> "**Live Assistance API**" means the Documented API that enables an Application or Corresponding Product to connect to an end user's FaceTime call.

그리고 사용 조항인 3.3.8(J) 전문임.

> **J. Live Assistance API**
>
> At an end user's invitation, Your Application or Corresponding Product may use the Live Assistance API solely for the purpose of enabling an accessibility service provider to join the end user's FaceTime call to provide accessibility services (e.g. captioning, translation, etc).

갈라 읽을 지점이 셋임.

- **`solely for the purpose of`.** 용도가 열거가 아니라 한정임. 괄호 예시가 자막과 번역이라 생성형 요약이나 회의록, 영업 코칭 같은 것은 이 문언 안에 없음
- **`accessibility service provider`.** 계약서가 이 표현을 정의하는 곳을 찾지 못했음. 프레임워크 쪽 열거형이 `signIn` 케이스의 설명에서 **VRS(Video Relay Service) 제공자**를 명시하므로, 문언이 겨냥하는 1차 대상은 영상 중계 서비스로 읽는 편이 맞음
- **`At an end user's invitation`.** 앱이 먼저 통화에 붙는 경로는 없음. 통화 중인 사람이 내 앱을 골라야 시작됨

**Live Assistance API가 3.3.11(AI·머신러닝)이 아니라 3.3.8(기타 API)에 들어간 것**도 그냥 넘길 게 아님. 자막과 번역을 예시로 들면서도 Apple 자신은 이걸 AI 조항으로 묶지 않았음. 3.3.8의 이웃이 Xcode Cloud, WeatherKit, iWork API, Now Playing, Critical Messaging, EnergyKit, Wi-Fi Infrastructure 같은 것들임.

## 확장이 실제로 받는 것

프레임워크 문서의 설명임.

> "To provide interpreter, captioning, or other services that work with FaceTime calls, create a container app that uses the LiveAssistance APIs, as well as an app extension that conforms to this protocol."

> "The extension performs most of the work involved in joining an assistance provider to a FaceTime call. When a person on a call chooses your app to provide assistance, the framework launches the extension and calls the prepareAssistanceRequest(_:) method. If your extension determines it has everything it needs — the person is signed in and has provided all needed information — it can retrieve a URL from the framework that the service provider uses to join the call."

구성은 **컨테이너 앱 + 앱 확장** 둘임. 확장이 구현하는 프로토콜은 이것임.

```swift
@MainActor protocol LiveAssistanceExtension : AppExtension
```

요구 메서드 둘, 그리고 요청 타입이 가진 것 전부.

| 심볼 | 선언 |
|---|---|
| `prepareAssistanceRequest(_:)` | `func prepareAssistanceRequest(_: LiveAssistanceRequest) async throws -> LiveAssistanceRequest.Response` |
| `resumeRequest(_:)` | `func resumeRequest(_: LiveAssistanceRequest) async throws -> LiveAssistanceRequest.Response` |
| `LiveAssistanceRequest.id` | `let id: UUID` |
| `LiveAssistanceRequest.url` | `let url: URL` |

`LiveAssistanceRequest`는 "value type sent by the framework to the request. All properties are read-only"로 적혀 있고 **프로퍼티가 위 둘뿐임.** `id`는 FaceTime 쪽 대화와 요청을 묶는 식별자, `url`은 "The FaceTime-generated URL for this request"임.

**이게 이 API의 성격을 결정함.** 오디오 버퍼도, 참가자 정보도, 트랜스크립트도 확장에 안 들어옴. 확장이 하는 일은 **인증과 설정을 확인하고 URL을 받아 제공자에게 넘기는 것**이고, 통화에 실제로 참가하는 주체는 그 URL로 접속하는 제공자임. 그래서 설계 질문이 "온디바이스 모델을 어디 꽂나"가 아니라 **"내 서비스가 통화에 접속해서 무엇을 가져가고 어디에 남기나"**가 됨.

## 응답과 에러를 가르는 선

반환 타입은 케이스 둘임. 문서가 **둘 다 "요청이 계속되어야 한다는 결과"**라고 명시함.

| 케이스 | 뜻 |
|---|---|
| `proceed` | 확장이 요청을 처리했고 설정을 진행해도 됨 |
| `requiresUserInput(reason:)` | 컨테이너 앱에서 할 일이 남아 있음. 시스템이 앱을 띄움 |

진행이 불가능할 때는 반환이 아니라 **throw**임. 문서가 선을 분명히 그음.

> "Throwing is terminal; the request ends immediately, although the person using FaceTime can initiate a new one."

> "If your container app can resolve the situation by requesting further input, don't throw an error. Instead, return `requiresUserInput(reason:)` to give the person a chance to sign in or otherwise provide the needed information."

에러 케이스는 `cannotFulfill(message: String?)`과 `unknown(message: String)` 둘임. `LiveAssistanceRequestError`와 `Response`는 둘 다 `frozen`이 아니라 `switch`에 `@unknown default`가 필요함.

**통화 중에 로그인 세션이 끊긴 상황에서 throw를 하면 사용자가 통화를 붙들고 처음부터 다시 해야 함.** 미로그인·설정 미완료는 전부 `requiresUserInput` 쪽으로 보내는 것이 이 API에서 제일 먼저 정해야 할 분기임.

컨테이너 앱을 띄우는 이유는 열거형 둘로만 옴.

| 케이스 | 뜻 |
|---|---|
| `signIn` | 사용자가 **VRS 제공자 서비스에 로그인되어 있지 않음** |
| `additionalInfo` | 인증과 무관한 정보가 더 필요함 |

문서가 단서를 하나 붙임. 컨테이너 앱이 `NSUserActivity`의 `userInfo`로 받는 **이유 문자열은 `LiveAssistanceLaunchReason`을 만들어 비교하는 용도에만 쓰고 UI에 그대로 보여주지 말라**는 것임.

## 컨테이너 앱 쪽 배선

문서가 지시하는 순서임.

1. Info.plist의 `NSUserActivityTypes`에 **`com.apple.conversation-accessibility.container-app-launch`**를 넣음
2. SwiftUI면 `onContinueUserActivity(_:perform:)`, UIKit이면 `application(_:continue:restorationHandler:)`로 받음
3. 활동 타입이 `LiveAssistance.containerAppLaunchActivityType`인지 확인
4. `userInfo`에서 `containerAppLaunchRequestIDKey`와 `containerAppLaunchReasonKey`로 요청 식별자와 이유를 꺼냄
5. 해결한 뒤 `LiveAssistance.returnToCall(requestID:)`를 호출하면 확장의 `resumeRequest(_:)`가 다시 불림

확장 쪽에 걸리는 제약 둘이 더 있고 둘 다 상태 설계를 바꿈.

> "Each invocation of your extension runs in a fresh process context. This means you can't rely on instance state persisting across calls to prepareAssistanceRequest(_:) and resumeRequest(_:), since the framework may have killed the extension and relaunched it in the interim. Instead, persist cross-invocation state in the keychain or App Group defaults."

> "If the extension process crashes mid-invocation, the request fails immediately. The framework doesn't re-invoke the handler in this scenario."

즉 **`prepareAssistanceRequest`와 `resumeRequest` 사이에 인스턴스 상태가 없음.** 토큰과 진행 상태를 키체인이나 App Group defaults에 넣어야 하고, 확장이 죽으면 재시도도 없음. 토큰 갱신이나 네트워크 호출을 확장 안에서 하는 설계라면 실패 경로를 전부 `cannotFulfill`로 닫아야 함.

## 지금 쓸 수 있나

| 항목 | 상태 |
|---|---|
| `LiveAssistanceExtension`·`LiveAssistance` 심볼 문서 | **iOS·iPadOS·Mac Catalyst·macOS·visionOS 27.2+, 전부 Beta** |
| 라이선스 계약 3.3.8(J) | **2026-10-13 개정으로 발효** |
| 27.2 베타 최신 빌드 | iOS·iPadOS 27.2 beta 4 (`24B5110a`), 2026-10-13 |

**계약서 조항이 API보다 먼저 섰음.** 프레임워크는 아직 베타 심볼인데 계약서에는 용도 제한이 들어가 있음. 설계를 시작하기에 늦지 않았다는 뜻이면서, 동시에 **정식 출시 때 심볼이 바뀌어도 계약 문언은 이미 고정됐다**는 뜻임.

⚠️ **27.2 베타 릴리스 노트에 이 프레임워크 항목이 없음.** iOS·iPadOS 27.2 베타 노트에 `LiveCommunicationKit`이나 Live Assistance를 다루는 절이 없고, 프레임워크 자신의 `LiveCommunicationKit updates` 페이지는 마지막 항목이 **2025년 6월**임. 공식 경로에서 이 기능의 존재를 알리는 것은 **심볼 문서와 계약서 개정 둘뿐**임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | 그 문서가 **5.1.2(i)**로 정리한 "서드파티로 나가는 개인정보" 판단이 여기 그대로 걸림. 통화 내용이 제공자 서버로 가는 구조라 온디바이스·PCC·서드파티 분류에서 서드파티 칸임. 아래에 그 문서가 못 읽은 **3.3.11 전문**을 옮겨 둠 |
| [iOS 27 · Xcode 27 정식 출시](./2026-09-14-ios-27-ai-apis-shipped.md) | 그 문서가 정리한 27 계열 API 표면에 이 확장점은 없었음. 27.2에 와서 붙은 것임 |
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 온디바이스 컨텍스트 제약과 무관한 자리임. 이 API는 모델을 기기에 올리는 경로가 아니라 **제공자를 통화에 접속시키는 경로**라 그쪽 제약을 받지 않음 |

### 3.3.11 AI·머신러닝 조항 전문

[App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) 문서가 Apple 공지 요약만 보고 "계약서 전문은 확인하지 못했음"으로 남겨둔 부분임. 2026-10-14 기준 문언이 이럼. 하위 조항은 **A·B·C 셋뿐이고 D는 없음.**

**A. Foundation Models Framework**

> "By accessing, prompting, or otherwise using the Foundation Models Framework, or a model accessed by the Foundation Models Framework, You agree to follow, and to maintain reasonable guardrails supporting, the Foundation Models Framework Acceptable Use Requirements."

조항 본문이 요건을 직접 적지 않고 **별도 문서인 Acceptable Use Requirements로 넘김.** 그리고 "reasonable guardrails supporting"이라는 의무가 따로 있음. 모델에 무엇을 보내는지만 관리하면 되는 게 아니라 **가드레일을 유지하는 것 자체가 계약 의무**라는 뜻임.

**B. SiriKit**은 (i)~(iii)으로, **C. App Intents**는 (i)~(iv)로 나뉨. AI 기능 설계에 바로 걸리는 문언만 추림.

- SiriKit (ii), App Intents (iii): 여기서 얻은 정보는 **지원 Apple 제품에서만** 쓰고, 관련 응답 제공·개선 또는 사용자 요청 수행에 필요한 범위를 넘어 **기기 밖으로 내보내거나 원격 접근·전송하면 안 됨.** 둘 다 "Notwithstanding anything to the contrary in Section 3.3.3(B)"로 시작해 다른 조항의 예외를 차단하고, 광고 게재 목적을 괄호로 명시해 금지함
- SiriKit (iii): SiriKit으로 오디오를 Apple에 처리시키면 **녹음된 오디오가 음성 인식·처리·전사 목적으로 Apple에 전송된다는 것과 Apple 제품·서비스 개선에 쓰일 수 있다는 것을 명확히 고지**해야 함
- App Intents (iv): iPhone 측면 버튼으로 실행되는 **음성 기반 대화형 앱**은 버튼을 누르면 (1) 음성 경험을 1차 모달리티로 띄우고 (2) 실행 후 질문·요청에 적절히 응답하고 동작을 수행해야 함

App Intents (iv)는 이 저장소에 없던 조건임. 측면 버튼 할당을 기능으로 잡고 있으면 **UI를 먼저 띄우고 음성을 선택지로 두는 구성이 계약 위반 쪽**이 됨.

## 짚어야 할 것

- **이 문서는 심볼 문서와 계약 문언까지임.** 샘플 코드도, "Joining a FaceTime call to provide assistance" 같은 기능 가이드도 찾지 못했음. 확장을 실제로 띄워 본 것이 아님
- **배포에 엔타이틀먼트나 Apple의 사전 승인이 필요한지 확인할 수 없었음.** 심볼 문서가 확장 포인트 식별자와 `NSUserActivityTypes` 키만 적고 엔타이틀먼트 신청 경로를 적지 않음. VRS 사업자를 1차 대상으로 보는 API라면 관문이 있을 가능성이 큰데 근거를 찾지 못함
- **`accessibility service provider`의 정의를 계약서에서 찾지 못했음.** 정의 조항에 이 표현이 따로 없어서, 어디까지가 접근성 서비스 제공자인지는 문언만으로 안 갈림. 자막 앱이 스스로를 그렇게 부르면 되는 것인지는 확인할 수 없었음
- **27.2가 정식으로 나왔을 때 이 심볼이 그대로 남는다는 보장이 없음.** 전부 Beta 표시이고 릴리스 노트에 항목이 없음
- **3.3.11 전문은 2026-10-14에 읽은 것이고 2026-06-08 개정판과 대조하지 않았음.** 지금 문언이 6월 개정 그대로인지 그 뒤에 또 바뀐 것인지는 확인하지 못했음
- 계약서 본문을 조항 단위로 읽었고 **전문을 통독하지 않았음.** 3.2의 보안 요건과 Schedule 쪽은 보지 않음

## 유효기간

**2026-10-14 확인 기준이고 iOS 27.2 beta 4(`24B5110a`) 시점임.** 계약 조항 번호와 문언은 개정될 수 있고, 개정은 [개발자 뉴스](https://developer.apple.com/news/)에 공지로 올라옴. 다시 볼 때 확인할 것 셋. **27.2 정식에서 심볼의 Beta 표시가 떨어졌는지**, **엔타이틀먼트나 승인 절차가 문서에 생겼는지**, 그리고 **기능 가이드나 샘플이 추가돼 확장이 받는 데이터 범위가 넓어졌는지**. 마지막 것이 넓어지면 5.1.2(i) 판단이 통째로 달라짐.
