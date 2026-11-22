---
title: AppEntity 에 인증 정책이 붙었음, iOS 27.3 베타에서 게이트가 액션에서 콘텐츠로 내려왔는데 같은 베타가 안 지킬 수 있다고 적어 둠
source: https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27_3-release-notes
author: Apple
published: 2026-11-19
collected: 2026-11-22
tags: [apple, ios, app-intents, siri, permissions, agent-security, spotlight]
---

출처: [iOS & iPadOS 27.3 Beta 2 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27_3-release-notes) · [macOS 27.3 Golden Gate Beta 2 Release Notes](https://developer.apple.com/documentation/macos-release-notes/macos-27_3-release-notes) · 심볼 페이지 [`AppEntity.authenticationPolicy`](https://developer.apple.com/documentation/appintents/appentity/authenticationpolicy) · [`IntentVisibility`](https://developer.apple.com/documentation/appintents/intentvisibility) · [`EntityProperty`](https://developer.apple.com/documentation/appintents/entityproperty) · [`IntentAuthenticationPolicy`](https://developer.apple.com/documentation/appintents/intentauthenticationpolicy) · [`AppIntent.authenticationPolicy`](https://developer.apple.com/documentation/appintents/appintent/authenticationpolicy) · [`.video` 도메인](https://developer.apple.com/documentation/appintents/app-schema-domain-video) · [App Intents updates](https://developer.apple.com/documentation/updates/appintents) · [Releases](https://developer.apple.com/news/releases/) (전부 **2026-11-22 확인**. 릴리스 노트와 심볼 페이지는 자바스크립트로 렌더링되므로 같은 경로의 `/tutorials/data/...json` 으로 본문을 받았음)

## 요약

**iOS 27.3 베타가 `AppEntity` 와 그 속성에 인증 정책과 노출 범위를 붙임.** 심볼 셋이 전부 **27.3 도입에 베타 표시**이고 iOS·iPadOS·Mac Catalyst·macOS·tvOS·visionOS·watchOS **7개 플랫폼 전부**에 같은 표시가 달림. 새것은 `AppEntity.authenticationPolicy`, `EntityProperty` 의 `authenticationPolicy`·`visibility` 파라미터, 그리고 `IntentVisibility` 타입임.

**값어치는 게이트가 내려온 자리에 있음.** 인증 정책 타입(`IntentAuthenticationPolicy`) 자체는 **iOS 16·macOS 13·watchOS 9** 부터 있었는데 걸 수 있는 대상이 **앱 인텐트가 실행될 때** 하나였음. 27.3 에서 같은 타입을 **엔티티를 시스템이 읽을 때**로 옮겨 걸 수 있게 됨. Siri AI 가 앱 콘텐츠에 닿는 주된 경로는 인텐트 실행이 아니라 **온스크린 컨텍스트, Spotlight 색인, 도네이션으로 엔티티를 읽는 것**이므로, 액션에만 걸리던 잠금이 비로소 읽기 경로에 생긴 것임.

**그런데 같은 베타의 릴리스 노트가 그 게이트를 안 지킬 수 있다고 적어 둠.** iOS·iPadOS 27.3 beta 2 의 App Intents Known Issues 에 "잠금을 요구하는 `authenticationPolicy` 를 걸어도 Siri AI 가 엔티티를 표시하기 전에 잠금 해제를 요구하지 않을 수 있다"가 있음(**190573761**). macOS 27.3 beta 2 노트에는 이 항목이 없음. 즉 **27.3 베타에서 이것을 보안 경계로 취급하면 안 됨.** 둘 중 실제로 기댈 수 있는 쪽은 인증이 아니라 **`visibility: .internal` 로 아예 안 넘기는 쪽**이고, 그 판단은 이 저장소가 [승인 정확도 문서](./2026-08-05-agent-approval-miss-rates.md)에서 이미 한 번 내린 것과 같은 모양임.

덤으로 확인된 것 둘. **`.video` 스키마 도메인이 Siri AI 연동 대상으로 열렸음**(tvOS·iOS·macOS·visionOS, **189805051**). 그리고 **App Intents 갱신 페이지의 최신 절이 여전히 June 2026** 이라, 이 변경은 갱신 내역으로는 찾을 수 없고 심볼 가용성으로만 보임.

## 새로 생긴 심볼 셋

심볼 페이지의 `platforms` 배열에서 읽은 값임.

| 심볼 | 선언 | 도입 | 베타 |
|---|---|---|---|
| `AppEntity.authenticationPolicy` | `static var authenticationPolicy: IntentAuthenticationPolicy { get }` | 27.3 | **베타** |
| `EntityProperty.init(title:description:authenticationPolicy:visibility:)` | `convenience init(title: LocalizedStringResource? = nil, description: LocalizedStringResource? = nil, authenticationPolicy: IntentAuthenticationPolicy? = nil, visibility: IntentVisibility? = nil)` | 27.3 | **베타** |
| `IntentVisibility` | `struct IntentVisibility` | 27.3 | **베타** |
| `IntentVisibility.default` | `static var default: IntentVisibility { get }` | 27.3 | **베타** |
| `IntentVisibility.internal` | `static var internal: IntentVisibility { get }` | 27.3 | **베타** |
| `IntentAuthenticationPolicy` (타입 자체) | `enum IntentAuthenticationPolicy` | iOS 16.0 · macOS 13.0 · watchOS 9.0 | 아님 |
| `AppIntent.authenticationPolicy` | `static var authenticationPolicy: IntentAuthenticationPolicy { get }` | iOS 16.0 · macOS 13.0 · watchOS 9.0 | 아님 |

7개 플랫폼 전부에 **같은 `27.3` 베타 표시**가 달려 있음. watchOS 와 tvOS 가 빠지지 않은 것이 눈에 띄는데, 엔티티를 읽는 쪽이 플랫폼마다 다른 표면이어도 정책 선언은 공통으로 둔다는 뜻으로 읽힘.

`EntityProperty` 는 `@propertyWrapper final class EntityProperty<Value>` 이고 문서가 **"시스템에 해당 속성을 노출하는 프로퍼티 래퍼"** 로 설명함. `AppEntity.Property` 가 이것의 타입 별칭이므로 실제로 쓰는 모양은 `@Property(title:..., authenticationPolicy:..., visibility:...)` 임.

## 게이트가 액션에서 콘텐츠로 내려온 것

같은 `IntentAuthenticationPolicy` 를 두 자리에 걸 수 있게 된 것이고, 두 자리의 문언이 정확히 갈림.

| 거는 자리 | 문서가 적은 시점 | 도입 |
|---|---|---|
| `AppIntent.authenticationPolicy` | "the authentication policy to enforce **when running** the app intent" | iOS 16.0 |
| `AppEntity.authenticationPolicy` | "the authentication policy to enforce **when the system accesses** this entity" | 27.3 베타 |

케이스는 셋 그대로임. `alwaysAllowed`(기기가 잠겨 있어도 허용), `requiresAuthentication`(인증 요구), `requiresLocalDeviceAuthentication`(실행 기기의 잠금 해제 요구). **엔티티 쪽 기본값도 `alwaysAllowed`** 라고 문서가 명시함. 즉 아무것도 안 적은 기존 엔티티는 전부 열린 상태로 남음. 마이그레이션이 필요한 쪽이 아니라 **지금까지 열려 있었다는 사실이 드러난 쪽**임.

이 구분이 왜 중요한지는 [App Intents Testing 문서](../practices/2026-10-05-app-intents-testing-and-onscreen-context.md)가 정리한 경로를 보면 됨. 뷰에 엔티티를 붙이는 API(`appEntityIdentifier(_:)`, `appEntityUIElements(_:)`)와 Spotlight 색인(`IndexedEntity`)은 **인텐트를 하나도 실행하지 않고** 엔티티를 시스템에 넘김. 그 경로에는 인텐트의 `authenticationPolicy` 가 걸릴 자리가 없었음. 27.3 의 추가분은 그 공백을 메우는 것이고, [Apple Intelligence and Siri AI](https://developer.apple.com/documentation/appintents/apple-intelligence-and-siri-ai) 가 "사람이 Siri AI 로 화면에 보이는 콘텐츠를 다루는데 그 콘텐츠는 앱의 사적 자산"이라고 적어둔 자리와 같은 곳임.

## 엔티티 정책과 속성 정책의 합성 규칙

`AppEntity.authenticationPolicy` 의 Discussion 이 합성 규칙을 한 문장으로 적음.

> "This is the minimum policy for each of the entity's properties. A property's own `authenticationPolicy` can make it stricter; a weaker one is raised to this policy."

읽어야 할 것은 **방향이 한쪽**이라는 것임. 엔티티에 적은 정책이 **바닥**이고 속성은 그보다 조이기만 할 수 있음. 속성에 더 느슨한 값을 적어도 바닥까지 올라감. 그래서 설계 순서가 정해짐.

1. 엔티티 전체에 **가장 느슨하게 둬도 되는 값**을 적음. 여기를 조이면 그 엔티티의 모든 속성이 같이 잠김
2. 민감한 속성만 `@Property(authenticationPolicy:)` 로 따로 올림
3. 반대 방향, 즉 "엔티티는 잠고 속성 하나만 열기"는 **불가능함.** 그런 모양이 필요하면 엔티티를 쪼개는 게 맞음

3번이 실무에서 제일 먼저 걸림. 엔티티 하나에 공개 필드와 민감 필드를 같이 담아 두고 엔티티 레벨을 잠그면, Siri AI 가 그 엔티티를 **이름조차 못 쓰게** 됨. 공개해서 검색되게 하려던 것과 보호하려던 것이 한 타입에 섞여 있으면 정책을 둘 다 만족시킬 수 없음.

`visibility` 는 축이 다름. 인증을 요구하는 것이 아니라 **넘길지 말지**를 정함. `.default` 는 "앱과 Siri·Shortcuts 같은 시스템 기능에 노출", `.internal` 은 "앱과 그 익스텐션으로 제한"임. 문서가 `IntentVisibility` 의 기본값을 숫자나 이름으로 적어두지 않았고 이니셜라이저 파라미터는 `nil` 기본이라, **미지정 시 무엇으로 해석되는지는 확인할 수 없었음.**

## 같은 베타가 안 지킬 수 있다고 적어 둔 것

iOS·iPadOS 27.3 beta 2 릴리스 노트의 App Intents Known Issues 전문임.

> "If you define `authenticationPolicy` on an `AppEntity` or `AppEntity` property that requires unlock, Siri AI might not request an unlock before it displays these entities. (190573761)"

세 가지를 읽어야 함.

- **대상이 잠금 해제를 요구하는 정책임.** 즉 `requiresLocalDeviceAuthentication` 쪽이고, `requiresAuthentication` 이 같이 걸리는지는 문언에 없음
- **증상이 거부가 아니라 조용한 표시임.** 에러도 로그도 아니고 Siri AI 가 그냥 보여줌. 즉 기기에서 테스트해서 "동작한다"로 넘어가면 통과하는 종류의 결함임
- **macOS 27.3 beta 2 노트에는 이 항목이 없음.** 없다는 것이 "macOS 에서는 지켜진다"는 뜻은 아님. 노트에 안 올라온 것과 동작이 다른 것은 별개임

같은 사이클의 Siri 항목도 같이 봐야 함. macOS 쪽에는 **"서드파티 앱이 Spotlight 에 도네이션한 엔티티를 스키마에 맞추지 않으면 Siri 가 찾지 못한다"**(191273925)가 있고, iOS 쪽에는 **HomeView 의 Siri AI 설정이 제대로 안 될 수 있다**(192952891)가 있음. 셋을 나란히 두면 **27.3 베타의 엔티티 경로 전체가 아직 흔들리는 구간**임.

## 지금 코드에 넣을 것

베타이고 그중 하나는 문서가 스스로 안 지킬 수 있다고 적어 뒀으므로, 넣는 순서가 중요함.

1. **민감 필드를 먼저 `visibility: .internal` 로 뺌.** 인증을 요구하는 것보다 안 넘기는 쪽이 싸고 확실함. 이것이 [승인 정확도 문서](./2026-08-05-agent-approval-miss-rates.md)의 결론을 속성 단위로 적용한 것임. 단 `.internal` 쪽에 Known Issue 가 없다는 사실이 **지켜진다는 확인은 아님**
2. **`authenticationPolicy` 는 적되 보안 경계로 세지 않음.** 27.3 정식에서 190573761 이 Resolved 로 올라온 뒤에 경계로 승격하는 것이 맞음. 지금 적어 두는 값어치는 정식에서 자동으로 켜진다는 것까지임
3. **엔티티 레벨을 조이기 전에 그 엔티티가 검색·온스크린 컨텍스트로 쓰이는지 확인함.** 바닥을 올리면 모든 속성이 같이 잠기므로 Siri AI 가 그 엔티티를 못 쓰게 됨
4. **`#available` 분기를 27.3 으로 둠.** 심볼이 전부 27.3 베타 표시이므로 27.2 이하 기기에서는 이 정책을 걸 수단이 아예 없음. 즉 **같은 앱이 OS 버전에 따라 노출 범위가 달라짐**. 이 격차를 코드가 아니라 데이터 쪽에서 메우려면, 민감 필드를 엔티티에 아예 안 담는 선택이 남음
5. **기기 테스트로 확인했다고 적지 않음.** 이 문서는 심볼 가용성과 릴리스 노트만 대조한 것이고 실제 Siri AI 동작은 확인하지 않았음

## 덤으로 확인된 것, `.video` 스키마 도메인

iOS 와 macOS 27.3 beta 2 노트가 **같은 항목 번호(189805051)**로 적음. "The Video schema domain is now available for apps to integrate with Siri AI on tvOS, iOS, macOS, and visionOS."

도메인 페이지에서 확인한 것임. `.video` 는 **Primary domains** 에 들어 있고 인텐트 스키마가 넷임. `playContent`, `playTrailer`, `addToWatchList`, `removeFromWatchList`. 그리고 **use-case group** 규칙이 걸려 있음. 재생 그룹은 `playContent` 스키마와 `IntentValueQuery` 둘로 이뤄지고, 한쪽만 구현하면 **Xcode 가 빌드 경고**를 냄. Xcode 에서 `video_` 를 타면 템플릿이 생성됨.

비디오 앱 쪽에 직접 걸리는 항목이고, 위의 인증 정책과 같은 자리에서 만남. 재생 목록이나 시청 기록을 엔티티로 내보내면 그것이 바로 `visibility` 로 가릴지 말지 판단할 대상이 됨.

## 갱신 페이지는 아직 비어 있음

이 저장소가 재확인 과제로 적어둔 항목 둘에 그대로 답이 됨.

| 재확인 과제 | 이번 확인 결과 |
|---|---|
| [App Intents Testing](../practices/2026-10-05-app-intents-testing-and-onscreen-context.md): App Intents 갱신 내역에 2026-09 이후 묶음이 생겼는지 | **안 생겼음.** 최신 절이 여전히 **June 2026** 이고 거기에 `authenticationPolicy`·`IntentVisibility` 가 없음 |
| [iOS 27.2 정식 출시](../industry/2026-10-26-ios-27-2-foundation-models-ga.md): Foundation Models updates 에 27.2 절이 생겼는지, 그리고 27.3 베타에서 새 베타 표면이 생겼는지 | 앞은 **안 생겼음**(최신 절이 여전히 **June 2026**). 뒤는 **생겼음.** 이 문서의 심볼 셋이 그것임 |

즉 **27.3 베타의 App Intents 변경을 갱신 내역으로 찾는 것은 지금 불가능함.** 릴리스 노트의 New Features 에도 `.video` 도메인 한 줄만 있고 인증 정책 쪽은 없음. 이번 심볼 셋이 릴리스 노트에 등장하는 유일한 자리가 **Known Issues** 임. 새 API 가 "있다"는 서술 없이 "안 지켜질 수 있다"는 서술로만 노트에 올라온 상태임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [App Intents Testing](../practices/2026-10-05-app-intents-testing-and-onscreen-context.md) | 인텐트를 실행하지 않고 엔티티가 시스템으로 가는 경로. 그 경로에 27.3 에서 처음 게이트가 생김. 그 문서의 재확인 과제 둘에 답이 됨 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 승인을 정확하게 하는 대신 대상에서 빼는 선택. `visibility: .internal` 이 속성 단위의 같은 선택임 |
| [macOS Full Disk Access 제한 예고](./2026-10-02-macos-full-disk-access-controls.md) | 같은 사이클에 Apple 이 권한을 좁히는 방향. 그쪽은 OS 권한 층, 이쪽은 앱이 스스로 내보내는 범위 층임 |
| [iOS 27.2 정식 출시](../industry/2026-10-26-ios-27-2-foundation-models-ga.md) | 갱신 페이지의 공백이라는 같은 문제. 그 문서가 27.3 베타 표면을 재확인 과제로 남겨 둠 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md) | 사용자 동의의 형식. 심사 조항이 요구하는 것은 서드파티 전송 공개이고 이번 것은 시스템 기능 노출이라 층이 다름. 다만 무엇을 내보내는지 목록으로 들고 있어야 한다는 요구는 같음 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 같은 사이클의 릴리스 추적. 베타 딱지를 심볼 가용성으로 세는 방식이 그쪽에서 시작됨 |

## 짚어야 할 것

- **실제 동작을 확인하지 않았음.** 이 문서가 대조한 것은 심볼 페이지의 가용성 주석과 릴리스 노트 문언임. 27.3 베타 기기에서 Siri AI 가 잠금을 요구하는지, `.internal` 속성이 실제로 안 넘어가는지는 **확인할 수 없었음**
- **`visibility` 미지정 시의 해석을 확인할 수 없었음.** `IntentVisibility` 문서에 기본값 서술이 없고 이니셜라이저 파라미터가 `nil` 기본임. `.default` 가 기본으로 들어간다고 적지 않았음
- **`requiresAuthentication` 이 Known Issue 에 걸리는지 확인할 수 없었음.** 노트 문언은 "requires unlock" 까지이고 세 케이스 중 어느 것인지 이름으로 적지 않았음
- **사용 가이드가 없음.** [Apple Intelligence and Siri AI](https://developer.apple.com/documentation/appintents/apple-intelligence-and-siri-ai) 와 [App Intents updates](https://developer.apple.com/documentation/updates/appintents) 어느 쪽도 `authenticationPolicy`·`IntentVisibility` 를 언급하지 않음. 언제 어느 값을 쓰라는 Apple 쪽 권고는 **확인할 수 없었음**
- **27.3 베타의 App Intents 추가분 전체를 나열하지 못했음.** 갱신 페이지에 27.3 절이 없고 프레임워크 색인 페이지에서 심볼별 가용성을 일괄로 받을 경로를 찾지 못했음. 이 문서의 심볼 셋은 릴리스 노트가 지목한 `authenticationPolicy` 에서 출발해 인접 심볼로 넓힌 것이고 **완전한 목록이 아님**
- **발행일 근거.** 릴리스 노트 페이지에 자체 날짜가 없음. `2026-11-19` 는 [Releases](https://developer.apple.com/news/releases/) 피드의 **iOS·iPadOS 27.3 beta 2 v2 (24C5049f)** 날짜임. 같은 beta 2 가 Xcode(27C5035i)·macOS(26C5047e)·tvOS·visionOS·watchOS 는 **2026-11-16** 이고 iOS·iPadOS 만 v2 로 다시 올라옴. **재공개 이유는 피드에 없어 확인할 수 없었음**
- **2차 보도는 쓰지 않았음.** 이 문서의 모든 수치와 문언은 Apple 문서와 릴리스 노트에서 직접 받은 것임
- **원문에 지시문이나 명령문 형태의 문장은 없었음.** 릴리스 노트는 항목 서술이고 심볼 페이지는 API 설명임

## 유효기간

**2026-11-22 확인 기준이고 iOS·iPadOS 27.3 beta 2 v2 시점임.** 베타 표시는 릴리스마다 움직이므로 위 표를 고치려 들지 말고 심볼 페이지 JSON 의 `platforms` 배열을 다시 받는 편이 나음. 다시 볼 때 확인할 것 셋. **190573761 이 Resolved Issues 로 올라왔는지**(올라오면 이 문서의 "보안 경계로 세지 말 것"이 풀림), **세 심볼의 베타 표시가 27.3 정식에서 떨어졌는지**, 그리고 **App Intents 갱신 페이지에 27.3 절이 생겨 사용 권고가 붙었는지.** 마지막 것이 생기면 `visibility` 기본값과 `requiresAuthentication` 적용 범위의 유보 둘이 같이 풀릴 자리임.
