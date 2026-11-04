---
title: Siri AI Developer Preview, 앱 스킬이 Agent Skills 의 부분집합이라고 Apple 이 적었음
source: https://developer.apple.com/news/?id=gdk2umrf
author: Apple
published: 2026-11-03
collected: 2026-11-04
tags: [apple, ios, app-intents, skills, agent-skills, eval, xcode]
---

출처: [Siri AI is now open to all developers](https://developer.apple.com/news/?id=gdk2umrf) (2026-11-03) · [Siri AI Developer Preview](https://developer.apple.com/documentation/appintents/siri-ai-developer-preview) · [Bringing custom actions and content to Siri](https://developer.apple.com/documentation/appintents/bringing-custom-actions-and-content-to-siri) · [Orchestrating complex actions with app skills](https://developer.apple.com/documentation/appintents/orchestrating-complex-actions-with-app-skills) · [Siri Evaluations](https://developer.apple.com/documentation/sirievaluations) · 심볼 [`ToolIntent`](https://developer.apple.com/documentation/appintents/toolintent) · [`GenerableEntity`](https://developer.apple.com/documentation/appintents/generableentity) · [Releases](https://developer.apple.com/news/releases/) · [Xcode 27.3 beta Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_3-release-notes) · [App Intents updates](https://developer.apple.com/documentation/updates/appintents) (전부 **2026-11-04 확인**. 문서·릴리스 노트 페이지는 자바스크립트로 렌더링되므로 같은 경로의 `/tutorials/data/...json` 으로 본문을 받았음)

## 요약

**2026-11-03, Apple 이 Siri AI 를 모든 개발자에게 열었음.** 형태는 정식 API 가 아니라 **Developer Preview** 이고 **27.3 베타(iOS·iPadOS 24C5037i, macOS 26C5035r, visionOS 24N5286r, Xcode 27C5020n)** 와 같은 날 올라왔음. 붙이는 경로는 새 프레임워크가 아니라 **기존 App Intents** 이고, 문서가 **"엔타이틀먼트를 새로 추가하거나 별도 개발자 프로그램에 가입할 필요가 없다"**고 적음.

그런데 **지금 할 수 있는 것은 빌드와 내부 테스트까지임.** 프리뷰는 **외부 TestFlight 테스터와 App Store 에 올린 빌드에서는 동작하지 않고**, App Store 제출은 **봄에 열림**. 즉 이번 분기에 이 기능으로 출시 일정을 잡을 수는 없음.

이 저장소에 직접 걸리는 발견이 둘임. 첫째, **앱 스킬이 `SKILL.md` 를 쓰고 Apple 이 그것을 "Agent Skills 포맷의 부분집합"이라고 1차 문서에 적었음.** [Agent Skills 가 Linux Foundation 으로](../agents/2026-10-22-agent-skills-standard-aaif.md)가 "Apple 의 어느 문서도 Agent Skills 나 `SKILL.md` 를 이름으로 언급하지 않음"으로 적고 그것이 바뀌는지를 재확인 신호로 걸어뒀는데, **그 신호가 켜졌음.** 단 부분집합이 좁음. 프론트매터는 `name`·`description` 둘뿐이고 **`scripts/` 같은 보조 파일을 명시적으로 안 받음.** 둘째, **`SiriEvaluations`** 프레임워크가 생겨서 Siri 대화를 스크립트로 돌리고 어떤 인텐트를 불렀는지 채점함. 전부 **27.3 베타**이고, 프레임워크 자신이 **"Siri 응답이 실행마다 달라지므로 세트를 여러 번 돌려라"**고 적음. 머지 게이트로 쓸 수 있는 모양이 아님.

## 무엇이 언제 나왔나

Releases 목록에서 받은 빌드 번호와 날짜임. **전부 2026-11-03** 이고 같은 날 뉴스 항목이 올라왔음.

| 항목 | 버전 | 날짜 |
|---|---|---|
| Xcode 27.3 beta | 27C5020n | 2026-11-03 |
| iOS 27.3 beta / iPadOS 27.3 beta | 24C5037i (동일 빌드) | 2026-11-03 |
| macOS 27.3 beta | 26C5035r | 2026-11-03 |
| visionOS 27.3 beta | 24N5286r | 2026-11-03 |
| tvOS 27.3 beta | 24K5286g | 2026-11-03 |
| watchOS 27.3 beta | 24S5284h | 2026-11-03 |

프리뷰가 도는 플랫폼으로 문서가 적은 것은 **iOS·iPadOS·macOS·visionOS 넷**임. tvOS·watchOS 베타는 같이 나왔지만 이 목록에 없음. Xcode 27.3 beta 는 **Swift 6.4** 와 27.3 SDK 묶음이고 **macOS Tahoe 26.6 이상**을 요구함. 27.2 와 같은 요건이라 빌드 머신 OS 는 건드릴 일이 없음.

## 먼저 봐야 할 것은 기능이 아니라 유통 경계임

프리뷰 문서가 테스트 방법을 둘로 한정함. 개발 프로파일로 서명해 기기에서 돌리는 것, 그리고 **TestFlight 내부 테스터**임. 그다음 문장이 이 문서에서 제일 중요함.

> "The Siri AI Developer Preview isn't available to external testers in TestFlight or to versions of your app that you publish on the App Store."

그리고 뉴스 항목이 **App Store 제출은 봄에 열린다**고 적음. 고객에게 기능이 가는 것은 **몇 달 뒤**임. 정리하면 이럼.

| 경로 | 지금 되는가 |
|---|---|
| 개발 서명 + 기기 | 됨 |
| TestFlight 내부 테스터 | 됨 |
| TestFlight 외부 테스터 | **안 됨** |
| App Store 배포판 | **안 됨** |
| App Store 제출 | **봄에 열림** |

여기서 갈리는 판단이 있음. 프리뷰 기간에 쓸 시간을 **기능 구현**에 넣을지 **인텐트·엔티티 메타데이터와 평가 세트**에 넣을지임. 뒤쪽이 맞다고 보는 근거가 아래 Siri Evaluations 절임. 메타데이터 품질이 그대로 Siri 의 선택 정확도이고, 그것을 재는 도구가 이번에 같이 나왔음. 기능 쪽은 봄까지 어차피 못 내보냄.

Apple 쪽 지원 창구는 **Apple Intelligence Group Labs, 2026-11-04~05** 임. 이 문서를 읽는 시점에 이미 지났을 수 있음.

## 인텐트를 툴로 올리는 조건

핵심 매크로가 `@AppIntent(.tool)` 이고 이것이 **`ToolIntent`** 프로토콜 적합성을 붙임.

> "A protocol that identifies an app intent that Siri can call while handling a person's request."

가용성은 **iOS·iPadOS·Mac Catalyst·macOS·visionOS 27.3, 전부 beta** 임. 범위의 전제가 이 문장임.

> "Siri can only find custom content and perform custom actions an app makes explicitly available using App Intents."

즉 **노출 모델이 화이트리스트임.** 앱 인텐트를 많이 가진 쪽이 자동으로 유리해지는 구조가 아니고, 어느 인텐트를 툴로 올릴지 고르는 일이 새로 생김. 문서가 좋은 후보와 나쁜 후보를 갈라 적어 둠. 좋은 쪽은 **한 문장으로 요청되고 파라미터를 그 문장에서 채울 수 있는 동작**이고, 나쁜 쪽은 **위젯 설정 인텐트, 숏컷의 한 단계로만 의미가 있는 인텐트, 주문·결제처럼 사람이 명시적으로 시작해야 하는 부수효과**임.

**밟기 쉬운 자리가 하나 있음.** 스키마에 적합한 인텐트에 `.tool` 을 붙이는 것이 조건부임. Shortcuts 전용 스키마(`openBook` 같은 것)에는 붙이는 게 맞지만, **Siri 가 이미 지원하는 스키마(`MessagesIntent.sendMessage` 등)에 붙이면 빌드 에러**임. 런타임에 조용히 중복되는 쪽이 아니라 컴파일에서 막히는 쪽이라 발견은 빠름.

엔티티 쪽은 **`GenerableEntity`** 가 새로 생김.

> "An interface that tells a model it can create an entity and fill in its properties directly."

`@AppEntity(.generable)` 과 `TransientAppEntity` 를 같이 채택하고, 프로퍼티 타입이 **값 타입·`AppEnum`·다른 `GenerableEntity`·그것들의 배열** 중 하나여야 함. 아니면 컴파일 에러임. 역할 구분이 명확함. **generable 이면 모델이 요청에서 직접 채우고, 아니면 시스템이 엔티티 쿼리를 돌려 조회함.** 검색 왕복을 없애고 싶은 파라미터 타입에 쓰는 것임.

엔티티를 Siri 가 **찾게** 하는 경로는 Spotlight 인덱스가 1순위고, 인덱스에 없으면 `EnumerableEntityQuery` 로 넘어가는데 문서가 그것을 **"비교적 작은 엔티티의 짧은 목록"** 용도로 적음. 큰 카탈로그를 여기 얹으면 안 됨.

## 앱 스킬은 Agent Skills 의 부분집합임

이 절이 이번 발표에서 이 저장소에 가장 크게 걸리는 부분임. 복잡한 요청을 위해 Siri 에 도메인 지식을 주는 수단이 **앱 스킬**이고, 포맷에 대해 문서가 이렇게 적음.

> "App skills use a subset of the Agent Skills format"

링크가 `agentskills.io/specification` 로 직접 걸려 있음. [Agent Skills 가 Linux Foundation 으로](../agents/2026-10-22-agent-skills-standard-aaif.md)는 **"Apple 문서는 스킬을 슬래시 커맨드로 부르고 직접 만들 수 있다고 적지만 `SKILL.md` 도 Agent Skills 도 언급하지 않음"**으로 적고, 유효기간에 **"Apple 문서가 `SKILL.md` 를 이름으로 언급하기 시작하는지가 신호"**로 걸어뒀음. 그 신호가 켜진 것임. 다만 **켜진 자리가 Xcode 문서가 아니라 App Intents 문서**라는 점이 중요함. Xcode 의 코딩 에이전트 스킬과 Siri 의 앱 스킬은 이름이 같을 뿐 다른 소비자를 향함.

부분집합이 얼마나 좁은지가 실무 결론을 바꿈. 명세 쪽 필드는 그 문서의 표에 여섯 개로 정리돼 있고, Apple 이 받는 것은 둘임.

| 항목 | Agent Skills 명세 | Apple 앱 스킬 |
|---|---|---|
| `name` | 필수. 1~64자, 소문자·숫자·하이픈, 부모 디렉터리와 일치 | **같음** |
| `description` | 필수. 1~1024자 | 필수. **1024자 상한, 한 줄** |
| `license` · `compatibility` · `metadata` · `allowed-tools` | 선택 | **안 받음** |
| 본문 | 마크다운, 형식 제약 없음 | 마크다운, **10,000자 상한** |
| 보조 파일(`scripts/`, 참조 문서) | 3단 progressive disclosure 의 핵심 | **안 받음** |

문언이 이럼.

> "Siri doesn't support other frontmatter fields or supporting files like scripts or references."

> "Each directory in `Skills/` represents one skill and must contain a `SKILL.md` file"

**가져가야 할 판단은 이것임.** Agent Skills 로 쓴 스킬을 Siri 로 그대로 옮길 수 있다고 가정하면 안 됨. 받는 것은 프론트매터 두 줄과 본문뿐이라, 스크립트에 기대 동작하던 스킬은 **옮길 수 있는 부분이 설명문밖에 없음.** 반대로 이 저장소가 [Agent Skills 문서](../agents/2026-10-22-agent-skills-standard-aaif.md)에서 짚은 보안 문제, 즉 스킬이 `scripts/` 를 들고 오고 그것을 에이전트가 실행한다는 표면은 **Siri 쪽에는 없음.** 좁은 부분집합이 그 자리에서는 이득임.

배치와 제약은 이럼.

```text
Skills.skillsbundle/
└── Skills/
    ├── name-of-skill-01/
    │   └── SKILL.md
    └── name-of-skill-02/
        └── SKILL.md
```

- Xcode 템플릿(File > New > File from Template > Skills Bundle)으로 만들고 **앱 타깃의 Copy Bundle Resources** 에 들어가야 함
- **앱 타깃당 `.skillsbundle` 은 하나뿐임.** 번들이 Copy Bundle Resources 에서 빠졌거나 컴파일 시점 에러가 있으면 **스킬을 아예 제공하지 않음**. 에러가 아니라 침묵으로 떨어지는 실패라 눈에 안 보임
- 스킬 이름은 앱·익스텐션·임베드된 프레임워크를 통틀어 유일해야 하고, 겹치면 **"시스템이 그중 하나만 쓰는데 어느 것인지는 정의되지 않음"**
- `description` 은 Siri 가 **본문을 읽지 않고** 이 스킬이 해당하는지 판정하는 데 쓰임. 좁게 쓰면 안 걸리고 넓게 쓰면 엉뚱한 데 걸림

그리고 기대치를 깎는 문장이 본문에 박혀 있음.

> "You can't use app skills to enforce deterministic behavior. Instead, app skills provide additional information that Siri AI uses to respond to complex requests. Siri AI doesn't sequentially perform the tool intents that your skill lists."

**순서를 적을 수는 있지만 그 순서로 돈다는 보장이 없음.** 앱 스킬을 워크플로 정의로 읽으면 틀림. 프롬프트에 가까움.

## Siri Evaluations, 대상이 비결정적인 채점 프레임워크

**`SiriEvaluations`** 가 생겼음. 플랫폼은 **iOS·iPadOS·macOS·visionOS 27.3** 이고 페이지의 **모든 심볼이 beta** 임.

> "Evaluate how well Siri engages your app's intents and entities by running scripted conversations and checking what Siri does."

구조가 이럼. `SiriScenario` 가 한 세션에서 도는 멀티턴 상호작용이고, `UserTurn` 하나가 발화 하나와 그 턴의 기대치임. 기대치 타입이 다섯임.

| 타입 | 기대하는 것 |
|---|---|
| `Calls` | 특정 인텐트를 **부르는지 / 안 부르는지** |
| `Finds` | 특정 앱 엔티티를 검색·해석으로 찾아내는지 |
| `Loads` | **특정 앱 스킬을 로드하는지** |
| `OnScreen` | 특정 앱 엔티티가 화면에 있는지 |
| `SiriResponse` | 응답 텍스트에 특정 문구가 들어 있는지 |

`Loads` 가 눈에 걸림. 앞 절의 `description` 선택 문제, 즉 스킬이 걸려야 할 때 걸리는지를 **직접 재는 수단**임. 실행은 `SiriService` 가 하고 채점은 `SiriExpectationEvaluator` 가 하며, 기록은 `SiriTranscript` 로 남고 Xcode 가 턴별 리포트를 보여줌. [Apple Evaluations 프레임워크](../practices/2026-09-22-apple-evaluations-framework.md)와 Swift Testing 위에 얹히는 구조라 그 문서의 워크플로가 그대로 적용됨.

**이 저장소 관점에서 중요한 것은 수치가 아니라 프레임워크가 직접 적은 운용 지침임.**

> "Because Siri's responses can vary between runs, run the set more than once, and improve your intents, their parameter names and descriptions, and your App Skills until more of the set passes, and passes consistently."

개별 비교는 결정적이라고 문서가 적는데, **채점 대상이 비결정적임.** [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md)이 다룬 것은 판정자가 흔들리는 쪽이었고, 여기는 **시스템 under test 가 흔들리는 쪽**임. 층위가 다른 같은 문제임. 실무 결론 둘이 따라옴.

1. **단일 실행의 통과/실패를 머지 게이트로 쓰면 안 됨.** 게이트로 쓸 값은 반복 실행의 통과율이고, 임계값은 "틀리면 얼마나 아픈가"로 정하는 그 문서의 기준이 그대로 필요함
2. **개선 대상이 코드가 아니라 메타데이터임.** 문서가 고치라고 지목하는 것이 인텐트, 파라미터 이름과 설명, 앱 스킬임. [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)의 규칙, 즉 제안한 쪽이 기준선을 못 옮기게 하라는 것이 여기서 그대로 필요함. 시나리오 세트를 통과시키려고 시나리오를 고치는 것이 가장 쉬운 길이기 때문임

## 권한과 가시성에서 밟는 자리

엔티티는 **기본적으로 인증을 요구하지 않음.** 조이는 자리가 둘임.

- 엔티티 수준: `static let authenticationPolicy: IntentAuthenticationPolicy`
- 프로퍼티 수준: `@Property`·`@DeferredProperty`·`ComputedProperty` 의 `authenticationPolicy:` 인자. `.requiresAuthentication`, `.requiresLocalDeviceAuthentication` 등

여기에 함정이 하나 있고 문서가 직접 적음. **프로퍼티 수준 정책은 Siri 가 제공하는 응답에만 걸림.** 인텐트가 민감한 데이터를 **결과로 돌려주는** 구조면 정책을 **인텐트 수준**에 걸어야 함. 프로퍼티마다 꼼꼼히 걸어두고 안전하다고 생각하는 쪽이 정확히 이 자리에 걸림.

가시성은 별개 축임. `visibility: .internal` 인 프로퍼티는 앱과 익스텐션에만 보이고 **Siri·Shortcuts·다른 시스템 기능이 노출하지도 읽지도 않음.** 라우팅 번호처럼 내부 식별용 필드는 인증을 거는 게 아니라 여기로 빼는 것이 맞음.

## 갱신 페이지는 아직 6월에 멈춰 있음

**App Intents updates 페이지의 최신 절이 여전히 June 2026** 임(2026-11-04 확인). Siri AI Developer Preview, `ToolIntent`, `GenerableEntity`, 앱 스킬, `SiriEvaluations` 가 **이 목록에 하나도 없음.**

[iOS 27.2 정식 출시](./2026-10-26-ios-27-2-foundation-models-ga.md)가 똑같은 모양을 심볼 페이지와 갱신 페이지 사이에서 찾아 적었고, [App Intents Testing](../practices/2026-10-05-app-intents-testing-and-onscreen-context.md)이 유효기간에 **"App Intents 갱신 내역에 2026-09 이후 묶음이 생겼는지"**를 재확인 과제로 적어뒀음. **오늘 기준 답은 아직 아님.** 결론은 그 두 문서가 내린 것과 같음. **갱신 페이지를 단일 출처로 쓰면 이 릴리스의 변경을 통째로 놓침.** 베타 딱지와 도입 버전은 심볼 페이지 JSON 의 `platforms` 배열이 가장 빠른 신호임.

릴리스 노트 쪽도 같음. **Xcode 27.3 beta 노트의 Coding Intelligence 절에 Siri 관련 항목이 하나도 없음.** 프리뷰 문서가 이름으로 소개한 **Siri Integration Advisor**(Xcode 안에서 App Intents 구현을 보고 개선점을 제안하는 기능)가 노트에 안 나옴. 그 절에 새 기능으로 들어 있는 것은 하나임.

> "GitHub Copilot is now available in the coding assistant. (185837709)"

[Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)이 Gemini·Antigravity 추가를 두고 "3사로 세는 것은 이제 맞지 않음"으로 적었는데 한 곳이 더 늘었음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Agent Skills 가 Linux Foundation 으로](../agents/2026-10-22-agent-skills-standard-aaif.md) | 그 문서가 재확인 신호로 걸어둔 항목에 답한 것. **Apple 이 `SKILL.md` 와 Agent Skills 를 이름으로 언급했음.** 단 프론트매터 둘만 받고 `scripts/` 는 안 받는 부분집합임 |
| [App Intents Testing](../practices/2026-10-05-app-intents-testing-and-onscreen-context.md) | 같은 프레임워크의 테스트 경로. 그 문서의 재확인 과제인 갱신 페이지 공백은 아직 안 풀림. 온스크린 컨텍스트가 이제 커스텀 엔티티까지 같은 API 로 덮임 |
| [Apple Evaluations 프레임워크](../practices/2026-09-22-apple-evaluations-framework.md) | `SiriEvaluations` 가 그 프레임워크와 Swift Testing 위에 얹힘. 판정자 편향 쪽이 아니라 **대상의 비결정성** 쪽 문제임 |
| [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md) | 반복 실행 통과율을 게이트로 쓰는 판단. 단일 실행으로 게이트를 열면 안 되는 이유 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | 고치라고 지목된 것이 메타데이터라, 기준선을 제안한 쪽이 못 옮기게 하는 규칙이 그대로 필요함 |
| [iOS 27.2 정식 출시](./2026-10-26-ios-27-2-foundation-models-ga.md) | 갱신 페이지가 비어 있는 동안 심볼 페이지가 먼저 움직이는 같은 패턴 |
| [iOS 27 · Xcode 27 정식 출시](./2026-09-14-ios-27-ai-apis-shipped.md) | 그 문서가 적은 Siri AI 베타 조건(언어 확대, EU 미제공, 일일 한도)이 이번 공지에도 같은 문언으로 남아 있음 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 코딩 어시스턴트 목록에 GitHub Copilot 이 추가됨. macOS 요건은 26.6 이상 그대로 |

