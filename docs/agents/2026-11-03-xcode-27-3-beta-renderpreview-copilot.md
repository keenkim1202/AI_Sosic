---
title: Xcode 27.3 beta의 에이전트 표면, RenderPreview가 엉뚱한 프리뷰를 렌더하던 버그가 고쳐짐
source: https://developer.apple.com/documentation/xcode-release-notes/xcode-27_3-release-notes
author: Apple
published: 2026-11-03
collected: 2026-11-05
tags: [apple, xcode, coding-agent, mcp, agent-eval, tooling]
---

출처: [Xcode 27.3 Beta Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_3-release-notes) · [Xcode 27.2 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes) · [Xcode 27 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes) · [iOS & iPadOS 27.3 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27_3-release-notes) · [개발자 릴리스 목록](https://developer.apple.com/news/releases/) (2026-11-05 확인)

## 요약

**Xcode 27.3 beta(27C5020n, 2026-11-03)**의 Coding Intelligence 절에도 항목이 딱 하나 있고, 그건 **"GitHub Copilot is now available in the coding assistant"**(185837709)임. 소켓에 공급자 하나가 더 꽂힌 것이고 그 자체로는 한 줄짜리 소식임. 정작 에이전트 루프에 걸리는 변화는 **Previews 절**에 있음. **RenderPreview MCP 툴이 요청한 것과 다른 프리뷰를 렌더하던 버그**(188817121)와 **프리뷰 로딩이 끝나기 전에 "프리뷰 없음"을 보고하던 버그**(188424749)가 둘 다 27.3 에서 고쳐졌음. 이게 중요한 이유는 하나임. 에이전트가 자기 변경을 프리뷰 스냅샷으로 검증하게 해 뒀다면, **27.3 이전에는 스냅샷이 엉뚱한 프리뷰의 그림일 수 있었음.** 통과든 실패든 판정의 근거가 흔들렸다는 뜻이고, 코드 탓이 아닌 실패를 코드 탓으로 적어 둔 구간이 생김.

같은 노트에서 **Swift Testing 과 `XCUIAutomation` 으로 UI 테스트를 쓸 수 있게 됐고**(95593221), `XCUIAutomation` 에 **iPhone Duo 의 힌지 각도를 설정하는 `XCUIHinge`**(187479559)가 들어왔음. 그리고 이 저장소가 [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)에 미해결로 적어 둔 **두 손가락 터치 전송(169537162)이 해결 항목으로 올라왔음.**

한편 짚어야 할 공백이 하나 있음. **2026-11-05 기준으로 공개된 Xcode 릴리스 노트 어느 페이지에도 `GetCodeCoverage` 라는 문언이 없음.**

## Coding Intelligence 절은 여전히 한 줄

27.3 beta 의 Coding Intelligence 절은 New Features 항목 하나로 끝나고 Resolved Issues 도 Known Issues 도 없음.

> "GitHub Copilot is now available in the coding assistant." (185837709)

Xcode 27 정식 노트의 같은 절에 들어 있던 공급자 추가 항목은 둘이었음. **"Google Gemini is now available in the coding assistant."**(171990272)와 **"Google Antigravity is now available in the coding assistant."**(183074664). 거기에 26.3 이 붙인 Claude Agent SDK 와 OpenAI Codex 를 더하면 Copilot 이 다섯 번째 선택지가 됨.

여기서 새 소식은 "Copilot 이 된다"가 아니고, **공급자를 늘리는 쪽으로 계속 간다**는 것임. [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md)에 정리된 "프로토콜을 열고 구현체는 고르게 한다"가 에이전트 자리에서 또 반복됨.

그런데 공급자가 늘 때 같이 늘어나는 게 있음. Xcode 27 정식 노트의 Coding Intelligence 해결 항목에 **"Fixed: Apple-authored agent skills may not be available to Codex."**(179171480)가 있었음. Apple 제작 스킬이 특정 서드파티 에이전트에서 안 보이던 문제임. **어느 에이전트에 무엇이 노출되는지가 공급자마다 균일하지 않다**는 신호였고, 다섯 번째 공급자가 붙었다는 것은 그 비균일성을 확인할 조합이 하나 더 생겼다는 뜻임. 27.3 beta 노트는 Copilot 에 Apple 스킬이 노출되는지 **말하지 않음.**

## 진짜 변화는 Previews 절에 있음

27.3 beta 의 Previews 절은 New Features 둘, Resolved Issues 아홉으로 이 노트에서 가장 두꺼움. 그중 RenderPreview MCP 툴에 직접 걸리는 것이 넷임.

| 항목 | 내용 | 번호 |
|---|---|---|
| New | 스냅샷 **파일 이름과 메타데이터에 적용된 override 가 포함**됨 | 176531615 |
| New | macOS 프리뷰에 대해 **interactive delayed snapshot** 지원 | 181244878 |
| Fixed | 프리뷰 로딩이 끝나기 전에 **"프리뷰 없음"을 보고**할 수 있었음 | 188424749 |
| Fixed | **요청한 것과 다른 프리뷰를 렌더**할 수 있었음 | 188817121 |

마지막 줄이 이 문서를 쓰게 한 항목임.

[Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)에 정리된 대로, Xcode 의 MCP 툴 중 에이전트가 자기 변경을 눈으로 확인하는 수단은 `RenderPreview` 와 시뮬레이터 스크린샷임. 그리고 [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)의 요지는 **루프의 판정이 앵커에 걸려 있어야 한다**는 것임. 그 앵커가 스냅샷일 때 전제는 하나뿐임. **요청한 프리뷰의 그림이 돌아온다.**

188817121 은 그 전제가 간헐적으로 깨져 있었다고 적은 항목임. 라이트·다크 두 프리뷰 중 다크를 요청했는데 라이트가 돌아왔다면 에이전트는 자기가 안 고친 것을 보고 "됐다"고 하거나, 잘 고친 것을 보고 "안 된다"고 함. **둘 다 코드가 아니라 계측의 실패**인데, 루프 안에서는 구분이 안 됨. 188424749 도 방향은 같음. 로딩이 안 끝났을 뿐인데 "프리뷰 없음"이 돌아오면 에이전트는 프리뷰가 깨졌다고 판단함.

실무 판단은 이렇게 됨.

- **에이전트가 프리뷰 스냅샷으로 판정하는 루프를 27.3 이전에 돌렸다면, 그 결과는 다시 돌려야 함.** 특히 프리뷰가 여러 개인 파일과 override 를 쓰는 구성에서
- **176531615 가 그 재발을 막는 쪽임.** override 가 파일 이름과 메타데이터에 들어오면 돌아온 스냅샷이 내가 요청한 변형인지 **사람이 눈으로 대조할 수 있음.** 툴이 올바른 것을 보냈다는 보장이 아니라, 틀렸을 때 보이게 만드는 장치임. 앵커를 믿는 대신 앵커를 검산할 수단이 생긴 것
- **스냅샷 하나를 판정 근거로 쓰지 말 것.** 27.2 에서 RenderPreview 가 **render destination 목록을 돌려주고 호출자가 목적지를 지정**할 수 있게 됐음(186442676). 목적지와 override 를 명시해서 받고, 파일 이름으로 그걸 되읽는 게 지금 할 수 있는 최선임

## Swift Testing 으로 UI 테스트를 쓸 수 있게 됨

Swift Testing 절에 New Features 항목 하나가 있음.

> "Xcode now supports writing UI tests with Swift Testing and `XCUIAutomation`." (95593221)

이 저장소의 관심에서는 스냅샷 쪽과 짝이 됨. **스크린샷은 사람이 봐야 판정이 되는 산출물이고, 테스트는 에이전트가 혼자 통과·실패를 읽을 수 있는 산출물임.** 지금까지 UI 테스트는 XCTest 쪽에 남아 있어서, 에이전트에게 "유닛 테스트는 Swift Testing, UI 테스트는 XCTest"라는 두 가지 작법을 쥐여 줘야 했음. 그 분기가 없어짐.

단, 이게 왜 함정이기도 한지는 [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md)과 [Xcode 27.2 RC](./2026-10-19-xcode-27-2-rc-coverage-mcp.md)에 이미 적혀 있음. 에이전트가 쓰기 쉬운 판정 수단을 주면 **판정을 통과시키는 테스트**가 나옴. UI 테스트도 단정문 없이 화면을 띄우고 끝낼 수 있음. 작법이 하나로 합쳐진 것은 편의이고, 머지 게이트의 기준이 되는 건 여전히 "틀리면 얼마나 아픈가"임.

`XCUIAutomation` 쪽에는 항목 하나가 더 있음.

> "`XCUIAutomation` supports setting the hinge angle on iPhone Duo with a new `XCUIHinge` class." (187479559)

**검증할 축이 하나 늘었다는 뜻임.** 27.2 노트의 iPhone Duo 관련 알려진 문제들(내부·외부 디스플레이를 전환하면 Shift·Command 가 먹지 않을 수 있음 189465196, 부팅 후 몇 분간 스크린샷이 검게 나올 수 있음 187146039, Device Hub 안에서 VoiceOver 와 Accessibility Inspector 가 화면 내용을 전달하지 못함 187148389)과 같이 보면, **이 기기에서 에이전트 검증을 돌릴 때 접근성 경로와 스크린샷은 아직 믿을 구석이 아님.** 힌지 각도는 테스트 코드로 다룰 수 있게 됐고, 그 결과를 눈으로 확인하는 경로가 덜 익었음.

## Device Hub, 미해결로 적어 둔 항목이 닫힘

[Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)이 Device Hub 의 한계로 적어 둔 것이 **물리 Apple Vision Pro 화면 전송·입력 전달 미지원(142582218)과 두 손가락 터치 전송(169537162)** 둘이었음. 후자가 27.3 에서 해결 항목으로 올라왔음.

> "Device Hub now supports using option-drag to simulate two-finger touch gestures for iPhones and iPads." (169537162)

27.2 쪽에도 같은 방향의 New Features 가 있었음. Sidecar 나 Touch 지원 MacBook 에서 **iPad 화면의 터치를 원격 기기의 터치로 보내서 pinch·rotate 같은 멀티 터치 제스처를 쓸 수 있게 됨**(188634387).

다만 상한이 남아 있음. 27.3 의 알려진 문제에 **Sidecar 나 Touch 지원 Mac 으로는 터치가 두 개까지만 전달되고 셋 이상은 macOS 가 시스템 제스처로 해석**한다고 적혀 있음(185140491). 회피법으로 Touch Display 설정에서 터치 제스처를 끄는 것을 안내함.

그리고 27.3 Device Hub 의 New Features 는 하나임.

> "You can now drag items onto a device in Device Hub. Drop an app (`.app` or `.ipa`) to install it, or drop a link to open it on the device. On iOS simulators, drop any other file to save it with the Files app." (179246120)

**이건 사람이 쓰는 기능으로 적혀 있고, MCP 툴로 노출된다는 서술은 노트에 없음.** 142582218 쪽은 27.3 노트에 다시 나오지 않아서 상태를 확인하지 못했음.

## `GetCodeCoverage` 문언이 지금은 어디에도 없음

이 저장소의 [Xcode 27.2 RC](./2026-10-19-xcode-27-2-rc-coverage-mcp.md) 문서는 27.2 **RC 노트**의 Coding Assistant 절을 인용해서 `GetCodeCoverage` MCP 툴(181141715)을 정리했음. 2026-11-05 에 세 페이지를 다시 확인한 결과는 이것임.

| 페이지 | Coding Intelligence / Coding Assistant 절 | `coverage` 문자열 |
|---|---|---|
| Xcode 27 정식 | 있음 (Gemini·Antigravity 추가, Codex 스킬 해결 항목) | 없음 |
| **Xcode 27.2 정식** | **절 자체가 없음** | **없음** |
| Xcode 27.3 beta | 있음 (Copilot 추가 한 줄) | 없음 |

**Xcode 27.2 정식 노트에는 Coding Assistant 도 Coding Intelligence 도 절이 없음.** 그 페이지의 절 목록은 Device Hub, Instruments, Interface Builder, Mac Catalyst, Memory Gauge, Previews, Project Format, SDK, Simulator, XCTest UI Automation 임.

**여기서 결론을 당기면 안 됨.** 릴리스 노트에 문언이 없다는 것은 **노트에 없다**는 것뿐이고, 툴이 Xcode 에서 빠졌다는 뜻이 아님. Apple 이 RC 페이지를 정식으로 갱신할 때 항목을 정리하는 일은 흔함. 그리고 **RC 당시 페이지를 다시 열 수 없어서** 그쪽 문언과 지금을 직접 대조하지 못했음. 지금 말할 수 있는 것은 하나임. **`GetCodeCoverage` 를 쓰려는 사람이 릴리스 노트에서 근거를 찾을 수는 없는 상태이고, 실제 가용 여부는 Xcode 에서 MCP 툴 목록을 직접 띄워 확인해야 함.**

27.2 RC 문서가 유효기간에 재확인 항목으로 적어 둔 **"`GetCodeCoverage` 의 파라미터와 반환 형식이 문서화됐는지"**는 **여전히 안 됐음.** 릴리스 노트 바깥에 MCP 툴 레퍼런스가 생겼는지는 이 문서가 확인하지 못했음.

## iOS 27.3 beta 쪽에는 온디바이스 모델 항목이 없음

같은 날 나온 **iOS·iPadOS 27.3 beta(24C5037i)** 노트의 절은 App Intents, Live Caller ID Lookup, NEURLFilter, StoreKit, StoreKit Testing in Xcode, SwiftUI 임. **Foundation Models, Core AI, Apple Intelligence, Neural Engine 절은 없음.**

27.2 사이클에서도 같았음. [Xcode 27.2 RC](./2026-10-19-xcode-27-2-rc-coverage-mcp.md)가 "이번 사이클의 변화는 온디바이스 모델 쪽이 아니라 IDE 쪽에만 있음"으로 적어 둔 패턴이 **27.3 beta 에서도 반복됨.** [iOS 27.2 Foundation Models GA](../industry/2026-10-26-ios-27-2-foundation-models-ga.md) 이후 프레임워크 쪽 변화는 릴리스 노트에 올라오지 않고 있음.

## 환경 요구사항

| | 값 |
|---|---|
| 빌드 | **27C5020n** (2026-11-03) |
| Swift | **6.4** |
| 포함 SDK | iOS·iPadOS·tvOS·watchOS·macOS·visionOS **27.3** |
| 요구 macOS | **macOS Tahoe 26.6 이상** |
| 온디바이스 디버깅 | iOS 17 이상, tvOS 17 이상, watchOS 10 이상, visionOS |

요구 macOS 는 27.1·27.2 와 같은 **26.6 이상**이고, [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)이 beta 6 기준으로 적은 **26.4 이상**에서 올라온 뒤 그대로임. Swift 버전도 **6.4** 로 27.1 이후 안 바뀌었음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md) | 그 문서의 MCP 툴 표에서 `RenderPreview` 칸이 이 문서의 대상임. Device Hub 미해결 항목 중 169537162 가 닫혔고 142582218 은 확인하지 못했음. 요구 macOS 가 26.4 에서 26.6 으로 올라간 것도 그쪽 기준에서 바뀐 값 |
| [Xcode 27.2 RC, 에이전트가 커버리지를 읽게 됨](./2026-10-19-xcode-27-2-rc-coverage-mcp.md) | 같은 줄기의 다음 릴리스. 그 문서가 재확인 항목으로 남긴 셋 중 둘(정식 노트의 절 변화, 파라미터 문서화)을 이 문서가 대조함 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | 앵커가 흔들리는 루프의 구체적 사례. 188817121 은 지표를 속인 게 아니라 **계측 자체가 틀렸던** 경우라, 그쪽 7요소 중 앵커 항목에 직접 걸림 |
| [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md) | Swift Testing UI 테스트가 생겼을 때 그걸 머지 게이트로 쓸 것인가. 쓰기 쉬워진 것과 게이트가 될 자격은 별개 |
| [.xcproj JSON 프로젝트 포맷](./2026-09-16-xcproj-json-project-format.md) | 27.2 정식 노트의 Project Format 항목(184661114)이 RC 와 같은 문언을 유지함. 27.3 beta 노트에는 Project Format 절이 없음 |
| [iOS 27.2 Foundation Models GA](../industry/2026-10-26-ios-27-2-foundation-models-ga.md) | 27.3 beta 에서 온디바이스 모델 쪽 릴리스 노트 항목이 또 비었음 |

