---
title: Foundation Models 실전, 컨텍스트 4K에서 32K로 넘어가는 지점
source: https://developer.apple.com/documentation/foundationmodels
author: Apple
collected: 2026-08-21
tags: [apple, ios, foundation-models, on-device-llm, core-ai, swift, context-window]
---

출처: [Foundation Models 프레임워크 문서](https://developer.apple.com/documentation/foundationmodels) · [Adding server-side intelligence with Private Cloud Compute](https://developer.apple.com/documentation/foundationmodels/adding-server-side-intelligence-with-private-cloud-compute) · [Managing the context window](https://developer.apple.com/documentation/foundationmodels/managing-the-context-window) · [Running a Core AI model in a Foundation Models session](https://developer.apple.com/documentation/foundationmodels/running-a-core-ai-model-in-a-foundation-models-session) · [Introducing the Third Generation of Apple's Foundation Models](https://machinelearning.apple.com/research/introducing-third-generation-of-apple-foundation-models) (2026-06-08) (전부 베타 문서, 2026-08-21 확인)

## 요약

[Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md)이 발표 내용이라면 이 문서는 실제로 코드를 짤 때 부딪히는 것들임. 설계의 첫 분기점은 모델 성능이 아니라 **컨텍스트 윈도우**임. 온디바이스는 **4,096토큰**, Private Cloud Compute는 **32K**. 그런데 **한국어는 대략 한 글자가 한 토큰**이라 영어 기준 감각으로 잡으면 훨씬 빨리 참. `LanguageModel` 프로토콜 덕에 온디바이스 / PCC / 직접 내보낸 오픈소스 모델을 **세션 생성 한 줄만 바꿔서** 갈아끼울 수 있고, 그래서 결정을 나중으로 미룰 수 있음.

## 모델 라인업 (AFM 3, 2026-06-08)

Apple ML 연구 블로그 기준 3세대 구성임.

| 모델 | 위치 | 구조 |
|---|---|---|
| **AFM 3 Core** | 온디바이스 | **3B 밀집(dense)** 모델의 차세대 |
| **AFM 3 Core Advanced** | 온디바이스 | **20B 희소 구조인데 요청에 따라 1~4B만 활성.** 가중치를 DRAM이 아니라 **플래시에 두고** Instruction-Following Pruning(IFP)으로 상시 활성 shared expert + 입력 의존 routed expert를 씀 |
| **AFM 3 Cloud** | 서버 | 속도·효율 중심 주력 |
| **AFM 3 Cloud Pro** | 서버 | 가장 강력한 서버 모델. Google Cloud의 NVIDIA GPU에 최적화 |
| **ADM 3 Cloud (Image)** | 서버 | 이미지 생성·편집 전용 |

⚠️ 흔히 **"20B 희소, 1~4B 활성"을 서버 모델로 잘못 인용**하는데, 원문에서 그건 **온디바이스 Core Advanced**임. 플래시 저장이라는 설명이 붙는 것도 그래서임.

품질은 자체 사이드바이사이드 선호도로 제시됨. Core가 2025 기준선 대비 **45.6% 대 23.3%**, Cloud가 **64.7% 대 8.7%**, 전체 응답 만족도 **36% 상대 개선**. 자체 평가라 그대로 인용하면 안 되고 방향만 볼 것.

## 컨텍스트, 설계의 첫 분기점

| | `SystemLanguageModel` (온디바이스) | `PrivateCloudComputeLanguageModel` |
|---|---|---|
| 컨텍스트 | **4K** | **32K** |
| 오프라인 | 됨 | 안 됨 |
| 사용량 제한 | 무제한 | **1인당 일일 한도** |
| 추론(reasoning) | 미지원 | **light / moderate / deep** |
| 프라이버시 | 보존 | 보존 |

**컨텍스트 윈도우는 대화 히스토리만 먹는 게 아님.** 문서가 명시하는 소비 항목은 프롬프트, instructions, **툴 정의와 그 입출력**, **Generable 타입 스키마**, 그리고 모델 응답 전부임. 툴을 늘리면 대화를 시작하기도 전에 창이 줄어듦.

**한국어 앱이면 이 숫자를 다시 계산해야 함.** 문서가 직접 적음. 라틴 알파벳은 토큰 하나가 보통 **3~4자**인데, **중국어·일본어·한국어·베트남어는 토큰 하나가 보통 한 글자**임. 같은 4,096토큰이 영어에서 대략 1만 4천 자를 담는다면 한국어에서는 4천 자 남짓임. 영어권 샘플 코드가 넉넉해 보인다고 그대로 옮기면 실기기에서 터짐.

## 토큰을 관리하는 도구

프레임워크가 주는 계측 수단이 실제로 있음.

- `tokenCount(for:)`로 프롬프트·instructions·툴이 각각 몇 토큰인지 확인
- `contextSize`로 모델이 지원하는 최대 크기 조회
- 넘치면 `LanguageModelError.contextSizeExceeded(_:)`가 던져지고 **세션은 그 뒤로 응답을 못 함**
- Xcode Instruments에 **Foundation Models 템플릿**이 있음. Product > Profile에서 세션 요청·프롬프트·응답·툴 호출별 토큰 사용을 실시간으로 봄

문서가 권하는 절약 방법 중 실제로 효과가 큰 것들.

- **프롬프트는 3문단 이내.** 명령형 동사로 시작하고 배경 설명과 정책 문장을 빼기
- **Generable 타입을 단순하게.** 타입 구조가 JSON 스키마로 변환돼 모델에 전달되고, `@Guide` 설명도 스키마의 일부라 토큰을 먹음. 프로퍼티 이름이 명확하면 `@Guide` 없이 먼저 테스트하고 필요한 곳에만 붙이는 순서
- **툴은 요청당 3~5개까지.** 툴 정의(이름·설명·파라미터)가 전부 모델에 전달됨. 모델이 판단할 필요가 없는 정보는 툴로 만들지 말고 직접 조회해서 프롬프트에 넣기
- 배열 길이는 `@Guide(.maximumCount(_:))`로 제한. `maximumResponseTokens`는 **문장이 잘려 나올 수 있으니 최후 수단**으로만

## 창이 찼을 때

새 세션을 만들되 맥락을 다 버리지 않는 패턴을 문서가 직접 제시함. **트랜스크립트의 첫 엔트리와 마지막 엔트리만 남겨 재시드**하는 것.

```swift
func newContextualSession(with originalSession: LanguageModelSession) -> LanguageModelSession {
    let allEntries = originalSession.transcript
    let condensedEntries = [allEntries.first, allEntries.last].compactMap { $0 }
    let condensedTranscript = Transcript(entries: condensedEntries)
    let newSession = LanguageModelSession(transcript: condensedTranscript)
    newSession.prewarm()
    return newSession
}
```

근거가 단순함. **첫 엔트리에 instructions가 들어 있고 마지막 엔트리가 가장 최근 맥락**이라, 둘만 남겨도 연속성이 유지되면서 토큰이 크게 줄어듦. 긴 문서 요약처럼 애초에 창을 넘는 작업은 청크별로 세션을 나누고 **직전 청크의 요약을 다음 프롬프트에 넣어** 연속성을 유지하는 방식을 씀.

이 구조가 [Claude Cowork](./2026-08-10-claude-cowork.md)에서 다룬 "대화를 이어붙일까 재시작할까" 문제와 정확히 같은 것임. 다만 결론은 반대 방향으로 갈림. 그쪽은 **캐시가 살아 있는지**가 판단 기준이었는데, 여기는 **하드 리밋이 있어서** 선택의 문제가 아니라 반드시 잘라야 하는 문제임.

## PCC로 넘어가기

바꾸는 건 세션 생성 한 줄임. `PrivateCloudComputeLanguageModel`과 `SystemLanguageModel`이 둘 다 `LanguageModel` 프로토콜을 따르므로 `respond` 계열 메서드, 툴, instructions가 수정 없이 그대로 넘어감.

```swift
let session = LanguageModelSession(model: PrivateCloudComputeLanguageModel())
```

붙는 조건이 셋 있음.

- **`iOS 27` 이상.** 그 아래에서는 `#available` 분기로 온디바이스 폴백을 둬야 함
- **네트워크 필요.** 실패하면 온디바이스로 재시도하는 경로를 두라고 문서가 권함
- **개발자 자격 요건과 managed entitlement 승인이 필요함** (`com.apple.developer.private-cloud-compute`). "Accessing Private Cloud Compute" 절차를 거쳐야 함

그리고 **일일 사용 한도가 앱이 다뤄야 하는 UI 문제로 들어옴.** `model.quotaUsage`로 한도 도달·근접 상태를 읽고, `limitIncreaseSuggestion`으로 iCloud+ 업그레이드 옵션을 띄우게 돼 있음. 한도 초과 시 `quotaLimitReached(_:)` 에러가 던져지고 `resetDate`로 갱신 시점을 확인함. Xcode Scheme의 Run > Options에서 **"Approaching Quota Usage Limit" / "Quota Usage Limit Reached"를 시뮬레이션**할 수 있으니 테스트 경로가 막혀 있지는 않음.

⚠️ 이 저장소의 [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md)은 Small Business Program 참가자에게 PCC가 **무료**라고 정리했는데, 공식 문서 쪽 표현은 **개발자 자격 심사 + 사용자 1인당 일일 한도 + iCloud+ 업그레이드**임. 둘이 모순은 아니지만 **"무료"를 무제한으로 읽으면 안 됨.** 비용이 아니라 한도가 제약임.

추론 레벨은 PCC에서만 쓸 수 있고 `ContextOptions(reasoningLevel:)`로 지정함. 문서 권고는 **moderate에서 시작해서 필요할 때만 deep**. 추론 텍스트도 컨텍스트 창을 먹고 최종 응답에는 안 나타나지만, 트랜스크립트에서 읽을 수 있어 프롬프트 디버깅에 쓸 수 있음.

## Core AI로 오픈소스 모델 넣기

Apple 모델을 안 쓰고 직접 내보낸 모델을 같은 세션 API에 태우는 경로임. Apple Intelligence 미지원 기기 대응이나 크로스 플랫폼 유지가 목적일 때 씀.

- 오픈소스 Swift 패키지 **`apple/coreai-models`**가 내보내기 레시피와 `CoreAILanguageModel`을 제공함. 이 타입이 `LanguageModel`을 따르므로 세션 생성부만 달라짐
- 내보내기는 터미널에서 `uv run coreai.model.registry --list-models`로 지원 모델을 훑고 `uv run coreai.llm.export HF_ID --platform iOS`로 뽑음. 결과는 `.aimodel` 파일과 토크나이저가 든 리소스 폴더고 앱 번들에 넣음
- **모델은 대상 플랫폼별로 따로 내보내야 함.** macOS용과 iOS용이 다름
- 문서 권고는 **첫 시도로 0.6B급**. 빨리 받고 기기에서 무난히 돎
- 로딩이 비동기임. 프레임워크가 모델을 컴파일하고 토크나이저를 올린 뒤 첫 요청이 가능하므로 **1~2초 여유가 있을 때 미리 로드**할 것
- 추론 모델이면 chain-of-thought가 `Transcript.Entry.reasoning(_:)` 세그먼트로 분리돼 사용자에게 안 보임. 지원 여부는 `model.capabilities.contains(.reasoning)`으로 확인
- 실행 엔진(GPU / CPU / Neural Engine)은 Core AI가 자동 선택함
- **iOS 27, macOS 27, Xcode 27 이상 필요**

## 그 밖에 알아둘 API 표면

프레임워크 문서 목차 기준으로, 실제로 설계에 영향을 주는 것들.

- **Dynamic profiles**: `DynamicProfile`, `DynamicInstructions`, `Profile`, `SessionProperty`. 앱 상태에 따라 instructions와 툴을 **런타임에 갈아끼우는** 구조. 문서가 이걸로 "에이전트나 스킬 같은 추상화"를 만들 수 있다고 적음
- **멀티모달 프롬프트**: `Attachment`, `ImageAttachmentContent`, `ImageReference`로 이미지를 프롬프트에 붙임
- **평가**: `Evaluations` 프레임워크로 기능을 먼저 측정하고, 그 결과로 온디바이스에 머물지 PCC로 갈지 정하라는 게 문서의 명시적 순서임
- **KV 캐시**: "Optimizing key-value caching in language model sessions" 문서가 따로 있음. 턴 간 캐시 상태를 보존해 토큰 재처리를 막는 쪽
- **가용성**: 프레임워크 자체는 iOS 26.0+, watchOS 27.0+. PCC·Core AI·dynamic profiles·attachments는 전부 **Beta** 표시

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md) | 같은 주제의 발표 요약. 이 문서는 그 위에서 실제 API와 제약을 확인한 것 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md) | 온디바이스 / PCC / 서드파티 경계가 그쪽에서는 동의 요건 문제로 나타남 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | KV 캐시 보존과 트랜스크립트 재작성 비용이 같은 문제임 |
| [Claude Cowork](./2026-08-10-claude-cowork.md) | 대화를 이어붙일지 재시작할지. 여기는 하드 리밋이라 선택지가 아님 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | 프롬프트에서 뺄 것과 넣을 것. 4K 창에서는 그 절약이 훨씬 크게 효과를 냄 |

