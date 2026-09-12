---
title: Claude Fable 5.1, 벤치마크보다 캐시 read 단가가 본론임
source: https://www.anthropic.com/claude-fable-and-mythos-5-1
author: Anthropic
published: 2026-09-01
collected: 2026-09-12
tags: [claude, model-release, prompt-caching, llm-cost, effort, coding-agent, benchmark]
---

출처: [Introducing Claude Fable 5.1 and Claude Mythos 5.1](https://www.anthropic.com/claude-fable-and-mythos-5-1) (2026-09-01) · [Models overview](https://platform.claude.com/docs/en/about-claude/models/overview) · [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) (공식 문서는 2026-09-12 확인)

## 요약

Fable 5.1 릴리스에서 이 저장소 관점으로 제일 값어치 있는 항목은 벤치마크가 아니라 가격표 한 줄임. **프롬프트 캐시 read 배수가 기본 input 대비 0.1배에서 0.025배로 내려갔음.** Fable 5.1과 Mythos 5.1에만 적용되고 나머지 모델은 전부 0.1배 그대로임. 그런데 cache write 배수는 5분 1.25배, 1시간 2배로 **안 바뀌었기 때문에**, write와 read의 비율이 12.5배에서 **50배**로 벌어짐. 캐시를 깨는 행위의 상대 비용이 4배가 된 것임. 나머지 셋도 짚어둘 값어치가 있음. **thinking을 끌 수 없고**(adaptive always on), **신규 API 계정은 이전 컨텍스트 수동 편집이 막혔고**, 공식 문서가 직접 **"대부분의 워크로드는 Opus 5로 시작하라"**고 적음.

## 가격표에서 실제로 바뀐 것

공식 Pricing 문서 기준 수치임. Fable 5는 비교용으로 같이 놓음.

| | Fable 5.1 | Fable 5 | Opus 5 |
|---|---|---|---|
| 기본 input | $10 / MTok | $10 / MTok | $5 / MTok |
| 5분 cache write | $12.50 (1.25배) | $12.50 (1.25배) | $6.25 (1.25배) |
| 1시간 cache write | $20 (2배) | $20 (2배) | $10 (2배) |
| cache read | **$0.25 (0.025배)** | $1 (0.1배) | $0.50 (0.1배) |
| output | $50 / MTok | $50 / MTok | $25 / MTok |

문서 각주가 범위를 명확히 함.

> "Cache hits and refreshes on Claude Fable 5.1 and Claude Mythos 5.1 are priced at 0.025x the base input price. All other models use the standard 0.1x multiplier."

### write 대 read 비율이 바뀐 것이 요점임

배수만 남기고 나누면 이렇게 됨. 아래 비율은 위 두 배수를 나눈 **계산 값**이고 원문에 그대로 적힌 숫자가 아님.

| | 5분 write | cache read | write / read |
|---|---|---|---|
| 대부분의 모델 | 1.25배 | 0.1배 | 12.5배 |
| **Fable 5.1** | 1.25배 | **0.025배** | **50배** |

같은 캐시 미스가 Fable 5.1에서는 상대적으로 4배 아프다는 뜻임. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 정리한 8가지 캐시 파괴 원인(툴 로드아웃 변경, 동적 시스템 프롬프트, effort 변경, 브랜치 이동 등)의 기대 손실이 모델을 바꾼 것만으로 올라감. 히트할 때는 더 싸지고 깨질 때는 상대적으로 더 비싸지는, 분산이 커지는 방향의 변화임.

[Claude Cowork](../practices/2026-08-10-claude-cowork.md)에 넣어둔 "이어붙일까 truncate할까" 표를 같은 가정(prefix 5k, suffix 25k, 한 턴 더)으로 다시 계산하면 이렇게 갈림.

| 선택 | 0.1배 모델 | Fable 5.1 (0.025배) |
|---|---|---|
| 이어붙이기 | 30k × 0.1 = 3k | 30k × 0.025 = **0.75k** |
| truncate, 되돌릴 지점 캐시 히트 | 5k × 0.1 = 0.5k | 5k × 0.025 = **0.125k** |
| truncate, 그 지점 캐시 없음 | 5k × 1.25 = 6.25k | 5k × 1.25 = **6.25k** |

판단 기준 자체는 안 바뀜. **되돌릴 지점에 캐시가 살아 있는지**가 여전히 갈림길임. 다만 write 배수가 그대로라 세 번째 줄만 안 움직였고, 그래서 **"캐시 없는 지점으로 되돌리기"의 상대 손해가 커짐.** 이어붙이기 대비 2.1배였던 것이 8.3배가 됨.

## 스펙과 Opus 5 대비 위치

| | Fable 5.1 | Opus 5 |
|---|---|---|
| API ID | `claude-fable-5-1` | `claude-opus-5` |
| 컨텍스트 | 1M | 1M |
| 최대 출력 | 128K | 128K |
| thinking | **Adaptive (always on)** | Adaptive |
| 기본 effort | `high` | `high` |
| 신뢰 가능한 지식 컷오프 | 2026-06 | 2026-05 |
| 은퇴 예정 | 2027-09-01 이후 | 2027-07-24 이후 |
| 지연시간 | Slower | Moderate |

여기서 놓치면 안 되는 게 **공식 문서 자신이 Fable 5.1을 기본값으로 권하지 않는다**는 것임.

> "If you're unsure which model to use, start with Claude Opus 5 for most workloads. Use Claude Fable 5.1 for demanding reasoning and long-horizon agentic work, or when your evals on Claude Opus 5 at higher effort still fall short."

조건이 **"자기 eval에서 Opus 5를 높은 effort로 돌려도 모자랄 때"**임. [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) 레버 1이 말한 그대로고, Stripe가 Opus 4.7을 거절한 판단과 같은 자리임. 토큰 단가가 Opus 5의 2배(input $10 대 $5, output $50 대 $25)이고 지연시간도 더 느리니, 새 모델이 나왔다는 이유로 기본값을 옮기면 그냥 2배 내는 것임.

## thinking을 끌 수 없음

Fable 5.1의 thinking은 **adaptive이면서 always on**임. [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)에 정리해둔 "thinking을 끄고 쓸 때"의 회피책, 즉 툴 호출이 텍스트로 새는 문제와 `<thinking>` 태그 유출에 대한 완화 지시가 Fable 5.1에서는 **필요 없는 동시에 선택지도 아님.** Opus 5는 effort `high` 이하에서 끌 수 있었는데 여기는 그 스위치 자체가 없음.

문서는 별도로 이전 세대의 수동 thinking 모드(`thinking.type: "enabled"` + `budget_tokens`)가 Opus 4.6과 Sonnet 4.6에서 폐기됐고 **그 이후 모델에서는 아예 받지 않는다**고 적음. 레거시 하네스가 이 필드를 보내고 있으면 마이그레이션 대상임.

## 신규 API 계정의 컨텍스트 수동 편집 제한

원문이 밝힌 변경임. 신규 API 계정은 **멀티턴 대화에서 Claude의 이전 컨텍스트를 수동 편집하면서 이전 thinking 트랜스크립트를 보존하는 것**을 더 이상 못 함. 이유는 품질이나 안전이 아니라 **증류(distillation) 차단**이라고 명시함. 안전장치 없이 모델 능력을 뽑아내는 데 쓰인 기법이었다는 것.

기존 계정은 영향을 받지 않는다고 하지만, **이후 나오는 모든 모델 릴리스에 이 정책이 적용된다**고 적혀 있음. 트랜스크립트를 직접 조립해 재전송하는 하네스를 굴리고 있으면 여기가 걸릴 자리임. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 가지치기 항목, 즉 중간 내용을 지우고 prefix를 다시 쓰는 패턴이 **비용 문제 이전에 정책 문제가 될 수 있음.**

## 벤치마크

원문이 제시한 수치 중 코딩·에이전틱 항목만 옮김. 전부 **Anthropic 자체 발표 수치**임.

| 벤치마크 | Fable 5.1 | Fable 5 |
|---|---|---|
| Terminal-Bench 4.0 | 55.8% | 42.0% |
| Terminal-Bench-Science 0.1 | 52.6% | 24.7% |
| CursorBench 3.2.0 | 73.4% | 70.5% |
| OSWorld 2.0 | 77.9% (partial) / 41.7% (strict) | 72.9% / 36.1% |
| AutomationBench | 31.4% | 17.1% |

Terminal-Bench 4.0은 Mythos 5.1이 **60.9%**로 더 높음.

읽을 때 붙는 단서가 둘임. 원문이 Terminal-Bench-Science 자리에 **모델당 표준오차 ±3.5~4.5포인트**를 명시함. CursorBench의 73.4% 대 70.5% 같은 3포인트 차이는 그 폭 안에 들어감. 그리고 평가를 **프로덕션 안전장치를 켠 채로** 돌렸고, 안전장치가 개입한 과제에서는 **Fable 5.1과 Fable 5 둘 다 0점 처리**됐다고 적음. OSWorld 2.0 숫자가 낮은 이유가 여기 있을 수 있음.

비용 절감 주장도 벤더 추정치임. 일반적인 워크로드에서 Fable 5 대비 **약 25%**, 에이전틱 비중이 높은 작업에서 **최대 약 45%**라고 함. 근거가 되는 워크로드 구성은 밝히지 않았음.

안전장치 쪽에서 개발자가 체감할 항목이 하나 있음. 사이버 보안 도메인에서 **오탐이 이전 대비 60% 감소**했고 취약점 식별은 하되 익스플로잇 개발은 안 한다고 함. 생물학 쪽은 정상 요청에 대해 안전장치가 **85% 덜 발동**한다고 함. 둘 다 자체 측정치임.

## Mythos 5.1은 이 저장소 범위 밖임

Mythos 5.1은 **같은 기반 모델에 다른 안전장치를 붙인 것**이고, Cyber Verification Program과 Life Sciences Verification Program을 통해서만 접근 가능하며 현재 미국 조직으로 제한됨. 일반 개발 워크플로에 들어올 물건이 아니라 벤치마크 비교 용도로만 언급했음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 캐시 배수가 바뀌면 8가지 파괴 원인의 기대 손실이 같이 바뀜. 가지치기 절은 정책 제한까지 겹침 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 이어붙이기 대 truncate 비용표를 0.025배로 다시 계산한 것이 위에 있음 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 레버 1. 공식 문서 자신이 Opus 5로 시작하라고 적은 것이 근거가 됨 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | thinking을 끌 수 있다는 전제가 Fable 5.1에서는 성립하지 않음 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 툴 목록 결정적 정렬 권고. 캐시 미스의 상대 비용이 오르면 그 권고의 값어치도 오름 |

## 짚어야 할 것

- **벤더가 자기 모델을 발표한 글임.** 벤치마크, 비용 절감률, 안전장치 개선률이 전부 자체 측정치임. 원문이 외부 테스트를 Gray Swan 등에 위탁했다고 적었지만 그 결과 자체는 이 글에 없음
- **가격과 스펙은 공식 문서에서 교차 확인했음.** 캐시 배수, 컨텍스트, effort, thinking, 모델 ID는 발표문이 아니라 Pricing과 Models overview 문서에서 받은 값이고 2026-09-12 기준임
- **write 대 read 비율 50배와 Cowork 표 재계산은 이 문서의 계산임.** 원문에 그 숫자로 적혀 있지 않음. 배수 자체는 공식 문서 값이니 검산 가능함
- **원문이 스스로 한계를 밝힘.** "very long-context work and multi-agent settings"에서 한계가 남아 있다고 적음. 이 저장소가 다루는 서브에이전트 팬아웃 패턴이 정확히 그 구간임
- **실제로 돌려보고 쓴 글이 아님.** 발표문과 공식 문서만 읽고 정리했음. 자기 워크로드에서 Opus 5와 비교한 eval은 각자 돌려야 함
- Claude Code나 Cowork 같은 구독 제품에서 어느 모델을 어느 조건으로 고를 수 있는지는 **확인하지 못했음.** 위 수치는 전부 API 토큰 과금 기준임

## 유효기간

**2026-09-12 확인 기준**임. 가격과 모델 라인업은 계속 움직이는 표라서, 위 배수를 인용하기 전에 [Pricing](https://platform.claude.com/docs/en/about-claude/pricing)의 각주를 다시 봐야 함. 특히 **0.025배가 Fable 5.1과 Mythos 5.1에만 적용되는 예외인지, 이후 모델의 기본값이 되는지**가 이 문서에서 가장 빨리 낡을 부분임. 벤치마크 수치는 시점이 박힌 기록이라 낡지 않지만, 비교 대상이 Fable 5뿐이라 Opus 5 대비 성능은 이 글로 알 수 없음.