## 짚어야 할 것

- **전부 베타임.** `ToolIntent`·`GenerableEntity` 는 27.3 에 `beta: true` 로 들어 있고, `SiriEvaluations` 는 페이지의 모든 심볼이 베타임. 프리뷰 문서가 **"During the preview, App Intents API and Siri functionality may change"** 를 세 페이지에 반복해 박아 둠. 타입 이름과 시그니처를 전제로 코드를 넓게 깔면 안 되는 구간임
- **플랫폼 목록이 문서와 심볼에서 갈림.** 프리뷰 문서는 **iOS·iPadOS·macOS·visionOS** 넷으로 적는데, `ToolIntent`·`GenerableEntity` 의 가용성 배열에는 **Mac Catalyst** 가 추가로 들어 있음. Catalyst 앱에서 프리뷰가 실제로 도는지는 **확인할 수 없었음**
- **"부분집합"의 범위를 Apple 이 숫자로 갈라 적지 않았음.** 위 비교표는 Apple 쪽 제약(필드 둘, 1024자, 10,000자, 보조 파일 미지원)을 이 저장소의 명세 정리와 나란히 둔 것이고, **Apple 문서가 명세의 어느 버전을 가리키는지는 적지 않음.** 명세 자체에 버전 번호가 없다는 것은 그 문서가 이미 적어둔 사실임
- **Siri Integration Advisor 는 이름과 역할까지만 확인했음.** 프리뷰 문서가 한 문장으로 소개하고 Xcode 27.3 beta 릴리스 노트에는 없으며, 별도 문서 페이지를 찾지 못했음. 어떤 진단을 내는지, 결과를 CI 에서 받을 수 있는지는 **확인할 수 없었음**
- **인텐트를 툴로 올렸을 때의 비용과 한도를 확인할 수 없었음.** 서버측 Apple Intelligence 기능에 일일 한도가 걸린다는 것과 **확대 접근이 향후 유료**라는 것은 공지 각주에 있지만, 숫자도 과금 모형도 없음. 유료화 시점과 단가를 전제로 기획하면 안 됨
- **뉴스 항목 본문을 축조로 인용하지 못했음.** 이 문서가 공지에서 가져온 것은 날짜, 27.3 베타 네 플랫폼, App Intents 경로, 스킬 도입, Spotlight 의미 색인, 봄 제출 개방, Group Labs 일정, 그리고 각주의 베타·언어·EU·13세 미만·일일 한도 조건까지임. API 세부는 전부 위 문서 페이지에서 받았음
- **Spotlight 의미 색인 쪽은 공지 문언까지만임.** 스키마에 맞지 않는 엔티티도 Siri 가 꺼낼 수 있게 됐다고 공지가 적는데, 색인 조건과 `IndexedEntity` 채택 요건이 그래서 달라지는지는 이번에 대조하지 못했음
- **봄은 날짜가 아님.** 공지가 `spring` 으로만 적었고 월도 OS 버전도 없음. 27.3 정식에 맞춰 열린다는 서술은 **어디에도 없음**

## 유효기간

**2026-11-04 확인 기준이고 27.3 베타 1 시점임.** 날짜가 박힌 사실(공지일, 베타 빌드 번호)은 낡지 않지만 **프리뷰 전체가 가장 빨리 움직이는 구간**임. 다시 볼 때 확인할 것 넷. **App Store 제출 개방 시점이 날짜로 공지됐는지**(공지는 `spring` 까지임), **프리뷰가 외부 TestFlight 과 App Store 배포판으로 열렸는지**, **`ToolIntent`·`GenerableEntity`·`SiriEvaluations` 에서 베타 표시가 떨어졌는지**, 그리고 **App Intents updates 페이지에 2026-11 절이 생겼는지.** 넷째가 생기면 이 저장소가 세 문서에 걸쳐 추적해 온 갱신 페이지 공백이 메워지는 것임. 출발점은 [개발자 뉴스](https://developer.apple.com/news/)와 심볼 페이지 JSON 의 `platforms` 배열임. 앱 스킬 쪽은 **Apple 이 받는 프론트매터 필드가 늘었는지**가 명세와의 거리를 재는 가장 빠른 신호임.