## 짚어야 할 것

- **릴리스 노트만 읽었고 Xcode 27.3 beta 를 직접 돌리지 않았음.** 이슈 번호는 Apple 이 붙인 것을 그대로 옮긴 것임
- **188817121 이 어떤 조건에서 재현됐는지는 노트에 없음.** "could render a different preview than the one requested" 한 문장이 전부이고, 빈도·영향 범위·어느 버전부터였는지는 **확인할 수 없었음.** 위의 "27.3 이전 결과를 다시 돌려라"는 이 문서의 판단이고 Apple 의 권고가 아님
- **176531615 을 "앵커 검산 수단"으로 읽은 것도 이 문서의 판단임.** 노트는 override 가 파일 이름과 메타데이터에 포함된다는 사실만 적고 용도를 말하지 않음
- **Copilot 에 Apple 제작 스킬이 노출되는지 노트에 없음.** 27 정식의 Codex 관련 해결 항목(179171480)에서 유추한 우려이고 확인된 사실이 아님
- **`XCUIHinge` 의 API 형태와 iPhone Duo 의 기기 사양은 이 문서가 확인하지 않았음.** 릴리스 노트가 클래스 이름과 "힌지 각도 설정"까지만 적음
- **`GetCodeCoverage` 가 실제로 빠졌는지는 확인할 수 없었음.** 위 표는 릴리스 노트 페이지의 문언 유무만 말함. 27.2 RC 페이지는 정식으로 갱신돼 당시 문언을 다시 열 수 없었음
- **27.3 은 beta 임.** 정식에서 항목이 빠지거나 문언이 바뀔 수 있음. Coding Intelligence 절이 한 줄로 끝난 것도 beta 단계라 그럴 수 있음
- 이 문서는 27.3 beta·27.2 정식·27 정식·iOS 27.3 beta 네 페이지를 **같은 날(2026-11-05)** 열어 대조한 것임. Apple 이 페이지를 갱신하면 표의 "없음" 칸이 먼저 낡음

## 유효기간

**2026-11-05 확인 기준, Xcode 27.3 beta(27C5020n) 시점임.** 다시 볼 때 확인할 것 셋. **27.3 정식 노트의 Coding Intelligence 절이 늘었는지**(특히 Copilot 에 Apple 스킬이 노출되는지), **`GetCodeCoverage` 문언이 어느 페이지든 다시 나타나는지 또는 MCP 툴 레퍼런스 문서가 생겼는지**, 그리고 **Device Hub 의 상태 주입 기능과 드래그 설치가 MCP 툴로 올라오는지**. 셋째 항목이 올라오면 에이전트가 앱 설치와 기기 상태를 스스로 만들게 되므로 [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)의 권한 모델 절을 같이 다시 봐야 함.
