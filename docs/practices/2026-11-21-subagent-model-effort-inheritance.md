---
title: 서브에이전트의 모델과 effort 는 누구에게서 물려받나, 세션이 아니라 부모에서 옴
source: https://code.claude.com/docs/en/sub-agents
collected: 2026-11-21
tags: [claude-code, subagent, effort, llm-cost, orchestration, compaction, prompt-caching]
---

출처: [Subagents, Claude Code Docs](https://code.claude.com/docs/en/sub-agents) · [Model configuration](https://code.claude.com/docs/en/model-config) · [How Claude Code uses prompt caching](https://code.claude.com/docs/en/prompt-caching) (문서 세 건은 **2026-11-21 확인**) · Claude Code 릴리스 노트 [v2.1.317](https://github.com/anthropics/claude-code/releases/tag/v2.1.317) ~ [v2.1.330](https://github.com/anthropics/claude-code/releases/tag/v2.1.330) (릴리스 날짜는 `gh api repos/anthropics/claude-code/releases` 의 `published_at`(UTC) 기준이고 **2026-11-21 에 받음**)

## 요약

이 저장소는 [Claude Code 의 effort 해석 순서](./2026-10-27-claude-code-effort-resolution.md)에서 **세션 하나의** 레벨이 어디서 오는지를 정리했음. 비어 있던 쪽이 **서브에이전트·팀메이트·fork 가 어느 모델, 어느 레벨로 도느냐**임. 지금 문서를 보면 답이 직관과 다름. **모델은 세션이 아니라 자기를 띄운 부모에서 내려오고, effort 는 "부모와 같은 모델일 때만" 세션 레벨이 적용됨.** 모델이 다르면 세션에서 `/effort` 로 올려 둔 값이 **그 서브에이전트에는 아예 적용되지 않고**, 그 모델에 저장해 둔 레벨이나 모델 기본값으로 떨어짐.

그래서 Opus 세션에서 Haiku 리뷰어를 띄우면 그 리뷰어는 **Opus 를 물려받지 않고**, 그 리뷰어가 또 띄우는 에이전트도 **Opus 가 아니라 Haiku** 를 물려받음. 문서가 예시로 못을 박아 뒀음. 비용 쪽 결과가 셋인데 전부 눈에 안 보이는 자리임. **서브에이전트·팀메이트·fork·compaction 은 캐시 TTL 버킷이 메인 대화와 달라서 구독 플랜에서도 5분**을 받고, compaction 은 **메인 세션 창이 아니라 자기 모델 창**에서 걸리며, Haiku 5.5 를 쓰면 그 창이 약 **967K** 라 [100K 를 넘는 순간 단가가 5배가 되는 구간](../models/2026-10-07-claude-haiku-5-5.md)에서 대부분의 수명을 보냄.

그리고 이 귀속 경로가 **11월에 여덟 릴리스에 걸쳐 계속 고쳐졌음.** v2.1.317(2026-11-03)부터 v2.1.330(2026-11-20) 사이에 "서브에이전트가 엉뚱한 모델이나 엉뚱한 effort 로 돌던 것"을 고친 항목을 **열한 건** 셌음(분류는 이 문서가 한 것임). 그중 둘은 **요금이 달라지는** 버그였음. 결론은 기능을 쓰지 말라는 것이 아니고, **중첩 에이전트를 쓰는 구성에서는 모델과 effort 를 정의 파일에 못 박아 두라**는 것임. 물려받기에 기대면 지금은 버전마다 답이 달라짐.

## 서브에이전트의 모델이 결정되는 순서

문서가 네 단계로 적음.

| 순위 | 출처 |
|---|---|
| 1 | 호출별 `model` 파라미터. Agent 툴 호출에서는 **별칭만** 받고 풀 모델 ID 나 `inherit` 는 못 넘김 |
| 2 | 정의 파일의 `model` 필드(`inherit` 포함) |
| 3 | `CLAUDE_CODE_SUBAGENT_MODEL` 환경변수 |
| 4 | 메인 대화의 모델, **또는 중첩 서브에이전트라면 자기를 띄운 서브에이전트의 모델** |

4번 뒷부분이 이 문서의 출발점임. 문서가 `inherit` 를 이렇게 정의함.

> "**`inherit`**: the main conversation's model, or the spawning subagent's model for a nested subagent"

그리고 예시로 한 번 더 못을 박음.

> "For example, suppose your main conversation runs on Opus and starts a reviewer subagent on Haiku. By default, these subagents run on Haiku, not Opus, when the reviewer spawns them"

여기 해당하는 것이 **`model: inherit` 이거나 `model` 필드가 없는 서브에이전트, 내장 Explore·Plan·general-purpose, 그리고 fork** 임. 즉 `inherit` 라는 이름이 "세션을 따른다"로 읽히는데 실제로는 **"직전 부모를 따른다"** 임. 2계층까지는 같은 뜻이지만 3계층에서 갈림.

fork 는 이 순서 밖에 있음. 문서가 fork 와 비일반 서브에이전트를 표로 갈라 두고, fork 의 모델 칸을 "Same as the parent conversation" 으로 적음. `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` 를 켜도 fork 는 부모 모델에 남음.

| | fork | 그 외 서브에이전트 |
|---|---|---|
| 컨텍스트 | 대화 전체를 그대로 물려받음 | 넘긴 프롬프트로 새로 시작 |
| 시스템 프롬프트·툴 | 부모와 동일 | 정의 파일 기준, 백그라운드 실행에서는 걸러짐 |
| 모델 | **부모 대화와 동일** | 위 4단계 순서 |
| 프롬프트 캐시 | **부모와 공유** | 별도 캐시 |

마지막 줄이 값어치임. 문서가 "a fork costs less than a fresh subagent for tasks that need the same context" 라고 적음. 같은 맥락이 필요한 일이라면 **새 서브에이전트보다 fork 가 싸다**는 것이고, 이유는 첫 요청이 부모 캐시를 읽기 때문임. 반대로 fork 는 **또 fork 를 못 띄움**("A fork can't spawn further forks").

## effort 는 모델이 같을 때만 세션에서 내려옴

fork·forked skill·팀메이트를 뺀 서브에이전트의 순서를 문서가 이렇게 적음.

> "A subagent other than a fork, forked skill, or teammate runs at the first of these that applies to it:
> 1. The level in `CLAUDE_CODE_EFFORT_LEVEL`, if you set it
> 2. The `effort` set for the subagent that spawned it, or a level set only for the current turn, such as a skill's `effort`
> 3. The level in `CLAUDE_CODE_SUBAGENT_EFFORT`, if you set it
>
> With none of these, a subagent on the main conversation's model runs at the level you pick in the session, such as with `/effort`, or set with `--effort`. On another model, that level doesn't apply. To choose its level, set the `effort` field."

**2번이 부모의 effort 임.** 모델과 똑같이 세션이 아니라 부모에서 내려옴. 그리고 마지막 두 문장이 함정임.

- 모델이 같으면 세션 레벨이 내려옴
- **모델이 다르면 세션 레벨이 아예 적용되지 않음.** 그 모델에 저장해 둔 레벨이나 모델 기본값으로 떨어짐

Claude Code 의 모델 기본값은 **Fable 5.2, Opus 5.5, Sonnet 5.5, Haiku 5.5 가 `medium`, Opus 4.7 이 `xhigh`, 나머지가 `high`** 임(문서 문언). 그러니까 Opus 5.5 세션을 `/effort xhigh` 로 올려 두고 Haiku 리뷰어를 붙이면, 리뷰어는 `xhigh` 가 아니라 **`medium`** 에서 돎. 올려 둔 쪽은 비싼 레인에 있고, 실제로 판단을 하는 쪽은 기본값에 있음. **설계 의도가 그쪽이었다면 좋은 기본값이고, 모르고 있었다면 품질 사고임.** 둘을 가르는 것은 정의 파일에 `effort` 를 적었는지 하나뿐임.

`CLAUDE_CODE_SUBAGENT_EFFORT` 는 **v2.1.316(2026-10-31)** 에 들어왔고, 릴리스 노트가 "to set the effort level subagents run at when nothing else names one, instead of inheriting the session's" 로 적음. 위 3번 자리임. 호출별 `effort` 파라미터는 **v2.1.292** 이상이고, 정의 파일의 `effort` 필드를 덮으며 **서브에이전트가 resume 될 때도 유지**됨.

갈래가 넷이라 외우기보다 표로 두는 편이 맞음.

| 띄우는 것 | 모델 | effort |
|---|---|---|
| 일반 서브에이전트 | 4단계 순서. 중첩이면 부모 | 위 3단계 순서. 모델이 다르면 세션 레벨 **미적용** |
| fork | 부모 대화와 동일 | 위 순서의 적용 대상이 **아님**(문서가 명시적으로 뺌) |
| 팀메이트 | 서브에이전트와 같은 변수·필드를 씀 | 위 순서의 적용 대상이 **아님** |
| `context: fork` 스킬 | 스킬 프론트매터의 `model` 이 **fork 된 서브에이전트의** 모델을 정함 | 스킬의 `effort` 가 그 fork 의 실행 동안 적용 |

**fork 와 팀메이트의 effort 를 문서가 이 순서에서 빼 두기만 하고 대신 무엇이 적용되는지는 적지 않았음.** 확인할 수 없었음. 모드 쪽 단서는 하나 있음. v2.1.318(2026-11-04)이 mod 의 `$.model.fork` 에 `effort` 옵션을 붙였고 노트가 "so a quick side job can run at `low` whatever effort the session uses" 로 적음. 즉 fork 가 기본적으로 세션 effort 를 쓴다는 전제로 읽히는데, 그것은 mod API 쪽 서술이고 일반 fork 에 그대로 옮길 근거가 아님.

## 11월에 이 경로가 여덟 릴리스에서 고쳐짐

아래는 릴리스 노트에서 **모델·effort 귀속에 해당하는 항목만** 뽑은 것임. 분류와 묶음은 이 문서가 한 것이고 릴리스 노트는 평평한 목록으로 적음.

| 날짜 | 버전 | 무엇이 틀려 있었나 |
|---|---|---|
| 2026-11-03 | 2.1.317 | 다른 모델의 서브에이전트·팀메이트가 **세션 effort 를 쓰고 있었음.** 그 모델의 저장 레벨이나 기본값을 쓰도록 바꿈. 즉 위 "모델이 다르면 미적용" 규칙이 이날 생김 |
| 2026-11-12 | 2.1.323 | 전체화면 헤더가 **다른 모델의 서브에이전트에 세션 effort 레벨을 표시**함. `/model` 이 환경변수가 덮는 레벨을 고른 값으로 보여줌 |
| 2026-11-13 | 2.1.325 | 자기 모델이 지정된 서브에이전트가 `opusplan`·`haiku` 설정에서 **플랜 모드 모델로 끌려감.** 다른 모델의 서브에이전트가 **파일 읽기와 MCP 결과 크기를 메인 세션 모델 기준으로** 셈 |
| 2026-11-16 | 2.1.326 | 전체화면 밖 헤더가 다른 모델의 서브에이전트를 보는 중에 **세션 모델을 표시**함. 관리 설정의 서브에이전트 모델이 **Claude Desktop 의 Code 탭에서 적용되지 않음** |
| 2026-11-17 | 2.1.327 | 인프로세스 팀메이트가 **리드의 컨텍스트 한도에서, 엉뚱한 모델로 compaction** 함. 이제 자기 한도·자기 모델로 하고 **"at that model's rate"** 로 청구됨. 끝난 서브에이전트가 **메시지를 보낸 형제 서브에이전트의 모델과 effort 로** resume 됨 |
| 2026-11-19 | 2.1.328 | Bedrock·Agent Platform·Foundry 가 진행 요약의 effort 설정을 거부하면 **그 세션 동안 해당 모델의 effort 가 꺼져 있었음**(2.1.327 회귀) |
| 2026-11-19 | 2.1.329 | 에이전트 팀에서 **Agent 툴의 `effort` 파라미터가 이름 붙은 팀메이트에게 무시**됨 |
| 2026-11-20 | 2.1.330 | **서브에이전트의 서브에이전트와 fork 가** `opusplan`·`haiku` 플랜 모드에서 플랜 모드 모델로 돎(2.1.325 의 중첩판). `model` 설정이 붙은 스킬이 서브에이전트·fork·compaction·`/recap` 까지 끌고 가던 것을 세션 모델에 남기도록 바꿈. 서브에이전트의 모델 폴백이 **그 아래로 승계**되도록 바꿈 |

**2.1.325 와 2.1.330 이 같은 버그의 1계층판과 2계층판임.** 13일에 "서브에이전트가 플랜 모드 모델로 끌려감"을 고쳤는데, 20일에 "서브에이전트의 서브에이전트와 fork 가 똑같이 끌려감"을 다시 고쳤음. 중첩을 한 단 내려가면 같은 자리가 다시 비어 있었다는 뜻임. [2026-10-22 권한 우회 수정 묶음](../security/2026-10-22-claude-code-permission-bypass-fixes.md)이 셸 문자열 판정에서 본 것과 모양이 같음. **한 층에서 고친 규칙이 다음 층에 자동으로 적용되지 않는 구조**임.

판단은 이렇게 두는 편이 맞음. **열한 건 중 어느 것도 에러를 내지 않았음.** 엉뚱한 모델로 돌거나 엉뚱한 레벨로 돌거나 엉뚱한 한도에서 접혔고, 그중 둘은 청구액이 달라졌음. 그래서 중첩 에이전트 구성에서 **"물려받게 두는 것"은 지금 검증 비용이 가장 큰 선택지**임. 정의 파일에 `model` 과 `effort` 를 적어 두면 위 표의 절반은 애초에 안 걸림.

## 비용이 갈리는 세 자리

### 캐시 TTL 버킷이 다름

캐시 문서가 요청을 두 버킷으로 나눔.

> "**Everything else**: the requests Claude Code makes outside that conversation, such as subagents, workflows, in-process teammates, forks, compaction, and session titles"

| 버킷 | 구독 플랜(한도 내) | 사용 크레딧·API 키·클라우드 |
|---|---|---|
| 메인 대화 | 1시간 | 5분 |
| 그 외 전부 | **5분**(서버가 정하는 일부 헬퍼 요청만 1시간) | 5분 |

즉 **구독 플랜에서 1시간 TTL 을 받는 것은 메인 대화뿐이고, 서브에이전트·팀메이트·fork·compaction 은 5분임.** 서브에이전트가 5분 넘게 생각하거나 긴 빌드를 기다리면 다음 요청이 캐시 미스로 떨어짐. 바꾸는 레버가 따로 있음: `subagentPromptCacheTtl` 설정 또는 `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` 환경변수(둘 다 **v2.1.242** 이상), 그리고 서브에이전트 `experimental` 프론트매터의 `cacheTtl`(**v2.1.248** 이상). 마지막 것은 **구독이 사용 크레딧을 쓰는 동안에는 `1h` 가 무시됨.**

[Cache diagnostics](./2026-09-23-prompt-cache-diagnostics.md)가 히트율을 보는 법을 정리했는데, 거기에 더해 알아 둘 것이 **`/usage` 의 `Prompt cache (main)` 줄이 이름 그대로 메인 대화만 센다**는 것임. 서브에이전트를 많이 띄우는 구성에서는 그 줄이 좋아 보여도 비용이 다른 데서 나감.

### compaction 이 자기 모델 창에서 걸림

서브에이전트는 자기 컨텍스트 창이 차면 접힘. 정의 파일에 `autoCompactWindow` 를 쓸 수 있고(**v2.1.296** 이상), 주의할 점이 둘임.

- **프론트매터에서는 `200k` 가 200 으로 읽혀 최소값 미달이 됨.** 조용히 무시되고 필드가 없는 것처럼 돎. `200000` 으로 쓸 것. 무시됐는지는 `claude --debug` 의 로그에서 `invalid autoCompactWindow` 로 확인
- `CLAUDE_CODE_AUTO_COMPACT_WINDOW` 를 같이 쓰면 **프론트매터 값·모델 컨텍스트 창·변수 값 중 가장 작은 것**이 적용됨. **v2.1.312** 이전에는 변수가 창을 정하는 동안 프론트매터가 아예 안 먹었음

여기에 2.1.327 의 수정("팀메이트가 리드 한도에서 엉뚱한 모델로 compaction")과 2.1.326 의 변경을 겹쳐 봐야 함. 후자가 이럼. **플랜 모드의 자동 compaction 이 답하는 모델, 즉 `opusplan` 이면 Opus 를 쓰게 됐고 노트가 "so it can cost more" 라고 직접 적음.** compaction 은 요약 요청을 따로 보내는 일이라 어느 모델로 도는지가 그대로 요금임.

### Haiku 서브에이전트는 창의 앞 10%만 싸다

이게 조합에서 나오는 문제임. Haiku 5.5 는 **1M 창**을 가지고 기본적으로 **약 967K** 에서 접힘. 그리고 [Claude Haiku 5.5](../models/2026-10-07-claude-haiku-5-5.md)에 정리한 대로 **프롬프트가 100K 를 넘으면 입력·출력·캐시 단가가 전부 5배**로 갈림. 즉 긴 로그를 읽는 Haiku 서브에이전트를 기본값으로 두면, **100K 를 넘긴 뒤 867K 구간을 5배 요금으로 달린 다음에 접힘.** 싼 모델을 골랐다고 생각한 자리에서 요금이 가장 크게 어긋남.

막는 법은 위 두 절에 이미 있음. 정의 파일에 `autoCompactWindow: 100000` 을 적어 **가격 분기점 근처에서 접히게** 하는 것임. 다만 이것은 두 문서의 사실을 조합한 결론이고 **문서가 이 조합을 적어 두지는 않았음.** 토큰 수를 실제로 재서 확인한 것도 아님.

## 깊이 제한과 강제 설정

중첩이 무한하지 않음. 기본값이 **메인 대화 아래 3계층**이고 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` 로 바꿈(`1` 로 두면 중첩 꺼짐). 한계에서의 동작이 fork 만 다름.

> "At the depth limit, Claude Code withholds the `Agent` tool from every subagent except a fork, so a subagent at the limit does its delegated work itself and returns one summary. A fork at the limit keeps `Agent` in its inherited tool list, but the tool returns an error instead of spawning."

**한계에 닿은 일반 서브에이전트는 Agent 툴이 아예 안 보이므로 자기가 일을 하고, fork 는 툴이 보이는 채로 호출이 실패함.** fork 는 부모의 툴 풀을 그대로 물려받는 구조라 그렇게 된 것으로 읽히는데, 모델에게는 **있는 줄 알았던 툴이 에러를 내는** 모양이라 재시도 토큰을 쓸 자리임.

전부 한 모델로 묶으려면 `CLAUDE_CODE_SUBAGENT_MODEL` 에 모델을 넣고 `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` 를 `1` 로 둠(**v2.1.257** 이상). 켜 두면 정의 파일의 `model` 을 무시하고 Claude 도 모델을 못 넘김. 예외가 둘임.

- **fork**
- **`model: inherit` 로 서브에이전트에서 도는 스킬**

그리고 `FORCE` 만 켜고 모델을 안 정하면 서브에이전트는 메인 대화 모델로 돌되 **내장 Explore 는 자기 모델로 돎.** 비용 상한을 하네스 설정으로 긋는 구성이면 이 셋이 구멍임.

모델 폴백 쪽도 서브에이전트에 걸림. 문서가 적는 것이 둘인데 한쪽만 알림이 뜸.

> "In interactive sessions, Claude Code warns you when it substitutes a top-level subagent's model. It doesn't report a teammate's fallback."

**팀메이트의 폴백은 보고되지 않음.** 그리고 v2.1.330 이 폴백을 **그 아래 계층까지 승계**하도록 바꿨으므로, 팀메이트 하나가 조용히 폴백하면 그 아래 전부가 폴백 모델에서 도는 구성이 됨.

## iOS 작업에서 어디를 고정할까

Xcode 프로젝트에 붙이는 전형적인 모양이 "메인은 Opus, 빌드·테스트·로그 읽기는 싼 모델" 임. 위 규칙을 그 모양에 대고 읽으면 고정해야 할 자리가 넷임.

1. **빌드·로그 서브에이전트 정의에 `model` 과 `effort` 를 같이 적음.** 모델만 적으면 effort 는 그 모델 기본값으로 떨어지고, 세션에서 올린 값은 안 내려옴
2. **로그를 읽는 쪽에 `autoCompactWindow` 를 숫자로 적음.** `xcodebuild` 출력과 시뮬레이터 로그는 창을 가장 빨리 채우는 입력임. `200k` 가 아니라 `200000`
3. **중첩을 쓰면 3계층을 전제로 설계하지 말고 깊이를 명시적으로 줄임.** 리뷰어가 또 에이전트를 띄우는 구성이면 그 손자는 리뷰어 모델을 물려받음. 메인을 Opus 로 올려도 손자는 안 올라감
4. **같은 맥락을 다시 쓰는 일은 새 서브에이전트가 아니라 fork 로 보냄.** 부모 캐시를 읽으므로 문서가 직접 "costs less" 라고 적은 자리임. 단 fork 는 또 fork 를 못 띄움

[대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md)의 레버 2(라우팅)를 하네스 안에서 하는 것이 결국 이 설정들임. 그 글이 게이트웨이에서 라우팅하라고 적은 것과 층이 다르고, **라우팅 결과의 effort 와 compaction 창까지 같이 정하지 않으면 라우팅으로 깎은 만큼이 다른 데서 나간다**는 것이 이 문서가 더하는 부분임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Code 의 effort 해석 순서](./2026-10-27-claude-code-effort-resolution.md) | 세션 하나의 해석 순서. 그 문서가 `CLAUDE_CODE_SUBAGENT_EFFORT` 를 한 줄로 적어 둔 자리가 이 문서 전체임. 모델 기본값 목록에 Fable 5.2 가 `medium` 으로 추가됐음 |
| [Claude Haiku 5.5](../models/2026-10-07-claude-haiku-5-5.md) | 100K 가격 분기와 1M 창. 서브에이전트 기본 compaction 창과 겹쳐 보면 싼 모델이 비싸지는 구간이 나옴 |
| [Cache diagnostics](./2026-09-23-prompt-cache-diagnostics.md) | 히트율을 보는 법. `/usage` 의 캐시 줄이 메인 대화만 센다는 점이 여기 붙음 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 에이전트 캐시 설계. fork 가 부모 prefix 를 읽고 일반 서브에이전트가 별도 캐시를 쓰는 구조가 그 글의 전제와 같은 자리 |
| [대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md) | 라우팅 레버를 하네스 설정으로 내린 것. 라우팅만 하고 effort·compaction 창을 안 정하면 새는 자리 |
| [Claude Code 2.1.309 의 권한 우회 수정 묶음](../security/2026-10-22-claude-code-permission-bypass-fixes.md) | 한 층에서 고친 규칙이 다음 층에 적용되지 않아 계열째 재발하는 모양. 2.1.325 와 2.1.330 이 같은 버그의 1계층판·2계층판임 |
| [Managed Agents 다이내믹 워크플로](../agents/2026-10-09-managed-agents-dynamic-workflows.md) | 서버측 run 의 계층. 이쪽은 로컬 하네스라 깊이 제한과 기본값을 섞으면 안 됨 |
| [자기개선 에이전트 루프](./2026-08-10-self-improving-agent-loops.md) | 서브에이전트에 역할을 갈라 맡기는 구성. 역할을 갈랐으면 모델과 effort 도 같이 갈라 적어야 한다는 것이 여기서 설정 계층 문제로 바뀜 |

## 짚어야 할 것

- **공식 문서와 릴리스 노트만 읽었고 직접 돌려보지 않았음.** 중첩 서브에이전트가 실제로 어느 모델 ID 로 요청을 보내는지, 어느 레벨이 실려 나가는지는 요청을 들여다봐야 하고 그것은 하지 않았음. **확인할 수 없었음**
- **fork 와 팀메이트의 effort 가 무엇으로 정해지는지 문서가 적지 않음.** 서브에이전트 effort 순서에서 셋을 명시적으로 뺐을 뿐이고 대체 규칙이 없음. 이 문서가 든 mod 쪽 단서는 `$.model.fork` API 서술이라 일반 fork 에 옮길 근거가 아님
- **"열한 건"과 계열 분류는 이 문서가 한 것임.** 릴리스 노트는 묶어 주지 않고 평평한 목록으로 적음. 어디까지를 "모델·effort 귀속" 으로 볼지에 따라 수가 달라짐. 표시용 버그(헤더가 엉뚱한 모델·레벨을 보여준 것) 셋을 포함해 센 값이고, 빼면 여덟 건임
- **Haiku 서브에이전트가 5배 구간에서 867K 를 달린다는 계산은 두 문서의 사실을 조합한 것임.** 문서가 이 조합을 적어 두지 않았고 실제 토큰을 재지도 않았음. 100K 분기와 약 967K 기본 compaction 지점이 각각 1차 출처에 있다는 것까지가 확인된 것임
- **각 수정 항목이 언제부터 있던 동작인지는 노트에 없음.** 노출 기간을 계산할 수 없고, 특히 2.1.328 이 고친 effort 꺼짐은 노트가 "regression in 2.1.327" 이라고 적어 **이틀**이라는 상한만 확인됨
- **`CLAUDE_CODE_SUBAGENT_EFFORT` 가 model-config 문서에 없음.** 서브에이전트 문서의 순서 목록과 v2.1.316 릴리스 노트에만 있음. 두 문서의 effort 절이 같은 규칙을 다른 범위로 적고 있어, 어느 쪽이 낡았을 가능성을 배제하지 못함
- **릴리스 노트에 날짜가 없음.** 저장소 `CHANGELOG.md` 도 항목에 날짜가 없어서 이 문서의 날짜는 전부 `gh api` 로 받은 릴리스 `published_at`(UTC) 기준임
- 원문 문서 본문에 독자나 도구를 향한 지시문·명령문은 없었음

## 유효기간

**2026-11-21 확인 기준이고 Claude Code 최신 릴리스가 v2.1.330 시점임.** 버전 번호는 하루에 한두 개씩 올라가니 읽는 시점에 이미 낡았다고 보는 게 맞음. 위 수정 표를 갱신하려 들지 말고 `gh api repos/anthropics/claude-code/releases` 로 받아 `subagent`·`teammate`·`fork`·`effort`·`model` 줄만 훑는 편이 빠름. 규칙 쪽은 [Subagents](https://code.claude.com/docs/en/sub-agents) 의 모델·effort 순서 목록 둘을 다시 읽는 것으로 대조됨. 다시 볼 때 확인할 것 셋. **fork 와 팀메이트의 effort 규칙이 문서에 적혔는지**, **`CLAUDE_CODE_SUBAGENT_EFFORT` 가 model-config 쪽에도 올라가 두 문서가 맞춰졌는지**, 그리고 **중첩 귀속 수정이 11월 이후로도 계속 쌓이는지**. 셋째가 쌓인다면 "물려받게 두지 말고 적어 둔다"는 이 문서의 결론을 더 세게 읽어야 함.
