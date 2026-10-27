---
title: Claude Code 의 effort 해석 순서, ultrathink 가 한 턴만 xhigh 로 올리는 자리
source: https://code.claude.com/docs/en/model-config
author: Anthropic
collected: 2026-10-27
tags: [claude-code, effort, prompt-caching, llm-cost, coding-agent, context-engineering]
---

출처: [Model configuration, Claude Code Docs](https://code.claude.com/docs/en/model-config) · [Claude Code v2.1.311 릴리스 노트](https://github.com/anthropics/claude-code/releases/tag/v2.1.311) (`gh api` 로 받은 `published_at` 이 **2026-10-26T21:51:23Z**) · [Effort, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/effort) · [Prompt caching 의 무효화 표](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) (문서 확인 **2026-10-27**)

## 요약

이 저장소는 effort 를 **API 파라미터**로만 다뤄 왔음. [Claude Opus 5.5](../models/2026-09-22-claude-opus-5-5.md), [Claude Sonnet 5.5](../models/2026-09-28-claude-sonnet-5-5.md), [Claude Haiku 5.5](../models/2026-10-07-claude-haiku-5-5.md) 셋이 `output_config.effort` 를 각각 적었고, 캐시가 깨지는 조건도 [Cache diagnostics](./2026-09-23-prompt-cache-diagnostics.md)에 정리돼 있음. 비어 있던 쪽이 **하네스가 그 값을 어떻게 정하느냐**임. Claude Code 는 레벨을 네 군데에서 끌어와 한 줄로 줄이고, 그 과정에서 **API 기본값과 다른 값을 보내는 모델이 둘** 있음.

그리고 **v2.1.311 (2026-10-26) 에서 `ultrathink` 가 프롬프트 문구에서 effort 조작으로 올라갔음.** 지금까지는 "깊게 생각하라"는 **인컨텍스트 지시**를 더하는 키워드였는데, 이제 터미널 프롬프트에 치면 **그 턴만 `xhigh`** 로 돈다고 문서가 적음. 대상은 **Opus 5.5, Sonnet 5.5, Haiku 5.5, Fable 5.1** 이고 Anthropic API 경로 한정임. 실무에서 먼저 물어야 하는 것은 기능이 아니라 캐시임. **최상위 effort 를 턴 사이에 바꾸면 메시지 캐시가 통째로 깨지는데**, 한 턴만 올린다는 것은 올릴 때와 내릴 때 두 번 깨진다는 뜻이기도 함. 네 모델이 전부 per-message effort 지원 목록 안에 있다는 점이 그 경로를 썼으리라는 **정황**이지만, Claude Code 가 어느 경로로 보내는지는 **두 문서 어디에도 없고 확인할 수 없었음.**

## 레벨과 모델별 지원

Claude Code 문서가 표로 적어 둔 것임.

| 모델 | 받는 레벨 |
|---|---|
| Fable 5.1, Fable 5 | `low`, `medium`, `high`, `xhigh`, `max` |
| Opus 5.5, Sonnet 5.5, Haiku 5.5, Opus 5, Sonnet 5, Opus 4.8, Opus 4.7 | `low`, `medium`, `high`, `xhigh`, `max` |
| Opus 4.6, Sonnet 4.6 | `low`, `medium`, `high`, `max` |

마지막 줄이 함정임. Platform 쪽 문서가 못을 박아 둠.

> "Not every model that supports `max` supports `xhigh`."

즉 **Opus 4.6 과 Sonnet 4.6 은 `max` 는 받고 `xhigh` 는 안 받음.** 레벨이 순서대로 늘어선 눈금이라고 생각하면 이 둘에서 어긋남. 안 받는 레벨을 설정해도 에러가 아니라 조용히 내려감.

> "If you set a level the active model does not support, Claude Code falls back to the highest supported level at or below the one you set. For example, `xhigh` runs as `high` on Opus 4.6."

**`xhigh` 를 박아 두고 모델만 갈아타는 설정이 제일 위험함.** Opus 4.6 으로 떨어지면 `high` 로 조용히 내려가고, 로그에 `xhigh` 라고 적힌 채로 돎.

## 해석 순서

문서가 세 단계로 적고, 거기에 스킬·서브에이전트 프론트매터가 하나 더 얹힘.

| 순위 | 출처 |
|---|---|
| 1 | 명시적 선택: `CLAUDE_CODE_EFFORT_LEVEL` 환경변수, `--effort` 로 실행, 세션 안의 `/effort` |
| 2 | 설정: 모델별로 저장한 레벨 또는 `effortLevel` 키 |
| 3 | 모델의 기본 effort |

프론트매터는 이 줄 밖에 있음.

> "Frontmatter effort applies when that skill or subagent is active, overriding the session level but not the environment variable."

**환경변수는 못 이기고 세션 레벨은 이김.** 서브에이전트를 `low` 로 묶어 비용을 깎는 구성이 `CLAUDE_CODE_EFFORT_LEVEL` 을 쓰는 CI 러너에서는 안 먹는다는 뜻임. [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)가 "서브에이전트 상한을 지시로 적어라"고 한 항목이 여기서는 설정 계층 문제로 바뀜.

저장 범위도 갈림. 같은 `/effort` 가 확인 키에 따라 다르게 남음.

| 조작 | 남는 범위 |
|---|---|
| `/effort` 슬라이더나 `/model` 피커에서 `Enter`, 또는 `/effort <레벨>` 타이핑 | **기본값으로 저장됨.** 다음 세션에도 적용 |
| 슬라이더나 피커에서 `s` | 이 세션만. **v2.1.257 이상** 필요 |
| `-p` 실행 안의 `/effort` | 그 세션만. 기본값으로 저장되지 않음 |
| `/effort auto` | 활성 모델에 저장해 둔 레벨을 지움 |

`max` 는 따로 취급됨. `CLAUDE_CODE_EFFORT_LEVEL` 로 넣은 경우가 아니면 **현재 세션에만** 적용되고, `effortLevel` 과 `modelSettings` 키는 `max` 를 아예 안 받음. 그리고 Opus 5.5 에만 걸리는 예외가 하나 있음. **사용자 설정 파일의 최상위 `effortLevel` 은 Opus 5.5 에 안 세어짐.** 그 모델은 모델별로 지정하지 않으면 `medium` 에서 출발함.

## `ultrathink`, v2.1.311 에서 레벨을 건드리게 됨

문서가 두 동작을 갈라 적음.

> "Include `ultrathink` anywhere in your prompt to request deeper reasoning on that turn without changing your session effort setting. Claude Code recognizes the keyword and adds an in-context instruction. Claude Code passes other phrases such as "think", "think hard", and "think more" through as ordinary prompt text and doesn't recognize them as keywords."

> "Typed in the terminal prompt, the word can also raise that turn to `xhigh` effort. When it can, the word shows in rainbow colors. Requires Claude Code v2.1.311 or later."

릴리스 노트 쪽이 범위를 좁게 적음.

> "Changed `ultrathink` in the interactive terminal to run its turn at xhigh effort on Opus 5.5, Sonnet 5.5, Haiku 5.5 and Fable 5.1 through Anthropic's API; typed while Claude is working, it raises the rest of that turn"

세 가지가 여기서 갈림.

- **`think`, `think hard`, `think more` 는 키워드가 아님.** 평범한 프롬프트 텍스트로 지나감. 예전 하네스 관습을 그대로 들고 온 CLAUDE.md 나 스킬 본문에 이 문구가 남아 있으면 **아무 일도 하지 않는 토큰**임. [/doctor prompt-audit](../prompting/2026-09-25-claude-md-prompt-audit.md)이 찾는 "낡은 모델용 지시"의 전형이고, 감사가 이 패턴을 짚는지는 그 문서가 이미 미확인으로 남겨 둠
- **대상 모델이 네 개뿐이고 Anthropic API 경로 한정임.** 게이트웨이나 Bedrock·Vertex·Foundry 를 타는 세션에서는 레벨이 안 올라감. 인컨텍스트 지시만 붙음
- **일하는 중에 쳐도 됨.** "raises the rest of that turn" 이라 이미 돌고 있는 턴의 남은 구간을 올림. 같은 릴리스에서 무지개 표시를 **올릴 수 있는 자리에만** 띄우도록 고쳤음. 인용 안이나 파일 경로 안의 `ultrathink` 는 더 이상 색이 붙지 않음. 즉 **색이 안 붙으면 레벨은 안 올라간 것**이고, 이것이 지금 유일한 눈으로 확인하는 수단임

## 한 턴만 올리는 것이 캐시에 공짜인가

Platform 의 무효화 표가 effort 행을 이렇게 적음.

> "Changing the `output_config.effort` value always invalidates message blocks, with the same model-specific effect on tool and system caches as thinking parameters. Setting effort explicitly to the model's default is equivalent to omitting it and does not invalidate. On models that support per-message effort, an effort change carried in a `role: "system"` message inside `messages` leaves the cached prefix intact."

경로가 둘이고 대가가 다름.

| 경로 | 캐시 |
|---|---|
| 최상위 `output_config.effort` 를 다음 요청에서 바꿈 | **메시지 캐시 무효.** 올릴 때 한 번, 되돌릴 때 또 한 번 |
| `messages` 안에 `output_config` 만 담은 `role: "system"` 항목을 끼움 (베타 `mid-conversation-output-config-2026-07-01`) | **캐시된 prefix 유지** |

per-message effort 가 되는 모델은 Claude API 와 Google managed 기준 **Fable 5.1, Mythos 5.1, Opus 5.5, Opus 5, Sonnet 5.5, Haiku 5.5** 임. `ultrathink` 가 레벨을 올리는 네 모델(Opus 5.5, Sonnet 5.5, Haiku 5.5, Fable 5.1)이 **전부 이 목록 안에 있고**, 릴리스 노트가 범위를 "through Anthropic's API" 로 좁힌 것도 per-message 쪽 가용 범위와 겹침. 그래서 **per-message 경로를 쓴다고 보는 쪽이 정황이 맞음.**

⚠️ 다만 그것은 추론이고 문서 문언이 아님. 반대 방향 단서도 같은 릴리스에 있음. 2.1.311 이 고친 항목 중에 이것이 있음.

> "Fixed `/effort` not warning about a prompt cache miss in plan mode when the plan model differs from the session model"

즉 **`/effort` 로 레벨을 바꾸면 Claude Code 가 캐시 미스를 경고함.** 최상위 값을 바꾸는 경로라는 뜻이고, `ultrathink` 에 대해서는 그런 경고가 문서에도 릴리스 노트에도 없음. 경고가 없는 이유가 "안 깨지기 때문"인지 "경고를 안 붙였기 때문"인지는 **확인할 수 없었음.**

판단은 이렇게 갈라 두는 편이 안전함. **`/effort` 로 레벨을 바꾸는 것은 캐시 비용이 확인된 조작이고, `ultrathink` 는 비용이 미확인인 조작임.** 긴 세션에서 prefix 를 길게 들고 가는 작업이면, 확인되지 않은 쪽을 습관적으로 치지 말고 `/usage` 를 한 번 보는 편이 맞음. 같은 릴리스가 **`/usage` 가 큰 프롬프트 캐시 미스를 "한도 사용에 기여하는 것" 목록에 아예 안 띄우던 것**도 고쳤으니, 2.1.311 이전 화면으로 판단해 왔다면 그 수치부터 다시 봐야 함.

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 히트율이 낮을 때 볼 8가지에 effort 변경을 5번으로 넣어 둔 것이 이 자리임. 그 글의 지적은 지금도 그대로고, 바뀐 것은 **effort 를 바꾸는 트리거가 명령에서 프롬프트 안의 단어로 내려왔다**는 것임.

## Claude Code 의 기본값이 API 기본값과 갈리는 두 모델

여기가 이 문서에서 제일 실무적인 절임. 두 문서의 기본값 서술이 어긋남.

| 모델 | Claude Code 기본 | Claude API 기본 |
|---|---|---|
| Opus 5.5 | `medium` | `medium` |
| Haiku 5.5 | `medium` | `medium` |
| **Sonnet 5.5** | **`medium`** | **`high`** |
| **Opus 4.7** | **`xhigh`** | **`high`** |
| 그 외 effort 지원 모델 | `high` | `high` |

Claude Code 문서는 기본값을 "`high` on every model that supports effort, except that Opus 5.5, Sonnet 5.5, and Haiku 5.5 default to `medium`, Opus 4.7 defaults to `xhigh`" 로 적음. Platform 쪽은 `high` 행을 "The default on every model that supports effort except Claude Opus 5.5 and Claude Haiku 5.5" 로 적고, Sonnet 5.5 절에서 "`high` is the default on the Claude API", Opus 4.7 절에서 "The API default is `high`. To use `xhigh`, set `effort` explicitly" 라고 적음.

**어긋난 것이 아니라 Claude Code 가 명시적으로 다른 값을 보내는 것으로 읽힘.** 두 모델에서 결과가 각각 반대 방향임.

- **Sonnet 5.5**: Claude Code 에서 API 보다 **한 단 아래**로 돎. 이 저장소의 [Claude Sonnet 5.5](../models/2026-09-28-claude-sonnet-5-5.md)가 "기본 effort `high`" 로 적은 것은 **API 기준 사실**이고 Claude Code 세션에는 그대로 옮기면 안 됨. 그 문서가 인용한 Anthropic 각주, 즉 **max effort 에서 점수가 xhigh 보다 낮게 나온 원인이 하네스가 서브에이전트를 퍼뜨린 것**이었다는 대목과 같이 읽으면 방향이 맞음. 하네스 쪽에서 기본을 낮춰 둔 셈임
- **Opus 4.7**: Claude Code 에서 API 보다 **한 단 위**로 돎. Platform 의 Opus 4.7 권고가 "코딩·에이전틱 작업은 `xhigh` 에서 시작하라"이므로 하네스 기본값이 권고를 따라간 것으로 읽힘. 대신 **아무것도 설정하지 않은 세션이 가장 비싼 레인 근처에서 돈다**는 뜻이기도 함

실무 결론 하나. **같은 모델을 Claude Code 로 쓸 때와 API 로 직접 칠 때 사고량과 토큰 비용이 다를 수 있음.** 하네스에서 재 본 수치를 API 서비스의 예산 근거로 옮기거나 그 반대로 옮기면, 둘 중 한 쪽은 effort 가 한 단 다른 상태의 숫자임. [대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md)의 레버 4 를 적용할 때 가장 먼저 고정해야 하는 변수가 이것임.

## Ultracode 는 effort 가 아님

`/effort` 슬라이더에 같이 붙어 있어서 섞이기 쉬운데 성격이 다름.

> "Ultracode is a Claude Code setting rather than a model effort level: with it on, Claude orchestrates dynamic workflows for substantive tasks, at whichever effort level the session runs at."

갈리는 지점이 켜는 방법에 따라 다름.

| 켜는 방법 | effort 레벨 |
|---|---|
| `/effort ultracode`, 슬라이더의 `Tab` 토글, `ultracode` 설정 | **그대로 둠** |
| `--effort ultracode`, Agent SDK 의 `effortLevel: "ultracode"` | **`xhigh` 로 같이 올림** |

그리고 조용히 안 켜지는 조건이 둘임. **워크플로를 꺼 뒀거나 모델이 `xhigh` 를 안 받으면** ultracode 는 안 됨. 이때 `--effort ultracode` 는 실패하지 않고 **ultracode 를 끈 상태로, 모델과 상한이 허용하는 최고 레벨(`xhigh` 까지)로** 세션을 시작함. 위 레벨 표의 마지막 줄과 겹치는 곳이 여기임. **Opus 4.6 이나 Sonnet 4.6 으로 떨어진 세션에서는 ultracode 가 조용히 꺼짐.** `effortLevel` 설정과 `CLAUDE_CODE_EFFORT_LEVEL` 은 `ultracode` 값을 받지 않음.

`off` 형태(`/effort ultracode off`)는 **v2.1.284 이상**에서 됨.

## 게이트웨이나 서드파티에서 effort 가 조용히 사라짐

원인이 모델 ID 매칭임.

> "Claude Code enables features like effort levels and extended thinking by matching the model ID against known patterns. Provider-specific IDs such as Amazon Bedrock ARNs or custom deployment names often don't match these patterns, leaving supported features disabled."

**`/effort` 명령 자체가 안 보이는 상태**가 버그가 아니라 이 동작임. 선언으로 되돌림.

| capability 값 | 켜는 것 |
|---|---|
| `effort` | effort 레벨과 `/effort` 명령 |
| `xhigh_effort` | `xhigh` 레벨 |
| `max_effort` | `max` 레벨 |

`ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` 처럼 모델별 변수로 넘기고, **선언하면 적은 것만 켜지고 안 적은 것은 꺼짐.** 셋을 따로 둔 이유가 위 레벨 표와 같음. `effort` 만 적고 `xhigh_effort` 를 빼면 명령은 뜨는데 `xhigh` 가 없음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Sonnet 5.5](../models/2026-09-28-claude-sonnet-5-5.md) | 그 문서의 "기본 effort `high`" 는 API 기준임. Claude Code 는 `medium` 으로 돎. max effort 점수 하락의 원인이 하네스였다는 각주와 같은 방향 |
| [Claude Haiku 5.5](../models/2026-10-07-claude-haiku-5-5.md) | 기본 `medium` 과 레벨별로 깨지는 동작. thinking 을 끈 상태에서는 per-message effort 변경도 400 이라 위 캐시 표의 둘째 줄이 막힘 |
| [Claude Opus 5.5](../models/2026-09-22-claude-opus-5-5.md) | per-message effort 지원 모델이 늘어난 경과. 사용자 설정의 최상위 `effortLevel` 이 이 모델에만 안 세어지는 예외가 여기 붙음 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 대화 중 effort 변경을 캐시 보존 경로로 처음 적은 문서. `ultrathink` 대상 네 모델 중 하나임 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 히트율 점검 8가지의 5번이 effort 변경임. 그 트리거가 명령에서 프롬프트 안의 단어로 내려왔음 |
| [Cache diagnostics](./2026-09-23-prompt-cache-diagnostics.md) | effort 변경이 진단에서 이름 없이 `unavailable` 로 떨어지는 문제. `ultrathink` 가 깼는지 여부를 진단으로 가리기 어려운 이유 |
| [Fast mode 정식 공개](./2026-10-15-fast-mode-opus-5-5-ga.md) | 캐시를 가르는 다른 축. speed 는 tools 캐시를 살리고 effort 는 모델에 따라 갈림 |
| [/doctor prompt-audit](../prompting/2026-09-25-claude-md-prompt-audit.md) | 키워드가 아닌 `think hard` 류가 지시 파일에 남아 있는 경우. 감사가 이 패턴을 짚는지는 미확인으로 남아 있음 |
| [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md) | 같은 레벨 축을 Swift 쪽에서 다루는 경로. `fixedEffort` 가 요청별 힌트를 덮고, 이름 붙은 레벨이 `high` 에서 멈춰 `.custom("xhigh")` 를 넘겨야 함 |
| [Managed Agents 다이내믹 워크플로](../agents/2026-10-09-managed-agents-dynamic-workflows.md) | ultracode 가 켜는 것이 다이내믹 워크플로임. 단 그쪽은 서버측 Managed Agents 의 run 이고 이쪽은 로컬 하네스라 기본값과 상한을 섞으면 안 됨 |

