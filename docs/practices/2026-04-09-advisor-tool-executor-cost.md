---
title: Advisor tool, 싼 모델에 비싼 조언자를 붙이는 값과 그 청구서가 안 보이는 자리
source: https://claude.com/resources/articles/the-advisor-strategy
author: Anthropic
published: 2026-04-09
collected: 2026-11-12
tags: [claude, llm-cost, coding-agent, routing, tooling, prompt-caching, thinking]
---

출처: [The advisor strategy](https://claude.com/resources/articles/the-advisor-strategy) (2026-04-09) · [Advisor tool 문서](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) · [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api) (문서와 릴리스 노트는 **2026-11-12 확인**)

## 요약

**싼 모델이 일을 하고, 막히는 자리에서만 비싼 모델에게 물어본다.** 이 패턴을 하네스에 직접 짜는 대신 서버 툴 하나로 받는 것이 **advisor tool**임. executor 가 `advisor` 를 부르면 Anthropic 서버가 **대화 전체를 advisor 모델에 따로 태워** 계획이나 궤도 수정을 받아오고, executor 는 그걸 읽고 계속함. 전부 `/v1/messages` 요청 하나 안에서 일어남. 베타이고 헤더는 `advisor-tool-2026-03-01`, 툴 타입은 `advisor_20260301` 임.

발표가 든 수치는 **Sonnet 4.6 executor + Opus 4.6 advisor** 기준으로 SWE-bench Multilingual 점수 **2.7 포인트 상승**에 에이전트 작업당 비용 **11.9% 감소**, BrowseComp 에서 **Haiku 4.5 가 19.7% 에서 41.2% 로** 올라감. 다만 그 Haiku 조합은 Sonnet 단독보다 점수가 **29% 낮고 비용은 85% 쌈**임. 즉 이 툴이 파는 것은 "더 똑똑해짐"이 아니라 **점수와 비용 사이의 곡선을 한 칸 옮기는 것**임.

이 저장소 입장에서 중요한 것은 두 가지임. 첫째, **advisor 가 쓴 토큰은 응답 최상위 `usage` 에 없음.** `usage.iterations[]` 의 `advisor_message` 항목에만 있음. [서버측 compaction](2026-09-20-claude-api-compaction.md)과 [OpenTelemetry 내보내기](2026-10-14-claude-api-opentelemetry.md)에서 이미 두 번 나온 **"최상위 usage 는 청구서가 아니다"**가 세 번째로 반복되는 자리임. 둘째, **모델을 올리면 쓸 수 있는 조합이 좁아짐.** Fable 5.2 를 executor 로 쓰면 advisor 로 받아지는 것이 Fable 5.2 와 Mythos 5.2 둘뿐이라, 싼 executor 에 비싼 advisor 를 붙인다는 전제 자체가 그 줄에서는 성립하지 않음.

## 붙이는 모양

`tools` 에 한 항목을 더하는 것이 전부임. advisor 는 **파라미터를 받지 않음**. `server_tool_use` 의 `input` 은 항상 비어 있고, 넘어가는 것은 대화 전체임.

```python
tools = [
    {
        "type": "advisor_20260301",
        "name": "advisor",
        "model": "claude-opus-5-5",
        "max_uses": 3,
        "max_tokens": 2048,
    },
    # ... 나머지 툴
]
```

| 필드 | 기본값 | 걸리는 것 |
|---|---|---|
| `model` | 필수 | advisor 모델 ID. **그 모델 요율로 따로 청구됨** |
| `max_uses` | 무제한 | **턴당** 호출 수. 넘으면 결과가 `max_uses_exceeded` 로 옴 |
| `max_tokens` | advisor 모델의 출력 상한 | 호출당 thinking + 텍스트 합계. **최소 1024**, advisor 출력 상한을 넘기면 400 |
| `caching` | `null`(꺼짐) | `{"type": "ephemeral", "ttl": "5m"}` 또는 `"1h"`. 중단점 표시가 아니라 **on/off 스위치** |

결과는 두 모양으로 갈림. `advisor_result` 는 평문이고, `advisor_redacted_result` 는 `encrypted_content` 라 **클라이언트가 못 읽음**. executor 만 읽음. Opus 5.5·Opus 5·Sonnet 5.5·Haiku 5.5·Fable/Mythos 5.x 가 전부 암호화 쪽이므로, **요즘 조합에서는 조언 본문을 로그로 남길 수 없다고 보는 게 맞음.** 평문으로 오는 것은 Opus 4.8 같은 구형 advisor 임.

advisor 쪽 실패는 요청을 깨지 않음. `advisor_tool_result_error` 로 `max_uses_exceeded`·`too_many_requests`·`overloaded`·`prompt_too_long`·`execution_time_exceeded`·`model_not_found`·`unavailable` 중 하나가 오고 **executor 는 조언 없이 그냥 계속함.** advisor 의 레이트 리밋은 툴 결과 안에 들어오고, executor 의 레이트 리밋만 HTTP 429 로 나옴. 즉 **advisor 가 조용히 안 돌고 있는 상태를 429 로는 못 잡음.** 결과 블록을 직접 세야 함.

advisor 는 툴도 컨텍스트 관리도 없이 돌고, 출력에 thinking 블록을 담지 않음.

## 발표가 잰 조합과 지금 쓸 수 있는 조합

발표 수치는 **2026-04-09 시점**이고 전부 **Opus 4.6 을 advisor 로** 쓴 것임.

| 벤치마크 | 구성 | 결과 | 표본 |
|---|---|---|---|
| SWE-bench Multilingual | Sonnet 4.6 executor + Opus 4.6 | **+2.7 포인트**, 작업당 비용 **-11.9%** | 9개 언어 300문제 × 5회 평균 |
| BrowseComp | Haiku 4.5 executor + Opus 4.6 | **19.7% → 41.2%** | 1,266문제, 문제당 1회 |
| BrowseComp | 위 조합 대 Sonnet 4.6 단독 | 점수 **29% 낮음**, 비용 **85% 쌈** | 위와 같음 |
| Terminal-Bench 2.0 | Sonnet 4.6 executor + Opus 4.6 | 각주만 있고 수치는 이미지라 **읽지 못했음** | 89태스크 × 5회 평균 |

조건이 대칭이 아님. 각주에 따르면 SWE-bench 의 **Sonnet 4.6 단독은 adaptive thinking 을 켰고, advisor 조합은 thinking 을 끈 채 권장 시스템 프롬프트를 붙였음.** 즉 이 2.7 포인트는 advisor 단독 효과가 아니라 **프롬프트 교체까지 포함한 묶음의 효과**임.

그리고 **그 조합을 지금 모델로 그대로 올릴 수 없음.** 문서의 호환 규칙은 advisor 가 **Sonnet 4.6 이상이면서 executor 이상으로 유능해야 한다**는 것인데, 실제 허용 목록은 세대별로 끊겨 있음.

| executor | 받아지는 advisor | 읽는 법 |
|---|---|---|
| Fable 5.2 · Mythos 5.2 | **Fable 5.2, Mythos 5.2 둘뿐** | 싼 executor + 비싼 advisor 구도가 여기서 사라짐 |
| Opus 5.5 | Mythos/Fable 5.2·5.1·5, Opus 5.5, Opus 5 | 4.x 가 전부 빠짐 |
| Sonnet 5.5 | 위 목록 + Sonnet 5.5 | **4.x 가 하나도 없음.** Sonnet 5 도 없음 |
| Haiku 5.5 | 위 목록 + Opus 4.8, Opus 4.7, Sonnet 5, Haiku 5.5 | **Opus 4.6 은 빠짐** |
| Haiku 4.5 | 4.6 세대까지 전부 | 발표가 잰 Haiku 4.5 + Opus 4.6 은 **아직 유효한 조합** |

읽어야 할 것은 이럼. **executor 를 Haiku 4.5 에서 Haiku 5.5 로 올리는 순간 발표가 측정한 advisor(Opus 4.6)가 400 이 됨.** 잘못된 조합은 `400 invalid_request_error` 로 어느 조합이 안 되는지를 이름으로 돌려줌. 이 저장소의 [Claude Sonnet 5.5](../models/2026-09-28-claude-sonnet-5-5.md) 문서가 적어 둔 "Sonnet 5.5 executor 에 Opus 4.8·4.7·Sonnet 5 는 400" 이 이 표의 한 줄이었음.

플랫폼은 Claude API, Claude Platform on AWS, Claude Models on Google Cloud (Anthropic managed) 임. **Amazon Bedrock 과 Microsoft Foundry 는 목록에 없음.**

## 조언자 비용이 안 보이는 자리

최상위 `usage` 의 `input_tokens`·`output_tokens`·`cache_read_input_tokens` 는 **executor 토큰만** 담음. advisor 몫은 `usage.iterations[]` 안에 `type: "advisor_message"` 인 항목으로 들어오고 그 항목에만 `model` 이 붙음. `type: "message"` 가 executor 쪽임.

따라오는 함정이 셋임.

- **`count_tokens` 는 executor 의 첫 iteration 만 돌려줌.** advisor 비용을 미리 재려면 advisor 모델로 따로 한 번 더 불러야 함
- **Priority Tier 는 모델별로 걸림.** executor 로 약정을 잡아 뒀어도 advisor 는 그 약정에 안 들어감
- 최상위 `max_tokens` 는 **advisor 출력을 전혀 제한하지 않음.** 작업 예산에서 빠지지도 않음

[Usage and Cost Analytics API](2026-10-29-usage-cost-analytics-api.md) 쪽에서 보면 advisor 호출은 그냥 **advisor 모델의 사용량 행**으로 나타남. 즉 "Sonnet 으로 내렸는데 Opus 청구가 왜 남아 있지"가 되는 구조라, 모델별 지출만 보고 라우팅이 실패했다고 읽으면 안 되는 자리임.

## 조언 길이가 최대 비용 요인임

문서가 **advisor 출력이 advisor 비용의 가장 큰 동인**이라고 직접 적음. 가벼운 작업에서 조언은 보통 **텍스트 400~700 토큰**, thinking 까지 합쳐 **1,400~1,800 토큰** 정도인데, 어려운 추론 과제에서는 상한을 안 걸면 평균이 **4,200~5,900 토큰**까지 감.

| `max_tokens` | 평균 조언 출력 | 잘린 호출 비율 |
|---|---|---|
| 설정 안 함 | 약 **4,200~5,900** | 해당 없음 |
| **2048**(권장 시작점) | 약 **630~840** | 약 **0%** |
| 1024(최소값) | 약 **370~480** | 약 **10%** |

**표본이 구성당 40회**이고 정확도 차이는 그 표본 크기에서 노이즈 범위였다고 문서가 같이 적어 둠. 즉 "7배 줄었는데 품질은 그대로"를 일반 법칙으로 읽으면 안 되고, 자기 워크로드에서 다시 재야 하는 종류임.

상한에 걸리면 결과 블록에 `stop_reason: "max_tokens"` 가 붙고 조언 텍스트 뒤에 `[Advisor output truncated at max_tokens=2048.]` 가 덧붙음. 이 둘은 **툴 정의에 `max_tokens` 를 준 경우에만** 나타남.

부드러운 쪽 방법도 있음. **advisor 는 system 프롬프트와 user 메시지를 executor 작업에 대한 인용 맥락으로 받으므로, advisor 에게 직접 말을 거는 문장이 훨씬 잘 먹힌다**는 것이 문서의 관찰임. 가장 효과적이었던 위치가 user 메시지 한 줄이고, 소프트 제약이라 실제 상한의 **약 80%** 를 요구하라고 권함. 재미있는 것은 이 줄이 **조언 호출 빈도를 오히려 늘렸는데도 총비용은 내려갔다**는 것임. 짧은 조언을 여러 번이 긴 조언 한 번보다 쌌다는 뜻임.

## 호출 시점을 프롬프트로 옮길 때의 교환

문서가 모델별로 다른 시스템 프롬프트 블록을 주는데, **셋 다 공짜가 아님**을 같은 자리에 적어 둠.

- **Haiku 코딩용 대체 블록**: 내부 코딩 벤치마크에서 통과율 **약 7.5 포인트 상승**. 그런데 같은 블록이 브라우즈 이해 벤치마크(n=1,266)에서 **약 4 포인트 하락**. 코딩과 조회가 섞인 워크로드면 쓰지 말거나 워크로드 신호로 분기하라고 함
- **Opus executor 용 체크포인트 블록**: 호출을 빠뜨리던 과제에서 **7~10 포인트 상승**, 대신 계획이 필요 없는 과제에서 과호출. 혼합 워크로드에서 **순효과는 대략 0**. 문서가 **"기본값으로 넣지 말라"**고 명시함
- **effort 와의 짝**: Sonnet executor 를 **medium effort** 로 내리고 Opus advisor 를 붙이면 Sonnet 기본 effort 수준의 지능을 더 싸게 낸다고 적음. 최대 지능이 목표면 executor 를 기본 effort 로 두라고 함

이 세 줄이 [자기개선 에이전트 루프의 7가지 규칙](2026-08-10-self-improving-agent-loops.md)이 경고한 자리와 정확히 같음. **한 벤치마크의 점수를 올린 프롬프트 변경이 다른 축에서 깎인다는 것을 벤더가 자기 문서에 숫자로 적어 둔 사례**임. 이 저장소에서 이런 식으로 자기 손해를 같이 적어 둔 1차 출처는 흔하지 않음.

## 캐시와 히스토리에서 밟는 것

`caching` 을 켜면 호출마다 캐시 항목을 쓰고 다음 호출이 prefix 를 읽음. 손익분기가 **조언 호출 약 3회**라고 문서가 못 박음. 그 아래면 켜는 쪽이 손해임.

문제는 **`caching` 을 중간에 켜고 끄는 것이 `tools` 편집**이라는 것임. advisor 를 빼는 것도, `model` 을 바꾸는 것도 마찬가지임. 그러면 히스토리의 thinking 블록이 무효가 되고, prefix 검사가 걸리는 환경에서는 **400** 이 남. **executor 가 Fable 5.2 나 Mythos 5.2 면 계정 나이와 무관하게 항상 검사함.** 빠져나가는 길은 `thinking.block_binding.prefix_mismatch_behavior` 를 `"drop_block"` 으로 두는 것인데 그것도 베타(`thinking-binding-controls-2026-08-01`)이고, [thinking 블록의 세 가지 바인딩](2026-10-12-preserved-thinking-bindings.md)이 적어 둔 대로 **출력 토큰이 늘어나는 대가**가 붙음.

여기서 설계가 꼬이는 자리가 하나 더 있음. **`max_uses` 는 턴당이라 대화 전체 상한이 없음.** 문서가 제시하는 방법은 클라이언트가 호출 수를 세다가 한도에 닿으면 `tools` 에서 advisor 를 빼는 것인데, 그 "빼는" 행위가 위의 400 을 부름. **비용 상한을 거는 정석 방법이 그대로 에러 경로를 밟는 구조**임.

- `clear_thinking` 의 `keep` 을 `"all"` 이 아닌 값으로 두면 advisor 캐시가 미스남. 비용만 손해고 동작은 같음
- `clear_tool_uses` 는 advisor 블록과 **완전히 호환되지 않는다**고 문서가 적음
- `advisor_tool_result` 블록 자체는 히스토리에서 **떼어낼 필요가 없음**

## 스트리밍이 조용해지는 구간

**advisor 는 스트리밍하지 않음.** `server_tool_use` 블록의 `content_block_stop` 에서 스트림이 멈추고, 그동안 SSE `ping` 이 **약 30초마다** 오고, 조언이 끝나면 `content_block_start` 하나로 전체가 한 번에 떨어짐.

앱에서 토큰을 그대로 화면에 흘리고 있었으면 **그 구간이 멈춘 화면으로 보임.** [Claude Opus 5.5](../models/2026-09-22-claude-opus-5-5.md) 문서에 적어 둔 "진행 상황이 `text` 가 아니라 thinking 으로 와서 화면만 조용해짐"과 같은 계열이고, 이쪽은 길이가 조언 생성 시간만큼임. `ping` 을 받아 "검토 중" 상태를 띄우든지, 조언 호출을 사용자에게 보이는 턴 밖으로 빼든지 둘 중 하나를 골라야 함.

`pause_turn` 도 같이 봐야 함. 같은 턴에 클라이언트 툴 호출이 없으면 응답이 `stop_reason: "pause_turn"` 으로 끝나고 재개 시점에 advisor 가 돎. 재개할 때는 **assistant 메시지를 그대로 다시 보내면서 advisor 툴을 계속 포함**해야 하고, 빼면 400 임. 클라이언트 툴도 같이 불렀으면 `stop_reason` 은 `tool_use` 가 되고 advisor 는 **다음 요청 시작 시점**에 돎.

`tool_choice` 로 강제하는 길도 막혀 있음. `{"type": "tool", "name": "advisor"}` 는 수동 extended thinking 과 같이 못 쓰고(400), **Fable 5.2·Mythos 5.2·Opus 5.5·Sonnet 5.5·Fable 5.1·Mythos 5.1 을 executor 로 쓰면 `tool_choice` 의 `tool` 과 `any` 자체가 거부됨.** 요즘 모델에서는 프롬프트로 유도하는 수밖에 없다는 뜻임.

## Managed Agents 쪽은 모양이 다름

세션에서는 툴 정의가 아니라 에이전트 설정임. `multiagent` 블록에 `"advisor": {"type": "enabled", "model": ...}` 를 둠. **`max_uses`·`max_tokens`·`caching` 이 없고**, 조언은 응답 블록이 아니라 **세션 이벤트 스트림의 thread event** 로 옴. 즉 위에서 정리한 비용 상한 수단이 이 경로에는 하나도 없음.

[Managed Agents 다이내믹 워크플로](../agents/2026-10-09-managed-agents-dynamic-workflows.md)가 적어 둔 것, 즉 `multiagent` 를 통째로 다시 보내야 해서 `advisor` 를 빠뜨리면 어드바이저가 꺼진다는 것이 여기 붙음. 그쪽 문서가 적은 "어드바이저 스레드는 자식 스레드 25개 한도에 안 센다"도 같은 기능의 다른 면임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md) | 그 문서가 **한 줄로 이름만 적어 둔 Advisor Tool** 의 실물. 에스컬레이션 패턴 대 Devin Fusion 의 하청 패턴 비교가 그쪽에 있음 |
| [Claude Sonnet 5.5](../models/2026-09-28-claude-sonnet-5-5.md) | 그 문서의 advisor 조합 제한 한 줄이 이 문서의 호환 표 한 행이었음. 암호화된 결과를 못 읽는다는 서술도 같은 자리 |
| [Managed Agents 다이내믹 워크플로](../agents/2026-10-09-managed-agents-dynamic-workflows.md) | `multiagent` 블록의 `advisor`. 설정 모양과 상한 수단의 부재가 Messages API 쪽과 갈림 |
| [thinking 블록의 세 가지 바인딩](2026-10-12-preserved-thinking-bindings.md) | `tools` 편집이 히스토리를 무효화하는 경로. 비용 상한을 걸려다 400 을 밟는 구조가 여기서 설명됨 |
| [서버측 compaction](2026-09-20-claude-api-compaction.md) | 최상위 `usage` 가 실제 소비를 과소 보고하는 **같은 형태의 두 번째 사례**. 그쪽은 compaction 몫, 이쪽은 advisor 몫 |
| [Claude API OpenTelemetry 내보내기](2026-10-14-claude-api-opentelemetry.md) | 텔레메트리 층에서도 같은 누락이 반복됨. 스팬만 보고 advisor 비용을 셀 수 없음 |
| [최소 하네스 대 오케스트레이터](2026-09-30-minimal-harness-mle-agents.md) | 그 논문은 **위임을 켜면 성적이 떨어진다**고 쟀고, 이 툴은 위임으로 올린다고 함. 차이는 일을 넘기느냐 조언만 받느냐임 |
| [Usage and Cost Analytics API](2026-10-29-usage-cost-analytics-api.md) | advisor 지출이 advisor 모델의 사용량 행으로 나타남. 모델별 지출만 보고 라우팅 실패로 읽으면 안 되는 자리 |
| [자기개선 에이전트 루프의 7가지 규칙](2026-08-10-self-improving-agent-loops.md) | 프롬프트 블록이 한 벤치마크에서 올리고 다른 벤치마크에서 깎는다는 것을 벤더가 직접 적어 둔 사례 |

