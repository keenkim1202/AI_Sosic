---
title: /doctor prompt-audit, CLAUDE.md를 낡은 모델용 지시로 보고 감사하는 명령
source: https://code.claude.com/docs/en/memory
author: Anthropic
published: 2026-09-25
collected: 2026-09-30
tags: [claude-code, prompt-engineering, claude-md, context-engineering, mcp, agent-memory]
---

출처: [Audit your instruction files, How Claude remembers your project](https://code.claude.com/docs/en/memory#audit-your-instruction-files) · [`/doctor` 항목, Commands](https://code.claude.com/docs/en/commands) · [`CLAUDE_CODE_MAX_MCP_DESCRIPTION_LENGTH`, Environment variables](https://code.claude.com/docs/en/env-vars) · [CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) (문서 확인 **2026-09-30**)

## 요약

이 저장소에 [Prompting Claude Opus 5](./2026-08-10-prompting-claude-opus-5.md)가 있고, 그 글의 논지는 "지시를 더하지 말고 **예전 모델용으로 넣어둔 지시를 빼라**"였음. 그 작업을 손으로 하라고 마이그레이션 체크리스트까지 적어뒀는데, 그걸 명령 하나로 돌리는 게 붙었음. **`/doctor prompt-audit`**(별칭 `/checkup prompt-audit`)이고 **Claude Code v2.1.283 이상**에서 됨. v2.1.283 은 공식 Atom 피드 기준 **2026-09-25** 에 나왔음.

검사 대상은 코드가 아니라 **지시 파일**임. 공식 문서가 찾는다고 적은 것 셋은 **낡은 모델용으로 쓰인 지시**, **존재하지 않는 파일·명령 참조**, **서로 모순되는 파일**임. 결과는 제안 편집이 붙은 리포트로 나오고 **적용해 달라고 말하기 전까지 파일은 안 바뀜.** 판단을 사람에게 남기는 형태라 이 저장소가 [Memory Engineer](../practices/2026-08-02-memory-engineer.md)에서 정리한 태도("시스템은 드러내고, 결정은 사람이 함")와 방향이 같음.

대신 두 가지를 알고 써야 함. 이 감사는 **번들 스킬 `/claude-api` 를 타고 돌기 때문에** 컨텍스트를 아끼려고 번들 스킬을 꺼둔 환경에서는 아예 안 됨. 그리고 **`prompt-audit` 을 붙이면 평소의 `/doctor` 체크업은 돌지 않음.** 둘은 같은 명령의 두 모드임.

## 무엇을 검사하나

문서가 이렇게 적음.

> "Claude looks for problems such as instructions written for older models, references to files or commands that don't exist, and files that contradict each other."

셋 중 값어치가 갈림.

**존재하지 않는 경로·명령 참조.** 기계적으로 확인되는 것이라 오판이 적음. CLAUDE.md 에 적어둔 `npm run` 스크립트나 디렉터리가 리팩터 뒤에 사라졌는데 지시는 남아 있는 상태가 흔함. 이건 그냥 이득임.

**파일 간 모순.** 사용자 스코프와 프로젝트 스코프가 충돌하는 경우가 잡힘. 같은 문서의 다른 절이 적어둔 것이 이 검사의 근거임.

> "if a user rule and a project rule conflict, Claude may follow either one, so keep the two consistent."

즉 모순은 "둘 중 나쁜 쪽이 이긴다"가 아니라 **어느 쪽이 이길지 모른다**는 문제임. 재현이 안 되는 행동의 원인으로 제일 찾기 어려운 형태고, 그래서 이 항목이 셋 중 실무 가치가 가장 큼.

**낡은 모델용 지시.** 여기가 조심할 곳임. 무엇이 "낡았는지"는 모델의 판단이고, 겉보기에 군더더기인 지시가 실제로는 사람이 겪은 실패 하나를 막고 있는 경우가 있음. [Prompting Claude Opus 5](./2026-08-10-prompting-claude-opus-5.md)가 빼라고 한 목록(자기 검증 지시, 재확인 지시, "생각하지 마라")은 근거가 문서에 박힌 것들인데, 감사가 그 목록만 짚는다는 보장은 문서에 없음. 리포트를 승인 화면으로 대하지 말고 **왜 그 줄을 넣었는지 기억나지 않는 항목부터** 보는 편이 맞음.

그리고 이 감사가 하지 않는 것이 있음. **빼는 쪽만 봄.** Opus 5 가이드가 넣으라고 한 여섯 가지(간결성, 내레이션 카덴스, 문서 길이 보정, 스코프 제약, 서브에이전트 상한, 정정 내레이션 억제)는 지시 파일에 **없는 것**이라 감사 대상이 아님. 그쪽은 그 문서의 체크리스트가 계속 필요함.

## 기본 범위와 좁히는 방법

문서가 밝힌 기본 범위임.

| 범위 | 대상 |
|---|---|
| 지시 파일 | `CLAUDE.md`, `CLAUDE.local.md`, `AGENTS.md` |
| `.claude/` 와 `~/.claude/` 아래 | rules, skills, commands, subagents, output styles |

즉 CLAUDE.md 한 장이 아니라 **지시가 들어가는 표면 전체**를 봄. 스킬과 서브에이전트까지 범위인 것이 중요한데, 낡은 지시가 제일 오래 살아남는 자리가 한 번 만들고 잊는 스킬 본문임.

한 곳만 보게 하려면 경로를 넘김.

```text
/doctor prompt-audit .claude/skills/deploy
```

## 체크업과 같은 명령인데 서로 다른 모드임

`/doctor` 를 인자 없이 돌리면 하는 일이 따로 있고, 그중 컨텍스트 쪽 항목이 이럼.

- 쓰이지 않는 스킬·MCP 서버·플러그인을 **컨텍스트 비용과 대조해서** 찾음
- 느린 훅을 표시함
- 로컬 `CLAUDE.md` 를 체크인된 것과 **중복 제거**함
- 체크인된 `CLAUDE.md` 에서 **코드베이스에서 유도할 수 있는 내용을 잘라내는** 트림을 제안함
- 남은 상시 로드 지시를 스킬과 **하위 디렉터리 `CLAUDE.md`** 로 옮겨 필요할 때만 로드되게 함

문서가 트림의 기준을 갈라 적어 둔 것이 쓸모 있음. 디렉터리 배치, 의존성 목록, 아키텍처 개요는 자르고, **함정·근거·도구 기본값과 다른 규약**은 남김. 앞의 것들은 에이전트가 파일을 열어 알아낼 수 있고 뒤의 것들은 알아낼 수 없다는 구분임.

⚠️ **`prompt-audit` 을 붙이면 위 체크업은 돌지 않음.** 문서 문언이 "instead of running the checkup" 임. 하나를 돌렸다고 다른 쪽이 같이 확인된 것으로 착각하면 안 됨. 둘 다 보려면 두 번 돌려야 함.

여기서 한 가지가 서로 당김. 체크업은 **쓰이지 않는 스킬을 컨텍스트 비용 때문에 끄라**고 권하는데, `prompt-audit` 자신은 번들 스킬 `/claude-api` 를 타고 돎.

> "The audit runs through the bundled `/claude-api` skill. It's unavailable while that skill is turned off in `skillOverrides` or with `disableBundledSkills`."

`disableBundledSkills` 로 번들 스킬을 통째로 끊어둔 환경에서는 감사 자체가 없는 기능이 됨. 컨텍스트를 아끼려고 한 설정이 진단 도구를 같이 끄는 형태인데, **`/skill-doctor` 가 스킬별 컨텍스트 비용과 사용 빈도를 보여주므로** 통째로 끄는 대신 그걸로 골라 끄는 편이 나음. 다만 `/claude-api` 를 켜 두는 것이 컨텍스트에 얼마를 물리는지는 문서에 수치로 없고 **확인할 수 없었음.**

## 크기 경고는 이미 붙어 있음

감사와 별개로 문서가 적어 둔 상한들임. 지시 파일을 정리할 때 목표 숫자로 쓸 수 있음.

| 항목 | 값 |
|---|---|
| CLAUDE.md 권장 길이 | **200줄 미만** |
| Claude Code 가 통째로 읽는 상한 | **4 MiB**. 넘으면 파일을 건너뜀 |
| 자동 메모리 `MEMORY.md` 로드 | **첫 200줄 또는 첫 25KB** 중 먼저 걸리는 쪽 |

권장 길이를 넘은 파일이 있으면 **시작 시점과 `/status` 에 경고가 뜨고**, 파일들이 각자는 안 넘는데 합계가 넘는 경우에도 세션 시작에 경고가 뜸. 각 CLAUDE.md, rules 파일, `@path` 임포트가 **따로 세어짐**.

임포트에 대해 문서가 못을 박은 것이 하나 있음. `@path` 로 쪼개는 것은 **정리에는 도움이 되지만 컨텍스트 비용은 안 줄음.** 임포트된 파일도 시작 시점에 같이 로드되기 때문임. 줄이려면 경로 스코프 rules 나 스킬로 내려서 **필요할 때만 로드되게** 만들어야 함. 파일을 나눴다고 가벼워졌다고 생각하고 있었으면 여기서 갈림.

## 같은 릴리스 계열에 붙은 것, MCP 설명 2,048자 상한

지시 파일이 아니라 툴 설명 쪽인데 같이 봐야 하는 항목임.

> "Maximum length in characters of each MCP tool description and each MCP server's instructions that Claude Code sends to the model (default: 2048). Claude Code truncates longer text."

`CLAUDE_CODE_MAX_MCP_DESCRIPTION_LENGTH` 로 바꿀 수 있고 **v2.1.280 이상**에서 됨. 값은 양의 정수만 받고 그 외는 무시되며 기본값이 적용됨.

두 가지가 여기서 갈림.

**첫째, 기본값이 이미 잘라내고 있음.** 설명이 긴 MCP 서버를 붙여두고 에이전트가 툴을 잘못 쓴다고 여기던 상황이면, 원인이 설명 품질이 아니라 **잘린 뒤쪽**일 수 있음. 잘린 사실이 눈에 보이게 알려지는지, 잘리는 지점이 어디인지는 문서에서 확인할 수 없었음.

**둘째, 상한을 올리는 것이 공짜가 아님.** [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 정리한 대로 툴 정의는 프롬프트 맨 앞에 접혀 들어가는 prefix 임. 설명을 길게 허용하면 매 턴 실어 보내는 prefix 가 같이 커짐. 툴을 잘 쓰게 만들려고 상한을 올리는 것과 캐시 재사용 구간을 짧게 유지하는 것이 같은 방향은 아님.

**혼동하면 안 되는 것.** 이 2,048자는 Claude Code 가 모델에 보낼 때 적용하는 **클라이언트측 잘라내기**임. [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)에 적힌 `SpotlightSearchTool` 이슈, 즉 툴 하나의 기본 설정 설명과 파라미터 스키마만으로 온디바이스 모델의 컨텍스트 창이 넘어가는 것은 **모델 쪽 하드 상한**이라 성격이 다름. 방향은 같음. 툴 설명은 공짜가 아니고 붙이기 전에 재 보는 것이 맞음. 그러나 한쪽의 수치를 다른 쪽 근거로 옮기면 안 됨.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompting Claude Opus 5](./2026-08-10-prompting-claude-opus-5.md) | 그쪽이 "예전 모델용 지시를 빼라"를 손으로 할 체크리스트로 줬고, 이 명령이 그중 **빼는 쪽**을 자동화함. 넣어야 할 여섯 가지는 감사 대상이 아니라 체크리스트가 계속 필요함 |
| [Memory Engineer](../practices/2026-08-02-memory-engineer.md) | 저장소가 커지기 전에 망각 정책을 넣으라는 논지. 모순을 자동 병합하지 않고 드러내기만 한다는 태도가 이 명령의 리포트 우선 동작과 같음 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | MCP 설명 상한을 올리면 prefix 가 커짐. 툴 정의가 캐시 맨 앞에 있다는 구조가 그쪽에 정리돼 있음 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 툴 설명과 스키마가 컨텍스트를 먹는다는 같은 문제의 Apple 쪽 사례. 단 클라이언트측 잘라내기와 모델측 하드 상한을 섞으면 안 됨 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | Xcode 가 Claude Agent SDK 를 태우는 경로. 그 안에서 이 명령을 쓸 수 있는지는 아래 참고 |

## 짚어야 할 것

- **공식 문서만 읽었고 직접 돌려보지 않았음.** 리포트가 실제로 어떤 항목을 몇 개 집어내는지, 오탐이 얼마나 되는지는 이 문서로 말할 수 없음
- **"낡은 모델용 지시"의 판정 기준이 문서에 없음.** 어떤 패턴을 낡았다고 보는지, 그 목록이 [Prompting Claude Opus 5](./2026-08-10-prompting-claude-opus-5.md)의 항목과 일치하는지 확인할 수 없었음. 모델의 판단이므로 리포트를 근거 없이 일괄 승인하면 안 됨
- **날짜는 v2.1.283 의 릴리스 시점임.** 공식 CHANGELOG 의 Atom 피드에서 **2026-09-25T21:49:55Z** 로 확인했음. 문서 페이지 자체의 발행일은 페이지에 표시되지 않아 확인할 수 없었음
- **CHANGELOG 항목과 문서의 버전 표기가 한 곳에서 갈림.** `CLAUDE_CODE_MAX_MCP_DESCRIPTION_LENGTH` 를 환경변수 문서는 **v2.1.280 이상**으로 적음. 이 문서는 문서 쪽 표기를 따랐음
- **Xcode 27 안에서 되는지 확인하지 못했음.** [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)에 적힌 대로 Xcode 26.3 이상이 Claude Agent SDK 를 태우지만, IDE 호스팅 경로에서 번들 스킬과 `/doctor` 하위 명령이 노출되는지는 Apple 이나 Anthropic 문서에서 확인되지 않음
- **감사가 고치는 것은 지시 파일이지 행동이 아님.** 지시 파일은 강제 설정이 아니라 컨텍스트라고 문서가 직접 적어 둠. 반드시 걸려야 하는 것은 감사로 정리할 대상이 아니라 훅으로 내려야 하는 것임
- 원문 문서 본문에 독자나 도구를 향한 지시문·명령문은 없었음

## 유효기간

**2026-09-30 확인 기준**임. `prompt-audit` 은 나온 지 닷새 된 기능이라 범위와 동작이 움직일 여지가 큼. 다시 볼 때 확인할 것 셋. **기본 검사 범위에 항목이 더 붙었는지**, **번들 스킬 `/claude-api` 의존이 풀렸는지**, 그리고 **`/doctor` 체크업과 `prompt-audit` 이 한 번에 도는 형태로 합쳐졌는지**. 셋 다 [Commands](https://code.claude.com/docs/en/commands)의 `/doctor` 행과 [메모리 문서의 감사 절](https://code.claude.com/docs/en/memory#audit-your-instruction-files)만 대조하면 잡힘. 버전별 변경은 [CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)로 확인할 것.
