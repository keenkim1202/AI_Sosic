---
title: Apple Evaluations 프레임워크, judge 편향 대응을 Apple이 문서로 적은 것
source: https://developer.apple.com/documentation/evaluations
author: Apple
collected: 2026-09-22
tags: [apple, ios, eval, llm-judge, swift, foundation-models, agent]
---

출처: [Evaluations 프레임워크 문서](https://developer.apple.com/documentation/evaluations) · [Evaluating language model responses](https://developer.apple.com/documentation/evaluations/evaluating-language-model-responses) · [Designing effective evaluations](https://developer.apple.com/documentation/evaluations/designing-effective-evaluations) · [Designing effective model-judges](https://developer.apple.com/documentation/evaluations/designing-effective-model-judges) (2026-09-22 확인)

## 요약

[iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)에서 이름만 확인하고 "문서 본문은 확인하지 못했음"으로 남겨둔 그 프레임워크임. 이번에 본문을 읽었고, 결론은 예상보다 큼. **가용성이 iOS 27.0+ 이고 일곱 플랫폼 전부 `beta: false`**, 즉 베타 딱지 없이 나온 정식 API임. 내용은 API 레퍼런스가 아니라 **평가 방법론 문서**에 가깝고, 이 저장소가 [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md)과 [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md) 두 편에 걸쳐 정리해 둔 결론들이 Apple 1차 문서에 거의 그대로 적혀 있음. 특히 **모델 judge의 편향 네 가지를 이름 붙여 나열하고 각각의 완화책을 적어 둔 절**이 있음. 실행은 `@Test(.evaluates(...))` 트레이트로 **Swift Testing 안에서** 돌고, 결과는 Xcode Report navigator에 별도 리포트로 붙음.

## 구성, 네 조각

프레임워크가 요구하는 것은 `Evaluation` 프로토콜 하나이고 그 안에 네 조각이 들어감.

| 조각 | 무엇을 정의하는가 |
|---|---|
| 데이터셋 | `ModelSample(prompt:expected:)` 의 배열. `expected` 는 옵셔널 |
| subject | `subject(from:)`. **앱에서 부르는 것과 같은 방식으로** 기능을 호출하고 `ModelSubject` 로 출력과 transcript 를 함께 반환 |
| evaluators | 응답을 채점하는 것들. `Metric` 을 선언하고 `passing` / `failing` / `scoring(_:)` 중 하나를 돌려줌 |
| 집계 | `aggregateMetrics(using:)`. `computeMean`, `computeMedian`, `computeMaximum` 과 `group(_:)` 으로 묶음 |

subject 가 transcript 를 같이 반환한다는 게 이 API 의 설계 의도를 드러냄. 최종 출력만 보관하는 구조였으면 아래 경로 채점이 불가능함.

```swift
func subject(from sample: ModelSample<Int>) async throws -> ModelSubject<Int> {
    let session = LanguageModelSession()
    let response = try await session.respond(to: sample.prompt, generating: Int.self)
    return ModelSubject(
        value: response.content,
        transcript: session.transcript.structuredTranscript
    )
}
```

## 답이 아니라 경로를 채점하는 자리가 API 로 있음

[Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md) 3단계의 주장이 "최종 응답만 채점하면 에이전트가 망가진 시퀀스를 거쳐 정답에 도달해도 한 달 동안 아무도 모른다"였음. Evaluations 는 그 trajectory 층을 **샘플에 붙이는 기대값**으로 표현함.

```swift
ModelSample(
    prompt: "Count the letter 'r' in 'strawberry'.",
    expected: 3,
    expectations: TrajectoryExpectation(
        ordered: [
            ToolExpectation(
                "count_letters",
                arguments: [
                    .exact(argumentName: "letter", value: .string("r")),
                    .exact(argumentName: "word", value: .string("strawberry")),
                ]
            ),
        ]
    )
)
```

그리고 `ToolCallEvaluator(allPass:percentagePass:)` 를 evaluators 에 넣으면 호출 순서와 인자가 기대와 맞는지가 지표 둘로 나옴. `allPass` 는 전부 맞았는지, `percentagePass` 는 몇 퍼센트가 맞았는지임. 즉 **"맞는 도구에 맞는 인자"(tool parameter accuracy)를 직접 재는 평가자가 표준 라이브러리에 들어 있음.** 직접 만들던 층이 프레임워크로 내려온 것임.

문서가 붙인 수치도 이 층의 값어치를 보여줌. 같은 letter-counting 과제를 툴 없이 / 툴 붙여서 돌리면 mean Exact Match 가 **58%에서 100%로** 올라감. 툴을 붙였더니 좋아졌다는 얘기가 아니라, **같은 평가를 구성만 바꿔 돌려 요약 행을 나란히 놓고 비교하는 것**이 기본 워크플로라는 뜻으로 읽어야 함.

## judge 편향을 Apple 문서가 직접 나열함

이 저장소에서 가장 자주 인용된 수치가 같은 출력 세트에 판정자만 바꿔 **93.3% 대 39.5%**가 나왔다는 것임. Apple 문서는 그 수치를 인용하지는 않지만, 편향의 종류와 완화책을 표로 정리해 둠.

| 편향 | 내용 | 문서가 제시한 완화책 |
|---|---|---|
| Verbosity bias | 길이가 정보를 더하지 않아도 긴 응답에 높은 점수를 줌 | 간결함을 별도 기준으로 명시하거나, 길이가 점수를 보장하지 않는다고 지시 |
| Leniency bias | 보정이 없으면 점수가 가운데로 몰림 | **짝수 눈금**을 써서 중립값을 없앰, 모든 레벨(특히 양 극단)에 서술을 붙임, few-shot 예시로 낮은 점수가 정당한 경우를 보여줌 |
| Self-enhancement bias | 자기 계열 모델의 출력에 후한 점수를 줌 | **다른 모델을, 되도록 더 강한 모델을** judge 로 씀 |
| Position bias | pairwise 비교에서 먼저 읽은 쪽을 선호함 | 순서를 뒤집어 두 번 돌리고 **양쪽이 일치할 때만** 판정을 신뢰 |

self-enhancement 항목이 [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md)의 "생성하는 모델이랑 다른 계열의 판정자를 쓴다"와 같은 규칙임. 그 문서는 이걸 커뮤니티 경험칙으로 적었는데 여기서는 플랫폼 벤더가 자기 API 문서에 적은 것이 됨.

눈금 설계 권고도 구체적임. 안전·규정·포맷·사실 확인처럼 잘라 말할 수 있는 것은 **binary(pass/fail)** 가 신뢰도가 가장 높고, 톤·명료성 같은 주관적 품질은 **1~4 같은 작은 짝수 눈금**을 권함. 홀수 눈금(1~5)이 "안전한 중간값"을 만든다는 게 이유임. 한 번의 judge 호출에서 채점하는 차원은 **3~4개**가 적정이고 **4~5개를 넘으면 attention decay 로 품질이 떨어진다**고 적음. 안전과 품질처럼 성격이 다른 관심사는 평가자 호출을 나누라고 함.

레벨 서술은 느낌이 아니라 관찰 가능한 특징으로 쓰라고 함. "Excellent limerick" 이 아니라 "Perfect AABBA rhyme, strong meter, and surprising punchline" 쪽임. 판단 기준은 **서로 다른 리뷰어 둘이 경계에 동의할 수 있는가**임.

## judge 를 사람과 맞추는 절차가 숫자로 적혀 있음

여기가 이 문서에서 제일 실무적인 부분임. judge 를 믿기 전에 밟으라는 순서가 이럼.

1. 사람 채점자 **2~3명**이 같은 기준으로 응답 **20~50개**를 채점
2. 같은 세트에 모델 judge 를 돌림
3. 체계적으로 어긋나는 지점에서 기준·눈금·지시를 고침. 평가 단계나 few-shot 예시를 추가
4. 일치도를 **Cohen's Kappa** 로 잼. 원시 일치율(raw agreement)은 분포가 치우치면 오해를 부른다고 못박음
5. **모델과 사람의 일치도가 사람과 사람의 일치도에 닿을 때까지** 반복

사람끼리의 일치도를 상한으로 두는 것이 요점임. 주관적 기준에서 judge 에게 사람보다 나은 일관성을 기대하지 말라는 뜻이고, 이 저장소가 [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md)에서 "검증자를 믿기 전에 검증자를 테스트한다"로 적어둔 것의 수치 버전임.

수학·사실 회상·코드·도구 선택처럼 정답이 있는 과제에는 **reference-guided judging**, 즉 judge 에게 기대 답을 같이 주라고 함. 근거는 **answer contamination** 임. 평가 컨텍스트에 틀린 답이 들어 있으면 judge 가 그 틀린 추론을 자기 chain of thought 에 복사해 온다는 것.

## Swift Testing 안에서 돎

실행 경로가 별도 러너가 아니라 테스트임.

```swift
@Test(.evaluates(Self.evaluation))
func letterCounting() async throws {
    let result = EvaluationContext.current.result
    let score = result.aggregateValue(.mean(of: Self.evaluation.exactMatch))
    #expect(score > 0.8)
}
```

`EvaluationTrait` 가 데이터셋을 돌리고 평가자를 적용하고 지표를 집계한 다음 `EvaluationContext` 로 결과를 넘김. 결과는 뷰 셋으로 나옴. `summary`(실행 간 비교용 집계), `detailed`(샘플별 점수와 질의·응답·모든 지표값), `groupedSummary`(집계에서 만든 그룹별). 실행이 끝나면 Report navigator 의 테스트 실행 아래에 Evaluations 항목이 생김.

`detailed` 에서 실패한 샘플을 뽑아내는 것도 타입 있는 컬럼 디스크립터로 됨(`inputColumn`, `expectedColumn`, `[metric:]` 서브스크립트). 문서가 권하는 용도가 **실패 샘플을 뽑아 후속 데이터셋으로 만드는 것**임.

## 과적합과 데이터셋 운영

문서가 명시적으로 경고하는 것이 **overfitting** 임. 테스트 케이스 50개를 통과하게 다듬은 프롬프트가 51번째에서 깨지는 형태임. 방어는 **holdout set**, 즉 프롬프트 개발 중에는 절대 쓰지 않는 몫을 떼어두고 마일스톤마다만 돌리는 것. 판정 기준은 **개발용 점수는 계속 오르는데 holdout 점수가 정체하거나 내려가면 과적합**임.

성장 일정도 숫자로 제시함.

| 시점 | 규모 |
|---|---|
| 첫 주 | 평가자 **2~3개**, 핵심 기능을 대표하는 golden set **10~20 샘플** |
| 첫 달 | 실패 모드를 발견할 때마다 challenge set 추가, 주관적 차원에 model-judge 도입, 데이터셋 **30~50 샘플** |
| 이후 | 사용자 상호작용을 동의 아래 편입, `SampleGenerator` 로 볼륨 테스트, 신호를 못 주는 평가자는 제거 |

[Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md)이 인용한 "집계 수치를 믿기 전에 최소 500케이스"와 나란히 놓으면 간극이 큰데, 그쪽 수치는 원문이 출처를 밝히지 않은 것이고 그 문서 자신이 보편 하한으로 쓰지 말라고 적어 둠. 여기 10~20은 **시작 규모**이고 500은 **집계를 믿는 기준**이라 층위가 다름. 둘을 같은 축에 놓고 비교하면 안 됨.

지표 자체를 의심하라는 경고도 있음.

> "Always verify that your metric is measuring what you think it is. A perfect score can be misleading: a bias evaluation that returns zero bias might mean your model is unbiased, or it might mean your model isn't answering the questions at all."

[자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md)의 "점수는 오르고 동작은 나빠진 에이전트"와 같은 실패임.

## 언제 안 쓰는 게 맞는가

- **온디바이스 모델을 judge 로 쓰는 구성은 self-enhancement 방어와 충돌함.** 문서는 judge 로 더 강한 다른 모델을 권하는데, 앱이 온디바이스 AFM 을 쓰고 judge 도 `SystemLanguageModel.default` 로 두면 생성과 판정이 같은 계열임. PCC 나 `LanguageModel` 프로토콜로 붙인 외부 모델을 judge 쪽에 두는 편이 권고에 맞음. 다만 그러면 평가가 네트워크와 한도에 묶임
- **정답이 있는 것에 judge 를 쓰지 말 것.** 포맷·스키마·정확 일치는 `Evaluator` 로 코드가 채점함. 문서 표현으로 코드 평가자는 "instant, free, and perfectly reproducible" 임
- 평가가 테스트 타깃 안에 있으므로 **CI 러닝타임에 그대로 얹힘.** model-judge 는 호출 비용과 시간이 붙음. 전체 스위트를 매 변경마다 돌리라는 문서 권고와 러닝타임은 직접 부딪히는 자리이고, 문서는 이 긴장을 다루지 않음

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md) | judge 편향과 경로 채점. 그쪽이 커뮤니티 경험칙으로 적은 것을 Apple 이 자기 API 문서에 적음 |
| [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md) | 판정 기준을 루프 바깥에 두는 문제. `Evaluation` 이 데이터셋·평가자·집계를 한 타입에 묶어 버전 관리 대상으로 만듦 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 그 문서가 이름만 확인하고 남겨둔 항목. 가용성이 **베타가 아님**을 이번에 확인함 |
| [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md) | subject 가 여는 `LanguageModelSession` 이 그쪽 컨텍스트 제약을 그대로 받음. 툴을 붙인 구성을 평가하면 툴 정의가 창을 먹는 문제가 평가 대상 안으로 들어옴 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | 근거를 모델 바깥에 두라는 원칙. 자기 검증 지시를 빼는 것과 외부 평가를 두는 것은 같은 방향임 |

