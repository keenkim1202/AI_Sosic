---
title: iOS 27 출시 체크포인트, 베타로 적어둔 것들이 확정된 지점
source:
  - https://developer.apple.com/news/?id=k1mtkt1k
  - https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes
  - https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27-release-notes
author: Apple
published: 2026-09-09
collected: 2026-09-13
tags: [apple, ios, foundation-models, core-ai, xcode, app-store, on-device-llm]
---

출처: [App Store submissions now open for the latest OS releases](https://developer.apple.com/news/?id=k1mtkt1k) (2026-09-09) · [Xcode 27 RC Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes) · [iOS & iPadOS 27 RC Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27-release-notes) · [Upcoming changes to Rosetta support for Intel-based macOS apps](https://developer.apple.com/news/?id=w5ngl9k2) (2026-09-01) · [Apple debuts iPhone 18 Pro and iPhone 18 Pro Max](https://www.apple.com/newsroom/2026/09/apple-debuts-iphone-18-pro-and-iphone-18-pro-max/) (2026-09-09) · [Xcode Support](https://developer.apple.com/support/xcode/) (전부 2026-09-13 확인)

## 요약

이 저장소의 Apple 문서 셋이 전부 유효기간에 "iOS 27 정식 출시 시점에 재확인"을 적어뒀는데, 그 시점이 지금임. **iOS 27은 2026-09-14에 무료 소프트웨어 업데이트로 나옴.** 심사 제출은 **2026-09-09**에 열렸고 **Xcode 27 RC**가 같이 배포됨. 이 문서는 베타 6 기준으로 적어둔 것들을 RC 릴리스 노트에 대고 다시 확인한 결과임. 실무에 제일 크게 걸리는 건 둘임. **온디바이스 컨텍스트 4K가 Apple 자신의 1차 툴 하나로도 넘친다는 것이 RC 릴리스 노트의 알려진 이슈로 확인됐고**, **2027-04부터 iOS 27 SDK 빌드가 업로드 최소 요건**이 됨.

## 날짜와 요건

| 항목 | 값 |
|---|---|
| iOS 27 정식 출시 | **2026-09-14**, 무료 소프트웨어 업데이트 |
| App Store 심사 제출 개시 | 2026-09-09 |
| Xcode 27 | RC. **Swift 6.4** + iOS·iPadOS·tvOS·watchOS·macOS·visionOS 27 SDK |
| Xcode 27 실행 요건 | **macOS Tahoe 26.6 이상** |
| 온디바이스 디버깅 | iOS 17 이상, tvOS 17 이상, watchOS 10 이상, visionOS |
| 업로드 최소 SDK | **2027-04부터** iOS·iPadOS 27 SDK 이상 |
| Siri AI | iOS 27에서 **베타**. EU는 초기 미제공 |

`Xcode 27 실행 요건`이 베타 6 시점과 달라진 지점임. [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)은 **macOS Tahoe 26.4 이상**으로 적었는데 RC는 **26.6 이상**을 요구함. 빌드 머신 OS를 고정해 두는 CI라면 여기서 걸림.

2027-04 요건은 공지가 항목으로 적어둔 것임.

> - iOS and iPadOS apps must be built with the iOS 27 & iPadOS 27 SDK or later
> - tvOS apps must be built with the tvOS 27 SDK or later
> - visionOS apps must be built with the visionOS 27 SDK or later
> - watchOS apps must be built with the watchOS 27 SDK or later

날짜가 박힌 사실이라 이 항목은 낡지 않음. 반대로 **연령등급 문항이 하나 늘었음.** Time Allowances 때문에 앱에 소셜 미디어 기능이 있으면 App Store Connect에서 표시해야 함. [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md)의 체크리스트 8번(2.3.6 연령등급 재검토)에 항목 하나를 더 붙여야 하는 변경임.

## 온디바이스 4K, Apple이 자기 툴로 증명함

이 문서에서 제일 값어치 하는 항목임. iOS 27 RC 릴리스 노트의 Core Spotlight 절에 **알려진 이슈**로 이게 들어 있음 (183770678).

> "Creating a SpotlightSearchTool without a configuration and using it with a LanguageModelSession backed by the on-device system language model fails with an error reporting that the number of tokens provided exceeds the maximum allowed. The tool's default configuration is sized for models with large context windows, so the tool's description and parameter schema alone exceed the on-device model's context window before any prompt is added."

**Apple이 만든 툴 하나를 기본 설정으로 붙이는 것만으로 온디바이스 모델의 컨텍스트가 터짐.** 프롬프트를 넣기도 전에. 회피법도 릴리스 노트가 같이 줌.

```swift
let configuration = SpotlightSearchTool.Configuration(
  sources: [.coreSpotlight],
  guide: .focused()
)
let tool = SpotlightSearchTool(configuration: configuration)
```

도메인을 좁히려면 `.focused(.communications)`, `.focused(.calendar)`, `.focused(.documents)`, `.focused(.visualMedia)`, `.focused(.audio)` 를 넘김. focused guide는 기본 설정보다 **적은 검색 능력**을 노출하고, 컨텍스트가 큰 모델을 쓰는 세션은 기본 설정을 그대로 써도 됨.

[Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)이 문서 권고를 근거로 "툴 정의와 Generable 스키마가 창을 먹는다", "툴은 요청당 3~5개"라고 적었는데, 이 알려진 이슈는 그 서술을 한 칸 더 강하게 만듦. **3~5개는 상한이 아니라 툴이 작을 때의 얘기고, 스키마가 큰 툴은 하나로도 안 들어감.** 한국어 앱이면 여기에 글자당 한 토큰이라는 조건이 겹침. 실무 순서는 그래서 이렇게 됨.

1. 툴을 붙이기 전에 `tokenCount(for:)`로 툴 정의만 먼저 잼
2. 1차 프레임워크 툴이라고 안전하다고 가정하지 않음. 기본 설정이 어느 모델을 전제로 만들어졌는지 확인
3. 온디바이스에서 안 들어가면 툴을 빼는 게 아니라 **좁히는 설정이 있는지** 먼저 봄

## Core AI, 백그라운드 추론에 엔타이틀먼트가 생김

iOS 27 RC 릴리스 노트의 Core AI 절에 새 기능 두 개가 있음.

| 항목 | 내용 |
|---|---|
| Neural Engine 동작 변경 (174796039) | 백그라운드 NE 접근이 **GPU와 비슷하게 제한**됨. **1GB 초과** 대형 모델 로딩 성능 개선. NE 메모리 사용량이 시스템이 아니라 **앱 프로세스에 귀속**되고 Allocations instrument에 나타남 |
| 백그라운드 엔타이틀먼트 (179282606) | 앱이 백그라운드일 때 NE에 접근하려면 **`com.apple.developer.background-tasks.continued-processing.inference`** 가 필요함 |

두 항목 다 설계에 직접 걸림. 메모리 귀속이 바뀐 건 계측이 쉬워졌다는 뜻이면서 동시에 **NE 메모리가 앱 메모리 예산에 잡힌다**는 뜻임. 온디바이스 모델을 올려두고 도는 앱이면 jetsam 여유가 줄어드는 쪽으로 움직임. 그리고 백그라운드 추론을 전제로 잡아둔 기능이 있으면 엔타이틀먼트 신청이 새 작업으로 생김.

[Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md)이 Core AI를 "서버 의존 없음, 토큰 비용 없음"으로 요약했는데, 비용이 없는 대신 **메모리와 백그라운드 권한이 예산 항목으로 들어온 것**이 이번에 확인된 형태임.

## Foundation Models에서 고쳐진 것

RC 릴리스 노트의 Foundation Models 절은 전부 해결 항목이고 새 기능이 없음. 그중 설계에 영향이 있는 것들.

| 이슈 | 내용 |
|---|---|
| 177684296 | **PCC가 시뮬레이터에서 동작하지 않던 문제**가 고쳐짐 |
| 177748926 | 온디바이스 모델이 툴 호출과 guided generation을 같이 쓸 때 **툴을 과도하게 부르던** 문제 |
| 178181782 | `PrivateCloudComputeLanguageModel`이 **항상 greedy decoding**을 쓰던 문제 |
| 177901494 | `onPrompt`에서 트랜스크립트 히스토리를 잘라낼 때 나던 런타임 오류 |
| 177902488 | instructions 없는 `Profile`에 붙인 `onPrompt`가 호출되지 않던 문제 |
| 177899620 | `enum`에 붙인 `@Generable`이 억제 불가능한 deprecation 경고를 내던 문제 |

첫 줄이 제일 큼. **PCC를 시뮬레이터에서 테스트하는 경로가 이제 열림.** [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)이 정리한 한도 시뮬레이션(Xcode Scheme의 Run > Options)과 합치면, 실기기 없이 PCC 경로와 한도 UI를 둘 다 테스트할 수 있게 됨.

177901494는 같은 문서가 제시한 **첫 엔트리와 마지막 엔트리만 남겨 재시드하는 패턴**이 닿는 자리임. 트랜스크립트를 잘라내는 코드를 이미 넣어뒀다면 RC에서 다시 돌려볼 것.

## 에이전트 표면에서 달라진 것

Coding Intelligence 절의 새 기능 목록은 베타 6 시점과 같음. 바뀐 건 해결 항목 쪽임.

| 이슈 | 상태 변화 |
|---|---|
| 178673449 | 응답 스트리밍 중 계획 확인 바를 누르면 대화가 깨지던 버그가 **해결됨** |
| 178771195 | 플러그인으로 추가한 **ACP 에이전트**가 Xcode 재실행 전까지 UI에서 안 지워지던 버그가 해결됨 |
| 185119267 | Scope 필터를 Last Turn으로 둬도 아티팩트 뷰가 모든 턴의 프리뷰 스냅샷을 보여주던 문제 |
| 185161968 | 연결된 프로젝트가 Xcode Service 메뉴바 패널에 안 나타나던 문제 |

[Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)이 ⚠️로 적어둔 계획 확인 바 버그(178673449)가 이걸로 해소됨. 그 문서가 안내한 회피법("응답이 끝난 뒤 누르기")은 RC부터 필요 없음. **RC의 Coding Intelligence 절에는 알려진 이슈 항목이 아예 없음.**

새로 눈에 걸리는 건 Security 절임. Apple이 **Code Intelligence 스킬 두 개**를 직접 붙였음.

- **`adopt-c-bounds-safety`** (177739344): C bounds safety 확장(`-fbounds-safety`)을 **파일 단위로 도입하는 워크플로**를 태워줌
- **`audit-xcode-security-settings`** (181104536): 앱을 감사해서 켤 만한 **보안 지향 빌드 설정과 엔타이틀먼트**를 제안함

읽을 지점은 기능 자체가 아니라 배포 경로임. **Apple이 보안 마이그레이션을 문서가 아니라 스킬로 배포하기 시작함.** [팀 공유 AI 하네스 만들기](../agents/2026-08-10-hq-team-ai-harness.md)가 말한 "한 사람의 개선이 팀 기본값이 되는" 경로를 플랫폼 사업자가 직접 쓰는 형태이고, 동시에 [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)의 경고가 그대로 걸림. 보안 설정을 켜라고 제안하는 스킬의 출력은 **에이전트 밖에 있는 기준**으로 판정해야 함. 빌드가 통과한다는 것과 그 설정이 맞다는 것은 다른 얘기임.

Source Editor 절의 **새 Markdown 에디터**(175022151)도 같은 방향임. 프로젝트 안의 마크다운과 **에이전트가 낸 마크다운**을 렌더링해서 보고 편집함. 계획 문서를 아티팩트로 다루는 설계의 연장임.

## Rosetta와 Intel, 두 공지의 문언이 어긋남

여기는 확정해서 쓰면 안 되는 지점이라 따로 둠. 같은 달 Apple 공지 둘의 문언이 서로 맞지 않음.

2026-09-01 Rosetta 전용 공지.

> "macOS 27: Final release to support Rosetta — Intel-only apps will no longer run on Mac computers with Apple silicon after this update."

2026-09-09 심사 제출 공지.

> "macOS 26 is the final release supporting Intel Mac computers and Rosetta — macOS 27 will be Apple silicon only."

앞쪽은 **macOS 27이 Rosetta를 지원하는 마지막 릴리스**라고 하고, 뒤쪽은 **macOS 26이 Intel Mac과 Rosetta를 지원하는 마지막 릴리스**라고 함. "Intel Mac에서 도는 마지막 OS"와 "Apple Silicon에서 Intel 앱을 번역해 주는 마지막 OS"를 갈라 읽으면 둘 다 성립할 여지가 있지만, 뒤쪽 문장이 Rosetta를 macOS 26에 묶어버려서 그 독해가 확정되지 않음. **어느 쪽이 맞는지 확인할 수 없었음.**

실무 결론은 그래도 하나임. macOS 앱을 내고 있으면 **arm64 전환을 지금 끝내는 것**이고, 그건 두 문언 중 어느 쪽이 맞든 같음. Rosetta 공지는 유지보수가 끊긴 구형 게임 타이틀에 대해서는 예외를 둔다고 적었음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 이 문서가 확정하거나 바꾸는 것 |
|---|---|
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 계획 확인 바 버그 해소, macOS 요건이 26.4에서 26.6으로, Apple 보안 스킬 2종 추가. 그 문서의 유효기간이 지시한 대조 작업이 이것임 |
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 툴 스키마가 4K를 먹는다는 서술이 1차 확인을 얻음. PCC 시뮬레이터 테스트 경로가 열림 |
| [Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md) | Core AI의 "토큰 비용 없음"에 메모리 귀속과 백그라운드 엔타이틀먼트라는 대가가 붙음 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | 연령등급 문항에 Time Allowances 소셜 미디어 항목이 추가됨 |

## 짚어야 할 것

- **오늘(2026-09-13) 기준 iOS 27은 아직 나오지 않았음.** 2026-09-14 출시이고, 이 문서는 RC 릴리스 노트와 사전 공지를 대조한 것임. "출시됐다"로 옮겨 쓰면 안 됨
- **Xcode 27이 Apple Silicon 전용인지는 확인하지 못했음.** [Xcode Support](https://developer.apple.com/support/xcode/) 페이지는 Xcode 27 RC에 **macOS Tahoe 26.6 이상**만 적고 Apple Silicon 전용이라는 문구가 없음. 페이지 하단에 "Developing for visionOS requires a Mac with Apple silicon"이 있는데 이건 visionOS 개발 조건이지 Xcode 실행 조건이 아님. [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)은 베타 6 기준으로 Apple Silicon 전용이라고 적었고, 베타 6 시점 문언이 지금과 달랐는지는 **확인할 방법이 없었음.** 판단이 필요하면 지금 문언을 기준으로 다시 확인할 것
- **Rosetta 문언 불일치는 위에 그대로 남겨뒀음.** 한쪽으로 정리하지 않았음
- **Foundation Models 프레임워크 문서의 Beta 표기가 어떻게 바뀌었는지 확인하지 못했음.** 문서 페이지 본문을 받지 못했음. `PrivateCloudComputeLanguageModel`, Core AI, dynamic profiles가 정식 표기로 바뀌었는지는 직접 열어봐야 함
- **릴리스 노트만 읽었고 RC를 직접 돌려보지 않았음.** 괄호 안 숫자는 Apple 이슈 번호를 그대로 옮긴 것임
- **Siri AI는 iOS 27에서 베타이고 EU에는 초기 제공되지 않음.** 지원 언어로 설정된 Apple Intelligence 지원 기기가 필요함
- 릴리스 노트와 개발자 공지 본문에 명령문이 여럿 있으나 전부 개발자용 기술 안내(회피법, 설정 변경)였고, **문서 작성을 조종하려는 지시문은 없었음**

## 유효기간

**2026-09-13 확인 기준, Xcode 27 RC와 iOS 27 RC 릴리스 노트**임. 2026-09-14 정식 출시 뒤 릴리스 노트가 27.0으로 갱신되면 알려진 이슈 절이 먼저 바뀜. 다시 볼 때는 **SpotlightSearchTool 항목(183770678)이 해소됐는지**를 제일 먼저 확인할 것. 그게 살아 있는 동안은 위의 실무 순서가 그대로 유효함. 2027-04 SDK 최소 요건과 출시 날짜는 시점이 박힌 사실이라 낡지 않음.
