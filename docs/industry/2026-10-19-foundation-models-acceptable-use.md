---
title: Foundation Models 허용 사용 요건, 계약서가 앱에 가드레일을 요구하는 지점
source: https://developer.apple.com/apple-intelligence/acceptable-use-requirements-for-the-foundation-models-framework
author: Apple
collected: 2026-10-19
tags: [apple, ios, foundation-models, app-store, compliance, on-device-llm]
---

출처: [Acceptable use requirements for the Foundation Models framework](https://developer.apple.com/apple-intelligence/acceptable-use-requirements-for-the-foundation-models-framework) · [Apple Developer Program License Agreement](https://developer.apple.com/support/terms/apple-developer-program-license-agreement/) (3.3.11(A), 3.3.8(j), 3.2, 1.2 정의 전문 확인) · [Updated Apple Developer Program License Agreement now available](https://developer.apple.com/news/?id=17mwd36z) (2026-10-13) (전부 **2026-10-19 확인**)

## 요약

이 저장소의 [App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) 문서가 **"DPLA 3.3.11 계열은 공지 요약만 확인했고 계약서 전문은 못 읽었음"**으로 남겨둔 자리를 메운 것임. 계약서 본문을 열어 보니 **3.3.11(A)는 한 문장**인데 요구가 둘임. 허용 사용 요건을 **따르는 것**, 그리고 그것을 **"받치는 합리적 가드레일을 유지하는 것"**. 후자는 정책 준수가 아니라 **앱이 만들어야 하는 구현물**임. 요건 페이지 자체는 금지 항목 **20개** 목록이고, 그중 여럿은 악용이 아니라 **평범한 앱 기획이 그대로 밟는 것들**임(사용자 의존 유발, 사람 감독 없는 고위험 도메인 판단, 생체 데이터로 민감 속성 추론). 여기에 **3.2 의 여덟 번째 항목(h)**이 **Apple 모델 출력으로 다른 AI 모델을 학습·파인튜닝·개선하는 것을 금지**함. 계약은 **2026-10-13 에 개정**됐고 AI 쪽 변경은 **3.2 보안 요건**과 **3.3.8(j) Live Assistance API** 둘임.

## 3.3.11(A) 전문, 한 문장에 요구가 둘

계약서 3.3.11 은 `A. Foundation Models Framework`, `B. SiriKit`, `C. App Intents` 세 하위 조항으로 되어 있고 (A) 본문은 이것 하나임.

> "By accessing, prompting, or otherwise using the Foundation Models Framework, or a model accessed by the Foundation Models Framework, You agree to follow, and to maintain reasonable guardrails supporting, the Foundation Models Framework Acceptable Use Requirements."

읽을 때 갈라야 할 것이 `follow` 와 `maintain reasonable guardrails supporting` 임.

| 요구 | 성격 | 앱에 생기는 작업 |
|---|---|---|
| `follow` | 금지 항목을 위반하지 않음 | 기획 단계의 범위 결정. 코드가 없어도 지킬 수 있음 |
| `maintain reasonable guardrails supporting` | 요건을 **받치는** 장치를 **유지**함 | 입출력 필터, 거부 경로, 고위험 질의 차단, 그 장치가 계속 도는지 확인하는 수단 |

두 번째가 이 조항의 무게임. **금지 항목을 피하는 것으로 끝나지 않고, 사용자가 그 방향으로 몰고 갈 때 막는 쪽을 앱이 들고 있어야 한다**는 뜻으로 읽힘. 온디바이스 모델만 쓰고 서버가 없는 앱에도 똑같이 걸림. 서버가 없으면 필터를 둘 자리도 클라이언트뿐임.

⚠️ `reasonable` 의 기준은 **계약서도 요건 페이지도 정의하지 않음.** 어느 수준이 합리적인지, Apple 내장 가드레일에 기대는 것으로 충족되는지는 **확인할 수 없었음.** 요건 목록에 "프레임워크에 통합된 안전 정책·가드레일·제약을 우회하지 말 것"이 따로 들어 있는 것을 보면 Apple 쪽 가드레일은 전제이고 개발자 쪽 가드레일은 그 위에 얹는 것으로 보이지만, 그 해석을 뒷받침하는 문언은 없음.

## 금지 항목 20개 중 앱 설계가 밟기 쉬운 것

요건 페이지는 `Prohibited uses` 한 절이고, "다음과 같은 방식으로 프레임워크를 쓰거나 프롬프트하거나 노출하지 말 것, 남이 그렇게 하도록 권하거나 가능하게 하지도 말 것"으로 묶음. **`encourage or enable others to do the same` 이 붙어 있어서 사용자가 하게 만드는 구조까지 범위에 들어감.**

악용 금지로 당연한 항목(아동 착취, 폭력, 무기, 혐오, 사기)은 여기서 생략함. 기능 기획이 의도 없이 밟는 쪽만 모았음.

| 금지 문언 | 밟기 쉬운 기획 |
|---|---|
| "Enables dependency or spiraling user interactions detrimental to a user's mental health" | 체류 시간을 지표로 잡은 대화형 동반자·일기·상담 톤 기능. **리텐션 최적화가 그대로 이 문언에 닿음** |
| "Provides inaccurate or dangerous outputs, or makes decisions without human supervision that have a material impact on individual rights, in high-risk domains, including in employment, medical, legal, or finance" | 건강·법률·금융 질의에 모델 답을 그대로 띄우는 화면. **사람 감독 없이 결정**하는 구조가 핵심 조건임 |
| "Classifies individuals based on biometric data to infer sensitive attributes" | 사진·음성에서 속성을 추정하는 기능. Vision 툴을 모델에 붙인 경로도 포함됨 |
| "Evaluates or classifies individuals based on social behavior or personal traits that leads to detrimental or unfavorable treatment" | 사용자 점수·등급 매기기, 커뮤니티 신뢰도 자동 산정 |
| "Creates or promotes adult content such as erotic chat, pornography and content meant to arouse sexual excitement" | 캐릭터 대화 앱에서 사용자가 끌고 가는 방향. **`enable others` 가 걸리는 전형적 자리** |
| "Circumvents any safety policies, guardrails, or restrictions integrated into the Foundation Models framework" | 거부를 피하려고 프롬프트를 바꿔 재시도하는 자동 재시도 로직 |
| "Generates, reverse engineers, summarizes, translates or otherwise reproduces the Apple artificial intelligence model training data" / "Generates citations to or otherwise identifies the Apple artificial intelligence model training data" | 모델이 무엇을 봤는지 캐보는 평가·프로빙. 학습 데이터 추정은 **평가 작업에서 해 볼 만한 일로 보이는데 금지 항목임** |
| "Removes watermarks or other content credentials intended to identify intellectual property rights or to label content as AI-generated" | 생성 이미지 후처리 파이프라인에서 메타데이터를 떨어뜨리는 것 |
| "Shows Apple or its products in a false or derogatory light" | 모델 비교·벤치마크 화면을 앱 안에 담을 때 |

아래 둘은 성격이 달라서 따로 떼어 둠.

- "Engages in assessing the risk of an individual committing a criminal offense, or for official use in connection with law enforcement or criminal justice": 범위가 넓음. 업무용 앱이 수사·형사 절차에 쓰이면 용도 자체로 걸림
- "Infringes or violates the intellectual property, publicity, or privacy rights of another (including Apple)": `including Apple` 이 붙은 것이 눈에 걸림

**재시도 로직 항목이 실무에서 제일 조용하게 걸리는 자리임.** 온디바이스 모델이 거부하거나 guided generation 이 실패했을 때 프롬프트를 바꿔 다시 부르는 코드는 품질 개선으로 들어가는데, 그 재시도가 거부를 뚫는 방향이면 문언상 우회에 해당함. 재시도를 넣을 때 **왜 실패했는지로 분기**하지 않으면 구분이 안 됨.

## 3.2(h), Apple 모델 출력으로 다른 모델을 키우지 말 것

3.2 의 여덟 번째 항목임. 2026-10-13 개정에서 **3.2 전체가 "보안 및 관련 요건 갱신"으로 지목된 절**이므로 문언을 그대로 옮김.

> "You will not (1) use or prompt Apple Software or Services in a manner that You know or reasonably should know generates, or is intended to generate, content that is unlawful, harmful or infringes, misappropriates or violates the rights of Apple or others, (2) use output generated from an Apple model to train, fine-tune, or improve another artificial intelligence model, or (3) programmatically access or use Apple models through Apple Software or Services except as expressly permitted. Furthermore, You warrant You have all necessary rights and licenses to use the information that You input to Apple Software or Services with such Software and Services."

(2)가 이 저장소 문서 여러 개와 바로 맞물림. **온디바이스 모델이나 PCC 로 합성 데이터를 만들어 자체 모델을 학습시키는 경로가 막힘.** 무료 구간이라서 단가 계산이 사라졌다는 [Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md)의 서술은 **추론 비용 얘기지 데이터 생산 라이선스 얘기가 아님**이 여기서 확인됨.

⚠️ **`improve another artificial intelligence model` 의 경계는 확인할 수 없었음.** 학습과 파인튜닝은 명확한데, 자체 모델의 출력을 Apple 모델 출력과 비교해 프롬프트나 설정을 고치는 것이 `improve` 에 들어가는지는 문언이 가르지 않음. 평가 레이블을 Apple 모델로 만드는 구성이 여기 걸릴 수 있으니, 그런 파이프라인을 잡고 있으면 문의로 확인하는 편이 맞음.

(3)의 `programmatically access ... except as expressly permitted` 는 프레임워크를 통하지 않는 접근 경로를 막는 문언으로 읽힘. 프레임워크 API 로 호출하는 것 자체는 명시적 허용 범위임.

## 2026-10-13 개정에서 AI 쪽에 걸린 것

공지가 나열한 변경 중 AI·ML 과 닿는 것은 둘임. 나머지(구독 그룹 구매, Apple Business·School Manager 볼륨 구매, 세무 정보 보고 제한, HomeView 위젯·CarPlay)는 이 저장소 범위가 아님.

| 조항 | 변경 |
|---|---|
| 3.2 | "Updated security and related requirements for using Apple's software and services". 위 (h) 항목이 이 절 안에 있음 |
| 정의, 3.3.8(j) | Live Assistance API 사용 조건 신설 |

**이 개정에서 AI 관련 조항 번호가 밀리지는 않았음.** 3.3.11(A)와 3.2 의 (h)는 위에 옮긴 문언 그대로임. 즉 [App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) 문서가 2026-06-08 개정을 근거로 적어둔 **조항 번호가 이번 개정 뒤에도 유효함.**

3.3.8(j)는 FaceTime 통화에 붙는 Live Assistance API 쪽이라 이 문서의 주제(Foundation Models 라이선스 층)와 갈림. 여기서는 **같은 개정에 들어온 또 하나의 AI 인접 조항**이라는 것까지만 적어둠.

## 어디까지가 "프레임워크가 접근하는 모델"인가

iOS 27 에서 `LanguageModel` 프로토콜이 열려 Claude·Gemini·자체 모델이 같은 Swift API 뒤에 들어옴([iOS 27 · Xcode 27 정식 출시](./2026-09-14-ios-27-ai-apis-shipped.md)). 그러면 3.3.11(A)의 `a model accessed by the Foundation Models Framework` 가 **서드파티 백엔드까지 가리키는지**가 실무 질문이 됨. 요건 20개를 Claude 백엔드 세션에도 적용해야 하는지가 갈리기 때문임.

계약서 정의는 이렇게 되어 있음.

> "Foundation Models Framework" means the Documented APIs that enable You to access, prompt, or otherwise use the Foundation Models framework, including the model accessed by the framework, in any manner, including through an Adapter.

> "Adapter" means a model adapter that fine-tunes the model accessed by the Foundation Models framework for a purpose permitted by the Foundation Models Framework Acceptable Use Requirements.

**단수 `the model` 로 적혀 있고 그것이 Apple 모델인지 아닌지를 밝히지 않음.** 범위를 좁히는 문언도, 넓히는 문언도 없음. **확인할 수 없었음.** 다만 `in any manner` 가 붙어 있어서 좁게 읽는 쪽이 근거가 약함. 실무에서는 **백엔드를 갈아끼워도 요건 20개는 그대로 적용된다고 가정하고 설계하는 편이 안전함.** 어차피 같은 항목 대부분이 App Review Guidelines 1.2 와 5.1.2 로도 걸림.

`Adapter` 정의가 계약서에 살아 있는 것은 별건으로 값어치가 있음. [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)이 **"온디바이스 파인튜닝(어댑터)은 문서 목차에서 확인되지 않았음"**으로 유보해 둔 항목인데, **계약서는 어댑터를 정의하고 그 용도를 요건 페이지로 묶어 둠.** 기능이 어떤 API 로 노출되는지는 여전히 프레임워크 문서로 확인해야 하지만, 계약상 개념으로는 유지되고 있음.

## 실무 순서

[App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) 체크리스트의 7번("DPLA 3.3.11(A), 3.2(h) 본문 확인")을 펼치면 이렇게 됨.

1. 기능 명세를 금지 항목 20개에 대고 읽음. 의도가 아니라 **사용자가 끌고 갈 수 있는 방향**으로 읽을 것. `enable others` 가 붙어 있음
2. 의존 유발과 고위험 도메인 둘은 **지표 설계에서 갈림.** 체류 시간을 목표로 잡았는지, 건강·법률·금융 답을 사람 확인 없이 띄우는지
3. `maintain reasonable guardrails supporting` 에 대응하는 **구현물을 하나라도 지목할 수 있는지** 확인. 못 하면 조항의 절반을 안 한 것임
4. 거부·실패 시 재시도 로직이 **실패 원인으로 분기하는지** 확인. 거부를 뚫는 재시도는 우회 항목에 닿음
5. Apple 모델 출력을 **다른 모델의 학습·평가 데이터로 쓰는 경로가 있는지** 전부 찾음 (3.2 의 (h))
6. 요건 페이지가 `as may be updated from time-to-time` 으로 묶여 있으므로, **릴리스마다 다시 열어보는 일을 일정에 넣음**

6번이 이 문서에서 제일 오래 남는 부분임. 계약이 가리키는 것은 URL 이고, **그 URL 의 내용은 계약 개정 없이 바뀜.** 심사 지침처럼 개정 공지가 따라오지 않으므로 바뀌어도 알 방법이 없음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | 그 문서가 공지 요약만으로 남겨둔 3.3.11(A)·3.2(h)의 **전문이 이 문서에 있음.** 심사 지침 대 계약서로 층이 갈림 |
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 어댑터 유보 항목. 계약서가 `Adapter` 를 정의하고 용도를 요건 페이지로 묶어 둠 |
| [Apple의 2026 AI 플랫폼](./2026-06-08-apple-ai-platform-for-ios.md) | PCC 무료 구간. 비용이 0 이라는 것이 **데이터 생산 라이선스까지 주는 것은 아님**이 3.2 의 (h)로 확인됨 |
| [iOS 27 · Xcode 27 정식 출시](./2026-09-14-ios-27-ai-apis-shipped.md) | 백엔드 교체 가능해진 것이 요건 적용 범위 질문을 만듦. Vision 툴을 모델에 붙이는 경로는 생체 추론 항목에 닿음 |

## 짚어야 할 것

- **요건 페이지에 발행일·갱신일 표시가 없음.** 금지 항목 20개는 **2026-10-19 에 받은 내용**이고, 이 목록이 언제부터 이 상태였는지는 확인할 수 없었음. 계약서가 `as may be updated from time-to-time` 으로 묶어 둔 문서라 이 점이 구조적인 한계임
- **계약서 개정판 확인 경로가 깔끔하지 않았음.** [Agreements and Guidelines](https://developer.apple.com/support/terms/) 색인은 DPLA 의 `Last update` 를 **2026-08-18** 로 표시하는데, 2026-10-13 공지는 그 뒤의 개정을 알림. 본문에서 공지가 신설이라고 적은 **3.3.8(j)가 실제로 읽혔으므로 이 문서가 인용한 문언은 개정 후 영문판으로 판단함.** 다만 색인 날짜와 어긋나므로 확정이 아님. 공지는 번역본이 한 달 내 올라온다고 적어둠
- **`reasonable guardrails` 의 수준, `improve another artificial intelligence model` 의 경계, 서드파티 백엔드에 요건이 적용되는지 셋 다 문언으로 갈리지 않음.** 이 문서는 세 지점을 유보로 남긴 것이고 답을 만들지 않았음
- **금지 항목 수 20 은 이 문서가 받은 페이지의 불릿을 센 것임.** Apple 이 번호를 붙여둔 목록이 아님
- 계약서는 660KB 분량의 단일 페이지라 **인용한 네 곳(1.2 정의, 3.2 의 (h), 3.3.8(j), 3.3.11(A)) 외에는 읽지 않았음.** AI 와 닿는 조항이 다른 절에 더 있을 수 있음
- 이 문서는 **문언 정리임.** 어떤 가드레일이 심사를 통과하는지, 위반이 실제로 어떻게 적발되는지는 다루지 않음

## 유효기간

**2026-10-19 확인 기준**임. 인용한 계약서 문언은 2026-10-13 개정판으로 판단한 것이고 다음 개정에서 번호가 밀릴 수 있음. 다시 볼 때는 [개발자 뉴스](https://developer.apple.com/news/)에서 DPLA 개정 공지를 찾아 변경 목록에 **3.2·3.3.11 이 들어 있는지**부터 보면 됨. **요건 페이지 쪽은 공지가 붙지 않으므로 금지 항목 수와 문언을 직접 대조해야 함.** 20개에서 늘거나 줄었는지가 제일 빠른 신호임.
