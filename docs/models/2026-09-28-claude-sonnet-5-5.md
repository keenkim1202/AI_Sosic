---
title: Claude Sonnet 5.5, thinking 블록이 계정에 묶이고 effort를 올리면 점수가 내려감
source: https://www.anthropic.com/claude-sonnet-5-5
author: Anthropic
published: 2026-09-28
collected: 2026-09-29
tags: [claude, model-release, thinking, effort, prompt-caching, coding-agent, llm-cost]
---

출처: [Introducing Claude Sonnet 5.5](https://www.anthropic.com/claude-sonnet-5-5) (2026-09-28) · [What's new in Claude Sonnet 5.5](https://platform.claude.com/docs/en/models/sonnet-5-5/whats-new-sonnet-5-5) · [모델 페이지](https://platform.claude.com/docs/en/models/sonnet-5-5/overview) · [Preserved thinking](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking) · [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api) (2026-09-29 확인)

## 요약

2026-09-28 릴리스. 가격은 Sonnet 5와 **같음**($2 / $10)인데 깨지는 코드가 다섯 군데임. 이 저장소 관점에서 값어치 있는 건 셋임. 첫째, **thinking 블록이 그것을 만든 계정 밖에서는 통하지 않음.** 다른 계정이 그 블록을 보내면 API가 조용히 버리고 요청은 성공함. Claude 모델에서 이런 제약이 붙은 건 이 모델이 처음이고, 트랜스크립트를 계정 경계 너머로 옮기는 설계가 여기서 막힘. 둘째, thinking을 끄는 방법이 `disabled`에서 **`between_tools`**로 바뀌었고 그게 **`high` 이하 effort에서만** 받아짐. 셋째, 발표 각주가 **max effort에서 점수가 xhigh보다 낮게 나온 이유**를 직접 적었는데, 원인이 모델이 아니라 **하네스가 서브에이전트를 퍼뜨린 것**임. 그리고 [Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md)이 유보로 남겼던 "히스토리 편집 검사가 전체 계정으로 확대됐는지"는 **아직 안 풀림.** 여전히 2026-08-31 이후 계정에만 기본 강제임.

## 기본 정보

| | |
|---|---|
| 모델 ID | `claude-sonnet-5-5` (Bedrock은 `anthropic.claude-sonnet-5-5`) |
| 릴리스 | 2026-09-28. 은퇴는 **2027-09-28 이전은 아님** |
| 컨텍스트 / 최대 출력 | 1M 토큰 / 128K 토큰 (Batch API는 `output-300k-2026-03-24` 헤더로 300K) |
| 가격 | input **$2** / output **$10** / 5m cache write $2.50 / 1h cache write $4 / **cache read $0.10** (MTok) |
| Sonnet 5 대비 가격 | **캐시 읽기만 다름.** input·output·cache write·배치 요율은 같고 **cache read 가 $0.20 → $0.10 으로 절반**임 (입력 단가의 10% 가 아니라 5%) |
| thinking | adaptive. 최저 설정이 `between_tools` |
| 기본 effort | `high` |
| 지식 컷오프 | 2026-06 (reliable, training 둘 다) |
| 토크나이저 | **Sonnet 5와 같음.** 같은 텍스트가 같은 토큰 수로 나옴 |
| 최소 캐시 길이 | **512토큰.** Sonnet 5는 1,024였음 |

`temperature`·`top_p`·`top_k` 는 **값을 보내는 것 자체가 400** 임. 기본값을 명시해 보내던 코드도 필드를 지워야 함. 최근 Claude 모델과 같음.

## thinking을 끄는 문법이 바뀜

`thinking: {"type": "disabled"}`를 보내면 **400**이고, 에러 메시지가 `between_tools`를 가리킴. 대체 설정이 이것임.

```json
{"thinking": {"type": "between_tools"}}
```

베타 헤더가 필요 없고 Sonnet 5.5를 제공하는 모든 플랫폼에서 됨. 다만 조건이 촘촘해서 옮길 때 걸리는 자리가 여럿임.

| 조건 | 결과 |
|---|---|
| effort `low`·`medium`·`high` | 받아짐 |
| effort `xhigh`·`max` | **400.** 그 구간을 쓰려면 adaptive로 가야 함 |
| 대화 중간 effort 변경 | **400.** 지금 레벨과 다른 `output_config.effort`를 끼우면 거부됨 |
| `display`·`budget_tokens`·`block_binding` 동봉 | **400.** `between_tools`는 다른 필드를 안 받음 |
| `thinking: {"type": "enabled", "budget_tokens": N}` | **400.** 수동 예산 모드가 없음 |

즉 **`between_tools`는 "싸게 가는 고정 레인"**이고, 턴마다 effort를 조절하는 설계와는 같이 못 씀. 둘 다 원하면 adaptive를 쓰고 `display`로 노출을 조절하는 쪽이 맞음.

`between_tools`를 켜도 툴 호출 사이 **진행 업데이트는 여전히 `thinking` 블록으로 옴.** 그 블록은 손대지 말고 assistant 턴과 함께 그대로 돌려보내야 하는데, 문서가 디테일을 하나 덧붙임. 돌려보낸 진행 업데이트 블록은 모델에게 **요약이 아니라 원래 쓴 전문**으로 읽힘. 툴을 안 쓰는 요청이면 응답에 텍스트만 옴.

## 툴 호출 사이 텍스트가 조용해짐

에러가 안 나면서 UI만 바뀌는 항목이라 제일 늦게 발견되는 쪽임.

> "Between tool calls, notes longer than a sentence or two come back as progress-update `thinking` blocks. Shorter remarks stay `text`."

기본값 `display: "omitted"`에서는 그 블록의 텍스트가 **빈 값**으로 옴. 툴 호출 사이 진행 상황을 사용자에게 스트리밍하던 앱은 그 구간이 **아무 에러 없이 조용해짐**. 푸는 방법은 둘. adaptive 를 쓰면서 `thinking.display` 를 설정하거나, `between_tools` 로 끄는 것. **둘 다 블록 타입은 `thinking` 그대로이고 그 안의 요약 텍스트가 채워져서 올 뿐임.** `text` 블록만 렌더하는 클라이언트는 어느 쪽을 켜도 여전히 아무것도 못 보여줌. 원문도 `between_tools` 에서 진행 메모가 "still come back as `thinking` blocks with their summary text" 라고 적음.

길이로 갈린다는 게 함정임. 한두 문장짜리 짧은 말은 `text`로 그대로 오므로, 개발 중 짧은 툴 체인만 돌려보면 **증상이 안 나타남.**

## thinking 블록이 계정에 묶임

이번 릴리스에서 가장 새로운 제약임. 릴리스 노트가 별도 항목으로 뺐음.

> "Thinking blocks that Claude Sonnet 5.5 produces work only in the account that produced them, or in an account linked to it. When another account sends one of these blocks, the API drops the block before the model sees it, and the request succeeds."

**실패하지 않고 조용히 버려진다**는 게 핵심임. 요청은 200으로 돌아오고 모델만 그 추론 없이 답함. Claude API와 Google Cloud에서 `thinking-binding-controls-2026-08-01` 헤더를 붙이면 `input_transformations`에 `reason: "organization_binding_mismatch"`로 잡힘. 이전 모델이 만든 블록은 해당 없음.

걸리는 설계가 구체적으로 이럼.

- 트랜스크립트를 **덤프해서 다른 계정의 재현 환경이나 eval 러너에 먹이는 것.** 에러가 안 나니 "재현이 안 되네"로만 보임
- 조직을 나눠 쓰는 팀에서 **한 계정이 만든 대화를 다른 계정이 이어받는 것**
- 고객사 계정과 자사 계정을 오가는 게이트웨이

점검 방법은 위 베타 헤더를 붙이고 `input_transformations`를 로깅하는 것뿐임. 헤더 없이는 **구분이 안 됨.**

## 어느 모델이 어느 모델의 thinking을 읽는가

모델을 갈아타는 라우터나 폴백이 있으면 이 표가 실무임. Sonnet 5.5 기준임.

| 방향 | 내용 |
|---|---|
| Sonnet 5.5가 읽는 것 | Sonnet 5, Opus 4.8, Haiku 4.5, 그 이전 모델 |
| Sonnet 5.5가 못 읽는 것 | **Opus 5, Opus 5.5, Fable·Mythos 계열 전부** |
| Sonnet 5.5의 블록을 읽는 모델 | **Opus 5.5 하나.** 단 **Claude API 와 Claude Models on Google Cloud (Google managed) 에서만** |

읽히지 않는 블록은 **거부가 아니라 폐기**임. 요청은 성공하고 과금도 안 됨. 그래서 방향이 이렇게 갈림. Sonnet 5에서 Sonnet 5.5로 올라오는 대화는 추론을 들고 올라오지만, **Sonnet 5.5 에서 Opus 5.5 로 올라가는 경로도 추론을 들고 감.** 단 그 두 플랫폼에서만이고, 그 밖의 어떤 전환이든 전환 이후 턴을 추론 없이 시작함. 비용 라우팅으로 상위 모델과 Sonnet 5.5를 오가게 설계했다면 여기서 품질이 조용히 깎임.

## 히스토리 편집 검사, Fable 5.1의 유보가 아직 안 풀림

[Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md)이 유효기간에 "히스토리 편집 검사가 전체 계정으로 확대됐는지 확인하라"고 적어뒀음. **이번 문서 기준으로는 아직임.**

> "It enforces that check by default for accounts created on or after August 31, 2026, 00:00 UTC, on the Claude API, Amazon Bedrock, and Google Cloud."

즉 기준일이 그대로임. 검사 내용도 같음. thinking 블록 앞에 있는 top-level `system`, `tools`, 그리고 앞선 메시지들이 prefix이고 그게 바뀌면 재생 시 400임. 옛 계정은 `thinking.block_binding.prefix_mismatch_behavior`를 지정할 때만 검사가 돎.

Sonnet 5.5에서 추가로 갈라야 할 게 둘임.

- **`block_binding`은 `thinking: {"type": "adaptive"}`에서만 동작함.** `between_tools`를 쓰면 이 노브가 없으므로, 히스토리를 append-only로 유지하거나 편집 지점부터 뒤의 thinking 블록을 걷어내야 함
- 계정 바인딩(위 절)과 prefix 바인딩은 **다른 검사**임. 전자는 조용히 폐기, 후자는 새 계정에서 400임. `input_transformations`의 `reason`이 `organization_binding_mismatch`인지 `prefix_binding_mismatch`인지로 갈림

## 그 밖의 깨지는 변경

| 변경 | 내용 |
|---|---|
| **강제 툴 호출 제거** | `tool_choice`가 `{"type": "any"}` 또는 `{"type": "tool", ...}`이면 400. 메시지는 `tool_choice: type "tool" and "any" are not supported for this model.` 임. **토큰 카운팅 엔드포인트에도 같은 검사가 걸림.** 대안은 `auto` + strict tool use, 또는 structured outputs |
| **computer use 툴셋 강제** | Claude API와 Google Cloud에서는 `computer_toolset_20260801`만 받고 이전 `computer_20251124`는 400. **Bedrock에서는 옛 툴이 그대로 동작함.** 플랫폼별로 갈리므로 멀티 클라우드면 분기 필요 |
| **advisor 조합 제한** | Sonnet 5.5를 executor로 쓸 때 Opus 4.8·Opus 4.7·Sonnet 5는 advisor로 400임. 받아지는 advisor는 Mythos 5.1, Fable 5.1, Mythos 5, Fable 5, Opus 5.5, Opus 5, 그리고 Sonnet 5.5 자신임. 받아지는 조합도 조언이 **암호화된 `advisor_redacted_result`**로 와서 클라이언트가 본문을 못 읽음 |

기능 지원 쪽은 Sonnet 5보다 넓음. 대화 중간 시스템 메시지, 대화 중간 툴 변경(베타), 메시지별 effort(베타)가 Sonnet 5에는 없고 이 모델에는 있음. 서버측 compaction은 [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md)에 정리한 그대로 쓸 수 있고, 문서가 이 모델에서 그게 더 중요한 이유를 따로 적음. **thinking 블록이 대화에 묶여 있으니, 클라이언트가 히스토리를 직접 자르는 대신 서명된 `compaction` 블록으로 바꿔 끼우면 남긴 턴의 thinking이 살아남을 수 있기 때문임.**

## effort를 올렸더니 점수가 내려간 자리

발표의 FrontierCode 각주가 이 저장소에서 제일 값어치 있는 대목임. 벤치마크 수치보다 **원인 서술**이 그럼.

> "Sonnet 5.5 scores lower at Max effort than at Xhigh. FrontierCode evaluates whether a code change could be merged without human edits. It penalizes out-of-scope changes, even if they are high-quality or helpful. At Max effort, Sonnet 5.5 more often ran Claude Code's code-review skill, which splits the review across many subagents, and in two cases Cognition examined, this led to a timeout or to extra edits beyond the task's scope, and therefore to a lower score."

읽을 것이 셋임.

**effort를 올린 결과가 모델 안에서 끝나지 않음.** 올린 effort가 **스킬을 더 자주 호출하게** 만들었고, 그 스킬이 서브에이전트를 퍼뜨렸고, 거기서 타임아웃과 범위 밖 수정이 나왔음. 즉 이 점수 하락은 추론 품질이 아니라 **하네스 동작의 연쇄**임. [Graph Engineering](../practices/2026-07-20-graph-engineering.md)이 말한 "병렬로 펼치면 벽시계 시간을 버는 대신 실패 표면이 늘어난다"가 벤더 자체 벤치마크 각주로 확인된 사례임.

**채점 기준이 "머지 가능한가"라서 도움이 되는 수정이 감점임.** [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md)이 게이트를 "확신이 아니라 틀리면 얼마나 아픈가"로 열라고 한 것과 같은 구조인데, 여기서는 그 기준이 **모델 설정의 최적점을 뒤집음.** 자기 워크로드의 채점 기준이 다르면 max가 최적일 수도 있다는 뜻이기도 함.

**그래서 effort는 "높을수록 좋다"가 아니라 재봐야 하는 값임.** 문서도 같은 말을 함. 레벨이 재조정돼서 Sonnet 5의 설정을 그대로 들고 오면 안 되고, effort 스윕을 다시 돌리라고 적음. 출발점 권고는 에이전틱 코딩과 멀티스텝 툴 사용이 `medium`(어렵거나 길면 `high`), 지연시간이 중요한 대화형은 `medium`이나 `low`임. 기본값은 `high`임.

## 벤치마크

**Anthropic 자체 발표 수치**임. 아래는 발표 표에서 Sonnet 5.5와 Sonnet 5 두 열만 옮긴 것이고, 원문 표에는 Opus 5.5와 GPT-6 Sol 열이 더 있음.

| 벤치마크 | Sonnet 5.5 | Sonnet 5 |
|---|---|---|
| Terminal-Bench 4.0 | 70.6% | 10.3% |
| FrontierCode 1.1 (Main) | 46.2% (Max) | 42.4% |
| CursorBench 4.0 | 55.5% | 34.1% |
| GDPval-AA v2.1 | 1844 | 1449 |
| AA-Briefcase v1.1 | 1811 | 1359 |
| Humanity's Last Exam (툴 사용) | 64.5% | 54.9% |
| OSWorld 2.1 (partial) | 80.1% | 57.0% |
| Chartography (툴 없음) | 61.6% | 15.6% |

지식 업무 두 줄에서 Sonnet 5.5가 **1844 / 1811**인데 발표가 같은 표에 적은 Opus 5.5는 **1846 / 1822**임. 단가가 $2 / $10 대 $4 / $20이므로, 이 축에서는 **절반 가격에 비슷한 Elo**라는 그림이 됨. 다만 아래 짚어야 할 것에 적은 대로 이 표는 자체 발표이고 측정 조건에 각주가 붙어 있음.

속도와 비용 주장은 이렇게 적혀 있음.

> "Sonnet 5.5 generates outputs 30%+ faster than Sonnet 5, making it our fastest Sonnet model to date."

> "In our testing, it costs up to 30% less per task than its predecessor."

**input·output 단가는 Sonnet 5와 같으므로 이 30%는 그쪽 요율에서 나올 수 없음.** 남는 출처는 태스크당 토큰 수, 그리고 **절반이 된 캐시 읽기 요율**임. 캐시 적중이 많은 에이전트 워크로드라면 뒤쪽이 실제로 기여함. 발표가 그 산출 근거를 적지 않았고 확인하지 못했음. [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md)의 레버 1이 말한 대로, 새 모델이 싸다는 주장은 자기 워크로드에 대고 다시 재는 것 말고는 확인할 방법이 없음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md) | 히스토리 편집 400과 강제 툴 호출 제거가 같은 계열임. 그 문서의 유효기간이 지목한 "전체 계정 확대"는 이번에도 안 일어났음. 반대로 캐시 읽기 0.025배는 그 모델만의 것이고 여기엔 없음 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 최소 캐시 길이가 512로 내려간 것과, prefix가 바뀌면 깨지는 목록이 이제 에러 목록과 겹치는 것 |
| [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md) | 클라이언트가 직접 자르는 대신 서명된 블록으로 바꿔 끼우면 남긴 턴의 thinking이 유효하게 남을 수 있음 |
| [Graph Engineering](../practices/2026-07-20-graph-engineering.md) | max effort에서 점수가 내려간 원인이 서브에이전트 확산과 타임아웃이었음. 펼치기의 대가가 벤더 각주로 나온 사례 |
| [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md) | 채점 기준이 "사람 수정 없이 머지 가능한가"라서 도움이 되는 범위 밖 수정이 감점됨 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 레버 1 그대로임. 단가가 같은데 "최대 30% 저렴"이라는 주장은 자기 워크로드로 재야 확인됨 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | 모델별 프롬프팅 가이드가 같은 형식으로 나옴. effort 레벨이 모델 간에 같은 사고량을 뜻하지 않는다는 점이 여기서도 반복됨 |

