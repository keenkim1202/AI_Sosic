---
title: Claude Haiku 5.5, 프롬프트가 10만 토큰을 넘는 순간 단가가 5배가 되는 모델
source: https://platform.claude.com/docs/en/models/haiku-5-5/whats-new-haiku-5-5
author: Anthropic
published: 2026-10-07
collected: 2026-10-08
tags: [claude, model-release, llm-cost, prompt-caching, effort, thinking, migration]
---

출처: [What's new in Claude Haiku 5.5](https://platform.claude.com/docs/en/models/haiku-5-5/whats-new-haiku-5-5) · [모델 개요](https://platform.claude.com/docs/en/models/haiku-5-5/overview) · [마이그레이션 가이드](https://platform.claude.com/docs/en/models/haiku-5-5/migration-guide) · [프롬프팅 가이드](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5) · [공지](https://www.anthropic.com/claude-haiku-5-5) · [가격](https://platform.claude.com/docs/en/about-claude/pricing) (2026-10-08 확인)

## 요약

**2026-10-07** 에 나온 `claude-haiku-5-5`. 분류·추출·라우팅처럼 양이 많고 지연이 중요한 일을 겨냥했고, Haiku 계열 최초로 **effort** 를 받음. 컨텍스트가 **1M**, 최대 출력이 **128K** 로 Haiku 4.5 의 200K / 64K 에서 올라감. 가격이 이 저장소가 지금까지 본 적 없는 모양인데, **프롬프트가 100,000 토큰을 넘으면 입력·출력·캐시 단가가 전부 5배**로 갈림. 즉 1M 창을 다 쓰라고 주면서 그 창의 **앞 10%만 싼 요금**임. Anthropic 자신은 평균 **약 75% 저렴**, 100K 이하 요청에서는 **90% 저렴**하다고 적었고 그 계산에 토크나이저 변경을 반영했다고 밝힘. 앱에 붙일 때 실제로 먼저 터지는 것은 가격이 아니라 **깨지는 요청 다섯 가지**인데, `temperature` 와 assistant prefill 이 둘 다 **400** 이 됨.

## 기본 정보

| 항목 | 값 |
|---|---|
| 모델 ID | `claude-haiku-5-5` (날짜 접미사 없는 고정 ID, 별도 alias 없음) |
| 릴리스 | 2026-10-07 |
| 컨텍스트 | 1M 토큰 |
| 최대 출력 | 128K (Batch API 는 `output-300k-2026-03-24` 베타 헤더로 300K) |
| thinking | adaptive, 기본 켜짐. 기본 effort 는 `medium` |
| 지식 컷오프 | 2026-06 (reliable knowledge cutoff, training data cutoff 동일) |
| 입출력 | 텍스트·이미지 입력, 텍스트 출력 |
| 플랫폼 | Claude API, Amazon Bedrock, Google Cloud, Microsoft Foundry, Claude Platform on AWS |
| Priority Tier | **미지원.** Haiku 4.5 에 Priority Tier 약정이 있으면 용량 계획을 따로 세우라고 가이드가 적음 |
| 은퇴 | 2027-10-07 이전은 아님 |

## 10만 토큰 경계가 가격표를 둘로 가름

단가가 프롬프트 길이로 갈리는 Claude 모델은 이게 처음임. 가격 문서가 Haiku 5.5 를 두 줄로 나눠 적어 뒀음.

| 항목 | 프롬프트 100K 이하 | 프롬프트 100K 초과 | Haiku 4.5 |
|---|---|---|---|
| 입력 | $0.10 / MTok | $0.50 / MTok | $1 / MTok |
| 출력 | $0.50 / MTok | $2.50 / MTok | $5 / MTok |
| 5분 캐시 쓰기 | $0.125 / MTok | $0.625 / MTok | $1.25 / MTok |
| 1시간 캐시 쓰기 | $0.20 / MTok | $1 / MTok | $2 / MTok |
| 캐시 읽기 | $0.01 / MTok | $0.05 / MTok | $0.10 / MTok |

읽을 때 틀리기 쉬운 지점 둘.

- **갈림의 기준은 프롬프트 길이 하나**임. 출력 단가도 프롬프트가 100K 를 넘었는지로 결정됨. 긴 문서를 한 번 밀어넣으면 그 요청의 **출력까지 5배**로 청구됨. 짧은 답을 받는다고 피할 수 있는 게 아님
- **캐시 읽기 배수는 두 구간 모두 0.1배**임($0.01/$0.10, $0.05/$0.50). Fable 5.1 의 0.025배나 Opus 5.5·Sonnet 5.5 의 0.05배 같은 예외가 아니라 표준 배수고, 싸게 느껴지는 것은 기본 입력가가 낮아서임

그리고 장문 컨텍스트 요금 문서가 이 모델을 명시적으로 예외로 빼 둠. Claude 4.6 이후 모델은 1M 창을 **표준 요금으로** 쓰는데(90만 토큰 요청이 9천 토큰 요청과 같은 단가), "**except Claude Haiku 5.5**" 라고 적혀 있음. 1M 창을 가진 모델 중에서 창을 쓰는 데 돈이 더 드는 유일한 모델임.

## 토크나이저가 그 경계를 앞당김

Haiku 5.5 는 Claude 4.7 이후와 같은 새 토크나이저를 씀. 같은 텍스트가 Haiku 4.5 대비 **약 30% 더 많은 토큰**으로 세어짐. 공지는 저렴해졌다는 계산에 이 증가를 반영했다고 밝혔는데, 반영이 안 되는 곳이 하나 있음. **100,000 이라는 경계값 자체**임.

**이 문서의 계산:** 약 30% 증가를 그대로 적용하면 Haiku 4.5 에서 약 **77,000 토큰**이던 프롬프트가 Haiku 5.5 에서 경계에 닿음(100,000 / 1.3 ≒ 76,900). 4.5 에서 8만 토큰짜리 프롬프트를 쓰고 있었다면 모델 ID 만 바꿔도 비싼 구간으로 넘어갈 수 있음. 증가율은 내용에 따라 다르다고 원문이 못 박아 뒀으니 이건 추정이고, 확인하는 방법은 가이드가 적어 둔 그대로임. **`model` 을 `claude-haiku-5-5` 로 두고 token counting 을 다시 돌릴 것.** 4.5 에서 재던 수를 재활용하지 말라고 명시돼 있음.

공지의 각주는 Haiku 4.5 요청 중 **약 90%가 100,000 토큰 미만**이었다고 적음. 뒤집으면 10%는 넘었다는 뜻이고, 그 10%가 이번에 5배 구간으로 들어감. 평균 75% 절감이라는 수치는 이 분포를 가정한 값이라서, 내 앱의 프롬프트 분포가 다르면 그대로 쓸 수 없음.

## 깨지는 요청 다섯 가지

마이그레이션 가이드가 breaking 으로 분류한 것이 다섯 개임. 앱에서 쓰던 패턴을 정확히 때리는 것이 둘 섞여 있음.

| 바뀐 것 | 결과 | 해야 할 것 |
|---|---|---|
| `thinking: {"type": "enabled", "budget_tokens": N}` | 400 | `{"type": "adaptive"}` 로 바꾸고 `output_config.effort` 로 조절 |
| `temperature`, `top_p`, `top_k` | 기본값 외 값은 400 | 세 개 다 빼기. `temperature` 는 `1`, `top_p` 는 `0.99` 만 허용되고 **`top_p: 1` 도 400**. 둘을 같이 보내도 400 |
| assistant prefill (마지막 턴을 assistant 로 끝내기) | 400 | `messages` 를 user 턴으로 끝내기. 포맷 강제는 structured outputs 나 enum 필드 툴로 |
| `computer_20250124` (Claude API·Google Cloud) | 400 | `computer_toolset_20260801` 툴셋으로 교체하고 `computer-use-2025-01-24` 헤더 제거 |
| thinking 블록을 되돌려 보내면서 앞선 턴을 고치기 | 400 | 대화를 append-only 로 유지 |

`temperature: 0` 과 assistant prefill 은 앱에서 **JSON 을 안정적으로 받아내려고** 쓰던 두 손잡이임. 둘 다 사라졌으므로 같은 보장을 원하면 structured outputs 로 옮기는 수밖에 없음. Bedrock 은 structured outputs 가 없어서 툴로 대신하라고 가이드가 따로 적어 뒀음.

breaking 은 아니지만 응답 파싱을 조용히 깨는 것도 있음. adaptive thinking 이 기본이라 **응답이 `thinking` 블록으로 시작할 수 있음.** 첫 블록을 답으로 읽는 코드는 `type` 으로 골라야 함. 게다가 thinking 블록은 기본적으로 **본문이 비어 있고 `signature` 만** 들어옴. 요약된 사고를 받으려면 `thinking: {"type": "adaptive", "display": "summarized"}` 를 명시해야 하는데, Haiku 4.5 는 요약을 그냥 줬음.

## thinking 을 끌 수 있다는 점에서 상위 모델과 갈림

이 저장소가 최신 Claude 모델에 대해 정리해 둔 결론 중 몇 개가 Haiku 5.5 에서는 성립하지 않음.

- **`thinking: {"type": "disabled"}` 가 받아들여짐.** 단 `low`·`medium`·`high` 에서만이고 `xhigh`·`max` 는 400. 끄는 것보다 effort 를 낮추는 쪽이 권고임
- **강제 `tool_choice`(`any`, 특정 툴)가 받아들여짐.** 대신 응답이 툴 호출로 시작하고 thinking 블록이 없음. 생각을 먼저 시키려면 `auto` 로 두고 프롬프트에 언제 쓸지 적으라고 함
- effort 기본값이 `medium`. Sonnet 5.5 와 Fable 5.1 의 `high` 보다 낮음

프롬프팅 가이드가 effort 별로 무엇이 깨지는지를 꽤 구체적으로 적어 둠. `low` 에서는 긴 에이전트 프롬프트일 때 검색을 건너뛰거나 **일을 끝내기 전에 사용자에게 되넘기는** 일이 생기고, `low`·`medium` 에서는 코드 변경을 확인 없이 완료로 보고하는 일이 생김. `low` 에서 `medium` 으로 올리면 조기 종료가 대략 절반으로 줄었지만 시도당 출력 토큰이 **두 배 이상**이 됐다고 적음. `xhigh` 에서는 멀티턴 채팅에서 답 전체를 thinking 안에 쓰고 **본문이 비어 있는 턴**이 나오는 경우가 있다고 함.

한 가지 더. **최상위 `effort` 값을 요청 사이에 바꾸면 그 대화의 메시지 프롬프트 캐시가 무효화됨.** 턴별로 바꾸려면 베타 헤더 `mid-conversation-output-config-2026-07-01` 을 쓴 per-message effort 변경이어야 하고, thinking 을 끈 상태에서는 그것도 400 임.

## 거절이 사용자 화면까지 그대로 올라옴

Haiku 4.5 에는 없던 것인데, Haiku 5.5 는 안전 분류기를 돌려 요청을 거절할 수 있음. 응답이 `stop_reason: "refusal"` 로 오고 `stop_details.category` 가 `cyber`·`frontier_llm`·`bio`·`general_harms` 중 하나를 담음. 가이드가 두 문장을 명시적으로 적어 둠. **"Claude Haiku 5.5 has no server-side fallback (beta)."** 그리고 같은 요청을 다시 보내도 보통 또 거절된다는 것.

앱 입장에서 이게 뜻하는 바는 분명함. 상위 모델에서 서버측 fallback 에 기대고 있었다면 Haiku 5.5 에서는 **클라이언트가 직접 처리해야 하는 실패 경로가 하나 늘어남.** 재시도로는 안 풀리므로 다른 모델로 넘기든 사용자에게 설명하든 분기를 만들어야 함. `cyber` 와 `general_harms` 는 **무해한 작업도 걸릴 수 있다**고 문서가 스스로 적어 뒀으니, 보안 관련 텍스트를 다루는 앱이면 특히 미리 넣어 볼 값어치가 있음.

## 벤치마크

공지의 표를 그대로 옮김. 전부 Anthropic 이 돌린 값임.

| 벤치마크 | Haiku 5.5 | Haiku 4.5 | GPT-6 Luna | Sonnet 5.5 |
|---|---|---|---|---|
| GDPval-AA v2.1 (Elo) | 1620 | 735 | 1437 | 1840 |
| AA-Briefcase v1.1 (Elo) | 1578 | 614 | 1336 | 1824 |
| OSWorld 2.1 (offline subset) | 72.4% | 15.7% | 48.9% | 83.9% |
| Humanity's Last Exam (툴 없음) | 45.9% | 10.2% | 확인할 수 없었음 | 56.9% |
| Humanity's Last Exam (툴 있음) | 57.4% | 18.7% | 확인할 수 없었음 | 64.5% |
| Terminal-Bench 4.0 | 39.2% | 0.0% | 16.4% | 70.6% |
| FrontierCode 1.1 (Main) | 46.4% | 확인할 수 없었음 | 42.4% | 52.1% (xhigh) |
| Chartography (툴 없음) | 46.4% | 6.4% | 29.1% | 61.6% |

읽을 때 조심할 것. Haiku 4.5 가 Terminal-Bench 4.0 에서 **0.0%**, OSWorld 에서 15.7% 임. 바닥에 붙은 점수에서 올라온 배수는 능력 차이의 크기가 아니라 **그 벤치마크가 Haiku 4.5 의 사정권 밖이었다**는 뜻에 가까움. 세대 간 배수로 인용하면 과장이 됨. 반면 Sonnet 5.5 열과 나란히 보면 격차가 코딩·터미널 쪽에서 가장 크게 남아 있음(39.2% 대 70.6%). 공지도 복잡한 에이전틱 코딩은 Sonnet 5.5·Opus 5.5 쪽이 낫다고 적었고, Haiku 5.5 의 자리를 **서브에이전트**로 잡았음.

## 같은 날 Sonnet 5.5 캐시 읽기가 절반이 됨

같은 릴리스 노트 항목에 붙어 있는 변경이라 같이 적어 둠. **Sonnet 5.5 의 캐시 읽기가 $0.20 에서 $0.10 / MTok 으로 내려갔고**, 기본 입력가 $2 에 대한 **0.05배**가 됨. 캐시 쓰기와 나머지 가격은 그대로임. 공지는 이것으로 대부분의 에이전틱 작업에서 Sonnet 5.5 가 **약 20% 저렴**해진다고 적음.

가격 문서의 각주가 지금은 이렇게 정리돼 있음. **Fable 5.1 과 Mythos 5.1 이 0.025배, Opus 5.5 와 Sonnet 5.5 가 0.05배, 나머지 전부가 0.1배.** 캐시 산술을 모델별로 나눠 둔 이 저장소의 문서들은 이 분기를 기준으로 다시 봐야 함.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 것 | 갈리는 것 |
|---|---|---|
| [Claude Fable 5.1](2026-09-01-claude-fable-5-1.md) | 캐시 읽기 배수가 모델마다 다르다는 결론 | 그 문서는 0.025배가 Fable 5.1 **단독 예외**라고 적었음. 2026-10-07 이후로는 Opus 5.5·Sonnet 5.5 의 0.05배가 끼어들어 배수가 세 갈래임 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 캐시 히트율과 미스 비용 계산 | 그쪽은 배수가 0.1배 하나라는 전제로 쓴 산술임. Haiku 5.5 는 0.1배가 맞지만 **구간이 둘**이라 기준 단가가 요청마다 갈림 |
| [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md) | 공지가 Haiku 5.5 의 용도로 compaction 과 요약을 직접 지목함 | 그 문서는 서버가 돌리는 compaction 을 다루고, 여기는 요약을 직접 돌릴 때 쓸 모델 쪽임 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | "더 싼 모델로 옮기기"와 태스크 라우팅 레버 | 그 문서의 라우팅은 모델 단위 분기임. Haiku 5.5 는 **같은 모델 안에서 프롬프트 길이로 단가가 갈리므로** 라우팅 규칙에 길이 조건이 하나 더 필요해짐 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | "스스로 검증해라"를 프롬프트에서 빼라는 결론 | Haiku 5.5 프롬프팅 가이드는 반대로 **코드 변경을 확인하라는 문단을 넣으라**고 함. 작은 모델에서는 그 지시가 값을 함 |

## 짚어야 할 것

- **75%·90% 절감은 Anthropic 의 계산임.** 각주가 전제를 밝혀 뒀고(100K 미만 90%, 초과 50%, 과거 요청의 약 90%가 100K 미만, 토크나이저 증가 반영) 그 전제가 내 워크로드와 같을 이유는 없음. 내 프롬프트 분포로 다시 계산해야 하는 수치임
- **약 77,000 토큰이라는 경계는 이 문서가 계산한 값임.** 원문에 없음. 약 30% 증가와 100,000 경계라는 두 사실에서 나온 것이고, 증가율이 내용에 따라 다르다고 원문이 적어 뒀으므로 실제 경계는 프롬프트마다 다름
- **GPT-6 Luna 비교는 Anthropic 이 돌린 값임.** 경쟁 모델 열은 벤더 자체 측정이라 그대로 인용하기 전에 측정 조건을 봐야 하고, Humanity's Last Exam 두 행과 FrontierCode 의 Haiku 4.5 칸은 공지 표에 값이 없어 **확인할 수 없었음**
- **시스템 카드는 읽지 않았음.** 공지가 [Claude Haiku 5.5 system card](https://www.anthropic.com/document/claude-haiku-5-5-system-card) 를 링크하고 안전 평가 방법이 거기 있다고 적었는데, 이 문서는 공지에 요약된 수준까지만 확인함. 사이버 세이프가드가 Haiku 4.5 보다 엄하고 Sonnet 5.5 보다 느슨하다는 서술도 공지 쪽 문장임
- **고객 사례 수치는 고객이 보고한 값임.** Asana 의 지연 30% 이상 감소, Box 의 약 절반 지연, AlphaSense 의 0.84 대 0.76, HubSpot 의 92.8% 는 각 회사가 자기 평가 스위트에서 낸 값이고 방법이 공개돼 있지 않음
- 공지의 **Max·Team 플랜 월 API 크레디트**(Max 5x $100, Max 20x $200, Team 최대 $500 풀)는 요금제 쪽 변경이라 이 저장소의 관심 밖임. 기록만 해 둠

## 유효기간

**2026-10-08 확인 기준이고 모델 공개 다음 날임.** 가격 구간과 effort 동작은 계속 움직이니 다시 볼 때는 [가격](https://platform.claude.com/docs/en/about-claude/pricing)의 Haiku 5.5 두 줄과 캐시 각주, 그리고 [릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)의 Haiku 항목부터 대조할 것. 특히 확인할 것 셋. **100,000 토큰 경계가 그대로인지**(구간제 자체가 이 모델에서 처음 나온 것이라 조정될 수 있음), **Priority Tier 가 붙었는지**, 그리고 **거절에 서버측 fallback 이 생겼는지**. 셋 다 생기면 이 문서의 판단이 바뀜. 벤치마크 표와 고객 사례는 공지 시점에 고정된 기록이라 낡지 않음.
