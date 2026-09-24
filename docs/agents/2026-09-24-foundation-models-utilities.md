---
title: Foundation Models framework utilities, Apple 이 직접 낸 컨텍스트 관리·스킬 패키지
source: https://github.com/apple/foundation-models-utilities
author: Apple
collected: 2026-09-24
tags: [apple, foundation-models, on-device-llm, skills, context-engineering, kv-cache, swift]
---

출처: [apple/foundation-models-utilities](https://github.com/apple/foundation-models-utilities) (README·`Package.swift`·`Sources/`·`skills/` 직접 확인, 2026-09-24) · [Foundation Models framework](https://developer.apple.com/documentation/FoundationModels)

## 요약

Apple 이 **Apache-2.0** 으로 낸 Swift 패키지임. Foundation Models 프레임워크 자체가 아니라 그 위에 붙는 유틸리티 셋이고, 들어 있는 것은 셋임. **`/chat/completions` 를 쓰는 아무 서버나 `LanguageModelSession` 에 꽂는 `ChatCompletionsLanguageModel`**, 트랜스크립트가 컨텍스트 창을 넘지 않게 깎는 **History 모디파이어 3종**, 그리고 모델이 툴 호출로 직접 켜고 끄는 **`Skills`** 임. 이 저장소가 [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)에서 정리한 온디바이스 **4,096토큰** 문제에 Apple 이 내놓은 공식 대응이 이 패키지라고 읽으면 됨. 값어치가 제일 큰 부분은 **스킬을 prompt 로 넣을 때와 instructions 로 넣을 때 KV 캐시 영향이 갈린다는 것을 Apple 이 문서에 표로 박아 둔 것**임. 다만 **정식 태그가 아직 없고 최신이 `1.1.0-beta1`(2026-09-21)** 인데 README 의 설치 예제는 `from: "1.0.0"` 이라 그대로 붙이면 해석되지 않음. 패키지가 같이 배포하는 `skills/SKILL.md` 가 README 와 어긋나는 지점도 둘 있음.

## 기본 정보

`gh api repos/apple/foundation-models-utilities` 로 **2026-09-24** 에 받은 값임.

| 항목 | 값 |
|---|---|
| 라이선스 | **Apache-2.0** |
| stars / forks | **507** / 33 |
| open issues | 0 |
| 저장소 생성 | 2026-06-08 |
| 마지막 푸시 | **2026-09-21** |
| 최신 태그 | **`1.1.0-beta1`** (2026-09-21). 그 앞은 `1.0.0-beta5`(2026-08-12), `1.0.0-beta1`(2026-06-08) |
| 정식 릴리스 | **없음.** GitHub Releases 가 비어 있고 태그는 전부 프리릴리스 |
| 플랫폼 요건 | `Package.swift` 기준 **macOS / iOS / visionOS / watchOS 27.0** 이상. **tvOS 는 목록에 없음** |
| 툴체인 | swift-tools-version **6.2**, `swiftLanguageModes: [.v6]` |
| 이슈 창구 | GitHub Issues 가 아니라 **Apple Developer Forums** 로 안내함 |

저장소 생성일이 **2026-06-08**, 즉 WWDC26 발표일과 같은 날이고 `1.0.0-beta1` 태그도 같은 날 붙음. 발표와 동시에 공개된 패키지임.

## ChatCompletionsLanguageModel, 프레임워크에 남의 모델을 꽂는 실제 경로

[iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)에서 확인한 "`LanguageModel` 프로토콜을 따르면 아무 모델이나 붙는다"는 문구의 구현체가 이것임. OpenAI 호환 `/chat/completions` 를 말하는 서버면 `SystemLanguageModel` 자리에 그대로 들어감.

```swift
let model = ChatCompletionsLanguageModel(
  name: "minimax-m2.5",
  url: URL(string: "http://localhost/v1:8000")!,
)

let session = LanguageModelSession(model: model)
let response = try await session.respond(to: "How many folds does it take to make a paper crane?")
```

서버 능력을 초기화 시점에 꺼 둘 수 있음. 로컬 LLM 서버는 guided generation 을 못 하는 경우가 많아서임.

```swift
let model = ChatCompletionsLanguageModel(
  name: "minimax-m2.5",
  url: URL(string: "http://localhost/v1:8000")!,
  supportsGuidedGeneration: false
)
```

⚠️ **`supportsGuidedGeneration` 의 기본값이 `true` 임.** 패키지가 같이 배포하는 스킬 문서가 함정으로 직접 지목한 항목인데, 서버가 `response_format` 을 모르는데도 기본값으로 두면 `respond(to:generating:)` 호출이 막히지 않고 통과하고 **자유 형식 출력이 그대로 돌아옴.** 스키마 위반이 예외가 아니라 조용한 오염으로 나타난다는 뜻임.

베이스 URL 처리 규칙도 적혀 있음. 경로 요소 어딘가에 `v1` 이 있으면 `/chat/completions` 만 붙이고, 없으면 `/v1/chat/completions` 를 붙임. `additionalHeaders` 는 더하는 게 아니라 **덮어쓰는** 동작이라 `Content-Type` 이나 `User-Agent` 를 여기 넣으면 패키지 기본값이 사라짐. `Authorization` 만 넣는 용도로 쓰는 게 맞음.

소스에 `canImport(FoundationNetworking)` 분기가 있고 README 가 "Apple platforms and select Linux distributions like Ubuntu" 를 지원 플랫폼으로 적음. 다만 `Package.swift` 의 `platforms:` 에 Linux 항목은 없는데, SwiftPM 이 명시 안 된 플랫폼을 막지는 않으므로 이것만으로 모순이라고 볼 수는 없음.

## History 모디파이어 3종, 온디바이스판 compaction

`DynamicProfile` 에 붙는 모디파이어 셋임. 셋 다 매 생성 전에 트랜스크립트를 변형함.

| 모디파이어 | 하는 일 |
|---|---|
| `droppingCompletedToolCalls()` | 이미 끝난 tool call·tool output 항목을 걷어냄. **가장 최근 tool call 교환과 툴 아닌 항목은 남김** |
| `rollingWindow(entries:)` | 최근 N개 항목만 유지. 내부적으로는 `rollingWindow(size: .entries(n))` 로 감 |
| `summarizeHistory(entryThreshold:model:instructions:summaryPostamble:)` | 항목 수가 임계값을 넘으면 **별도 모델 세션**으로 과거를 요약해 한 항목으로 갈아끼움 |

README 예제가 셋을 겹쳐 씀. **모디파이어는 바깥에서 안으로 적용되므로 소스에서 마지막에 쓴 것이 런타임에 먼저 돎.**

```swift
Profile {
  Instructions("A conversation between a user and a helpful assistant.")
  ToggleDarkModeTool()
}
.summarizeHistory(entryThreshold: 10, model: status.summarizerModel)
.rollingWindow(entries: 10)
.droppingCompletedToolCalls()
```

즉 툴 호출을 먼저 버리고, 창을 자르고, 그래도 크면 요약함. 비싼 것을 안쪽에 두라는 순서고 소스 주석도 같은 말을 함.

`summarizeHistory` 에서 눈여겨볼 것은 **`summaryPostamble`** 임. 기본값이 이럼.

> "Do not begin with phrases like "Based on the context", "Based on the facts", "Based on the summary", or any reference to a summary or the facts provided. Treat the summary and facts above as things you naturally remember."

요약을 끼워 넣으면 모델이 "제공된 요약에 따르면" 으로 답을 시작해 **요약이 있다는 사실 자체가 사용자에게 새는** 문제가 생기는데, 그걸 막는 문장이 기본으로 들어가 있음. 빈 문자열을 넘기면 뺄 수 있음.

[서버측 compaction](../practices/2026-09-20-claude-api-compaction.md)과 같은 문제를 푸는데 층위가 반대임. 그쪽은 서버가 요약을 만들고 서명해서 돌려주고 캐시 재사용 구간을 서버가 보장함. 이쪽은 **전부 앱 프로세스 안에서 벌어지고 요약 모델도 앱이 고름.** 온디바이스라 비용은 안 나가지만 요약 품질과 요약에 드는 지연이 전부 앱 책임이 됨. 요약 모델로 온디바이스 모델을 그대로 쓰면 4K 창 안에서 요약을 만드는 셈이라 자기 꼬리를 무는 구조가 되므로, PCC 나 서버 모델을 요약 전용으로 두는 조합을 먼저 재 보는 편이 나음.

## Skills, prompt 로 넣을지 instructions 로 넣을지가 캐시를 가름

`Skills` 는 `DynamicInstructions` 이고 `SkillActivations` 로 활성 상태를 추적함. 활성화 전에는 **스킬의 이름과 설명만 프롬프트에 있고 본문은 밖에 있음.** 모델이 툴 호출을 내서 켬.

Apple 이 스킬 두 종류의 차이를 표로 적어 뒀는데 이 문서에서 제일 값어치가 큰 부분임.

| 종류 | 본문이 들어가는 자리 | KV 캐시 영향 |
|---|---|---|
| prompt 기반 | 활성화 툴 호출의 **tool output** 으로 반환됨. 새 턴 안에 머묾 | 없음. 앞선 트랜스크립트 바이트가 안 바뀜 |
| instructions 기반 | 트랜스크립트 맨 앞 instructions 항목에 **끼워 넣음**. 활성인 동안 유지됨 | **대화 전체의 KV 캐시가 무효화됨** (prefix 가 바뀜) |

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 정리한 "prefix 앞쪽을 건드리면 그 뒤가 전부 날아간다" 는 규칙이 온디바이스 KV 캐시에도 그대로 적용된다는 것을 **Apple 이 자기 API 문서로 확인해 준 셈**임. 이 저장소가 클라우드 API 쪽에서 여러 번 적어둔 결론인데 온디바이스에서 1차 출처로 확인된 것은 이번이 처음임.

실무 판단은 이렇게 갈림. 본문이 크거나 한 턴만 필요하면(스타일 가이드, 레퍼런스 문서) prompt 기반. 본문이 짧고 여러 턴에 걸쳐 시스템 지시로 작동해야 하면 instructions 기반. 후자는 `allowsDeactivation: true` 로 모델이 두 번째 툴 호출을 내서 본문을 다시 뺄 수 있고, `droppingCompletedToolCalls()` 와 묶으면 활성화·비활성화 툴 호출 쌍까지 히스토리에서 사라짐.

⚠️ **prompt 기반 스킬도 툴 호출 쌍을 트랜스크립트에 남김.** 캐시는 안 깨지지만 항목 수는 늘어남. `summarizeHistory` 의 임계값이 항목 수 기준이라 스킬을 자주 켜는 세션은 의도보다 빨리 요약이 도는 쪽으로 움직임.

`SkillActivations` 는 `Observable` 이면서 `RandomAccessCollection<String>` 이라 SwiftUI 뷰가 그대로 순회하고 모델이 스킬을 켤 때 다시 그림. 내부 변경은 `Mutex` 로 보호됨. **참조 타입이므로 세션 하나당 하나를 들고 있어야 하고, 렌더마다 새로 만들면 활성 상태가 날아감.**

## 코딩 에이전트용 스킬을 패키지가 같이 배포함

`skills/` 아래 `SKILL.md` 두 개가 들어 있음. "이 패키지를 쓰는 법" 을 사람이 아니라 **코딩 에이전트에게 가르치는 문서**임.

| 파일 | 크기 | 다루는 것 |
|---|---|---|
| `skills/foundation-models-utilities/SKILL.md` | 20,255 B | 이 패키지의 세 기능 영역, 함정 목록, 패키지 레이아웃 |
| `skills/foundation-models-language-model-protocol/SKILL.md` | 50,750 B | `LanguageModel` 프로토콜을 직접 구현하는 법 |

[Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)에서 확인한 "플러그인이 skills 를 담고 스킬은 슬래시 커맨드" 구조의 공급 쪽이 이것임. **라이브러리가 자기 사용법 스킬을 같이 배포하는 형태**가 Apple 1차 패키지에서 나온 것이라, 사내 패키지에 같은 걸 붙이는 근거로 쓸 만함. 50KB 짜리 프로토콜 구현 가이드는 사람이 읽어도 되는 분량이 아니고 애초에 에이전트 컨텍스트에 들어가라고 만든 것임.

다만 이 스킬 문서가 README·소스와 어긋나는 지점이 있어 아래에 따로 적음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 온디바이스 4K 창과 툴 정의가 창을 먹는 문제. 이 패키지가 그 문제의 공식 대응 도구임 |
| [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md) | 같은 트랜스크립트 압축 문제. 서버가 하느냐 앱이 하느냐로 갈림 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | prefix 를 건드리면 캐시가 날아간다는 규칙. instructions 기반 스킬이 그 사례임 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | `LanguageModel` 프로토콜 개방과 Dynamic Profiles. 이 패키지가 둘의 실제 사용례 |
| [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md) | 스킬 배포 경로. 패키지가 `SKILL.md` 를 같이 실어 보냄 |
| [Memory Engineer](../practices/2026-08-02-memory-engineer.md) | 무엇을 잊게 만들 것인가. History 모디파이어 3종이 그 선택지를 API 로 준 형태 |

## 짚어야 할 것

- **README 의 설치 예제가 그대로는 안 됨.** `from: "1.0.0"` 으로 적혀 있는데 저장소에 `1.0.0` 태그가 없음. 존재하는 태그는 `1.0.0-beta1`·`1.0.0-beta3`·`1.0.0-beta5`·`1.1.0-beta1` 뿐이고 SwiftPM 의 `from:` 은 프리릴리스를 집지 않음. 정확한 태그를 직접 지정해야 함
- **정식 릴리스가 없는 패키지임.** 저장소 설명이 스스로 "Emerging and experimental patterns" 라고 적음. API 이름이 바뀔 것을 전제로 붙여야 함
- **`SKILL.md` 가 말하는 SwiftPM trait 게이팅을 `Package.swift` 에서 확인하지 못했음.** 스킬 문서는 세 기능 영역이 각각 `ChatCompletions`·`Skills`·`History` trait 으로 갈리고 소스가 `#if ChatCompletions` 로 묶여 있다고 적는데, `main` 의 `Package.swift` 에 `traits:` 선언이 없고 `Skill.swift`·`RollingWindow.swift` 에도 해당 `#if` 가 없음. `ChatCompletionsLanguageModel.swift` 의 `#if` 는 전부 `canImport` 임. 문서가 앞서 나간 것인지 이후 되돌린 것인지는 판단할 근거가 없음
- **README 본문과 API 가 어긋나는 지점이 하나 있음.** README 가 요약 예제를 "the rolling window of 10 entries exceeds 5000 tokens" 로 설명하는데, `summarizeHistory` 의 파라미터는 `entryThreshold: Int` 하나뿐이고 소스 주석이 "transcript entry count" 라고 못 박음. 토큰을 재는 경로가 없음. 이건 추측이 아니라 **패키지가 같이 싣는 `SKILL.md` 자신이 그 문구를 "aspirational" 이라고 적어 둔 것**임. 토큰 기준으로 자르려면 `rollingWindow` 를 같이 걸어야 함
- **`RollingWindowSize` 는 지금 케이스가 하나뿐임.** `.entries(Int)` 만 있는 enum 인데 굳이 전략 타입으로 열어 둔 것은 토큰 기준을 나중에 넣겠다는 신호로 읽힘. 확인된 것은 아님
- **README 예제의 URL 이 잘못돼 있음.** `URL(string: "http://localhost/v1:8000")!` 인데 포트가 경로 뒤에 붙어 있어 호스트 `localhost` 에 경로 `/v1:8000` 으로 파싱됨. 복붙하면 안 됨
- **`summarizeHistory` 는 트랜스크립트 마지막 항목이 `.prompt` 일 때만 돎.** 다른 종류면 아무것도 안 함. 툴 호출이 진행 중인 구간에서 압축이 걸릴 거라고 가정하면 안 됨
- **실제로 빌드해 보지 않았음.** 이 문서는 저장소의 README, `Package.swift`, `Sources/` 소스와 주석, `skills/SKILL.md`, GitHub API 응답을 읽은 것까지임. iOS 27 실기기나 시뮬레이터에서 동작을 확인한 것이 아님
- **Anthropic·Google 이 같은 프로토콜로 Swift 패키지를 낸다는 발표가 있었으나, 그 패키지들은 이 점검에서 확인하지 못했음.** 이 문서가 다루는 것은 Apple 자신이 낸 것 하나임

## 유효기간

**2026-09-24 확인 기준, 최신 태그 `1.1.0-beta1`(2026-09-21) 시점임.** stars 와 이슈 수는 며칠 단위로 낡으므로 갱신하지 말고 `gh api repos/apple/foundation-models-utilities` 로 다시 받을 것. 다시 볼 때 확인할 것 셋. **`1.0.0` 정식 태그가 붙어 README 의 `from: "1.0.0"` 이 실제로 해석되는지**, **`RollingWindowSize` 에 토큰 기준 케이스가 추가됐는지**, 그리고 **`Package.swift` 에 trait 선언이 들어와 `SKILL.md` 의 서술과 맞춰졌는지**. 셋 다 이 문서가 미해결로 남긴 항목임.