## 짚어야 할 것

- **문서만 읽었고 직접 돌려보지 않았음.** 시그니처는 문서의 코드 샘플을 그대로 옮긴 것이고, 컴파일 여부는 확인하지 못했음
- **58%에서 100%는 Apple 의 예제 데이터셋 5샘플에 대한 수치임.** 문서가 제시한 letter-counting 예제의 결과이고 일반적인 툴 도입 효과가 아님
- **`SampleGenerator` 로 합성 데이터셋을 만드는 문서는 읽지 못했음.** `generating-synthetic-datasets` 경로가 404 였고 다른 경로를 찾지 못했음. 모델이 만든 샘플로 모델을 평가할 때 생기는 문제를 Apple 이 어떻게 다루는지는 **확인할 수 없었음**
- **평가 비용과 러닝타임에 대한 서술이 없음.** model-judge 를 PCC 나 외부 모델로 돌릴 때 일일 한도에 어떻게 잡히는지도 이 문서들에는 없음
- Cohen's Kappa 를 계산하는 API 가 프레임워크에 있는지는 심볼 목록에서 확인되지 않음. 문서는 지표로 쓰라고만 적음
- 문서에 나열된 편향 완화책은 **모두 설정 권고이고, 프레임워크가 강제하는 것은 없음.** 짝수 눈금을 쓰지 않아도 `ScoringScale.numeric` 은 그대로 동작함

## 유효기간

**2026-09-22 확인 기준이고 iOS 27.0 정식 문서임.** 방법론 절(편향 목록, 보정 절차, 과적합 방어)은 도구 버전과 무관하게 오래 감. 반면 타입 이름과 시그니처는 27.1 이후 갱신될 수 있고, 특히 `SampleGenerator` 쪽은 이번에 본문을 못 봐서 비어 있음. 다시 볼 때는 [프레임워크 문서](https://developer.apple.com/documentation/evaluations)의 Datasets 절에서 합성 데이터셋 문서가 살아났는지부터 확인할 것.