## 짚어야 할 것

- **벤치마크는 전부 Anthropic 자체 발표임.** 게다가 Terminal-Bench 4.0에서 Sonnet 5가 **10.3%**로 나온 것은 같은 표의 다른 줄과 비교해도 폭이 비정상적으로 큼. 하네스 조건이 갈렸을 가능성이 있는데 **발표에 그 설명이 없고 확인하지 못했음**
- **GDPval-AA와 AA-Briefcase는 사전 릴리스 배포에서 측정됐음.** 발표 각주가 그 배포에 structured outputs 응답을 저하시키는 버그가 있었고 이후 고쳐졌다고 밝힘. GPT-6 Sol 열에도 별도 각주가 붙어 있어 **모델 간 비교로 쓰기엔 조건이 정렬돼 있지 않음**
- **"최대 30% 저렴"의 근거를 확인하지 못했음.** 단가가 Sonnet 5와 동일하므로 태스크당 토큰 감소가 전제인데, 측정 조건이 발표에 없음
- **계정 바인딩이 조용히 동작함.** 에러가 아니라 폐기이므로, 베타 헤더를 붙여 `input_transformations`를 보지 않으면 품질 저하로만 보임. 트랜스크립트를 계정 밖으로 옮기는 파이프라인이 있으면 여기부터 점검해야 함
- **`between_tools`와 effort 조절은 같이 못 씀.** 비용을 낮추려고 `between_tools`로 갔다가 특정 턴만 effort를 올리는 기존 설계가 있으면 400으로 걸림
- **Claude Code나 구독 플랜에서 이 모델이 어떻게 노출되는지는 확인하지 않았음.** 위 내용은 전부 Claude API 기준임
- **시스템 카드는 링크만 확인했고 본문은 읽지 않았음**

## 유효기간

**2026-09-29 확인 기준**임. 확인할 것 셋. **`between_tools`가 `xhigh`·`max`까지 열렸는지**, **계정 바인딩이 다른 모델로도 확대됐는지**, 그리고 **히스토리 편집 검사가 2026-08-31 이전 계정까지 확대됐는지**. 셋째는 [Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md)이 먼저 적어둔 유보이고 이번에도 안 풀렸음. 다시 볼 때는 [What's new](https://platform.claude.com/docs/en/models/sonnet-5-5/whats-new-sonnet-5-5)의 베타 표시와 [Preserved thinking](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking)의 모델 호환 목록부터 대조할 것. 발표가 **Claude Haiku 5.5를 "coming in the coming weeks"**로 예고했으므로 모델 표는 곧 한 줄 더 늘어남.
