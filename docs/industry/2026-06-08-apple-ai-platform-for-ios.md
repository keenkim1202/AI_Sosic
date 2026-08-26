---
title: Apple의 2026 AI 플랫폼, iOS 개발자가 실제로 쓰게 되는 것들
source: https://www.apple.com/newsroom/2026/06/apple-aids-app-development-with-new-intelligence-frameworks-and-advanced-tools/
author: Apple
published: 2026-06-08
collected: 2026-08-19
tags: [apple, ios, foundation-models, core-ai, mlx, on-device-llm, swift]
---

출처: [Apple aids app development with new intelligence frameworks and advanced tools](https://www.apple.com/newsroom/2026/06/apple-aids-app-development-with-new-intelligence-frameworks-and-advanced-tools/) (2026-06-08) · [WWDC26 세션 339, Bring an LLM provider to the Foundation Models framework](https://developer.apple.com/videos/play/wwdc2026/339/) · [WWDC26 세션 241, What's new in the Foundation Models framework](https://developer.apple.com/videos/play/wwdc2026/241/) · [Apple Launches Core AI (InfoQ)](https://www.infoq.com/news/2026/06/apple-core-ai-wwdc/)

## 요약

iOS 26의 Foundation Models는 **"Apple 온디바이스 3B 모델 아니면 없음"** 이었음. WWDC26에서 그 규칙이 사라짐. 어떤 LLM이든 구현할 수 있는 **공개 프로토콜 계층**이 추가되고, 프레임워크 자체도 오픈소스화됨. 같은 발표에서 Core ML의 후계인 **Core AI**, Private Cloud Compute 서버 모델 **무료 구간**, HuggingFace 모델을 그대로 로드하는 **MLXLanguageModel** 이 함께 나옴.

올해 축은 "Apple 모델을 쓴다"가 아니라 **"Apple의 Swift API 위에서 아무 모델이나 쓴다"** 로 옮겨감. iOS 앱 코드에서 모델 선택이 아키텍처 결정이 아니게 됐다는 게 이 변화의 전부임. iOS 27 기능이라 지금은 전부 베타지만, API 표면이 유지되는 구조라 iOS 26 코드가 헛수고가 되지는 않음.

## Foundation Models가 공개 프로토콜로 열림

WWDC26 세션 339의 내용. 클라우드 API, 오픈소스 로컬 모델, 자기가 호스팅하는 파인튜닝 모델이 전부 백엔드 후보가 됨. 프레임워크는 오픈소스화되고, OS 릴리스 사이에도 갱신되는 `Foundation Models framework utilities` 패키지가 별도로 나옴.

실무에서 달라지는 건 이것뿐임.

```
LanguageModelSession 코드는 그대로 두고
백엔드만  Apple 온디바이스 ↔ 서버 모델 ↔ 자체 모델  로 갈아끼움
```

기존 API 표면(`SystemLanguageModel`, `LanguageModelSession`, `Tool` 프로토콜, `@Generable` 구조화 출력)은 유지됨. iOS 26에 이미 붙여둔 코드가 있으면 백엔드 교체만 남음.

여기에 이미지 입력, 서버 모델 호출, 커스텀 스킬, 멀티에이전트 워크플로용 **Dynamic Profiles** 가 추가됨. Apple Foundation Models 자체는 Google Gemini와 공동 구축했다고 밝힘.

## Core AI, Core ML의 후계

Apple Silicon의 통합 메모리와 Neural Engine에 맞춰 재설계된 저수준 추론 프레임워크. Core ML의 공식 후계로 발표됨.

| | Core ML | Core AI | Foundation Models | MLX |
|---|---|---|---|---|
| 성격 | 전통 ML 모델 배포 | 저수준 온디바이스 추론 엔진 | Swift LLM API (프로토콜) | 오픈소스 배열 프레임워크 |
| 상태 | 유지되지만 신규 작업의 방향은 아님 | iOS/macOS 27 신규 기본값 | 백엔드 교체 가능해짐 | Foundation Models 백엔드로 편입 |

커스텀 변환한 PyTorch 모델과 사전 최적화된 오픈소스 모델을 둘 다 받음. 3B급 비전 모델부터 70B급 추론 모델까지 iPhone·iPad·Mac·Vision Pro에서 로컬 실행하는 것을 목표로 함. 서버 의존 없음, 토큰 비용 없음이 홍보 문구임.

당분간은 공존함. 다만 신규 뉴럴넷 작업을 Core ML로 시작할 이유는 없어졌으니, 기존 Core ML 파이프라인이 있으면 마이그레이션 계획을 세울 시점임.

## Private Cloud Compute 서버 모델 무료

Small Business Program 참가자, 즉 첫 다운로드 **200만 미만** 개발자는 PCC에서 도는 Apple Foundation Models를 무료로 호출할 수 있음.

온디바이스 3B로 안 되는 작업에 서버 추론을 붙이는 데 인프라 비용도 API 키도 필요 없다는 뜻임. 인디 앱이 AI 기능을 넣을 때 늘 걸리던 단가 계산이 사라짐. 같은 Swift API로 Claude, Gemini 같은 서드파티 모델도 호출 가능함.

⚠️ 다만 **개발자에게 청구되는 비용이 없다는 것이지 무제한이라는 뜻은 아님.** 공식 문서 기준으로 사용자 한 명에게 **일일 요청 한도**가 걸리고, 한도에 다가가거나 초과했을 때 앱이 직접 상태를 보여주고 iCloud+ 업그레이드 경로를 띄우게 돼 있음. 조건과 API는 [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)에 정리했음.

## MLXLanguageModel, HuggingFace 모델 드롭인

Foundation Models 프로토콜을 구현한 MLX 백엔드. `mlx-community` 의 **약 4,800개** 모델을 Foundation Models Swift API에 그대로 로드함. 파인튜닝한 자체 모델을 Apple 기본 모델 자리에 끼워 넣는 게 코드 몇 줄이 됨.

성능 쪽 참고 수치는 2차 출처 인용이라 조건 확인이 필요함.

- 14B 미만 구간에서 MLX가 대안 런타임 대비 **20~87%** 빠름
- 27B 이상에서는 메모리 대역폭이 병목이라 MLX와 llama.cpp가 수렴
- 밀집(dense) 모델은 Core AI가 MLX와 대등하거나 더 나음. MLX의 확실한 우위는 MoE 전문가 디스패치 쪽

## 지금 쓸 수 있나

| 항목 | 상태 |
|---|---|
| iOS 26 Foundation Models (온디바이스 3B) | **출시됨** |
| Foundation Models 프로토콜 개방 | 베타 (iOS 27) |
| Core AI | 베타 (iOS/macOS 27) |
| PCC 서버 모델 무료 | 베타 (iOS 27과 함께) |
| MLXLanguageModel | 베타 |

지금 할 것은 셋임.

1. iOS 26 Foundation Models를 아직 안 붙였으면 지금 붙임. API 표면이 유지되니 버려지는 작업이 아님
2. Core ML 파이프라인 목록을 만들어 둠. Core AI 마이그레이션 대상이 뭔지부터 알아야 함
3. 다운로드 200만 미만이면 PCC 무료 구간을 전제로 기능 범위를 다시 잡아볼 만함

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode에 에이전틱 코딩이 정식 탑재됨](../agents/2026-02-03-xcode-agentic-coding.md) | 같은 Apple 2026 발표의 도구 쪽. 백엔드를 교체 가능하게 만드는 패턴이 에이전트에도 똑같이 적용됨 |
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 여기 적은 것들을 실제 API와 제약으로 확인한 문서 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 같은 발표의 도구 쪽이 베타 6까지 와서 실제로 무엇이 되는지 |

## 짚어야 할 것

- iOS 27 기능은 전부 베타임. 가을 정식 전까지 API가 바뀔 수 있음
- MLX 성능 수치는 2차 출처 인용임. 자기 모델·자기 기기로 재측정하지 않고 인용하면 안 됨
- "70B 온디바이스"는 발표 문구지 모든 기기에서 된다는 뜻이 아님. 메모리 구간별 확인이 필요함
- WWDC26 세션 영상 두 건은 제목과 발표 내용만 확인했고 세션 본편은 보지 않았음

## 유효기간

2026-08-19 기준 스냅샷임. iOS 27 정식 출시(2026 가을) 시점에 API 표면과 무료 구간 조건을 재확인해야 함.
