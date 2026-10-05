---
title: App Intents Testing, iOS 27에서 새로 생긴 것은 온스크린 컨텍스트가 아니라 그걸 검증하는 경로임
source: https://developer.apple.com/documentation/appintentstesting
author: Apple
collected: 2026-10-05
tags: [apple, ios, app-intents, siri, testing, spotlight, on-device-llm]
---

출처: [App Intents Testing](https://developer.apple.com/documentation/appintentstesting) · [Testing your App Intents code](https://developer.apple.com/documentation/appintentstesting/testing-your-app-intents-code) · [Verifying your App Intents implementation](https://developer.apple.com/documentation/appintents/verifying-your-app-intents-implementation) · [Providing contextual cues to Apple Intelligence and Siri](https://developer.apple.com/documentation/appintents/providing-contextual-cues-to-apple-intelligence-and-siri) · [Apple Intelligence and Siri AI](https://developer.apple.com/documentation/appintents/apple-intelligence-and-siri-ai) · [App Intents updates](https://developer.apple.com/documentation/updates/appintents) · [`appEntityIdentifier(_:)`](<https://developer.apple.com/documentation/swiftui/view/appentityidentifier(_:)>) · [`appEntityUIElements(_:)`](<https://developer.apple.com/documentation/swiftui/view/appentityuielements(_:)>) · [`AppEntityUIElement`](https://developer.apple.com/documentation/appintents/appentityuielement) · [`ViewAnnotation`](https://developer.apple.com/documentation/appintentstesting/viewannotation) (2026-10-05 확인)

## 요약

[iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)가 출시 문구를 근거로 **View Annotations API**와 **App Intents Testing**을 "새로 확인된 API 표면"으로 적어두고, 가용성 표시는 프레임워크 문서로 다시 확인해야 한다고 유보를 걸어뒀음. 그 유보를 푼 것이 이 문서임. 결론이 갈림. **뷰에 엔티티를 붙이는 API는 iOS 27이 아니라 iOS 18.4에 들어온 것**이고, `appEntityIdentifier(_:)`·`appEntityUIElements(_:)`·`AppEntityUIElement`가 전부 **18.4** 표시임. App Intents 갱신 내역의 **2026-06(iOS 27) 항목에 온스크린 컨텍스트가 아예 없음.** 반면 **`AppIntentsTesting` 프레임워크는 통째로 iOS 27.0 신규**이고, 그 안에 **`viewAnnotations()`와 `ViewAnnotation`**이 들어 있음. 즉 iOS 27이 실제로 더한 것은 온스크린 컨텍스트 자체가 아니라 **그 컨텍스트가 제대로 붙었는지 코드로 확인하는 수단**임.

그리고 이 경로는 앱이 직접 여는 `LanguageModelSession`과 다른 층임. 문서 어디에도 뷰 주석이 앱 자신의 세션 프롬프트를 채워준다는 서술이 없음. 받는 쪽은 **Apple Intelligence와 Siri**임.

## 가용성을 직접 대조한 표

문서 페이지의 Availability 표시를 그대로 옮긴 것임.

| 심볼 | 도입 (iOS) |
|---|---|
| `appEntityIdentifier(_:)` (SwiftUI) | **18.4** |
| `appEntityUIElements(_:)` (SwiftUI) | **18.4** |
| `AppEntityUIElement` | **18.4** |
| Providing contextual cues 문서 전체 | 18.2 |
| `AppIntentsTesting` 프레임워크 | **27.0** |
| `IntentDefinitions` | **27.0** |
| `AppEntityDefinition` | **27.0** |
| `ViewAnnotation` | **27.0** |

App Intents 갱신 내역이 **2026-06** 묶음으로 나열하는 것은 다음임. `SyncableEntity`, `OwnershipProvidingEntity`와 `EntityOwnership`, `RelevantEntities`, `IntentValueRepresentation`, `RunSystemShortcutIntent`, `LongRunningIntent`, `CancellableIntent`와 `IntentCancellationReason`, `UndoableIntent`, `IntentModes`와 `IntentSystemContext.currentMode`, `IntentExecutionTargets`, `EntityCollection`, `AppUnionValue`, `IndexedEntityQuery`, `AppIntentError.init(description:)`. **온스크린 컨텍스트 관련 항목은 이 목록에 없음.**

**판단.** 출시 문구가 "The new View Annotations API"라고 부른 것은 홍보 쪽 명명이고, 프레임워크 문서가 쓰는 말은 `onscreen context`와 `entity annotations`임. `View Annotations`라는 이름의 심볼은 App Intents 프레임워크에서 찾지 못했고, 그 이름이 실제로 붙어 있는 곳은 테스트 프레임워크의 `ViewAnnotation` 하나임. 배포 타깃을 잡을 때 쓸 숫자는 홍보 문구가 아니라 위 표임.

**다만 "18.4부터 API가 있다"와 "18.4부터 시스템이 그 컨텍스트를 쓴다"는 다른 층임.** 전자는 확인했고 후자는 확인하지 못했음. Siri AI 자체가 iOS 27 기능이라 실제 소비 시점은 갈릴 수 있음.

## 온스크린 컨텍스트를 붙이는 세 가지 방법

문서가 정리하는 선택지가 셋이고, 고르는 기준은 **화면에 엔티티가 몇 개 있느냐**임.

| 상황 | 쓰는 것 |
|---|---|
| 화면의 뷰 하나가 엔티티 하나에 대응 | `appEntityIdentifier(_:)` (SwiftUI) 또는 responder의 `appEntityIdentifier` (UIKit·AppKit) |
| 한 화면에 엔티티가 여럿이고 뷰와 일대일이 아님 | `appEntityUIElements(_:)`로 `AppEntityUIElement`를 요청받을 때 만들어 줌 |
| 화면이 엔티티 하나만 담고 Handoff 대상 | `NSUserActivity`의 `appEntityIdentifier` |

가운데 것이 이 저장소 관점에서 볼 만함. 뷰마다 수식어를 다는 대신 **클로저가 요청을 받아 그때그때 목록을 만듦**. 요청은 `visible(let rect)`와 `selected` 둘로 갈리고, 캔버스나 Core Animation 레이어처럼 뷰 계층이 내용과 일대일이 아닌 화면에 쓰라고 적혀 있음.

```swift
.appEntityUIElements { context in
    stickyNotes.compactMap { note in
        let includeNote = context.requests.contains { request in
            switch request {
                case .visible(let rect): return note.frame.intersects(rect)
                case .selected: return note.isSelected
                @unknown default: return false
            }
        }
        guard includeNote else { return nil }
        return AppEntityUIElement(
            identifier: EntityIdentifier(for: StickyNote.self, identifier: note.id),
            bounds: note.frame,
            state: State(isSelected: note.isSelected)
        )
    }
}
```

문서가 경고로 박아둔 줄이 하나 있음.

> "Attach app entities to views to reflect the data those views show. Don't attach unrelated app entities to your interface."

관련 없는 엔티티를 붙이면 시스템이 "이거"를 잘못 푼다는 뜻임. 컨텍스트를 많이 주는 게 이득이 아니라는 점에서 [Memory Engineer](./2026-08-02-memory-engineer.md)가 정리한 "밀도가 부피를 이긴다"와 같은 방향임.

시스템 타입 쪽에도 같은 주석을 다는 경로가 있음. `UserNotifications`의 가변 설정 객체, `MPNowPlayingInfoCenter`의 `MPNowPlayingInfoPropertyAppEntityIdentifiers` 키, `AlarmKit`의 `AlarmConfiguration`이 `AppEntityAnnotatable`을 따름. 문서가 바로 뒤에 단서를 붙임. **프로토콜을 따르지 않는 시스템 타입에 내가 적합성을 추가해도 추가 컨텍스트가 전달되지 않음.**

## AppIntentsTesting, 타입을 import하지 않고 이름으로 부름

프레임워크가 iOS 27.0에 통째로 새로 생겼고, 구조가 보통의 단위 테스트와 다름.

- 테스트는 **UI Testing 번들**에 둠. 앱 프로세스를 실제로 띄우고 거기에 붙기 때문임
- 앱 타깃에 링크하지 않음. `IntentDefinitions(bundleIdentifier:)`로 카탈로그를 만들고 **문자열 이름으로 조회**함
- 실행은 out-of-process. 문서 표현으로 **"the same way Siri or Shortcuts perform them"**
- 반환 타입이 `@dynamicMemberLookup`이라 `result.value.count`처럼 앱 타입 없이 속성을 읽음

```swift
let app = XCUIApplication()
var definitions: IntentDefinitions!

override func setUp() async throws {
    app.launch()
    definitions = IntentDefinitions(bundleIdentifier: "com.example.my-app")
}
```

`IntentDefinitions`가 들고 있는 컬렉션은 `intents`, `entities`, `enums`, `valueQueries`, `transientEntities` 다섯임. 엔티티 쪽 `AppEntityDefinition`에 실제로 쓸 메서드가 몰려 있음.

| 메서드 | 하는 일 |
|---|---|
| `entities(matching:)` | 시스템이 발화된 이름을 푸는 것과 같은 경로로 문자열 검색 |
| `makeReference(identifier:)` | 알고 있는 식별자로 엔티티 참조를 만들어 인텐트 파라미터로 넘김 |
| `spotlightQuery(_:)` | Spotlight 색인을 직접 질의. `nil`을 주면 해당 타입의 색인된 전부 |
| `viewAnnotations()` | **지금 화면에 보이는 엔티티 목록** |
| `resolved(from:)` | `exported(as:)`로 내보낸 것을 되읽어 `Transferable` 왕복 검증 |
| `allEntities()` · `suggestedEntities()` | 전체·추천 엔티티 |

**이름으로 부르는 구조의 대가가 하나 있음.** 인텐트나 엔티티 타입의 이름을 바꾸면 컴파일이 아니라 테스트 실행 시점에 깨짐. 리네임을 컴파일러가 잡아주지 않는 테스트를 늘리는 것이므로, 이름 자체를 계약으로 취급해야 함.

## viewAnnotations(), 온스크린 컨텍스트에 처음 생긴 검증 수단

`ViewAnnotation`은 필드가 둘뿐임. `entity: AnyAppEntity`와 `isSelected: Bool`. 그런데 이게 중요한 이유는 **그 전까지 뷰 주석이 제대로 붙었는지 확인할 방법이 Siri에 말을 걸어보는 것뿐이었기 때문**임.

```swift
func testEventDetailShowsAnnotation() async throws {
    try await definitions.intents["NavigateToEvent"]
        .makeIntent(eventId: "design-review-id")
        .run()
    let annotations = try await definitions.entities["EventEntity"].viewAnnotations()
    XCTAssertFalse(annotations.isEmpty)
    let title: String = try annotations[0].entity.title
    XCTAssertEqual(title, "Design Review")
    XCTAssertTrue(annotations[0].isSelected)
}
```

화면 이동을 UI 자동화로 하지 않고 **테스트 전용 인텐트**로 하는 것이 문서가 미는 패턴임. `isDiscoverable = false`로 Shortcuts·Siri에 안 보이게 하고 `#if DEBUG`로 릴리스 빌드에서 뺌.

```swift
#if DEBUG
struct ResetDatabaseIntent: AppIntent {
    static let title: LocalizedStringResource = "Reset Database"
    static let isDiscoverable = false
    func perform() async throws -> some IntentResult { /* seed */ }
}
#endif
```

**판단.** [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md)이 말한 것, 즉 판정 기준을 루프 바깥에 두라는 요구가 여기서 구체적인 형태를 얻음. 에이전트에게 App Intents 구현을 맡기면 "Siri가 잘 알아듣는다"는 평가를 에이전트 자신이 할 수 없는데, `viewAnnotations()`와 `spotlightQuery(_:)`는 **사람도 에이전트도 아닌 시스템이 내놓는 답**임. [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)이 정리한 시뮬레이터 스크린샷 검증보다 이쪽이 앵커로 쓸 값어치가 큼. 스크린샷은 다시 모델이 읽어야 하지만 이 둘은 값으로 비교됨.

## 검증 4단계, 각 단계가 잡는 것이 다름

Verifying 문서가 네 단계를 쌓는 순서로 제시하고, **각 단계가 앞 단계로는 못 잡는 것을 적어둔 부분**이 실제로 쓸모 있음.

| 단계 | 잡는 것 | 못 잡는 것 (문서 표현) |
|---|---|---|
| App Intents Testing 통합 테스트 | `perform()`이 기대한 결과를 내는가 | Shortcuts에서 파라미터 요약이 어색하게 읽히는 것 |
| Shortcuts 앱에서 확인 | 파라미터 표현과 결과 타입, 액션 라이브러리 노출 | 엔티티가 발견 가능한지 |
| Spotlight 검색 | 엔티티가 색인에 들어갔는지 | Siri가 발화를 맞게 처리하는지 |
| Siri 종단 확인 | 실제 사람이 쓰는 말로 동작하는지 | |

Spotlight 단계에 단서가 붙음. **색인은 기기 동작에 의존하므로 시뮬레이터와 다를 수 있고 실기기에서 확인해야 함.** 색인을 날리고 처음부터 돌리는 것은 기기 개발자 설정의 CoreSpotlight Testing 항목으로 함.

Siri 단계에서 문서가 권하는 것 중 둘이 이 저장소 주제와 붙음. **온스크린 컨텍스트가 있을 때와 없을 때 같은 요청을 둘 다 해 볼 것**, 그리고 **AirPods처럼 화면이 없는 상황에서 결과 발화가 핵심 정보를 담는지 볼 것.** 후자는 인텐트 결과를 짧게 만드는 압력이라, [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)의 간결성 지시와 같은 문제를 반대쪽에서 만나는 셈임.

## 앱이 직접 여는 세션과는 다른 경로임

[iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)가 남긴 질문이 이것이었음. 뷰 주석이 앱 자신의 `LanguageModelSession` 프롬프트나 컨텍스트를 대신 채워주는가. **문서 기준으로 그런 서술이 없음.** Apple Intelligence and Siri AI 페이지가 받는 쪽을 이렇게 적음.

> "Associating views, user activities, or other visible content with app entities gives Apple Intelligence onscreen context."

소비자는 Apple Intelligence와 Siri이고, 앱이 `LanguageModelSession`을 직접 열어 쓰는 경로는 여전히 프롬프트와 툴로 컨텍스트를 만들어 넣는 쪽임. 둘이 만나는 지점은 **엔티티라는 공용 타입** 하나고, 앱 안에 자체 대화형 표면을 만든다면 같은 `AppEntity`를 재료로 쓰되 세션에는 직접 넣어야 함.

그리고 그 직접 넣는 쪽은 창이 좁음. 같은 문서가 정리한 대로 온디바이스 컨텍스트는 **4,096토큰**이고 툴 정의와 스키마가 그 창을 먼저 먹음([Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md)). iOS 27 정식 노트의 `SpotlightSearchTool` 알려진 이슈는 **Apple이 만든 툴 하나를 기본 설정으로 붙이는 것만으로 창이 넘친다**는 것이었음. 즉 설계 순서가 이렇게 됨.

1. 사용자가 시스템 경로(Siri·Shortcuts·Spotlight)로 하길 바라는 일이면 **엔티티와 스키마와 뷰 주석**으로 감. 컨텍스트 창 문제가 없음
2. 앱 안의 자체 대화형 표면이 필요한 일만 `LanguageModelSession`으로 내림. 여기서는 4K 예산을 직접 셈
3. 1번을 2번으로 대신하려 들지 않음. 온스크린 컨텍스트는 1번 경로에만 흐름

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 그 문서가 "가용성 표시는 재확인하지 못했음"으로 남긴 유보를 App Intents 쪽에 한해 푼 것. 결론이 갈림 |
| [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md) | 온디바이스 4K 예산. 온스크린 컨텍스트는 그 예산을 쓰지 않는 다른 경로임 |
| [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md) | 판정 기준을 루프 바깥에 두는 문제. `viewAnnotations()`와 `spotlightQuery(_:)`가 그 바깥에 해당함 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 에이전트가 시뮬레이터로 자기 변경을 검증하는 구조. 스크린샷보다 값 비교가 앵커로 나음 |
| [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md) | "틀리면 얼마나 아픈가"로 게이트를 여는 것. Spotlight 색인 회귀는 조용히 깨지는 쪽이라 게이트를 걸 값어치가 있음 |

## 짚어야 할 것

- **프레임워크 문서만 읽었고 직접 돌려보지 않았음.** 테스트 타깃을 만들어 `viewAnnotations()`가 실제로 무엇을 돌려주는지는 확인하지 않았음
- **`View Annotations`라는 이름의 심볼을 App Intents 프레임워크에서 찾지 못했음.** 출시 문구의 명명이고 문서는 `onscreen context`·`entity annotations`로 부름. 검색으로 자료를 찾을 때 이름이 갈린다는 점을 알고 있어야 함
- **"iOS 18.4에 API가 있다"가 "iOS 18.4 기기에서 시스템이 그 컨텍스트를 쓴다"는 뜻은 아님.** 가용성 표시만 확인했고 시스템이 실제로 소비하기 시작한 시점은 확인하지 못했음
- **`AppIntentsTesting`에 베타 표시는 보이지 않았으나 명시적으로 정식이라는 서술도 없음.** 심볼 페이지의 Availability가 `27.0`으로만 적혀 있는 것까지 확인함
- **테스트가 UI Testing 타깃에서 돌고 앱 프로세스를 띄움.** 문서는 "fast enough that you can include them in frequent continuous-integration workflows"라고 적는데 **측정값은 없음.** CI 시간 예산은 직접 재야 함
- **WWDC26 세션 두 건(295 `Validate your App Intents adoption with AppIntentsTesting`, 343 `Explore advanced App Intents features for Siri and Apple Intelligence`)은 제목만 확인했고 본편을 보지 않았음.** 이 문서의 근거는 전부 문서 페이지임
- 온스크린 컨텍스트가 프라이버시 측면에서 무엇을 시스템에 넘기는지는 이 페이지들이 다루지 않음. [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md)의 공개·동의 요건과 어떻게 만나는지는 **확인하지 못했음**

## 유효기간

**2026-10-05 확인 기준**임. 가용성 표는 심볼별로 박힌 값이라 낡지 않지만, iOS 27.1·27.2에서 `AppIntentsTesting`에 심볼이 추가되면 표가 늘어남. 다시 볼 때 확인할 것 둘. **App Intents 갱신 내역에 2026-09 이후 묶음이 생겼는지**, 그리고 **온스크린 컨텍스트 쪽에 iOS 27 표시를 단 심볼이 실제로 추가됐는지**. 후자가 생기면 이 문서의 결론인 "iOS 27이 더한 것은 테스트 경로뿐"이 바뀜.
