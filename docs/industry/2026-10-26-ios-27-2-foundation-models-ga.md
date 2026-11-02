---
title: iOS 27.2 정식 출시, 기기에는 정식 API 가 들어갔는데 Xcode 는 RC 로만 올라와 있음
source: https://developer.apple.com/news/releases/
author: Apple
published: 2026-10-26
collected: 2026-10-29
tags: [apple, ios, xcode, foundation-models, on-device-llm, app-intents]
---

출처: [Releases](https://developer.apple.com/news/releases/) · [iOS & iPadOS 27.2 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27_2-release-notes) · [macOS 27.2 Release Notes](https://developer.apple.com/documentation/macos-release-notes/macos-27_2-release-notes) · [Xcode 27.2 RC Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes) · [Foundation Models updates](https://developer.apple.com/documentation/updates/foundationmodels) · [App Intents updates](https://developer.apple.com/documentation/updates/appintents) · [Updates 색인](https://developer.apple.com/documentation/updates) · 심볼 페이지 [`DataAttachmentRepresentable`](https://developer.apple.com/documentation/foundationmodels/dataattachmentrepresentable) · [`DataEntryRepresentable`](https://developer.apple.com/documentation/foundationmodels/dataentryrepresentable) · [`supportsDataAttachmentType(_:)`](https://developer.apple.com/documentation/foundationmodels/languagemodel/supportsdataattachmenttype(_:)) · [Private Cloud Compute](https://developer.apple.com/private-cloud-compute/) (전부 **2026-10-29 확인**. 릴리스 노트와 심볼 페이지는 자바스크립트로 렌더링되므로 같은 경로의 `/tutorials/data/...json` 으로 본문을 받았음)

## 요약

**2026-10-26 에 iOS·iPadOS·macOS·tvOS·visionOS·watchOS 27.2 가 정식으로 나왔음.** 이 저장소가 27.2 를 조건으로 걸어둔 재확인 항목이 다섯 개 있었고, 이 문서는 그것을 심볼 페이지의 가용성 주석과 릴리스 노트로 직접 대조한 것임.

결과가 갈림. 풀린 것 하나. **데이터 첨부 경로 세 심볼에서 베타 표시가 사라졌음.** [Foundation Models 가용성 대조](../practices/2026-09-28-foundation-models-availability-audit.md)가 "지금 베타인 것은 데이터 첨부 경로 하나"로 적어뒀는데, 그 칸이 이제 비었음. 안 풀린 것 넷. **27.2 OS 릴리스 노트에 Foundation Models·Core AI·App Intents 절이 아예 없고**, Foundation Models updates 와 App Intents updates 페이지는 최신 절이 여전히 **June 2026** 이며, Updates 색인에 **Core AI 항목이 아직 없음.**

그리고 실무에 제일 크게 걸리는 것이 하나 더 나옴. **Apple 이 Releases 에 올려둔 최신 Xcode 는 2026-10-19 의 `Xcode 27.2 RC (27B48)` 이고 27.2 정식 항목이 없음.** iOS 노트가 **"The SDK comes bundled with Xcode 27.2"** 라고 적으므로, 27.2 전용 API 를 쓰려면 지금은 RC 툴체인이 유일한 경로임. 기기 쪽은 정식인데 빌드하는 쪽이 RC 인 상태가 며칠째 유지되고 있음.

## 무엇이 언제 나왔나

Releases 목록에서 받은 빌드 번호와 날짜임.

| 항목 | 버전 | 날짜 |
|---|---|---|
| iOS 27.2 | 24B91020 · 24B119 | **2026-10-26** |
| iPadOS 27.2 | 24B119 | 2026-10-26 |
| macOS 27.2 | 26B123 | 2026-10-26 |
| tvOS 27.2 | 24K122 | 2026-10-26 |
| visionOS 27.2 | 24N120 | 2026-10-26 |
| watchOS 27.2 | 24S118 | 2026-10-26 |
| Xcode 27.2 RC | 27B48 | 2026-10-19 |
| Xcode 27.1 | 27A9275 | 2026-10-15 |

iOS 27.2 는 목록에 **두 줄로** 올라와 있음. `24B91020` 과 `24B119` 가 각각 한 줄씩이고 페이지가 이유를 적지 않음. 같은 날 iPadOS 26.7.2(`23H128`), iOS·iPadOS 18.7.11(`22H384`) 도 같이 나왔음.

## 베타 딱지가 비었음

심볼 페이지 JSON 의 `platforms` 배열에서 읽은 값임. 09-28 대조와 같은 방법·같은 출처라 비교가 성립함.

| 심볼 | 도입 | 2026-09-28 | 2026-10-29 |
|---|---|---|---|
| `DataAttachmentRepresentable` | 27.2 | 베타 | **베타 표시 없음** |
| `DataEntryRepresentable` | 27.2 | 베타 | **베타 표시 없음** |
| `LanguageModel.supportsDataAttachmentType(_:)` | 27.2 | 베타 | **베타 표시 없음** |

세 심볼 모두 `iOS`·`iPadOS`·`Mac Catalyst`·`macOS`·`visionOS`·`watchOS` 여섯 플랫폼에 `27.2` 로 들어 있고, 배열에 베타 플래그가 없으며 페이지 본문에 `beta` 라는 단어가 나오지 않음. 메서드 선언도 그대로임.

```swift
nonisolated(nonsending) func supportsDataAttachmentType(_ type: UTType) async throws -> Bool
```

실무 결론은 이럼. **임의 타입을 첨부로 넘기는 코드는 이제 베타 API 에 의존하지 않음.** 대신 그 코드가 요구하는 것이 베타에서 툴체인으로 옮겨감. 아래 Xcode 항목과 같이 봐야 함.

## PCC 자격 요건은 문언 그대로

같은 문서가 재확인 과제로 적어둔 둘째 항목임. 개발자 페이지의 세 조건과 이탈 시 기한이 09-28 과 같은 문장으로 남아 있음.

> "Access to PCC is available to developers who meet the following criteria:
> * Are enrolled in the App Store Small Business Program.
> * Have fewer than 2 million first-time app downloads from any of their apps on the App Store.
> * Have the Private Cloud Compute entitlement assigned to their account."

> "If any app subsequently exceeds the 2 million first-time downloads threshold, or the developer is no longer enrolled in the App Store Small Business Program, the developer will be notified and must migrate to an alternative solution within 6 months."

**엔타이틀먼트 할당이 포함된 세 조건과 6개월 기한이 그대로임.** 무료 구간을 전제로 기능 범위를 잡아둔 쪽은 다시 계산할 것이 없음.

## 27.2 노트에 AI 절이 하나도 없음

iOS·iPadOS 27.2 노트의 절은 여덟 개고 전부 AI 밖임. `App Tracking Transparency`, `Health`, `Nearby Interaction`, `Quest Lab Appointments`, `Shortcuts`, `Simulator`, `StoreKit Testing in Xcode`, `UIKit` 임. Foundation Models, Core AI, Apple Intelligence, `LanguageModelSession`, `SpotlightSearchTool`, Core Spotlight, App Intents, Evaluations, Neural Engine 을 언급하는 항목이 하나도 없음.

macOS 27.2 노트도 같음. 절이 `General`·`AppKit`·`Display Settings`·`Notifications`·`Shortcuts`·`StoreKit Testing in Xcode` 여섯이고, [macOS Full Disk Access 제한 예고](../security/2026-10-02-macos-full-disk-access-controls.md)가 재확인 과제로 적어둔 **권한 통제의 형태와 적용 시점은 여기에 없음.** 권한 쪽에 가까운 항목은 둘뿐임.

- `AppKit` 폐기 항목: `EnablePasteboardPrivacyDeveloperPreview` 사용자 기본값이 제거됨 (186955507)
- `Notifications` 알려진 이슈: 이 빌드를 클린 설치한 Mac 이 일부 기본 앱의 알림 허용을 묻지 않을 수 있음 (187312007)

**절이 없다는 것을 변경이 없다는 뜻으로 읽으면 안 됨.** 데이터 첨부 세 심볼의 베타 표시가 떨어진 것은 심볼 페이지에서 확인된 사실인데, 그 변경이 27.2 노트에는 적혀 있지 않음. 즉 **이 릴리스에서 Foundation Models 표면이 움직인 것을 릴리스 노트만 보고는 알 수 없었음.** [iOS 27 · Xcode 27 정식 출시](./2026-09-14-ios-27-ai-apis-shipped.md)가 "릴리스 노트는 릴리스 주기 전체를 누적하는 문서"라고 적어둔 것의 반대편 사례임. 그 문서는 베타 시절 문구가 남는 쪽을 경고했고, 이번은 정식 시점의 변경이 올라오지 않는 쪽임.

누적 문서라는 성질 자체도 그대로 확인됨. 27.2 가 정식으로 나온 뒤에도 `Health` 절에 **"The redesigned Health app is not included in this beta"** (188843443) 가 남아 있고, macOS 쪽 해결 항목에 **"Xcode might crash when using code completion on macOS 27.2 beta"** (186939138) 가 그대로 있음.

## 에이전트로 테스트를 병렬로 돌리면 걸리는 것

27.2 노트에서 이 저장소 독자에게 직접 걸리는 항목은 `Simulator` 절의 알려진 이슈 하나임 (188407822, FB24939277).

> "Cloning a simulator device running runtime 27.2 might fail due to incorrect file permissions."

시뮬레이터 복제는 에이전트를 워크트리마다 띄워 테스트를 동시에 돌릴 때 쓰는 경로임. 27.2 런타임으로 올린 머신에서 복제가 깨지면 **병렬 테스트 레인이 통째로 멈추는데, 원인이 코드가 아니라 런타임 쪽**이라 찾는 데 시간이 걸리는 부류임. 노트가 주는 회피법은 소스 기기의 샌드박스 임시 파일을 지우는 것임.

```
find ~/Library/Developer/CoreSimulator/Devices/<SOURCE_UDID>/data -name '*.sb-????????-??????' -delete
```

`<SOURCE_UDID>` 를 복제 원본 기기로 바꿔 씀. 아래 짚어야 할 것에 적었듯 이건 파일을 지우는 명령이라 경로를 먼저 확인하고 돌리는 쪽이 맞음.

## Xcode 만 RC 에 멈춰 있음

이 문서에서 설계 판단에 제일 크게 걸리는 항목임. 확인한 것 넷임.

| 확인한 것 | 내용 |
|---|---|
| Releases 목록 | 최신 Xcode 항목이 `Xcode 27.2 RC (27B48)` 이고 날짜가 **2026-10-19**. 그 아래가 `Xcode 27.1 (27A9275)`. **27.2 정식 항목이 없음** |
| 릴리스 노트 목차 | Xcode 릴리스 노트 색인의 첫 항목 제목이 `Xcode 27.2 RC Release Notes` 임 |
| Xcode 27.2 노트 Overview | "Xcode 27.2 RC includes Swift 6.4 and SDKs for iOS 27.2, iPadOS 27.2, tvOS 27.2, watchOS 27.2, macOS 27.2, and visionOS 27.2." 그리고 **macOS Tahoe 26.6 이상** 요구 |
| iOS 27.2 노트 Overview | "The SDK comes bundled with Xcode 27.2. For information on the compatibility requirements for Xcode 27.2, see Xcode 27.2 Release Notes." |

개발자 뉴스 쪽에 27.2 SDK 제출 개방을 알리는 공지를 찾지 못했음. 2026-10-18 부터 2026-10-29 사이의 뉴스 항목은 둘뿐이고 둘 다 이 주제와 무관함(10-28 `Say hello to Touch on Mac`, 10-22 `Offer your subscriptions to businesses and schools`). 9월 사이클에서는 정식 출시 **5일 전**에 제출 개방 공지가 따로 나왔는데, 이번 마이너 릴리스에는 같은 형태의 공지가 안 보임.

**그래서 단정하지 않는 것이 맞음.** Xcode 27.2 가 정식으로 나오지 않았다는 증명이 아니라, **Apple 이 Releases 와 릴리스 노트 제목으로는 아직 그것을 RC 로만 제시하고 있다**는 것까지가 확인된 범위임.

실무 결론은 그래도 분명함. 데이터 첨부 경로처럼 **27.2 도입 API 를 쓰는 코드를 지금 머지하면 CI 의 Xcode 를 RC 로 올려야 함.** 그리고 정식 노트가 나오면 한 번 더 올리게 됨. [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)이 적어둔 macOS 요건이 27.2 RC 에서도 **26.6 이상 그대로**라 빌드 머신 OS 는 이번에 건드릴 일이 없는데, 툴체인 버전은 두 번 움직이는 모양이 됨. 27.2 전용 API 가 급하지 않으면 **정식 Xcode 항목이 올라온 뒤에 한 번만 올리는 쪽이 싸게 끝남.**

## 갱신 페이지 셋은 아직 안 열림

세 문서가 재확인 과제로 적어둔 항목이 전부 "아직" 으로 유지됨.

| 페이지 | 2026-10-29 상태 |
|---|---|
| Foundation Models updates | 최신 절이 **June 2026** 임. 27.2 절이 없음 |
| App Intents updates | 최신 절이 **June 2026** 임. 2026-09 이후 묶음이 없음 |
| Core AI updates | Updates 색인의 기술·프레임워크 목록에 **항목 자체가 없음.** 본문 링크로만 이름이 나오고 `/tutorials/data/documentation/updates/coreai.json` 은 404 |

Foundation Models updates 의 June 2026 절은 `DynamicProfile`, `LanguageModel` 프로토콜, `PrivateCloudComputeLanguageModel`, Vision 툴(`OCRTool`·`BarcodeReaderTool`), `GenerationOptions.ToolCallingMode`, 그리고 오픈소스 쪽 `CoreAILanguageModel`·`MLXLanguageModel` 을 그대로 담고 있음. **데이터 첨부 경로는 이 목록에 없음.** 심볼 페이지에서는 정식이 됐는데 갱신 페이지에는 등장하지 않은 상태라, 갱신 페이지를 단일 출처로 쓰면 이 변경을 놓침.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Foundation Models 가용성 대조](../practices/2026-09-28-foundation-models-availability-audit.md) | 그 문서의 재확인 항목 둘에 답한 것. 데이터 첨부 세 심볼의 베타 표시와 PCC 자격 요건 문언 |
| [iOS 27 · Xcode 27 정식 출시](./2026-09-14-ios-27-ai-apis-shipped.md) | 누적 릴리스 노트의 성질. 그 문서는 베타 문구가 남는 쪽, 이 문서는 변경이 안 올라오는 쪽 |
| [Xcode 27.2 RC, 에이전트가 커버리지를 읽게 됨](../agents/2026-10-19-xcode-27-2-rc-coverage-mcp.md) | 그 문서가 읽은 RC 노트가 OS 정식 출시 뒤에도 같은 제목으로 남아 있음. `GetCodeCoverage` 항목도 그대로 하나임 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 빌드 머신 macOS 요건. 27.2 RC 에서도 26.6 이상임 |
| [App Intents Testing](../practices/2026-10-05-app-intents-testing-and-onscreen-context.md) | App Intents 갱신 내역에 새 묶음이 생겼는지. 안 생겼음 |
| [Core AI 로 직접 만든 모델 올리기](../practices/2026-10-07-core-ai-model-deployment.md) | Core AI updates 페이지가 열렸는지. 색인에 항목이 없음 |
| [macOS Full Disk Access 제한 예고](../security/2026-10-02-macos-full-disk-access-controls.md) | 27.2 노트에 통제의 형태와 시점이 적혔는지. 안 적혔음 |

