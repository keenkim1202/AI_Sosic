---
title: Claude for Foundation Models, LanguageModelSession 으로 Claude 를 호출하는 Swift 패키지
source: https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/apple-foundation-models
author: Anthropic
collected: 2026-10-25
tags: [apple, foundation-models, swift, claude, ios, tooling, opensource]
---

출처: [Apple Foundation Models, Claude Platform Docs](https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/apple-foundation-models) · [anthropics/ClaudeForFoundationModels](https://github.com/anthropics/ClaudeForFoundationModels) (README 와 릴리스 노트) · [Claude Platform release notes](https://platform.claude.com/docs/en/release-notes/overview) (패키지 공개 항목은 **2026-06-09**) · 레포 수치는 `gh api`로 2026-10-25 에 받음

## 요약

Anthropic 이 **Claude 를 Apple Foundation Models 프레임워크의 `LanguageModel` 로 꽂아주는 Swift 패키지**를 공개해 둠. 즉 `LanguageModelSession` 하나로 온디바이스 모델과 Claude 를 **`model:` 인자만 바꿔** 갈아끼울 수 있고, `respond(to:)`·스트리밍·`@Generable` 유도 생성·툴 호출이 그대로 동작함. 이 저장소의 [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)이 "서드파티 프로바이더 구현의 세부는 타입 이름까지만 확인했다"고 남겨둔 자리의 **실물 구현**이고, [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)가 인용한 "cloud models like Claude and Gemini" 문구의 벤더 쪽 답임. 실무에서 제일 큰 항목은 기능이 아니라 **인증**임. **App Attest 로 앱에 키도 백엔드도 없이** 과금을 자기 워크스페이스로 붙일 수 있음. 대가가 둘 있음. 요청이 앱에서 Anthropic 으로 **직접** 가고(Apple 은 경로에 없음), 그리고 **프롬프트 캐시 제어·토큰 카운팅·배치·베타 헤더가 이 경로에서는 아예 없음.**

## 기본 정보

| 항목 | 값 |
|---|---|
| 레포 | [anthropics/ClaudeForFoundationModels](https://github.com/anthropics/ClaudeForFoundationModels) |
| 별 / 포크 / 열린 이슈 | **335 / 39 / 7** (`gh api`, 2026-10-25) |
| 언어 / 라이선스 | Swift / **Apache-2.0** |
| 생성 / 최근 푸시 | 2026-05-20 / 2026-10-07 |
| 최신 릴리스 | **0.2.3** (2026-10-07). `claude-haiku-5-5` 상수 추가 하나짜리 릴리스임 |
| 설치 | SwiftPM, `from: "0.1.0"` |
| 요구 사항 | iOS·macOS·visionOS·watchOS **27**, Xcode **27** |
| 상태 | **베타.** 유지보수는 best effort 이고 외부 PR 은 베타 기간 중 받지 않음 |

릴리스 흐름을 보면 이 패키지가 무엇인지가 더 분명함. 0.2.1 은 `claude-opus-5-5`, 0.2.2 는 `claude-sonnet-5-5`, 0.2.3 은 `claude-haiku-5-5` 상수를 더한 것임. **모델 ID 가 패키지 상수로 들어오는 구조**라서, 새 모델을 쓰려면 패키지를 올리거나 아래 `ClaudeModel` 직접 생성 경로를 써야 함.

## 세션 생성 한 줄만 달라짐

```swift
import FoundationModels
import ClaudeForFoundationModels

let model = ClaudeLanguageModel(
  name: .sonnet5,
  auth: .apiKey(ProcessInfo.processInfo.environment["ANTHROPIC_API_KEY"] ?? "")
)

let session = LanguageModelSession(model: model)
let response = try await session.respond(to: "Plan a 4-day trip to Buenos Aires.")
print(response.content)
```

[Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)이 `SystemLanguageModel` 과 `PrivateCloudComputeLanguageModel` 사이에서 하던 교체가 그대로 한 칸 더 늘어난 것임. 설계 판단이 바뀌는 지점은 컨텍스트임. 그쪽 문서의 분기점이 **온디바이스 4,096 토큰 / PCC 32K** 였는데, 이 경로에서는 선택한 Claude 모델의 컨텍스트가 그대로 적용됨. 다만 그 숫자를 얻는 대가가 과금과 네트워크임.

> "Requests go directly from your app to the Claude API; Apple is not in the request path and does not see prompts or responses."

과금은 Anthropic 계정의 표준 API 가격이고, 크레딧이나 결제 수단이 있어야 함. **PCC 무료 구간 같은 것은 없음.** [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md)이 적어둔 "AI 기능 단가 계산이 사라진다"는 조건이 이 경로에는 적용되지 않음.

## 인증, 이 문서에서 제일 값어치 있는 절

세 가지임.

| 모드 | 무엇이 필요한가 | 쓰는 자리 |
|---|---|---|
| `.appAttest(clientID:)` | 콘솔의 app integration 등록, App Attest 엔타이틀먼트, **실기기** | 출시. 앱에 키 없음, 백엔드 없음 |
| `.proxied(headers:)` + `baseURL` | 자기 백엔드 릴레이 | 출시. 키를 서버가 붙임 |
| `.apiKey` | 키 문자열 | **개발 전용** |

`.apiKey` 에는 문서가 경고를 직접 붙여 뒀음.

> "A key bundled into an app is extractable from the shipping binary, and anyone who extracts it can make requests billed to your account."

### App Attest 가 실제로 바꾸는 것

앱 설치마다 Apple 의 App Attest 로 "등록된 그 앱의 변조되지 않은 빌드"임을 증명하면, Anthropic 이 그 기기에 **짧은 수명의 액세스 토큰**을 발급해 워크스페이스로 과금함. 즉 **키를 숨길 백엔드를 세우지 않아도 됨.** 개인 개발자나 소규모 앱에서 이 선택지의 값이 큼.

조건과 한계를 정확히 봐야 함.

- 설정은 셋임. Xcode 에서 **App Attest capability** 추가, 콘솔 app integrations 에서 **Apple Developer Team ID 와 번들 ID(최대 32개)** 등록, 발급된 `clid_...` 를 앱에 넣기. 등록에는 조직의 admin·owner·primary owner 권한이 필요함
- **시뮬레이터에서는 안 됨.** Secure Enclave 가 없는 하드웨어도 안 되고, 그 경우 `ClaudeError.attestationUnsupported` 가 남. 그래서 개발 중에는 `.apiKey` 를 병행하라고 문서가 적음
- 첫 요청에서 attestation 이 수 초 걸림. `try await model.authenticateIfNeeded()` 로 앱 시작 때 미리 치르면 실패를 사용자 첫 프롬프트보다 먼저 알 수 있음. `session.prewarm()` 도 시작시키지만 **오류 보고가 없는 힌트**임
- 이후에는 Keychain 의 캐시 토큰을 재사용하고, 갱신은 로컬 Secure Enclave 서명과 짧은 왕복 한 번임. 자격은 기기에 묶이고 동기화·백업되지 않음
- 토큰의 범위가 좁음

> "Tokens are scoped to your workspace, expire after 1 hour, and authorize only Messages API calls. They carry no end-user identity: App Attest identifies your app, not the person using it, so handle any per-user logic in your app."

마지막 문장이 운영에서 걸림. **사용자 식별이 토큰에 없으므로 1인당 한도는 앱이 만들어야 함.** [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)이 PCC 무료 구간에 대해 "1인당 일일 한도 UI 를 앱이 만들어야 함"이라고 적은 것과 같은 숙제가, 이유만 바꿔서 그대로 남음.

그리고 **App Attest 는 Claude API 직접 호출에서만 됨.** Amazon Bedrock·Google Cloud·Microsoft Foundry 경유에는 없음. 사용자별 귀속이 필요하면 패키지가 `userProfileID:` 를 받아 `anthropic-user-profile-id` 헤더와 `user-profiles-2026-08-18` 베타를 보내는데, **프로필은 백엔드가 `/v1/user_profiles` 로 미리 만들어야** 하므로 이 경우엔 백엔드가 다시 등장함. 폐기는 콘솔에서 integration 을 revoke 하는 것이고 **되돌릴 수 없음**(새로 만들어야 함).

## 모델 선택, capabilities, effort

`ClaudeModel` 상수는 ID 와 **그 모델이 받는 필드 목록**을 함께 들고 있음. 이유가 명시돼 있음.

> "The package uses this to determine which request fields to send, because sending a field a model rejects is a hard error."

컴파일된 상수에 없는 ID 를 쓰려면 capabilities 를 **직접 선언**해야 하고, 추측해 주는 축약은 의도적으로 없음.

```swift
let model = ClaudeModel(
  id: "claude-experimental-x",
  capabilities: .init(samplingParams: false, effortLevels: [.low, .high])
)
```

effort 는 두 경로가 겹침. `fixedEffort:` 로 모든 요청에 못 박으면 **프레임워크의 요청별 reasoning 힌트보다 우선**함. 프레임워크의 이름 붙은 레벨은 `high` 에서 멈추므로, 한 요청만 더 올리려면 `.custom("xhigh")` 나 `.custom("max")` 를 넘김. 모델이 받지 않는 레벨은 조용히 **드롭**됨(에러가 아님). 다섯 레벨 중 무엇을 받는지는 모델마다 다르고, 아예 안 받는 모델도 있음.

거절 폴백도 패키지 표면에 있음. `fallbacks:` 로 최대 3개를 이름 지어 두면 거절된 요청을 같은 요청 안에서 순서대로 재시도하고, `fallbacks: .serverDefault` 로 서버 기본 구성에 맡길 수도 있음. 모델이 답을 **시작한 뒤** 거절하면 폴백 모델이 그 텍스트에서 이어 쓰고, 빈 텍스트 세그먼트가 인계 지점을 표시함(`entry.claudeHandovers`, `entry.claudeModelID`). 인계 뒤에는 그 대화의 다음 요청들이 **약 1시간 동안** 폴백 모델로 바로 감. 스키마나 이미지를 쓰면 체인의 **모든** 모델이 그것을 지원해야 하고, 샘플링 파라미터는 전원이 받을 때만 전송됨. `.proxied` 를 쓰면 릴레이가 `anthropic-beta` 헤더를 **전달해야** 폴백이 돎.

## 서버 툴과 전사(transcript)

프레임워크의 `tools:` 배열은 그대로 동작함. 기기에서 도는 클라이언트측 툴임. 반면 웹 검색·웹 페치·코드 실행은 Anthropic 쪽에서 **한 번의 왕복 안에서** 돌고, 기기가 호출할 것이 없으므로 세션이 아니라 **모델에 붙임**.

```swift
let model = ClaudeLanguageModel(
  name: .sonnet5,
  auth: auth,
  serverTools: [
    .webSearch(maxUses: 5),
    .codeExecution,
  ]
)
```

세션이 아니라 모델에 붙는 이유는 `LanguageModelSession` 이 Apple 타입이라 손댈 수 없기 때문임. 그래서 **대화마다 서버 툴 조합을 달리하려면 `ClaudeLanguageModel` 인스턴스를 여러 개 만들어야 함.** 검색·페치는 도메인 허용·차단 목록과 `maxUses` 를 받음.

서버 툴 활동은 전사에서 `ClaudeServerToolSegment` 커스텀 세그먼트로 보이고, 모델이 호출을 내는 즉시 자리를 잡고 결과가 같은 응답 안에서 채워짐. 스트리밍 중에도 스냅샷의 엔트리를 같은 방식으로 훑어 검색 활동을 인라인으로 그릴 수 있음. 다음 턴에 API 가 다시 받아야 하는 것들(thinking 서명, 검색 결과, 인용)은 전사 엔트리의 **예약 메타데이터 키 `claude.content`** 에 보관돼 자동 재생되는데, 문서는 그 값을 **불투명하게 취급하라**고 적음. 전사를 직렬화해 저장하는 앱이면 이 키를 건드리지 말아야 함.

구조화 출력은 `@Generable` 과 `generating:` 그대로임. 모델이 구조화 출력을 지원하지 않으면 **조용히 품질을 낮추지 않고** `LanguageModelError.unsupportedGenerationGuide` 를 던짐.

에러는 Apple 의 `LanguageModelError` 로 매핑되는 것만 매핑됨. 컨텍스트 초과는 `.contextSizeExceeded`, HTTP 429 는 `.rateLimited`, 타임아웃은 `.timeout`. 프레임워크에 대응이 없는 것은 `ClaudeError` 로 옴. 문서가 권하는 패턴이 쓸모 있음. **`.rateLimited` 를 잡아 그 턴만 `SystemLanguageModel` 로 떨어뜨리기.** 온디바이스 모델이 폴백으로 남아 있는 구조가 이 프레임워크의 이점임.

## 이 경로에서 사라지는 것

프레임워크의 프로바이더 프로토콜이 표현하지 못하는 Messages API 기능은 **없는 것**이 됨. 문서가 직접 열거함.

| 빠지는 것 | 이 저장소에서 걸리는 지점 |
|---|---|
| 프롬프트 캐시 제어 | 캐시는 **자동 적용**되지만 TTL 과 중단점 배치를 지정할 수 없음 |
| 스톱 시퀀스 | 출력 절단을 프롬프트 밖에서 막는 수단이 없음 |
| 배치 처리 | 대량 비동기 작업은 다른 경로로 빼야 함 |
| Files API | 파일 업로드 기반 흐름이 안 됨 |
| 토큰 카운팅 | 요청 전 토큰 계산을 못 함. 온디바이스의 `tokenCount(for:)` 같은 수단이 없음 |
| 베타 헤더 | 베타 기능을 **직접 켤 수 없음** |

첫 줄이 이 저장소에서 가장 아픈 항목임. [Prompt Caching In Agents](./2026-07-22-prompt-caching-in-agents.md)가 정리한 레버(중단점을 어디에 두는가, TTL 을 1시간으로 올리는가)가 전부 **쓸 수 없는 상태로 고정**됨. 캐시가 자동이라는 서술만 있고 배치 전략은 공개돼 있지 않으므로, 이 경로의 캐시 적중률은 **측정도 조정도 앱에서 할 수 없다**고 보는 편이 안전함. 토큰 카운팅이 없다는 것과 합치면, 비용 관측을 하려면 콘솔이나 서버측 집계에 의존해야 함.

베타 헤더 항목도 뒤집어 읽을 거리임. 사용자가 보낼 수는 없지만 **패키지 자신은 보냄**(사용자 프로필의 `user-profiles-2026-08-18`, 폴백의 `anthropic-beta`). 즉 "베타 기능 불가"가 아니라 **"패키지가 지원하는 베타만 가능"**이 정확한 서술임.

## 언제 안 쓰는 게 맞나

- **온디바이스로 충분한 작업**: 요약·분류·추출처럼 짧은 작업은 무료·오프라인·프라이버시 쪽이 그대로 이김. 문서 자신이 "larger context, frontier reasoning, 서버 툴이 필요할 때 올려라"로 적음
- **캐시 비용을 설계로 다뤄야 하는 앱**: 위 표의 첫 줄 때문임. 프롬프트 prefix 를 길게 들고 가는 설계라면 Messages API 를 직접 치는 쪽이 통제권이 큼
- **Messages API 를 일반적으로 쓰려는 경우**: 이 패키지는 그 용도가 아님. 공개 표면이 `ClaudeLanguageModel`, `ClaudeModel`, `AuthMode`, `ClaudeServerTool` 뿐임
- **Bedrock·Google Cloud·Foundry 경유가 조직 요건인 경우**: App Attest 이점이 사라져서 백엔드를 다시 세워야 함

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 그 문서가 "본문을 읽지 못했음"으로 남긴 서드파티 프로바이더 구현의 실물. 컨텍스트 분기점(4,096 / 32K)에 선택지가 하나 더 붙음 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 그 문서가 인용한 "cloud models like Claude and Gemini" 문구의 벤더측 구현 |
| [Prompt Caching In Agents](./2026-07-22-prompt-caching-in-agents.md) | 캐시 중단점·TTL 레버가 이 경로에서는 노출되지 않음 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md) | 요청이 앱에서 Anthropic 으로 직접 가므로 제공자 이름과 데이터 항목을 대는 사전 동의 요건에 걸림 |
| [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md) | PCC 무료 구간의 단가 계산이 이 경로에는 적용되지 않음 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | effort 와 캐시 배수 판단. 베타 헤더를 직접 못 보내므로 그쪽이 적은 "대화 중 effort 변경(베타)"을 이 경로에서 켤 수 없음 |

## 짚어야 할 것

- **베타 문언이 멈춰 있음.** 문서와 README 가 요구 OS 를 "OS 27 betas", "Xcode 27 (beta)" 로 적지만 [iOS 27 과 Xcode 27 은 2026-09-14 에 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)됐음. 요구 사항은 27 이상으로 읽으면 되고, **패키지 자신이 베타라는 것은 별개의 사실**임. 문서의 Note 는 그대로 유효함

> "Beta. This package targets the Foundation Models server-side language model API introduced in the OS 27 betas. APIs might change during the beta."

- **effort 기본값 서술이 두 1차 출처에서 어긋남.** 레포 README 는 "The API defaults to `high` when no effort is sent" 라고 적고, 문서 페이지는 "모델의 기본 레벨이며 모델마다 다르다"고 적음. [thinking 문서](https://platform.claude.com/docs/en/build-with-claude/thinking)는 Claude Opus 5.5 와 Claude Haiku 5.5 의 기본이 `medium` 이라고 적음. **README 쪽이 낡은 것으로 판단**하지만 패키지가 어느 쪽으로 동작하는지는 코드로 확인하지 못했음
- **App Attest 의 도입 시점을 확인할 수 없었음.** 릴리스 노트의 패키지 공개 항목(2026-06-09)에는 App Attest 언급이 없고, 콘솔 app integrations 페이지에 대한 릴리스 노트 항목도 찾지 못했음. 0.2.0 의 breaking change 는 Xcode 27 beta 5 채택이고 App Attest 가 아님
- **직접 써보지 않았음.** 예제 실행에 macOS 27 호스트가 필요하고 App Attest 에는 실기기가 필요함. 이 문서는 공식 문서·README·릴리스 노트·GitHub API 로만 확인한 것임
- **캐시 거동이 공개돼 있지 않음.** "자동으로 적용한다"는 서술만 있음. 중단점을 어디에 두는지, 대화가 길어질 때 어떻게 되는지는 확인할 수 없었음
- **유지보수 상태를 README 가 스스로 낮춰 적음.** best effort, as is, 베타 기간 중 외부 PR 미접수. 출시 앱의 의존성으로 넣을지는 이 문언을 보고 판단해야 함
- 레포의 별·포크·이슈 수는 2026-10-25 에 받은 값임. 수치가 낡으면 갱신하지 말고 다시 받을 것

## 유효기간

**2026-10-25 확인 기준이고 패키지 0.2.3 시점임.** 모델 상수는 릴리스마다 늘어나므로 위 표를 고치지 말고 `gh api repos/anthropics/ClaudeForFoundationModels/releases` 로 다시 받을 것. 다시 볼 때 확인할 것 둘. **패키지의 베타가 끝났는지**(문서 Note 와 README 의 "before general availability" 문언), 그리고 **빠지는 기능 표가 줄었는지**. 특히 프롬프트 캐시 제어와 토큰 카운팅이 생기면 비용 설계가 달라짐. App Attest 가 Bedrock·Google Cloud·Foundry 로 확대되는지도 같이 봐야 함.
