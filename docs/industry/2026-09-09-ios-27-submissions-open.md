---
title: iOS 27 제출 개시, 베타 딱지가 떨어지는 날과 새로 생긴 기한 둘
source: https://developer.apple.com/news/
author: Apple
published: 2026-09-09
collected: 2026-09-10
tags: [apple, ios, xcode, app-store, foundation-models, app-review]
---

출처: [Apple Developer News](https://developer.apple.com/news/)의 2026-09-09 "App Store submissions now open for the latest OS releases"와 2026-09-01 "Upcoming changes to Rosetta support for Intel-based macOS apps" 항목 · [Releases](https://developer.apple.com/news/releases/) · [Apple debuts iPhone 18 Pro and iPhone 18 Pro Max](https://www.apple.com/newsroom/2026/09/apple-debuts-iphone-18-pro-and-iphone-18-pro-max/) (전부 2026-09-10 확인)

## 요약

베타가 끝남. **2026-09-09에 iOS 27·iPadOS 27·macOS 27·tvOS 27·visionOS 27·watchOS 27 앱 제출이 열렸고**, 같은 날 **Xcode 27 RC(27A266a)** 와 각 OS의 RC가 함께 올라왔음. iOS 27 자체는 **2026-09-14 무료 소프트웨어 업데이트**로 배포됨. 이 저장소가 "베타"라고 적어둔 것들, 즉 Foundation Models의 PCC 경로와 Core AI, Xcode의 에이전트 표면이 이번 주에 정식 표면이 됨. 대신 기한이 둘 생겼음. **2027년 4월부터 iOS 27 SDK 이상으로 빌드하지 않으면 업로드가 막히고**, macOS 쪽은 Intel과 Rosetta 지원이 끝나는 구간에 들어감. 연령등급 문항에는 **Time Allowances와 소셜미디어 기능 신고**가 추가됐음.

## 2026-09-09에 올라온 것

Releases 페이지에서 확인한 빌드임. 전부 같은 날짜에 RC로 올라왔음.

| 항목 | 빌드 |
|---|---|
| Xcode 27 RC | 27A266a |
| iOS 27.0 RC / iPadOS 27.0 RC | 24A435 |
| macOS 27.0 RC | 26A428 |
| tvOS 27.0 RC | 24J360 |
| visionOS 27.0 RC | 24M362 |
| watchOS 27.0 RC | 24R363 |

공지가 요구하는 순서는 단순함. Xcode 27 RC를 받아 최신 SDK로 빌드하고, TestFlight로 테스트하고, 심사에 제출하는 것. 공지는 제출 대상으로 Apple Intelligence와 **Foundation Models 프레임워크**를 이름으로 꼽음.

## 새로 생긴 기한 둘

**1. 2027년 4월부터 최소 SDK 요건.** 그 시점 이후 App Store Connect에 업로드하는 앱과 게임은 아래를 만족해야 함.

| 플랫폼 | 요건 |
|---|---|
| iOS · iPadOS | **iOS 27 & iPadOS 27 SDK 이상**으로 빌드 |
| tvOS | tvOS 27 SDK 이상 |
| visionOS | visionOS 27 SDK 이상 |
| watchOS | watchOS 27 SDK 이상 |

즉 오늘 당장 27 SDK로 옮기지 않아도 되지만, 다음 봄까지는 옮겨야 함. 기존 앱이 있으면 이번 사이클에 Xcode 27로 한 번 빌드해 보고 깨지는 곳을 목록으로 만들어 두는 편이 싸게 먹힘.

**2. macOS의 Intel과 Rosetta 종료.** 9월 1일 공지가 개발자 쪽 조치를 이렇게 정리함.

- **macOS 26.4 이상**: Rosetta에 의존하는 앱을 실행하면 사용자에게 시스템 알림이 뜰 수 있음. 네이티브 버전으로 갱신하라는 안내임
- **macOS 27**: Rosetta를 지원하는 마지막 릴리스. 이후로는 Intel 전용 앱이 Apple Silicon Mac에서 돌지 않음
- 예외로 **Intel 기반 프레임워크에 의존하는 오래된 게임 타이틀**에 대한 Rosetta 지원은 계속됨

9월 9일 제출 공지 쪽은 여기에 한 줄을 더 붙임. Apple Silicon Mac으로 배포 대상을 좁히려면 **Xcode 빌드 아키텍처를 `arm64`만으로 설정하고 다시 빌드해 제출**하라는 것.

## 연령등급 문항이 바뀜

제출 공지가 연령등급 항목을 따로 뗐음. 둘임.

- **Time Allowances**: 보호자가 Entertainment, Games, Social Media 같은 카테고리별로 사용 시간을 조절할 수 있게 하는 장치
- **소셜미디어 기능 신고**: 앱이나 게임에 소셜미디어 성격의 기능이 있으면 **App Store Connect에 그것을 표시해야 함**

[앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md)에서 정리한 **2.3.6(연령등급 정확성)** 이 이번 사이클에 실제 문항 변경으로 내려온 것임. 그쪽 문서가 "잘못 매기면 규제 기관 조회로 이어질 수 있다고 지침이 직접 적음"이라고 적어둔 항목이라, 생성형 대화 기능을 붙인 앱이면 이번 제출에서 문항을 다시 읽어야 함. 사용자가 만든 콘텐츠가 다른 사용자에게 닿는 구조면 소셜미디어 신고 대상인지도 같이 판단해야 함.

제품 페이지 쪽 변경도 함께 안내됐음. 새 **product page header와 검색 결과 에셋** 규격이 생겼고, App Store Connect의 미리보기 도구는 **coming soon**으로 적혀 있음.

## 이 저장소의 다른 문서가 이 시점에 다시 걸림

이 저장소의 Apple 관련 문서 넷이 전부 "iOS 27 정식 출시 때 재확인"을 유효기간에 적어뒀음. 그 시점이 이번 주임.

| 문서 | 이번 주에 확인할 것 |
|---|---|
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 4K·32K 컨텍스트 값, PCC entitlement 승인 절차, `quotaUsage` 계열 API가 베타 문서와 같은지 |
| [Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md) | Foundation Models 프로토콜 개방, Core AI, PCC 무료 구간 조건이 정식에서 그대로인지 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 베타 6 기준으로 정리한 플러그인·MCP·ACP 항목이 RC에서 살아남았는지 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | 연령등급 문항 변경과 소셜미디어 신고 요건 |

## 짚어야 할 것

- **Xcode 27 RC 릴리스 노트 본문은 읽지 못했음.** 문서 페이지가 존재하는 것과 제목이 `Xcode 27 RC Release Notes`라는 것까지만 확인했고, RC에서 무엇이 해결되고 무엇이 남았는지는 확인할 수 없었음. 위 [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)이 베타 6 기준이라 대조가 필요한데, 그 대조는 아직 못 한 상태임
- **Rosetta에 대한 두 공지의 문언이 어긋남.** 9월 1일 공지는 "macOS 27: Final release to support Rosetta"라고 적고, 9월 9일 제출 공지는 "macOS 26 is the final release supporting Intel Mac computers and Rosetta"라고 적음. 앞은 Apple Silicon에서 도는 번역 계층의 마지막 버전을, 뒤는 Intel Mac에서 도는 마지막 OS를 말하는 것으로 읽히지만, 두 문장이 Rosetta를 같은 자리에 놓고 있어서 **어느 쪽이 정확한지 확인할 수 없었음.** macOS 앱을 배포 중이면 Apple에 직접 확인하는 편이 안전함
- **iOS 27의 9월 14일 날짜는 Apple 뉴스룸 문장에서 가져온 것임.** "iOS 27 will be available as a free software update on Monday, September 14." 개발자 뉴스 쪽 공지는 날짜를 적지 않고 "will soon be available"이라고만 함
- **Apple Intelligence와 Siri AI의 언어 일정이 다름.** 뉴스룸 기준으로 Apple Intelligence는 9월 14일에 한국어를 포함한 지원 언어에서 쓸 수 있고, Siri AI는 같은 날 **영어 베타**로만 시작해 한국어를 포함한 추가 언어는 **10월**에 붙음. 한국어 앱에서 Siri 쪽 연동을 전제로 기능을 잡고 있으면 이 간격을 계산에 넣어야 함
- 2027년 4월 요건은 **업로드 시점 기준**임. 이미 스토어에 올라가 있는 빌드가 그 날짜에 내려간다는 얘기가 아님
- 이 문서는 **공지 문언만 정리한 것**이고 RC를 실제로 받아 빌드해 본 기록이 아님

## 유효기간

**2026-09-10 확인 기준**임. RC 빌드 번호는 정식 출시(9월 14일 예정) 시점에 최종 빌드로 교체됨. 다시 볼 때는 [Releases](https://developer.apple.com/news/releases/)에서 RC가 정식으로 바뀌었는지 보고, Xcode 27 릴리스 노트의 Coding Intelligence 절을 베타 6 기준 정리와 대조하는 것이 남은 일임. 2027년 4월 SDK 요건은 그때까지 유효한 기한이라 낡지 않음.
