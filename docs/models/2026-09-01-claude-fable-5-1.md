---
title: Claude Fable 5.1, 캐시 읽기가 0.025배가 되고 히스토리 편집이 400이 됨
source: https://www.anthropic.com/claude-fable-and-mythos-5-1
author: Anthropic
published: 2026-09-01
collected: 2026-09-07
tags: [claude, model-release, prompt-caching, effort, thinking, llm-cost, coding-agent]
---

출처: [Introducing Claude Fable 5.1 and Claude Mythos 5.1](https://www.anthropic.com/claude-fable-and-mythos-5-1) (2026-09-01) · [What's new in Claude Fable 5.1](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1) · [모델 페이지](https://platform.claude.com/docs/en/models/fable-5-1/overview) · [Models overview](https://platform.claude.com/docs/en/about-claude/models/overview) · [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) · [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) (2026-09-07 확인)

## 요약

2026-09-01 릴리스. 벤치마크보다 이 저장소에 걸리는 게 셋임. 첫째, **캐시 읽기가 기본 input 가격의 0.025배**로 내려감. 다른 Claude 모델은 전부 0.1배인데 Fable 5.1과 Mythos 5.1만 예외임. 둘째, **대화 중간에 effort를 바꿔도 프롬프트 캐시가 깨지지 않음**(베타). 이 저장소가 여러 문서에서 "effort 변경은 툴 변경과 같은 효과"라고 적어둔 대목이 이 모델에서는 갈림. 셋째, **앞 턴을 편집하면 400 에러**가 남. 히스토리 append-only가 비용 권고가 아니라 API 계약이 됐음. 그리고 이 모델은 기본값이 아님. 공식 문서가 **"For most workloads, start with Claude Opus 5"**라고 적고, Fable 5.1은 장시간 에이전틱 작업이나 Opus 5로 eval이 안 나올 때 쓰라고 함.

## 기본 정보

| | |
|---|---|
| 모델 ID | `claude-fable-5-1` (Bedrock은 `anthropic.claude-fable-5-1`) |
| 릴리스 | 2026-09-01. 은퇴는 **2027-09-01 이전은 아님** |
| 컨텍스트 / 최대 출력 | 1M 토큰 / 128K 토큰 |
| 가격 | input **$10** / output **$50** / 5m cache write $12.50 / 1h cache write $20 / **cache read $0.25** (MTok) |
| thinking | adaptive **always on**. 끌 수 없음 |
| 기본 effort | `high`. `low`~`max` 존재 |
| 지식 컷오프 | 2026-06 |
| 데이터 보존 | **30일 고정.** 별도 승인 없이는 zero data retention으로 못 씀 |
| Mythos 5.1 | 같은 모델, 같은 가격. Project Glasswing 참가자 전용 |

Fable 5.1과 Mythos 5.1은 능력이 같고 **세이프가드 수준만 다름**. 이 저장소가 다루지 않는 축(사이버·생명과학 심사 프로그램)이라 아래는 전부 Fable 5.1 기준임.

## 캐시 읽기 0.025배, 이 저장소의 산술이 모델별로 갈림

공식 Pricing 문서 각주가 이렇게 적음.

> "Cache reads (hits and refreshes) cost 0.025 times the base input price on these models, compared with 0.1 on other Claude models."

배수를 표로 놓으면 이럼. 전부 기본 input 가격 대비임.

| 항목 | 다른 Claude 모델 | Fable 5.1 · Mythos 5.1 |
|---|---|---|
| cache read | 0.1배 | **0.025배** |
| 5분 cache write | 1.25배 | 1.25배 (그대로) |
| 1시간 cache write | 2배 | 2배 (그대로) |
| 최소 캐시 길이 | 512토큰 | 512토큰 (그대로) |

**쓰기는 그대로인데 읽기만 4분의 1이 됨.** 그래서 "긴 prefix를 오래 재사용하는" 워크로드에 유리하게 기울고, 반대로 히스토리를 잘라 다시 올리는 행위는 상대적으로 더 비싸짐. 문서가 그 함의를 직접 적음.

> "Because cache reads are now cheaper, compacting early to save cost may no longer be the right cost-intelligence tradeoff on Claude Fable 5.1, so experiment with later compaction points."

[Claude Cowork](../practices/2026-08-10-claude-cowork.md)에 "이어붙일까 truncate할까"를 캐시 배수로 따진 표가 있음. 그 표의 예시(prefix 5k, suffix 25k, 한 턴 더)에 **이 모델의 공식 배수를 다시 넣으면** 이렇게 됨. 아래는 원문 수치가 아니라 위 배수로 다시 계산한 값임.

| 선택 | 0.1배 read 모델 | Fable 5.1 (0.025배 read) |
|---|---|---|
| 이어붙이기 (30k read) | 3k | **0.75k** |
| truncate, 되돌릴 지점 캐시 히트 (5k read) | 0.5k | **0.125k** |
| truncate, 그 지점 캐시 없음 (5k write) | 6.25k | 6.25k (변화 없음) |

판단 기준 자체는 그대로임. **되돌릴 지점에 캐시가 살아 있느냐**가 갈림길이고, 살아 있으면 truncate가 쌈. 달라진 건 폭임. 캐시가 없는 지점으로 되돌릴 때의 손해가 이 모델에서 훨씬 커짐. 원리는 [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 "가지치기의 역설" 그대로인데, 계수가 바뀌었으니 자기 하네스에서 다시 재야 함.

## effort를 대화 중간에 바꿔도 캐시가 안 깨짐 (베타)

이 저장소에 여러 번 적어둔 문장이 있음. reasoning level(effort)을 바꾸면 툴 정의를 바꾼 것과 같은 효과로 프롬프트 캐시가 깨진다는 것. **Fable 5.1에서는 그렇지 않음.**

> "On Claude Fable 5.1 you can change the effort level mid-conversation without invalidating the prompt cache."

`messages` 안에 `output_config`만 담은 `role: "system"` 항목을 끼워 넣는 방식이고, 새 레벨은 **다음 user 턴부터** 적용됨. 베타 헤더 `mid-conversation-output-config-2026-07-01`이 필요하고, 지원 모델은 **Fable 5.1, Mythos 5.1, Opus 5**이며 Claude API 한정임.

```json
{"role": "system", "content": [], "output_config": {"effort": "low"}}
```

실무 의미가 큼. 어려운 단계만 올리고 반복 단계는 내리는 게 캐시 손해 없이 됨. 문서가 권하는 사용처도 그쪽임. `low`에서 검색 툴을 덜 부르는 문제를 **해당 턴만 effort를 올려서** 푸는 것.

⚠️ 다만 세 가지를 갈라야 함. 이건 **베타**이고, **Claude API에서만**이고, **모델 셋에만** 해당함. 다른 모델이나 게이트웨이를 쓰면 기존 결론(effort 변경은 prefix를 바꿈)이 그대로 유효함. 이 저장소의 다른 문서를 이 하나로 덮어쓰면 안 됨.

## 히스토리 편집이 비용 문제에서 에러로 바뀜

가장 크게 바뀐 지점임. Fable 5.1의 thinking 블록은 **그것을 만든 대화에만 유효**함. 시스템 프롬프트든 툴 목록이든 앞선 메시지든, thinking 블록보다 앞에 있는 것을 고치면 다음 요청이 400으로 떨어짐. 에러 메시지가 `The block is bound to a different conversation`임.

깨뜨리는 패턴을 문서가 나열함.

- 뒤 턴을 남긴 채 앞 턴을 편집·재정렬·삭제
- 요청마다 앞 턴에 리마인더나 상태 줄을 끼워 넣었다가 다음 요청에서 빼는 것
- 같은 대화 안에서 top-level `system`이나 `tools` 배열의 **내용이나 순서를 바꾸는 것**
- 나중 요청에서 다른 바이트를 내주는 이미지·문서 URL (URL이 아니라 **바이트**를 봄)

세 번째 항목은 오해하기 쉬움. 원문 표현은 "Rebuilding the top-level `system` prompt or `tools` array"인데, Messages API는 stateless라 **매 요청에 같은 `system`·`tools`를 다시 실어 보내는 것은 정상이고 그 자체는 위반이 아님.** 검사가 보는 것은 직렬화된 prefix가 앞 요청과 달라졌는지임. 원문도 처방 쪽에서는 "rather than editing `system` or `tools`", "move `system` and `tools` changes to mid-conversation system messages"라고 **바꾸는 것**을 문제로 적음. 이 줄을 "배열을 보내지 말라"로 읽으면 필수 정의를 빼서 오히려 망가짐.

반대로 유효하게 남는 것들도 명시함. 앞에서부터 연속으로 thinking 블록을 걷어내는 것, 서버측 compaction·context editing, `cache_control` 마커 이동, 그리고 **요청 사이에 `effort`를 바꾸는 것**.

여기서 이 저장소 관점의 핵심은 목록이 겹친다는 점임. 문서가 그걸 직접 말함.

> "The history edits that trip the check are the same ones that restart the prompt cache."

즉 [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 **비용 때문에 하지 말라**고 했던 행위 목록이, 이제 **에러가 나는 행위 목록**과 거의 같아짐. append 지향 트랜스크립트가 취향이나 절약이 아니라 API 계약이 된 것임.

적용 범위에 단서가 둘 있음. **2026-08-31 이후 생성된 계정에만 강제**되고, 그 이전 계정은 요청이 `thinking.block_binding.prefix_mismatch_behavior`를 지정할 때만 동작함. 그리고 Mythos 5.1은 이 검사를 돌리지 않음. 다만 프롬프팅 문서가 "Future models are expected to enforce this check for all accounts"라고 적었으니 지금 맞춰두는 게 맞음.

Claude Code, claude.ai, Claude Managed Agents, Claude Agent SDK는 prefix를 알아서 지켜줌. **직접 `messages` 배열을 만드는 코드만 점검하면 됨.** 점검 방법도 문서가 제시함. `prefix_mismatch_behavior: "drop_block"`으로 한 세션 돌리고 `input_transformations`를 로깅하는 것.

## 그 밖의 API 변경

| 변경 | 내용 |
|---|---|
| **강제 툴 호출 제거** | `tool_choice`가 `{"type": "any"}` 또는 `{"type": "tool", ...}`이면 400. thinking이 항상 켜져 있는데 강제 호출이 그걸 건너뛰기 때문이라고 설명함. 대안은 `auto` + strict tool use, 또는 structured outputs |
| **thinking 블록의 단방향성** | Fable 5.1은 이전 모델의 thinking을 읽지만 이전 모델은 Fable 5.1의 것을 못 읽음. 라우터나 폴백이 모델을 갈아타면 API가 블록을 **조용히 버림**(과금은 안 됨). `thinking-binding-controls-2026-08-01` 헤더를 붙여야 `input_transformations`로 보고됨 |
| **턴 한정 시스템 메시지** (베타) | `clear_at: "next_user_message"`. 한 턴만 시스템 권한으로 읽히고 그 뒤로는 렌더링되지 않는데 **`messages`에는 그대로 남음.** 그래서 캐시도 유지되고 thinking 블록도 안 깨짐. 헤더는 `mid-conversation-system-clear-at-2026-08-21` |
| **툴 호출 사이 진행 상황** (베타) | 기본 `thinking.display`가 `"omitted"`라 진행 업데이트 블록이 빈 값으로 옴. `display: "updates"`(헤더 `thinking-display-updates-2026-08-18`)로 받으면 추론은 감춘 채 상태 줄만 텍스트로 받음 |
| **콘텐츠 출처 표시** | 모든 텍스트 출력에 통계적 워터마크가 들어감. 코드 실행 툴이 만든 이미지·비디오·오디오는 Files API로 받을 때 **C2PA Content Credentials** 서명이 붙음. 토큰이 늘거나 숨은 문자가 들어가지는 않는다고 명시 |

`temperature`·`top_p`·`top_k` 비기본값 금지, assistant 프리필 금지, `thinking: {"type": "disabled"}` 금지는 Fable 5와 동일하게 유지됨.

## 토큰을 더 쓰게 되는 동작 변화

코드 한 줄 안 바꿔도 드러나는 차이들임. 품질 문제가 아니라 **비용·시간 문제**라 여기 따로 모음.

- **에이전트 루프에서 툴 호출이 턴당 하나로 쪼개질 수 있음.** 요청이 가져올 것들을 명시적으로 호명하면 병렬로 가는데, 다음 호출이 과제에서 암시만 되는 커스텀 코딩 에이전트·bash 하네스·computer use에서 하나씩 나감. 답 품질은 안 떨어지고 **턴·토큰·벽시계 시간만 듦.** 문서가 주는 한 줄 처방: `First privately list what you need next; then request every item that doesn't depend on another's result in this one response.`
- **작은 수정에도 파일 전체를 다시 쓰는 경향.** 결과 파일은 대개 같은데 출력 토큰과 시간이 더 듦. 처방도 한 줄임: `The number of tokens used to edit files is best minimized, all else being equal.`
- **`xhigh`·`max`에서 긴 산출물을 thinking 안에 한 번 쓰고 응답으로 또 씀.** 문서 권고는 `high`에서 시작하고 측정된 이득이 있을 때만 올리는 것
- **`low`에서 검색 툴을 덜 부르고 기억으로 답함.** 위의 대화 중 effort 변경이 여기 처방으로 제시됨

effort 자체도 다시 재야 함. **레벨 이름이 모델 간에 같은 사고량을 뜻하지 않음**이라고 명시하고, `medium`이 대략 Fable 5 수준을 더 싸게 낸다고 적음.

## 벤치마크

원문 발표가 실은 수치임. **최대 effort 기준**이고 **Anthropic 자체 발표**라 방향만 볼 것.

| 벤치마크 | Fable 5.1 | Fable 5 |
|---|---|---|
| Terminal-Bench-Science 0.1 | 52.6% | 24.7% |
| Terminal-Bench 4.0 | 55.8% | 42.0% |
| CursorBench 3.2.0 | 73.4% | 70.5% |
| Humanity's Last Exam (툴 사용) | 65.0% | 63.8% |
| OSWorld 2.0 (strict) | 41.7% | 36.1% |

비용은 "전형적 워크로드에서 Fable 5 대비 약 **25% 절감**", 에이전틱 비중이 높으면 "**최대 약 45%**"라고 적음. 근거는 입출력 가격이 아니라 **캐시 읽기 인하**임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 히트율 낮을 때 볼 8가지 중 "reasoning level 변경"이 이 모델에서는 해당 안 됨. 반대로 "메시지 수정 익스텐션"은 이제 비용이 아니라 400 에러가 됨 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 이어붙이기 대 truncate 계산의 계수가 바뀜. 판단 기준은 그대로고 폭만 커짐 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | 모델별 프롬프팅 가이드가 같은 형식으로 나옴. 다만 방향이 반대인 항목이 있음. Opus 5는 내레이션을 줄이라는 쪽인데 Fable 5.1은 오히려 **요청해야 나옴** |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 레버 1 그대로임. 새 모델이 나왔다고 갈아타는 게 아니라 자기 워크로드로 재보는 것. 공식 문서 자신이 Opus 5를 기본으로 권함 |
| [Memory Engineer](../practices/2026-08-02-memory-engineer.md) | 클라이언트측 요약·압축이 이제 에러 표면을 만듦. 서버측 compaction과 context editing은 검사를 통과함 |
| [Graph Engineering](../practices/2026-07-20-graph-engineering.md) | 서브에이전트를 기다리지 말고 리드 에이전트를 계속 돌리라는 권고가 문서에 들어옴. 조율을 코드로 접는 발상과 같은 방향 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 대화 중간에 툴을 바꾸는 경로가 `tools` 배열 재작성이 아니라 mid-conversation 툴 변경으로 정리됨. 점진적 툴 발견 논의와 붙여 읽을 것 |

## 짚어야 할 것

- **이 모델은 기본값이 아님.** 공식 문서가 대부분의 워크로드에는 Opus 5로 시작하라고 적음. Fable 5.1은 input이 **2배**($5 대 $10), output도 **2배**($25 대 $50)이고 지연시간도 더 김. 캐시 읽기만 싼 것이지 전반적으로 싼 모델이 아님
- **베타 헤더 셋에 의존하는 기능이 많음.** 대화 중 effort 변경, 턴 한정 시스템 메시지, 진행 상황 표시가 전부 베타임. 정식화 시점은 문서에 없음
- **히스토리 편집 검사는 계정 생성일로 갈림.** 2026-08-31 이후 계정에만 강제됨. 옛 계정에서 안 걸린다고 안전한 게 아니라 **아직 안 켜진 것**으로 읽어야 함
- **벤치마크는 자체 발표 수치임.** 대조군이 Fable 5 하나이고 Opus 5와의 비교표는 발표에서 확인하지 못했음
- **비용 절감 25%·45%는 "추정"으로 적혀 있음.** 워크로드 구성에 따라 갈리고, 캐시 히트율이 낮으면 성립하지 않음
- Claude Code나 claude.ai에서 이 모델을 어떻게 고르는지, 구독 플랜에서 어떤 한도로 쓰이는지는 **이번에 확인하지 않았음.** 위 내용은 전부 Claude API 기준임
- 시스템 카드는 링크만 확인했고 **본문은 읽지 않았음**

## 유효기간

**2026-09-07 확인 기준**임. 가격표와 베타 헤더는 계속 움직이니 다시 볼 때는 [Pricing](https://platform.claude.com/docs/en/about-claude/pricing)의 캐시 각주와 [What's new](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1)의 베타 표시부터 대조할 것. 특히 확인할 것 둘. **대화 중 effort 변경이 베타를 벗어났는지**, 그리고 **히스토리 편집 검사가 전체 계정으로 확대됐는지**. 후자는 문서가 이미 예고해뒀음.
