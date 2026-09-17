---
title: iOS 27 · Xcode 27 정식 출시, 출시 문구로 재확인한 AI API 표면
source: https://developer.apple.com/apple-intelligence/whats-new/
author: Apple
published: 2026-09-14
collected: 2026-09-15
tags: [apple, ios, foundation-models, core-ai, xcode, on-device-llm, app-intents]
---

출처: [Apple Intelligence, What's New](https://developer.apple.com/apple-intelligence/whats-new/) · [AI & Machine Learning, What's New](https://developer.apple.com/machine-learning/whats-new/) · [iOS, What's New](https://developer.apple.com/ios/whats-new/) · [Releases](https://developer.apple.com/news/releases/) · [Xcode 27 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes) · [iOS & iPadOS 27 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27-release-notes) · [App Store submissions now open for the latest OS releases](https://developer.apple.com/news/?id=k1mtkt1k) (2026-09-09) · [Upcoming changes to Rosetta support for Intel-based macOS apps](https://developer.apple.com/news/?id=w5ngl9k2) (2026-09-01) · [Major updates for Apple's software platforms are now available](https://www.apple.com/newsroom/2026/09/major-updates-for-apples-software-platforms-are-now-available/) (2026-09-14) (릴리스 노트 둘은 **2026-09-17 확인**, 나머지는 2026-09-15 확인)

## 요약

이 저장소의 Apple 문서 두 개가 유효기간에 **"iOS 27 정식 출시 시점에 재확인하라"**고 적어뒀는데, 그 시점이 **2026-09-14**로 지나감. 결론부터: 베타로 적어둔 항목들이 출시 OS의 What's New 문구에 그대로 들어 있고 베타 표시 없이 서술됨. **다만 이것이 각 API의 베타 딱지가 떨어졌다는 확인은 아님.** 프레임워크 문서의 가용성 표시는 이번에 대조하지 못했고, [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)은 PCC·Core AI·dynamic profiles·attachments 를 여전히 Beta 로 적어둔 상태임. **Xcode 27(27A266a)과 iOS 27.0(24A437)이 같은 날 정식**으로 나왔고, Foundation Models의 `LanguageModel` 프로토콜 개방과 Core AI, MLX 백엔드, Private Cloud Compute **무료 구간(Small Business Program 가입 + 다운로드 200만 미만)**이 출시 문구에 그대로 남아 있음. 새로 확인된 표면은 셋임. **View Annotations API**, **App Intents Testing**, **Evaluations**. 반면 사용자 쪽 **Siri AI는 여전히 베타**이고 **한국어는 10월**, EU의 iOS·iPadOS·watchOS에서는 초기 미제공임.

정식 릴리스 노트까지 대조해서 붙인 것이 넷임. **Xcode 27 이 요구하는 macOS 가 26.6 으로 올라갔고**, 워크스페이스 없이 도는 **MCP 서버는 승격됐다는 서술이 없고** (노트가 누적 문서라 이것이 승격되지 않았다는 증명은 아님), 파일시스템 접근 감시 계층은 **켜야 도는 것**임. 그리고 **Apple 이 만든 `SpotlightSearchTool` 을 기본 설정으로 붙이면 프롬프트를 넣기도 전에 온디바이스 컨텍스트가 넘침.**

## 무엇이 언제 나왔나

릴리스 목록에서 받은 빌드 번호와 날짜임.

| 항목 | 버전 | 날짜 |
|---|---|---|
| Xcode 27 | 27A266a | **2026-09-14** |
| iOS 27.0 / iPadOS 27.0 | 24A437 | 2026-09-14 |
| macOS 27.0 | 26A428 | 2026-09-14 |
| tvOS 27.0 | 24J361 | 2026-09-14 |
| visionOS 27.0 | 24M362 | 2026-09-14 |
| watchOS 27.0 | 24R364 | 2026-09-14 |
| App Store 제출 개방 + Xcode 27 RC | 27A266a | 2026-09-09 |

RC(09-11 기준 iOS 27.0 RC)와 정식의 빌드 번호가 `24A437`로 같음. 제출 개방은 정식 출시 **5일 전**이었음.

## 출시 문구에 그대로 남은 것

[Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md)에서 "지금 쓸 수 있나" 표에 **베타(iOS 27)**로 적어둔 항목들이 출시 OS의 What's New 문구에 베타 표시 없이 들어 있음. **6월 발표 문구와 출시 시점 문구가 갈리지 않는다는 것까지가 이번에 확인한 것이고, 각 API 의 가용성 표시가 실제로 바뀌었는지는 아님.**

**모델 백엔드 교체.** 프레임워크 페이지가 이렇게 적음.

> "You can now work with any language model, including Apple Foundation Models, cloud models like Claude and Gemini, or any other provider that conforms to the Language Model protocol."

**PCC 무료 구간의 조건이 그대로임.** 6월 발표와 문언이 같음.

> "If you're enrolled in the App Store Small Business Program and your app has fewer than 2 million total first-time App Store downloads, you can access the next generation of Apple Foundation Models running on Private Cloud Compute at no cloud API cost."

즉 **다운로드 200만 미만**이라는 기준과 **클라우드 API 비용이 0**이라는 범위가 출시 시점에도 유지됨. [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)에서 갈라둔 대로 이건 사용자 1인당 일일 한도가 없다는 뜻이 아님. 아래 사용자 쪽 조건 항목과 같이 봐야 함.

## 새로 확인된 API 표면

이 저장소에 아직 안 적힌 것들임.

**View Annotations API.** App Intents 쪽에 붙음.

> "The new View Annotations API adds on-screen awareness on top of all of this, letting you map your views to entities so people can reference and act on what's right in front of them conversationally."

화면에 떠 있는 뷰를 엔티티로 매핑해서 사용자가 "이거"라고 지칭할 수 있게 하는 것임. **App Intents 프레임워크에 속하고, 문구가 가리키는 것은 Siri·Shortcuts 같은 시스템 대화 경로임.** 앱이 직접 여는 `LanguageModelSession` 의 프롬프트나 컨텍스트를 이것이 대신 채워준다는 서술은 **원문에 없음.** 앱 안에 자체 대화형 표면을 만드는 경우와 이 API 가 어떻게 만나는지는 프레임워크 문서로 확인해야 함

**App Intents Testing.** Siri·Shortcuts·Spotlight 통합을 UI 자동화 없이 실제 시스템 경로로 검증하는 프레임워크가 생김. 에이전트가 만든 App Intents 구현을 사람이 손으로 눌러 확인하던 구간이 테스트로 내려옴.

**Evaluations.** 이름만 알려져 있던 것이 출시 문구에 들어옴.

> "With the Evaluations framework, you can verify that your AI features behave correctly across dynamic conditions, going beyond what unit tests alone can catch."

[자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)이 말한 것, 즉 **판정 기준을 루프 바깥에 두는 일**을 앱 개발 쪽에서 프레임워크로 하겠다는 것임. 규칙 6(제안한 쪽이 기준선을 못 옮기게)이 여기서 어떻게 보장되는지는 문서 본문을 봐야 알 수 있고, 이번에는 **확인하지 못했음.**

**멀티모달과 Vision 툴.** 이미지 입력이 되는 것까지는 6월에 나왔는데, 툴 쪽 서술이 구체적으로 바뀜.

> "Multimodal prompts let you pass images alongside text so your app can reason about visual content, and Vision framework tools like OCR and barcode readers are available for your model to call directly, all on-device."

OCR과 바코드 리더가 **모델이 직접 호출하는 툴**로 제공됨. 툴 정의가 컨텍스트 창을 먹는다는 제약은 그대로이므로, 요청당 툴 3~5개 권고 안에서 자리를 어떻게 나눌지가 설계 문제가 됨.

**Dynamic Profiles.** 세션을 유지한 채 모델·툴·instructions를 런타임에 갈아끼우는 구조로 서술됨. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 관점에서 보면 **사용 가능한 것의 목록을 세션 중간에 바꾸는 행위**라 캐시 prefix가 걸리는 자리인데, 온디바이스 KV 캐시에서 이게 어떻게 처리되는지는 이 페이지들로는 알 수 없었음.

## Core AI와 MLX

Core AI는 Core ML 후계로 6월에 발표된 그 프레임워크임. 출시 문구가 목적을 더 좁게 적음.

> "Core AI is a new framework built directly into the OS and purpose-built for Apple Silicon, providing the best way to bring your own models on-device"

메모리 안전한 Swift API, 하드웨어별 자동 특수화와 AOT 컴파일, 추론 메모리 제어와 zero-copy 데이터 경로, stateful 실행이 항목으로 나열됨. "서버 의존 0, 토큰 비용 0"이라는 문구도 유지됨.

MLX 쪽에 새 항목이 둘 있음. **Metal 4와 GPU Neural Accelerators 지원**, 그리고 **Thunderbolt 위 RDMA로 여러 대의 Mac에 학습을 분산**하는 것. 후자는 개인 개발자 규모에서 바로 쓸 일이 없지만, 파인튜닝을 사내 Mac으로 돌리는 선택지가 생겼다는 신호로는 읽을 만함.

## 사용자 쪽 조건이 앱 설계에 들어옴

프레임워크가 출시됐다는 것과 사용자 기기에서 기능이 돈다는 것은 다른 층임. 뉴스룸이 붙인 조건이 이럼.

> "Siri AI is now rolling out as a beta in English, with French, Japanese, Korean, Portuguese, and Spanish coming in October."

**한국어는 10월**임. 한국어 앱에서 Siri AI 경로를 전제로 기능을 잡아뒀다면 출시 일정이 한 달 밀리는 셈임.

> "Siri AI will not be available initially in the EU in iOS, iPadOS, and watchOS."

EU의 Mac과 Apple Vision Pro는 지원 언어로 설정하면 접근 가능하다고 같은 문단이 적음. 즉 **플랫폼별로 갈림.**

> "Certain Apple Intelligence features that rely on server-side models are subject to daily usage limits, including but not limited to Siri AI, intelligent photo editing tools, Image Playground, and AFM 3 Cloud models in Shortcuts."

**일일 한도가 공식 문구로 남아 있음.** [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)에서 정리한 `quotaUsage`·`limitIncreaseSuggestion`·`quotaLimitReached(_:)` 처리가 선택이 아니라 출시 요건이라는 뜻임. PCC를 붙일 계획이면 한도 도달 UI를 기능 범위에 포함시켜야 함.

Apple Intelligence 지원 언어 목록에 한국어가 들어 있고, 지원 기기는 iPhone 15 Pro·iPhone 16 이후·M1 이후 iPad와 Mac 계열로 적혀 있음. 전체 목록은 뉴스룸 각주에 있음.

## 제출 쪽에서 같이 바뀐 것

개발자 공지 쪽에 AI와 직접 관련은 없지만 같이 챙길 것이 셋 있음.

- **2027-04 부터 업로드 최소 SDK 요건이 생김.** 날짜가 박힌 사실이라 이 항목은 낡지 않음

| 플랫폼 | 요건 |
|---|---|
| iOS · iPadOS | `iOS 27 & iPadOS 27 SDK or later` |
| tvOS | `tvOS 27 SDK or later` |
| visionOS | `visionOS 27 SDK or later` |
| watchOS | `watchOS 27 SDK or later` |

오늘 당장 옮기지 않아도 되지만 다음 봄까지는 옮겨야 함. 기존 앱이 있으면 이번 사이클에 Xcode 27 로 한 번 빌드해 깨지는 곳을 목록으로 만들어 두는 편이 쌈.


- **Intel 과 Rosetta 는 종료 시점이 다름.** 09-09 제출 공지는 "macOS 26 is the final release supporting Intel Mac computers and Rosetta" 로 둘을 묶어 적었는데, 이 문장만 보면 Rosetta 가 macOS 26 에서 끝나는 것으로 읽힘. **Rosetta 전용 공지(2026-09-01)의 타임라인이 그렇지 않다는 것을 보여줌.** macOS 26.4 이상에서는 Rosetta 에 의존하는 앱을 실행할 때 네이티브 버전으로 갱신하라는 시스템 알림이 뜰 수 있고, **macOS 27 이 "Final release to support Rosetta"** 이며 그 이후로 Intel 전용 앱이 Apple Silicon Mac 에서 돌지 않음. 단계가 26.4 에서 27 로 이어지므로 Rosetta 는 macOS 27 까지 감. 갈리는 것은 **Intel Mac 에서 도는 마지막 OS(macOS 26)** 와 **Apple Silicon 에서 Intel 앱을 번역해 주는 마지막 OS(macOS 27)** 임. 제출 공지의 문장은 둘을 뭉쳐 놓은 요약으로 읽어야 함. 유지보수가 끊긴 구형 게임 타이틀은 예외로 계속 지원됨. 실무 결론은 그래도 같음. arm64 전환을 이번 사이클에 끝내는 것임
- **연령등급 문항에 소셜 미디어 기능 표시가 추가됨.** 새 Time Allowances 때문이고, 앱이 소셜 미디어 기능을 담으면 App Store Connect에서 밝혀야 함. [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md)의 2.3.6 항목에 걸리는 변경임. 생성형 대화 기능이 사용자 간 노출로 이어지는 구조면 이 문항을 다시 봐야 함

## 릴리스 노트가 답한 것

앞서 이 문서는 릴리스 노트 본문을 읽지 못했다고 적었음. 문서 페이지가 자바스크립트로 렌더링되기 때문인데, 같은 경로의 JSON(`/tutorials/data/...json`)으로 정식 노트 본문을 받아 아래를 확인했음. **RC 가 아니라 정식 노트 기준임.**

**Xcode 27 은 macOS Tahoe 26.6 이상을 요구함.**

> "Xcode 27 requires a Mac running macOS Tahoe 26.6 or later."

[Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)은 beta 6 기준으로 **26.4 이상**이라고 적어뒀음. 정식에서 한 칸 올라간 것이라 **빌드 머신 OS 를 고정해 둔 CI 는 여기서 걸림.** 그 문서는 이 PR 에서 손대지 않았음. Swift 6.4 와 27 SDK 묶음, 온디바이스 디버깅 대상(iOS 17 이상, tvOS 17 이상, watchOS 10 이상, visionOS)은 그대로임.

**워크스페이스 없이 도는 MCP 서버는 승격됐다는 서술이 없음.** 베타 6 문서가 "정식에서 빠지거나 바뀔 수 있다"고 지목한 바로 그 기능인데, 정식 노트에서 이 기능을 설명하는 문장은 여전히 이것 하나임 (181836944).

> "Xcode 27 Beta 5 adds a preview of a new MCP server experience that runs without requiring an open Xcode workspace."

> "In this early preview, some aspects of the [xcrun mcp-server] command line utility may not work in all configurations, and some settings or permissions may occasionally require relaunching Xcode or rebooting your machine to apply."

무인 환경에서 권한을 미리 전부 승인하는 구성에 대해 **"This is not a recommended configuration for at-desk use"** 라는 단서도 그대로임.

**다만 이것을 "정식에서도 preview 로 남았다" 는 확인으로 읽으면 안 됨.** 이 페이지는 릴리스 주기 전체를 누적하는 문서라 베타 시절 문구가 그대로 남음. 같은 페이지의 USDKit 절이 `Beta 1` 과 `Beta 2` 사이의 호환성 이야기를 아직 담고 있는 것이 그 증거임. 그러므로 위 문장이 남아 있다는 사실이 승격되지 않았음을 증명하지는 않음. **확인한 것은 정식 노트가 이 기능의 승격을 알리지 않았고, `xcrun mcp-server` 를 설명하는 문구가 early preview 단서를 단 채로 남아 있다는 것까지임.** 별도의 기능 문서를 찾지 못해 더 좁히지 못했음.

실무 판단은 그래도 같음. 코드 서명된 에이전트에게 디렉터리 트리 권한을 오래 주는 모델을 CI 에 넣을 계획이면, 승격이 확인되기 전에는 **preview 전제로 설계하는 편이 안전함.**

**파일시스템 접근 감시 계층은 기본값이 아님** (178289431).

> "Coding Intelligence now includes a new security layer that monitors and controls filesystem access by coding agents and any processes they spawn. This can be enabled in Coding Intelligence settings."

켜는 것이지 켜져 있는 것이 아님. 에이전트가 만드는 프로세스까지 범위에 들어간다는 것이 이 계층의 값어치인데, 설정에서 켜지 않으면 없는 것과 같음.

**둘 다 켜야 도는 것이고, 스위치가 서로 다른 데 있음.** 권한을 넓히는 쪽은 `xcrun mcp-server enable` 로 켜고 무인 환경의 일괄 승인은 `--unsafe-always-allow-all-agents` 라는 별도 플래그이며, 감시하는 쪽은 Coding Intelligence 설정에서 켬. 기본값으로 열려 있는 것은 없음.

문제가 되는 조합은 그래서 **한쪽만 켜는 것**임. CI 를 굴리려고 권한 쪽을 켜는 것은 목적이 분명해서 하게 되는데, 감시 쪽은 켜야 할 이유가 그 순간 눈에 보이지 않아 넘어가기 쉬움. [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md)가 내린 결론, 즉 승인 정확도를 올리는 것보다 위험한 경로를 아예 승인 대상에서 빼는 편이 싸다는 판단이 여기서 그대로 필요함. 권한 쪽을 켤 때 감시 쪽 설정도 같이 열어보면 끝나는 일임.

**베타에서 걸리던 것 셋이 해결 목록에 있음.** [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)이 경고로 적어둔 항목이 여기 포함됨.

| 이슈 | 내용 |
|---|---|
| 178673449 | 에이전트가 스트리밍 중인데 계획 확인 바가 떠서 버튼을 누르면 진행 중인 턴 위에 새 턴이 얹히던 문제. **회피법(응답이 끝난 뒤 누르기)을 더 지킬 필요가 없음** |
| 179171480 | `Apple-authored agent skills may not be available to Codex.` |
| 178771195 | 플러그인으로 추가한 ACP 에이전트를 제거해도 Xcode 를 재실행하기 전까지 UI 에 남던 문제 |

가운데 것이 값어치가 큼. 그 문서가 이 항목을 "어느 에이전트에 무엇이 노출되는지 균일하지 않을 수 있다" 는 신호로 읽으라고 적었는데, **신호 자체는 사라짐.** 다만 버그 하나가 고쳐진 것이지 노출 균일성이 보장된다는 서술은 노트 어디에도 없음. 서드파티 에이전트를 붙일 때 스킬이 실제로 보이는지는 여전히 직접 확인해야 함.

## 온디바이스 컨텍스트, Apple 자기 툴이 기본 설정으로는 안 들어감

iOS 27 정식 노트의 Core Spotlight 절에 **알려진 이슈**로 들어 있음 (183770678). 이 문서에서 실무에 제일 크게 걸리는 항목임.

> "Creating a SpotlightSearchTool without a configuration and using it with a LanguageModelSession backed by the on-device system language model fails with an error reporting that the number of tokens provided exceeds the maximum allowed. The tool's default configuration is sized for models with large context windows, so the tool's description and parameter schema alone exceed the on-device model's context window before any prompt is added."

**Apple 이 만든 툴 하나를 기본 설정으로 붙이는 것만으로 온디바이스 모델의 컨텍스트가 터짐.** 프롬프트를 넣기도 전에. 회피법도 노트가 같이 줌.

```swift
let configuration = SpotlightSearchTool.Configuration(
  sources: [.coreSpotlight],
  guide: .focused()
)
let tool = SpotlightSearchTool(configuration: configuration)
```

도메인을 좁히려면 `.focused()` 에 도메인을 넘김. focused guide 는 기본 설정보다 **적은 검색 능력**을 노출하고, 컨텍스트가 큰 모델을 쓰는 세션은 기본 설정을 그대로 써도 됨.

[Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)이 문서 권고를 근거로 "툴 정의와 Generable 스키마가 창을 먹는다", "툴은 요청당 3~5개"라고 적었는데, 이 알려진 이슈는 그 서술을 한 칸 더 강하게 만듦. **3~5개는 상한이 아니라 툴이 작을 때의 얘기고, 스키마가 큰 툴은 하나로도 안 들어감.** 한국어 앱이면 여기에 글자당 토큰이 더 붙는 조건이 겹침. 순서는 그래서 이렇게 됨.

1. 툴을 붙이기 전에 툴 정의만 먼저 재 봄
2. 1차 프레임워크 툴이라고 안전하다고 가정하지 않음. 기본 설정이 어느 모델을 전제로 만들어졌는지 확인
3. 온디바이스에서 안 들어가면 툴을 빼는 게 아니라 **좁히는 설정이 있는지** 먼저 봄

## Neural Engine, 백그라운드 접근에 엔타이틀먼트가 생김

정식 노트의 Core AI 절 새 기능 둘임.

| 항목 | 내용 |
|---|---|
| Neural Engine 동작 변경 (174796039) | 백그라운드 NE 접근이 **GPU 제한과 비슷하게 제한**됨. **1GB 초과** 대형 모델 로딩 성능 개선. NE 메모리 사용량이 시스템이 아니라 **앱 프로세스에 귀속**되고 Allocations instrument 에 나타남 |
| 백그라운드 엔타이틀먼트 (179282606) | 앱이 백그라운드일 때 NE 에 접근하려면 `com.apple.developer.background-tasks.continued-processing.inference` 가 필요함 |

둘 다 설계에 직접 걸림. 메모리 귀속이 바뀐 것은 계측이 쉬워졌다는 뜻이면서 동시에 **NE 메모리가 앱 메모리 예산에 잡힌다**는 뜻임. 온디바이스 모델을 올려두고 도는 앱이면 jetsam 여유가 줄어드는 쪽으로 움직임. 백그라운드 추론을 전제로 잡아둔 기능이 있으면 엔타이틀먼트 신청이 새 작업으로 생김.

[Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md)이 Core AI 를 "서버 의존 없음, 토큰 비용 없음"으로 요약했는데, 비용이 없는 대신 **메모리와 백그라운드 권한이 예산 항목으로 들어온 것**이 이번에 확인된 형태임.

## Foundation Models 는 새 기능 없이 수정만 있음

정식 노트의 Foundation Models 절은 전부 해결 항목이고 새 기능이 없음. 설계에 영향이 있는 것들.

| 이슈 | 내용 |
|---|---|
| 177684296 | **PCC 가 시뮬레이터에서 동작하지 않던 문제**가 고쳐짐 |
| 177748926 | 온디바이스 모델이 툴 호출과 guided generation 을 같이 쓸 때 **툴을 과도하게 부르던** 문제 |
| 178181782 | PCC 언어 모델이 **항상 greedy decoding** 을 쓰던 문제 |
| 177901494 | 트랜스크립트 히스토리를 잘라낼 때 나던 런타임 오류 |
| 177902488 | instructions 없는 프로필에 붙인 수정자가 호출되지 않던 문제 |
| 177899620 | `@Generable` 이 억제 불가능한 deprecation 경고를 내던 문제 |

시뮬레이터에서 PCC 가 돌게 된 것이 이 중 제일 큼. 베타 기간에는 PCC 경로를 실기기로만 확인할 수 있었음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md) | 6월 발표 시점의 예고. 그 문서의 유효기간이 지목한 재확인 시점이 이번임 |
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 베타 문서로 확인한 API 제약. 컨텍스트·한도 수치는 이번에 재확인하지 못했음 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | beta 6 기준 항목. macOS 요건이 26.4 에서 **26.6** 으로 올라갔고, 그 문서가 경고로 적어둔 버그 둘이 해결됨. MCP 서버는 정식 노트에 **승격을 알리는 서술이 없음** |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | 연령등급 문항 변경, 서드파티 전송 공개 요건 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | Evaluations 프레임워크가 같은 루프를 앱 개발 쪽으로 들여옴 |