## 짚어야 할 것

- **공식 문서와 릴리스 노트만 읽었고 직접 돌려보지 않았음.** `ultrathink` 가 실제로 어느 요청 모양으로 나가는지, 무지개 표시가 뜨는 조건이 문서 문언과 정확히 일치하는지는 **확인할 수 없었음**
- **`ultrathink` 가 per-message effort 경로를 쓰는지가 이 문서의 가장 큰 미확인 항목임.** 대상 모델 네 개가 전부 per-message 지원 목록 안에 있다는 것은 정황이고, Claude Code 문서도 Platform 문서도 경로를 적지 않음. **캐시가 깨지는지 아닌지를 이 문서로 단정하면 안 됨**
- **기본값 표의 Sonnet 5.5·Opus 4.7 행은 두 1차 출처의 문언을 나란히 놓은 것임.** 어느 쪽이 낡았을 가능성도 배제하지 못함. Claude Code 가 실제로 어떤 값을 전송하는지는 요청을 들여다봐야 하고 그것은 하지 않았음
- **`ultrathink` 의 인컨텍스트 지시 본문을 확인할 수 없었음.** 문서가 "adds an in-context instruction" 까지만 적고 문구를 공개하지 않음. 그래서 레벨이 안 올라가는 경로(게이트웨이, 서드파티)에서 이 키워드가 무엇을 바꾸는지는 측정할 수 없음
- **`[1m]` 접미사 처리와 compaction 임계값은 이 문서에서 다루지 않았음.** 같은 문서 페이지에 있고 v2.1.311 도 접미사 표기를 건드렸지만, effort 와는 별개 축임
- 레벨별 토큰·비용 배수는 문서에 수치로 없음. 이 문서는 숫자를 만들지 않았음
- 원문 문서 본문에 독자나 도구를 향한 지시문·명령문은 없었음

## 유효기간

**2026-10-27 확인 기준이고 Claude Code 최신 릴리스가 v2.1.311 시점임.** 버전 번호는 하루에 한두 개씩 올라가니 읽는 시점에 이미 낡았다고 보는 게 맞음. 위 표들을 고치려 들지 말고 [Model configuration](https://code.claude.com/docs/en/model-config)의 effort 절을 다시 읽는 편이 빠름. 다시 볼 때 확인할 것 셋. **`ultrathink` 가 쓰는 경로와 캐시 영향이 문서에 적혔는지**, **대상 모델 네 개가 늘었거나 Anthropic API 한정이 풀렸는지**, 그리고 **per-message effort 가 베타를 벗어났는지**. 셋째는 [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)이 이미 재확인 항목으로 적어 둔 것과 같은 항목임. 버전별 변경은 `gh api repos/anthropics/claude-code/releases` 로 받아 `effort`·`ultrathink`·`ultracode` 줄만 훑어도 대부분 잡힘.