## 짚어야 할 것

- **베타 표시는 심볼 페이지 JSON 의 `platforms` 배열에서 읽은 것임.** 09-28 대조와 같은 방법·같은 출처라 비교는 성립하지만, 플래그가 사라진 것이 동작이 바뀌지 않는다는 보장은 아님. 시그니처가 고정됐다는 것까지로 읽어야 함
- **Xcode 27.2 가 정식으로 나오지 않았다고 단정하는 것이 아님.** 확인한 것은 Releases 목록에 정식 항목이 없고 릴리스 노트 제목이 RC 라는 것까지임. 다운로드 페이지와 Mac App Store 쪽은 확인하지 못했음
- **iOS 27.2 가 Releases 목록에 빌드 두 개로 올라와 있는 이유를 확인할 수 없었음.** 페이지가 `24B91020` 과 `24B119` 를 각각 한 줄로 적고 설명을 달지 않음
- **확인한 심볼은 셋임.** `Attachment.init(_:)` 의 데이터 첨부 오버로드와 `supportsDataEntryType(_:)` 의 가용성은 개별 페이지로 열어보지 않았음. 가용성 대조 문서가 적어둔 계열 전체가 같은 상태라고 단정하면 안 됨
- **시뮬레이터 복제 회피법은 파일을 지우는 명령임.** 원문이 그대로 제시한 것을 옮겼고, `<SOURCE_UDID>` 를 치환하지 않은 채 돌리면 매치가 안 되지만, 경로를 잘못 넣으면 지우는 쪽으로 동작함. 돌리기 전에 경로를 먼저 확인할 것
- 릴리스 노트 본문은 자바스크립트 렌더링 때문에 `/tutorials/data/...json` 경로로 받았음. **문서 페이지에 보이는 것과 JSON 이 다를 가능성은 확인하지 못했음**
- 27.2 노트의 `App Tracking Transparency` 절에 EU 사용자용 확장 프롬프트와 연 단위 재질의가 새 기능으로 들어 있고 프랑스·독일·이탈리아·폴란드·루마니아에서는 확장 프롬프트가 필수라고 적혀 있음 (179504171). AI 와 무관하지만 같은 릴리스에 든 요건이라 적어 둠

## 유효기간

**2026-10-29 확인 기준이고 OS 정식 출시 사흘 뒤임.** 출시일과 빌드 번호는 고정된 기록이라 낡지 않음. 낡는 것은 Xcode 쪽 상태와 갱신 페이지의 공백임. 다시 볼 때 확인할 것 셋. **Releases 에 `Xcode 27.2` 정식 항목이 올라왔는지**, **Foundation Models updates 에 27.2 절이 생겨 데이터 첨부 경로가 목록에 들어왔는지**, 그리고 **27.3 베타에서 새 베타 표면이 생겼는지.** 셋째는 이 저장소가 베타 딱지를 추적해 온 자리가 다시 채워지는지의 문제고, 심볼 페이지 JSON 의 `platforms` 배열이 가장 빠른 신호임.