## 짚어야 할 것

- **릴리스 노트는 릴리스 주기 전체를 누적하는 문서임.** 베타 시절 문구가 남아 있을 수 있으므로, 어떤 문장이 남아 있다는 사실만으로 그 기능의 현재 상태를 단정하면 안 됨. 반대로 해결 항목은 베타 중에 고쳐졌더라도 정식 빌드에 들어 있음
- **릴리스 노트 대조는 위 항목들까지임.** macOS 요건, MCP 서버에 대한 서술, 파일시스템 계층을 켜야 한다는 것, 해결 항목은 확인했으나, [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)에 적은 beta 6 항목 **전체를 정식 노트와 대조하지는 않았음**
- **근거로 쓴 What's New 페이지들은 요약 성격임.** 타입 이름, 시그니처, 가용성 표시는 프레임워크 문서로 다시 확인해야 함. 이 문서는 "무엇이 출시됐다고 Apple이 말하는가"까지임
- **컨텍스트 4K / 32K, 추론 레벨, 일일 한도의 구체적 수치는 이번에도 재확인하지 않았음.** [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)의 값은 베타 문서 기준 그대로 남아 있음. 위 SpotlightSearchTool 이슈는 그 수치를 재확인해 준 것이 아니라 **기본 설정 툴 하나가 창을 넘긴다는 사실**을 알려주는 것임
- **릴리스 노트의 이슈 번호는 Apple 이 붙인 것을 그대로 옮겼음.** 각 항목이 어느 빌드에서 고쳐졌는지까지는 노트가 밝히지 않음
- Xcode의 코딩 에이전트 관련 페이지는 "원하는 모델로 에이전트를 돌린다"는 수준의 문구만 있고 플러그인·MCP·ACP를 언급하지 않음. **없어졌다는 뜻이 아니라 그 페이지가 다루지 않는 것**이므로 릴리스 노트로 확인해야 함
- 지원 기기 목록에 새 하드웨어 이름이 들어 있는데 이 문서는 하드웨어를 다루지 않음

## 유효기간

**2026-09-15 확인 기준이고, 릴리스 노트 대조는 2026-09-17 에 함.** 출시일과 빌드 번호는 고정된 기록이라 낡지 않음. 반면 Siri AI 언어 확대(10월 예정)와 EU 제공 여부는 움직이고, What's New 페이지 문구는 다음 OS 사이클에 통째로 교체됨. 다시 볼 때는 [개발자 뉴스](https://developer.apple.com/news/)와 릴리스 목록에서 27.1 이후 갱신을 먼저 확인할 것.
