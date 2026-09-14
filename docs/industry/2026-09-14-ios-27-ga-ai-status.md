---
title: iOS 27 정식 출시 시점 재확인, 베타 딱지가 떨어진 것과 안 떨어진 것
source:
  - https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes
  - https://developer.apple.com/news/?id=k1mtkt1k
  - https://developer.apple.com/documentation/foundationmodels
author: Apple
collected: 2026-09-14
tags: [apple, ios, foundation-models, core-ai, xcode, coding-agent, app-store]
---

출처: [Xcode 27 RC Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes) · [App Store submissions now open for the latest OS releases](https://developer.apple.com/news/?id=k1mtkt1k) (2026-09-09) · [Releases](https://developer.apple.com/news/releases/) · [Foundation Models 문서](https://developer.apple.com/documentation/foundationmodels) · [Core AI 문서](https://developer.apple.com/documentation/coreai) · [apple.com/os/ios](https://www.apple.com/os/ios/) (전부 2026-09-14 확인)

## 요약

이 저장소의 Apple 문서 셋은 전부 베타 기준으로 적혔고, 각자 `## 유효기간`에 "정식 출시 시점에 다시 대조하라"고 적어뒀음. 오늘이 그 시점임. **iOS 27이 2026-09-14 출시**됐고 Xcode 27 RC는 **2026-09-09**에 나왔음. 대조 결과 셋으로 갈림. 첫째, **Foundation Models의 PCC·dynamic profiles·attachments와 Core AI가 문서상 베타에서 벗어났음.** 둘째, Xcode 쪽은 **베타 6 문서가 알려진 버그로 적어둔 계획 확인 바 문제가 고쳐졌고 요구 macOS가 26.4에서 26.6으로 올라갔음.** 셋째, **워크스페이스 없이 도는 MCP 서버는 RC에서도 여전히 early preview**라 승격되지 않았음. 새 내용을 소개하는 문서가 아니라 **기존 문서의 베타 전제를 갱신하는 대조 기록**임.

## 무엇이 정식이 됐나

Apple 개발자 문서의 DocC 메타데이터에서 `beta` 플래그를 직접 확인한 결과임. 심볼 페이지의 availability 항목을 본 것이고, 전부 `false`로 돌아섰음.

| 대상 | 도입 버전 | `beta` 플래그 |
|---|---|---|
| Foundation Models 프레임워크 | iOS 26.0, watchOS 27.0 | `false` |
| `PrivateCloudComputeLanguageModel` | iOS·iPadOS·macOS·tvOS 계열 전부 27.0 | `false` |
| Core AI 프레임워크 | 전 플랫폼 27.0 | `false` |

[Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)이 "PCC·Core AI·dynamic profiles·attachments는 전부 **Beta** 표시"라고 적어둔 부분이 여기서 갱신됨. 그 문서의 API 표면 설명과 수치(온디바이스 4K, PCC 32K, 툴 3~5개, 한국어 한 글자당 한 토큰)는 그대로 유효하고, **달라진 건 베타 표기뿐임.** 설계를 미뤄둘 이유가 하나 사라졌다는 뜻으로 읽으면 됨.

Core AI 문서가 프레임워크를 이렇게 설명함.

> "Core AI helps you build, run, and deploy AI models in your app, designed with Apple silicon in mind to leverage the CPU, GPU, and Neural Engine."

[Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md)이 "Core AI는 베타(iOS/macOS 27)"로 적어둔 줄이 이제 27.0 정식임. 다만 그 문서가 인용한 **MLX 성능 수치는 여전히 2차 출처**이고 이번에 재확인하지 않았음.

## Xcode 27, 베타 6에서 RC까지

2026-09-14 기준으로 릴리스 노트 페이지는 **아직 "Xcode 27 RC"** 제목임. 정식 표기 노트는 나오지 않았음.

| 항목 | 값 |
|---|---|
| 빌드 | Xcode 27 RC (27A266a), 2026-09-09 |
| Swift | 6.4 |
| 포함 SDK | iOS·iPadOS·tvOS·watchOS·macOS·visionOS 27 |
| 요구 macOS | **macOS Tahoe 26.6 이상** |

요구 macOS가 [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)에 적힌 **26.4에서 26.6으로 올라갔음.** 빌드 머신 계획을 26.4 기준으로 잡아뒀으면 지금 고쳐야 함. 릴리스 노트 문장은 이것임.

> "Xcode 27 RC requires a Mac running macOS Tahoe 26.6 or later."

### 알려진 버그였던 것이 해결됨

베타 6 문서가 ⚠️ 표시로 적어둔 계획 확인 바 버그(178673449)가 RC에서 **Resolved 항목으로 옮겨갔음.**

> "Fixed: If the plan-mode confirmation bar ("Implement the plan?" with Yes/No buttons) appears while the agent is still streaming a response, clicking either button may trigger a new agent turn on top of the in-flight one, leaving the conversation in an inconsistent state."

즉 **"응답이 끝난 뒤 누르라"는 회피법은 더 이상 필요 없음.** 기존 문서의 그 경고는 베타 시점의 기록으로 남겨두면 됨.

### 에이전트 관련 Resolved 항목

RC의 해결 목록 중 에이전트 표면에 걸리는 것들임.

| 이슈 | 내용 |
|---|---|
| 178673449 | 계획 확인 바가 스트리밍 중 턴을 겹치게 만들던 문제 |
| 178771195 | 플러그인이 제공한 ACP 에이전트가 UI에서 제대로 제거되지 않던 문제 |
| 179171480 | Apple 제작 에이전트 스킬이 Codex에 노출되지 않던 문제 |
| 185119267 | 아티팩트 뷰가 프리뷰 스냅샷의 "Scope" 필터를 무시하던 문제 |
| 177462397 | VoiceOver가 코딩 어시스턴트 프롬프트 영역에 갇히던 문제 |

179171480은 베타 6 문서가 이미 "베타 2 해결 항목"으로 적어둔 것과 같은 건임. 다만 그 문서가 거기서 끌어낸 판단, 즉 **어느 에이전트에 무엇이 노출되는지가 균일하지 않을 수 있다**는 신호는 178771195가 같은 계열이라 여전히 유효함.

### 베타 6 문서에 없던 신규 항목

| 이슈 | 내용 |
|---|---|
| 176385678 | 상태 표시가 붙은 "New Conversation" 툴바 버튼 |
| 178470032 | 에이전트 플러그인이 `_meta`로 커스텀 아이콘과 툴 이름을 지정 |
| 185161968 | 연결된 프로젝트가 Xcode Service 메뉴 막대에 나타남 |
| 179126594 | 활성 개발자 도구 버전에서 딥링크 동작 |

전부 UI·편의 항목이고, 베타 6 문서가 정리한 **플러그인·MCP·ACP·권한 모델의 구조 자체는 RC에서 바뀌지 않았음.**

## 정식이 안 된 것

RC 릴리스 노트에서 **워크스페이스 없이 도는 MCP 서버(181836944) 항목의 문구가 그대로임.** "Xcode 27 Beta 5 adds a preview of..."로 시작하는 문장이 RC에서도 유지되고, 끝에 이 단서가 붙어 있음.

> "In this early preview, some aspects of the `xcrun mcp-server` command line utility may not work in all configurations, and some settings or permissions may occasionally require relaunching Xcode or rebooting your machine to apply."

**베타 6 문서가 "정식 출시 때 빠지거나 바뀔 수 있다"고 지목한 바로 그 기능이 승격되지 않고 preview로 남았음.** `sudo xcrun mcp-server enable --unsafe-always-allow-all-agents`와 "자리에 앉아 쓰는 용도로는 권장하지 않는 구성"이라는 단서도 그대로임. 코드 서명된 에이전트에게 디렉터리 트리 권한을 장기간 주는 모델을 지금 무인 CI에 넣을 계획이었다면, **정식 릴리스에 기대지 말고 preview 전제로 설계해야 함.**

파일시스템 접근 감시 보안 계층(178289431)의 문구도 RC에서 동일함. 여전히 **Coding Intelligence 설정에서 켜는 것**이고 기본값이 아님.

## 제출 쪽에서 새로 걸리는 것

2026-09-09자 개발자 뉴스에서 확인한 것 중 일정에 영향을 주는 항목임.

| 항목 | 내용 |
|---|---|
| 최소 SDK | **2027년 4월부터** iOS·iPadOS 앱은 iOS 27 / iPadOS 27 SDK 이상으로 빌드해야 업로드됨. tvOS·visionOS·watchOS도 각각 27 SDK 이상 |
| 아키텍처 | macOS 26이 Intel과 Rosetta를 지원하는 마지막 릴리스. macOS 27은 Apple Silicon 전용이고 빌드 아키텍처를 `arm64` 단독으로 둘 것 |
| 연령등급 문항 | 소셜 미디어 기능이 있는 앱은 App Store Connect에서 그 사실을 표시해야 함 (Time Allowances 관련) |

세 번째는 AI 조항이 아님. [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md)의 체크리스트 8번(연령등급 문항 재검토)에 항목이 하나 늘어난 것으로 읽으면 됨. **2026-09 중 AI 관련 심사 지침 개정은 확인되지 않았음.**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 이 문서가 갱신하는 지점 |
|---|---|
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | "전부 베타 문서임"이라는 전제. PCC와 attachments, dynamic profiles가 베타 표기에서 벗어남 |
| [Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md) | "지금 쓸 수 있나" 표의 베타 행들. Core AI가 27.0 정식 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 요구 macOS 26.4, 계획 확인 바 알려진 버그, 워크스페이스 없는 MCP 서버의 승격 여부 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | 연령등급 문항과 최소 SDK 마감 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 로드맵 2번이 겨냥한 로컬 stdio 서버. Xcode 쪽 진입점이 preview에 머물렀음 |

## 짚어야 할 것

- **릴리스 노트를 읽었고 Xcode 27을 직접 돌려보지 않았음.** 이슈 번호는 Apple 표기를 그대로 옮긴 것임
- **`beta` 플래그는 문서 메타데이터의 값임.** API가 앞으로 안 바뀐다는 보장이 아니라 Apple이 베타 표기를 뗐다는 사실까지만 말함
- **`CoreAILanguageModel` 심볼은 이번에 확인하지 못했음.** Foundation Models 프레임워크 목차에서 안 보였고, 기존 문서대로 오픈소스 `apple/coreai-models` 패키지 쪽에 있는 것으로 보이나 대조하지 못했음
- **Xcode 27 정식 표기 릴리스 노트는 아직 없음.** 오늘 확인한 것은 RC 노트이고, 정식 노트가 별도로 나오면 항목이 더 붙을 수 있음
- 개발자 뉴스 목록 상단에서 확인한 2026-09 항목은 **두 건**임. 목록이 더 있을 수 있으니 AI 관련 개정이 없었다는 결론은 그 범위 안에서만 유효함
- Apple Intelligence 지원 언어에 한국어가 포함되지만 **Siri AI는 영어로 먼저 나감.** apple.com 문구가 "Siri AI is rolling out in English"이고, EU의 iOS·iPadOS·watchOS에는 초기 미제공이라는 각주가 붙음. 이건 소비자 기능 쪽이라 이 저장소 범위 밖이고, **온디바이스 Foundation Models의 한국어 토큰 계산과는 무관한 얘기임**
- 원문 어디에도 읽는 쪽에 지시를 내리는 문장은 없었음

## 유효기간

**2026-09-14 확인 기준**임. 베타 표기 해제와 RC 릴리스 노트 대조는 시점이 박힌 기록이라 낡지 않음. 반면 두 가지는 계속 움직임. **Xcode 27 정식 릴리스 노트**가 나오면 위 Resolved·신규 항목을 다시 대조해야 하고, **워크스페이스 없는 MCP 서버**는 preview 딱지가 언제 떨어지는지 27.x 노트에서 따라가야 함. 최소 SDK 마감은 **2027년 4월**이라 그 전에 한 번 더 볼 일이 있음.