## 짚어야 할 것

- **전부 베타 문서임.** iOS 27 정식은 2026 가을 예정이고 API 표면이 바뀔 수 있음
- AFM 3 품질 수치는 **Apple 자체 사이드바이사이드 평가**임. 외부 벤치마크가 아님
- "Bring an LLM provider to the Foundation Models framework" 문서는 프레임워크 목차에서 제목만 확인했고 **본문을 읽지 못했음.** 서드파티 프로바이더 구현의 세부는 `LanguageModel`, `LanguageModelExecutor`, `LanguageModelExecutorGenerationRequest` 타입 이름까지만 확인한 상태임
- 온디바이스 파인튜닝(어댑터)은 이번 문서 목차에서 확인되지 않았음. 2025년 버전에 있던 기능이라 27에서 어떻게 되는지는 **확인 못 함**
- 4K·32K는 문서에 적힌 값이고, 모델 버전이 올라가면 바뀔 수 있음. `contextSize`로 런타임에 읽는 편이 안전함

## 유효기간

**2026-08-21 스냅샷**임. 베타 API라 타입 이름과 수치가 정식 출시 때 달라질 수 있음. 다시 볼 때는 [Foundation Models updates](https://developer.apple.com/documentation/foundationmodels) 문서로 변경점을 먼저 확인할 것.
