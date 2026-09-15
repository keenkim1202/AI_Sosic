---
title: iOS 27 · Xcode 27 정식 출시, 출시 문구로 재확인한 AI API 표면
source: https://developer.apple.com/apple-intelligence/whats-new/
author: Apple
published: 2026-09-14
collected: 2026-09-15
tags: [apple, ios, foundation-models, core-ai, xcode, on-device-llm, app-intents]
---

출처: [Apple Intelligence, What's New](https://developer.apple.com/apple-intelligence/whats-new/) · [AI & Machine Learning, What's New](https://developer.apple.com/machine-learning/whats-new/) · [iOS, What's New](https://developer.apple.com/ios/whats-new/) · [Releases](https://developer.apple.com/news/releases/) · [App Store submissions now open for the latest OS releases](https://developer.apple.com/news/?id=k1mtkt1k) (2026-09-09) · [Upcoming changes to Rosetta support for Intel-based macOS apps](https://developer.apple.com/news/?id=w5ngl9k2) (2026-09-01) · [Major updates for Apple's software platforms are now available](https://www.apple.com/newsroom/2026/09/major-updates-for-apples-software-platforms-are-now-available/) (2026-09-14) (전부 2026-09-15 확인)

## 요약

이 저장소의 Apple 문서 두 개가 유효기간에 **"iOS 27 정식 출시 시점에 재확인하라"**고 적어뒀는데, 그 시점이 **2026-09-14**로 지나감. 결론부터: 베타로 적어둔 항목들이 출시 OS의 What's New 문구에 그대로 들어 있고 베타 표시 없이 서술됨. **다만 이것이 각 API의 베타 딱지가 떨어졌다는 확인은 아님.** 프레임워크 문서의 가용성 표시는 이번에 대조하지 못했고, [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)은 PCC·Core AI·dynamic profiles·attachments 를 여전히 Beta 로 적어둔 상태임. **Xcode 27(27A266a)과 iOS 27.0(24A437)이 같은 날 정식**으로 나왔고, Foundation Models의 `LanguageModel` 프로토콜 개방과 Core AI, MLX 백엔드, Private Cloud Compute **무료 구간(Small Business Program 가입 + 다운로드 200만 미만)**이 출시 문구에 그대로 남아 있음. 새로 확인된 표면은 셋임. **View Annotations API**, **App Intents Testing**, **Evaluations**. 반면 사용자 쪽 **Siri AI는 여전히 베타**이고 **한국어는 10월**, EU의 iOS·iPadOS·watchOS에서는 초기 미제공임.

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

개발자 공지 쪽에 AI와 직접 관련은 없지만 같이 챙길 것이 둘 있음.

- **Intel·Rosetta 종료 시점은 두 공지의 문언이 어긋남.** 2026-09-09 제출 공지는 "macOS 26 is the final release supporting Intel Mac computers and Rosetta" 라고 적었는데, 2026-09-01 Rosetta 전용 공지는 "macOS 27: Final release to support Rosetta" 라고 적음. "Intel Mac 에서 도는 마지막 OS" 와 "Apple Silicon 에서 Intel 앱을 번역해 주는 마지막 OS" 를 갈라 읽으면 둘 다 설 여지가 있으나, 앞 문장이 Rosetta 를 macOS 26 에 묶어버려서 그 독해가 확정되지 않음. **어느 쪽이 맞는지 확인하지 못했음.** 실무 결론은 어느 쪽이든 같음. macOS 앱을 내고 있으면 arm64 전환을 지금 끝내는 것임. Rosetta 공지는 유지보수가 끊긴 구형 게임 타이틀에 예외를 둔다고 적었음
- **연령등급 문항에 소셜 미디어 기능 표시가 추가됨.** 새 Time Allowances 때문이고, 앱이 소셜 미디어 기능을 담으면 App Store Connect에서 밝혀야 함. [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md)의 2.3.6 항목에 걸리는 변경임. 생성형 대화 기능이 사용자 간 노출로 이어지는 구조면 이 문항을 다시 봐야 함

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md) | 6월 발표 시점의 예고. 그 문서의 유효기간이 지목한 재확인 시점이 이번임 |
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 베타 문서로 확인한 API 제약. 컨텍스트·한도 수치는 이번에 재확인하지 못했음 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | beta 6 기준 항목. 정식 릴리스 노트와의 대조는 못 했고 빌드 번호와 날짜만 확인함 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | 연령등급 문항 변경, 서드파티 전송 공개 요건 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | Evaluations 프레임워크가 같은 루프를 앱 개발 쪽으로 들여옴 |

## 짚어야 할 것

- **Xcode 27 정식 릴리스 노트 본문을 읽지 못했음.** 해당 문서 페이지가 자바스크립트로 렌더링돼 자동으로는 열리지 않음. 그래서 [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)에 적은 beta 6 항목 중 **무엇이 정식에서 빠졌거나 바뀌었는지 대조하지 못했음.** 워크스페이스 없이 도는 MCP 서버가 여전히 프리뷰인지, 파일시스템 감시 계층이 기본값이 됐는지가 그중 제일 궁금한 부분인데 확인 못 함
- **근거로 쓴 What's New 페이지들은 요약 성격임.** 타입 이름, 시그니처, 가용성 표시는 프레임워크 문서로 다시 확인해야 함. 이 문서는 "무엇이 출시됐다고 Apple이 말하는가"까지임
- **컨텍스트 4K / 32K, 추론 레벨, 일일 한도의 구체적 수치는 이번에 재확인하지 않았음.** [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)의 값은 베타 문서 기준 그대로 남아 있음
- Xcode의 코딩 에이전트 관련 페이지는 "원하는 모델로 에이전트를 돌린다"는 수준의 문구만 있고 플러그인·MCP·ACP를 언급하지 않음. **없어졌다는 뜻이 아니라 그 페이지가 다루지 않는 것**이므로 릴리스 노트로 확인해야 함
- 지원 기기 목록에 새 하드웨어 이름이 들어 있는데 이 문서는 하드웨어를 다루지 않음

## 유효기간

**2026-09-15 확인 기준**임. 출시일과 빌드 번호는 고정된 기록이라 낡지 않음. 반면 Siri AI 언어 확대(10월 예정)와 EU 제공 여부는 움직이고, What's New 페이지 문구는 다음 OS 사이클에 통째로 교체됨. 다시 볼 때는 [개발자 뉴스](https://developer.apple.com/news/)와 릴리스 목록에서 27.1 이후 갱신을 먼저 확인할 것.