## 짚어야 할 것

- **벤치마크 수치는 전부 Anthropic 자체 발표임.** 게다가 비교 조건이 대칭이 아님. SWE-bench 의 단독 구성은 adaptive thinking 을 켰고 advisor 구성은 thinking 을 끈 채 권장 시스템 프롬프트를 붙였음. **advisor 만의 기여분은 발표에서 분리되지 않음**
- **발표 본문의 결과 표가 이미지라 읽지 못했음.** 위 표의 수치는 본문 산문과 각주에서 확보한 것이고, Terminal-Bench 2.0 의 점수는 각주의 측정 조건만 있고 **수치를 확인하지 못했음**
- **토큰 단가가 어느 1차 출처에도 없음.** 문서는 "각 모델 요율로 청구된다"까지만 적음. 이 문서가 비용 비율을 직접 계산하지 않은 이유임
- **직접 호출해 보지 않았음.** 필드 이름, 에러 코드, 수치는 전부 문서 본문과 문서의 예시 JSON 에서 읽은 것임
- **`max_tokens` 실험의 표본이 구성당 40회**이고 문서 자신이 정확도 차이가 노이즈 범위라고 적음. 7배 절감은 그 과제에서의 값이고 일반 수치가 아님
- 발표에 고객 인용이 붙어 있으나 **측정 방법이 없는 추천사라 수치로 옮기지 않았음**
- **온디바이스 경로와는 무관함.** Foundation Models framework 쪽에는 대응물이 없고, 이 패턴은 서버를 두는 구성에서만 성립함
- **평문 advisor 가 구형 모델에만 남아 있음.** 조언 내용을 평가나 디버깅에 쓰려는 설계는 모델을 올리는 순간 재료를 잃음. 이 제약이 완화될 거라는 서술은 문서에 없음
- ZDR 은 Covered Models 를 빼고 적격이고 **HIPAA 는 비적격**임. advisor 출력은 응답에 담겨 돌아오고 서버에 보관되지 않는다고 적혀 있음

## 유효기간

**2026-11-12 확인 기준이고 베타임.** 발표 글은 **2026-04-09** 에 고정된 기록이라 수치는 안 바뀌지만, **호환 표는 모델이 나올 때마다 다시 그려짐.** 실제로 Fable 5.2 가 나온 2026-11-09 에 "Fable 5.2 executor 는 Fable 5.2·Mythos 5.2 만 advisor 로 받는다"는 줄이 추가됐음. 다시 볼 때 확인할 것 셋. **베타를 벗어났는지**, **Fable/Mythos 계열 executor 의 advisor 목록이 아래 세대로 다시 열렸는지**, 그리고 **대화 전체 단위의 호출 상한이 파라미터로 생겼는지**. 셋째가 생기면 위의 "상한을 걸려면 `tools` 를 편집해야 하고 그게 400 을 부른다"가 통째로 사라짐. 표를 손으로 고치지 말고 [호환 절](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool#model-compatibility)에서 새로 받을 것.
