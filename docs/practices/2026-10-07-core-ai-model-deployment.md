---
title: Core AI 로 직접 만든 모델 올리기, 비용은 추론이 아니라 specialization 과 캐시에 있음
source: https://developer.apple.com/documentation/coreai
author: Apple
collected: 2026-10-07
tags: [apple, ios, core-ai, on-device-llm, swift, xcode, migration]
---

출처: [Core AI](https://developer.apple.com/documentation/coreai) · [Integrating on-device AI models in your app with Core AI](https://developer.apple.com/documentation/coreai/integrating-on-device-ai-models-in-your-app-with-core-ai) · [Managing model specialization and caching](https://developer.apple.com/documentation/coreai/managing-model-specialization-and-caching) · [Compiling Core AI models ahead of time](https://developer.apple.com/documentation/coreai/compiling-core-ai-models-ahead-of-time) · [Inspecting, debugging, and profiling Core AI models](https://developer.apple.com/documentation/coreai/inspecting-debugging-and-profiling-core-ai-models) · 2026-10-07 확인

## 요약

이 저장소는 Core AI 를 6월 발표 요약과 정식 출시 노트로만 두 번 다뤘음. 이번은 **프레임워크 문서 본문**을 읽고 실제 API 표면을 확인한 것임. 가용성은 **iOS·iPadOS·macOS·tvOS·visionOS·watchOS 27.0+** 로 전 플랫폼 동시임. 핵심은 하나임. **`.aimodel` 파일은 기기에 맞춰 컴파일되기 전에는 못 돎**(Apple 용어로 specialization). 그래서 Core AI 를 쓰는 앱의 어려운 부분은 추론 호출이 아니라 **첫 로딩을 언제 치를지, 그 결과물을 OS 가 언제 지워가는지**를 다루는 쪽임. 문서가 그 비용을 숨기지 않고 **캐시 조회·사전 specialization·빌드 시점 컴파일·북마크 복구**를 각각 API 로 내놓았음. 그리고 **Core ML 은 폐기되지 않았음**. 이 저장소가 발표 기준으로 적어둔 "공식 후계"와 출시된 문서의 문언이 갈림.

## 프레임워크 표면

| 타입 | 역할 |
|---|---|
| `AIModelAsset` | 특수화되지 않은 원본 모델 애셋 |
| `AIModel` | 특정 기기용으로 특수화돼 추론할 수 있는 모델 |
| `InferenceFunction` | 입력을 받아 출력을 내는 함수. 모델 하나에 여럿 들어갈 수 있음 |
| `InferenceFunctionDescriptor` | 그 함수의 시그니처. 런타임에 이름·타입·shape 를 조회함 |
| `NDArray` / `NDArrayDescriptor` | 스칼라 다차원 배열과 그 shape·타입·메모리 레이아웃 기대값 |
| `ImageDescriptor` | 이미지 입출력의 크기와 픽셀 포맷 |
| `AIModelCache` | 특수화 산출물을 보관하는 캐시 |
| `SpecializationOptions` | 특수화 방식 설정. 기본값, `.cpuOnly`, 선호 compute unit |
| `ComputeUnitKind` | CPU·GPU·Neural Engine. `availableKinds` 로 기기별 가용성 확인 |
| `ComputeStream` | 비동기로 돌릴 작업 스트림 |

입력을 믿지 말고 descriptor 로 검증하라는 게 문서의 기본 자세임. `inputDescriptor(of:)` 로 받아서 `shape` 와 `scalarType` 을 확인하고, 이미지면 `width`·`height`·`pixelFormatType` 을 확인함. **`width` 나 `height` 가 `-1` 이면 동적 차원**이라는 규약이 있음. 모델을 교체 배포하는 앱이면 이 검증이 선택이 아님. 모델 버전이 바뀌면서 shape 가 달라지는 것을 코드가 먼저 알아채야 함.

## 프로젝트 설정에서 걸리는 것 둘

- `.aimodel` 파일을 드래그해 넣고 **Compile Sources 빌드 단계에 들어갔는지** 확인해야 함
- **Metal Toolchain 이 기본 설치가 아님.** Xcode > Settings > Components > Other Components 에서 받거나 `xcodebuild -downloadComponent MetalToolchain` 으로 받음. 없으면 빌드가 Metal 컴파일러 없음 에러로 깨짐

둘째가 CI 에 바로 걸림. 러너 이미지에 Xcode 만 깔아두고 Core AI 모델이 든 타깃을 빌드하면 로컬에서는 되고 CI 에서만 깨지는 형태로 나타남.

## specialization, 로딩이 비동기인 이유

`AIModel(contentsOf:)` 가 `async` 인 것은 네트워크 때문이 아님. 그 자리에서 **기기 하드웨어와 OS 버전에 묶인 실행 코드를 만들기** 때문임. 문서 표현으로 "모델 크기에 따라 상당한 시간이 걸릴 수 있음". 기본 동작은 한 번 만들고 캐시해서 다음부터 재사용하는 것임.

문서가 주는 통제 수단은 셋임.

1. **캐시 조회**: `AIModelCache.default` 에 `model(for:options:)` 를 물어봄. 특수화를 하지 않고 캐시 존재만 확인함. `nil` 이면 그때 사용자에게 "AI 기능 준비 중" 을 띄우고 특수화를 시작하라는 게 문서 예시 코드의 순서임
2. **사전 specialization**: `AIModel.specialize(contentsOf:options:cache:cachePolicy:)` 로 추론용 로드 없이 특수화만 미리 해 둠. 모델을 내려받은 직후나 기능을 켠 시점이 적기임
3. **빌드 시점 컴파일**: 아래 `coreai-build` 절

문서가 2와 3을 명확히 갈라 둠. **`specialize` 는 작업량을 줄이는 게 아니라 시점만 옮기는 것**이고, 빌드 시점 컴파일이 실제로 기기에서 할 일을 줄이는 쪽임. 이 구분을 놓치면 "미리 해두면 빨라진다" 로 뭉개게 됨.

동적 shape 모델에는 함정이 하나 더 있음. 기본 동작이 **새 입력 shape 마다 그 함수를 최적화**하는 것이라, 시퀀스가 토큰 하나씩 자라는 LLM 처럼 shape 가 계속 바뀌는 쪽에서는 최적화가 버는 시간보다 더 든다고 문서가 직접 적음. 그 경우 `SpecializationOptions.expectFrequentReshapes` 를 `true` 로 두어 **범용 동적 버전**을 쓰게 함.

compute unit 은 기본값을 건드리지 말라는 쪽임. 다만 **백그라운드에서 작은 모델을 돌리는 앱은 `.cpuOnly`** 를 써서 포그라운드 GPU 작업과 경합하지 않게 하라는 구체적 권고가 있음.

## 캐시가 사라지는 세 가지 조건

여기가 설계에 제일 크게 걸림. 특수화 산출물은 **시스템이 지워갈 수 있는 것**임.

| 조건 | 정책으로 막을 수 있나 |
|---|---|
| **OS 업데이트** | 못 막음. 산출물이 OS 버전에 묶여 있어 정책과 무관하게 항상 무효화됨 |
| 원본 `.aimodel` 수정·삭제 | 기본 정책에서는 무효화됨. `.persistent` 로 유지 |
| 저장공간 압박 | 기본 정책에서는 purgeable 로 잡혀 회수 대상. `.persistent` 로 유지 |

**OS 업데이트가 정책으로 못 막히는 줄**이라는 게 핵심임. 즉 사용자가 iOS 를 올리면 앱의 다음 첫 실행에서 그 긴 로딩이 한 번 더 돌아옴. 로딩 UI 를 "최초 설치 1회" 로 설계하면 틀림. **OS 업데이트 직후마다 다시 나타나는 상태**로 잡아야 함.

`.persistent` 에는 플랫폼 예외가 있음. **tvOS 에서는 `.persistent` 를 쓸 수 없음.** 저장공간이 빠듯해 `AIModelCache` 항목이 반드시 purgeable 이어야 한다는 이유를 문서가 밝혀 뒀음.

정리용 API 는 세 층위임. `deleteEntries(for:)` 는 특정 모델의 모든 옵션 조합, `deleteEntry(for:options:)` 는 한 조합, `deleteAll()` 은 전체임. 아직 쓰고 있는 `AIModel` 인스턴스가 있으면 **해제될 때까지 삭제를 미룬다**고 적혀 있어서, 모델 교체 코드가 인스턴스를 붙들고 있으면 조용히 디스크가 두 배로 잡힘.

앱 그룹을 쓰면 `AIModelCache(appGroup:)` 로 **앱·익스텐션 사이에 특수화 결과를 공유**할 수 있음. 같은 모델을 쓰는 앱이 둘 이상이면 중복 특수화를 피하는 유일한 경로임.

## 원본을 지우고도 쓰는 법

캐시 조회의 키가 **원본 `.aimodel` URL + `SpecializationOptions`** 조합임. 그래서 용량을 아끼려고 원본을 지우면 `AIModel(contentsOf:)` 도 `AIModelCache.model(for:options:)` 도 못 찾음. 문서가 내놓은 우회가 북마크임.

- 특수화 후 `model.bookmarkData` 를 받아 `UserDefaults` 같은 데 저장함
- 다음 실행에서 `AIModel(resolvingBookmark:)` 로 **원본을 거치지 않고** 캐시에서 바로 로드함
- 그다음에 원본 `.aimodel` 을 지워 용량을 회수함

단 북마크가 애셋의 생존을 보장하지 않음. 시스템이 회수했거나 OS 업데이트로 무효화되면 북마크 해결이 실패하고, 그때는 **모델을 다시 받아서 다시 특수화**해야 함. 즉 북마크 경로를 쓰기로 하면 재다운로드 경로를 같이 들고 있어야 하고, 원본을 지워 번 용량을 네트워크 비용으로 치르는 선택임.

## 빌드 시점으로 밀기, coreai-build

무거운 쪽을 Mac 으로 옮기는 경로임. `xcrun coreai-build compile MyModel.aimodel --platform iOS --min-deployment-version 27.0 --output compiled/` 로 `.aimodel` 을 **아키텍처별 `.aimodelc`** 로 뽑음. 파일 이름이 `MyModel.<arch>.aimodelc` 고 `<arch>` 는 런타임에 `AIModel.deviceArchitectureName` 으로 읽는 값과 같음.

- 로딩 코드는 안 바뀜. `.aimodelc` 도 같은 `AIModel(contentsOf:options:)` 로 로드함
- 각 `.aimodelc` 는 `--min-deployment-version` 이상의 모든 OS 버전에서 돎
- 기기는 자기 아키텍처 변종 하나만 쓰므로, 문서 권고는 **원격 호스팅 후 해당 변종만 내려받기**임. Background Assets 프레임워크를 같이 지목함
- **빌드 시점 컴파일로도 기기 특수화가 완전히 사라지지 않음.** 남는 양은 모델과 compute unit 에 따라 다르다고만 적혀 있음

제약이 둘 있고 둘 다 범위를 크게 깎음.

- **Apple Intelligence 지원 기기용으로만 컴파일함.** A17 Pro 이상 iPhone·iPad, M1 이상 Mac, M2 이상 Vision Pro
- **tvOS·watchOS 미지원**

즉 저사양 기기 대응이나 크로스 플랫폼 유지를 이유로 Core AI 를 고른 앱은 빌드 시점 컴파일의 이득을 **그 기기들에서는 못 받음**. 이게 역설적인 지점임. 이 저장소의 [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md)이 적어둔 Core AI 채택 이유 중 하나가 "Apple Intelligence 미지원 기기 대응" 인데, 로딩 최적화 경로가 정확히 그 기기들을 비켜감.

## 도구가 세 개 있고 역할이 갈림

| 도구 | 어디서 | 보는 것 |
|---|---|---|
| **Core AI Debugger** | Apple Developer 사이트에서 받는 **독립 macOS 앱** | `.aimodel` 자체. 연산 그래프, 각 연산을 만든 소스, 기기나 Mac 에서 실행, 기준 실행과 출력 비교 |
| **Core AI debug gauge** | Xcode 내장 | 디버그 세션 중 모델 로드·특수화·추론 이벤트를 실시간으로 |
| **Core AI instrument** | Instruments 템플릿 | CPU·GPU·Neural Engine 에 걸친 실행 타이밍 |

gauge 와 instrument 는 앱 안에서 도는 모델을, Debugger 는 파일 자체를 봄. gauge 에서 잡은 이벤트를 다른 둘로 넘길 수 있다고 적혀 있음.

Debugger 의 출력 비교 기능에는 문서가 직접 경고를 달아 뒀음. 그 비교가 말해주는 것은 **변환이나 최적화가 출력을 바꿨는지**이고, **모델이 내 과제를 잘 하는지가 아님**. 후자는 Evaluations 프레임워크로 자기 데이터에서 재야 한다고 넘김. 이 저장소가 [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md)에서 여러 번 적은 구분, 즉 지표가 재는 것과 재고 싶은 것이 다르다는 것이 Apple 문서에도 같은 모양으로 들어 있음.

gauge 로 확인하라는 항목 중 "특수화가 기대한 시점에 일어나는지" 와 "앱이 모델을 얼마나 자주 다시 로드하는지" 둘이 실무에서 제일 값어치 있음. 위 캐시 무효화 조건과 바로 짝임.

## Core ML 은 폐기되지 않았음

이 저장소의 [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md)이 Core AI 를 "Core ML 의 공식 후계" 로 적고 "신규 뉴럴넷 작업을 Core ML 로 시작할 이유가 없어짐" 까지 갔음. 출시된 문서를 보면 문언이 그보다 좁음.

- **Core ML 프레임워크 페이지에 deprecated 표시가 없음.** 가용성도 iOS 11.0+ 그대로임
- 그 페이지가 Core AI 를 가리키는 문장은 조건부임. "If your app integrates AI models using the latest architectures and inference techniques, see Core AI"
- 다만 [Core ML updates](https://developer.apple.com/documentation/updates/coreml) 페이지에 **날짜 절이 `June 2024` 하나뿐임.** 2026-10-07 확인 기준으로 그 뒤 갱신 항목이 없음

마지막 줄이 가장 말이 많은 사실임. 폐기 선언은 없는데 **갱신이 2년 넘게 멈춰 있음**. 실무 판단으로 옮기면 이렇게 됨. 돌고 있는 Core ML 파이프라인을 급히 뜯을 근거는 문서에 없음. 반대로 새로 시작하는 작업을 Core ML 로 잡으면 갱신이 끊긴 표면에 올리는 것이고, 특히 **최신 아키텍처나 LLM 급 모델이면 Core AI 가 문서상 지정된 자리**임.

## 도입 판단

Core AI 를 고를 자리는 좁음. 문서가 스스로 그렇게 적고 있음.

- 온디바이스로 **생성·언어 과제**를 하는데 Apple 모델로 충분하면 Foundation Models 가 위층이고, 거기 머무는 게 맞음. Core AI 는 그 아래
- 직접 내보낸 모델을 **언어 모델 세션 API 에 태우는 것**이 목적이면 Core AI 를 직접 쓰지 않고 `CoreAILanguageModel` 경로가 있음. 이 저장소는 그쪽을 [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md)에 정리해 뒀음
- Core AI 를 직접 만질 이유는 **언어 모델이 아닌 모델**(비전, 오디오, 분류 등)이거나, specialization·캐시·compute unit 을 직접 통제해야 할 때임
- 모델을 **앱 업데이트와 별개로 교체 배포**할 계획이면 이 문서의 캐시·북마크 절이 설계 요구사항으로 바로 들어옴

쓰기로 했으면 1일차에 정할 것 셋임. 첫 로딩을 어느 화면에서 치를지, OS 업데이트 후 재특수화를 어떻게 안내할지, 모델을 번들할지 내려받을지. 셋 다 나중에 고치면 비싼 쪽임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md) | 6월 발표 기준의 Core AI 서술. "Core ML 공식 후계" 와 4종 비교표가 거기 있고, 위 절에서 출시 문언과의 차이를 짚었음 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 정식 릴리스 노트 쪽의 Core AI. 백그라운드 Neural Engine 엔타이틀먼트(`com.apple.developer.background-tasks.continued-processing.inference`)와 NE 메모리가 앱 프로세스에 귀속되는 변경이 그쪽에 있음 |
| [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md) | `CoreAILanguageModel` 로 오픈소스 모델을 세션에 태우는 경로. 이 문서는 그 아래층인 프레임워크 자체를 봄 |
| [AFM 3](../models/2026-06-08-apple-foundation-models-3.md) | Apple 자기 모델이 어떤 구조로 기기에 사는지. 플래시 상주와 전문가 활성이 specialization 과는 다른 층위의 최적화임 |
| [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md) | Core AI Debugger 의 출력 비교가 품질 측정이 아니라는 경고. 무엇을 재고 있는지 혼동하는 같은 실패임 |

## 짚어야 할 것

- **이 문서는 프레임워크 문서 본문 기준임.** WWDC26 세션 영상(`324 Meet Core AI`, `326 Integrate on-device AI models into your app using Core AI`)은 제목만 확인했고 보지 않았음. 세션에만 있는 수치나 권고가 있을 수 있음
- **specialization 시간의 실측값이 문서에 없음.** "모델 크기에 따라 상당한 시간" 이 전부고, 초 단위 수치는 어디에도 없었음. 자기 모델로 재는 수밖에 없음. 빌드 시점 컴파일이 줄여주는 비율도 "모델과 compute unit 에 따라 다름" 까지만 적혀 있음
- **캐시가 디스크를 얼마나 쓰는지도 문서에 수치가 없음.** 삭제 API 와 purgeable 정책만 있음. 저장공간 경고를 띄울 임계값을 문서에서 끌어올 수 없음
- `Background Inference` 엔타이틀먼트는 키와 설명(`BGContinuedProcessingTask` 와 함께 동작하고, 백그라운드 Neural Engine 접근 전반에 필요함)까지만 확인했음. **승인 절차나 심사 요건은 확인할 수 없었음**
- [Core AI 업데이트 페이지](https://developer.apple.com/documentation/updates/coreai)는 **2026-10-07 확인 시점에 열리지 않았음**(404). 그래서 27.0 이후 추가·변경 항목을 updates 경로로 대조하지 못했고, 이 문서는 프레임워크 문서의 현재 상태만 반영함
- `apple/coreai-models` 수치는 **2026-10-07 에 `gh api` 로 받은 값**임. 별 **2,176**, 포크 **222**, 열린 이슈 **42**, 라이선스 **BSD-3-Clause**, 저장소 생성 **2026-06-08**, 마지막 푸시 **2026-10-06**. 모델 레지스트리의 내용은 이 문서에서 확인하지 않았음
- Core ML 에 대한 위 판단은 **문서 표면 관찰**임. Apple 이 폐기를 선언한 바 없고, 갱신 공백이 폐기 예고라는 근거도 없음. 그 둘을 섞어 읽지 말 것

## 유효기간

**2026-10-07 확인 기준이고 iOS 27.0 정식 표면임.** 타입 이름과 캐시 정책은 27.x 사이에 바뀔 가능성이 낮지만, 확인할 것 셋이 남음. **Core AI updates 페이지가 열리게 됐는지**(열리면 27.1·27.2 변경점을 그쪽으로 한 번에 대조할 수 있음), **빌드 시점 컴파일의 Apple Intelligence 기기 제한이 풀렸는지**, 그리고 **Core ML updates 에 `June 2024` 이후 절이 붙었는지**. 마지막 것은 Core ML 의 위치를 다시 읽어야 한다는 신호임.
