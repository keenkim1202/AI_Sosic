---
title: 스킬이 로드될 때 도는 셸 명령, PreToolUse 가 안 걸리던 자리에 전용 훅이 생김
source: https://github.com/anthropics/claude-code/releases/tag/v2.1.332
author: Anthropic
published: 2026-11-25
collected: 2026-11-26
tags: [agent-security, claude-code, permissions, skills, guardrail, human-in-the-loop]
---

출처: [Claude Code v2.1.332 릴리스 노트](https://github.com/anthropics/claude-code/releases/tag/v2.1.332) (2026-11-25) · [Hooks reference](https://code.claude.com/docs/en/hooks) · [Skills](https://code.claude.com/docs/en/skills) · [All settings](https://code.claude.com/docs/en/settings-reference) (문서 셋은 **2026-11-26 확인**. 릴리스 날짜는 `gh api` 의 `published_at`(UTC) 기준이고, npm 등록 시각은 2026-11-24T22:06Z 임)

## 요약

**2026-11-25 v2.1.332 에 `PreSkillShellCommand` 훅이 붙었음.** 이 훅이 생겼다는 사실보다 중요한 것은 **왜 필요했는지**임. 스킬과 슬래시 커맨드의 본문에 `` !`<command>` `` 로 박아 둔 셸 명령은 스킬이 로드되는 동안 실행되는데, 훅 레퍼런스가 그 명령에 대해 이렇게 적음. **"PreToolUse hooks don't fire for injected commands."** 즉 이 저장소가 [mods 문서](../agents/2026-10-04-claude-code-mods.md)와 [권한 우회 수정 묶음](./2026-10-22-claude-code-permission-bypass-fixes.md)에서 전제로 깔아 온 `PreToolUse` 기반 차단·기록 스크립트가, 스킬 하나를 부르는 순간 도는 셸에는 **처음부터 걸려 있지 않았음.**

권한 쪽이 전부 비어 있던 것은 아님. 스킬 문서가 **"Injected commands never prompt for permission while the skill renders"** 라고 적고, 대신 **룰 검사는 함.** `deny` 룰에 걸리면 스킬 호출 자체가 중단됨. 비어 있던 것은 **룰 바깥의 층**, 즉 훅으로 기록하거나 조건부로 막는 자리임.

실무 결론이 두 개임. 하나는 **차단용으로 쓸 수 있지만 조건이 네 개 붙는다**는 것(동기 타입, 매처 `Bash|PowerShell`, 설정 파일 또는 플러그인 위치, `onFailure: "block"`). 다른 하나는 **버전**임. 훅은 **v2.1.332 이상**을 요구하는데 2026-11-26 기준 npm `stable` 태그는 **2.1.326**(2026-11-16)에 머물러 있음. 즉 stable 채널을 쓰고 있으면 이 훅이 아직 없고, 그 상태에서 설정 파일에 훅 이름을 적으면 **터미널 세션을 열 때마다 경고 대화상자가 뜸.**

## 스킬 로드 중에 셸이 도는 경로

스킬 본문에서 두 가지 형태가 명령으로 해석됨.

| 형태 | 조건 |
|---|---|
| 인라인 `` !`<command>` `` | `!` 가 **줄 시작이거나 공백 바로 뒤**일 때만 인식됨. `` KEY=!`cmd` `` 는 리터럴 텍스트로 남고 실행되지 않음 |
| ` ```! ` 로 연 펜스 블록 | 여러 줄 명령용 |

치환은 **원본 파일 위에서 한 번만** 돌고, 명령 출력은 다시 스캔되지 않음. 그래서 명령이 또 다른 플레이스홀더를 뱉어도 2차 확장은 없음. 이 부분이 이 기능의 유일한 구조적 안전장치임.

실행 자체는 Claude 가 직접 부르는 셸 명령과 같은 조건에서 돎. 세션 셸의 **현재 작업 디렉터리**에서 돌고, 기본 `bash` 에서는 **stderr 가 stdout 에 합쳐져 주입 텍스트에 들어가고**, Bash 툴 기본 **2분** 타임아웃이 걸림. 실패의 파급은 생각보다 큼.

> "A failed command aborts the entire skill invocation, not just its own placeholder. Claude never sees the skill content for that invocation."

기본 `bash` 에서는 0 이 아닌 종료 코드가 전부 실패임. 검색·비교 계열 명령의 **종료 코드 1 만 예외**로 정상 처리되고, 2 이상은 그 계열도 실패임. 문제를 찾으면 1 로 끝나는 점검 스크립트를 스킬에 박아 두면 스킬이 통째로 안 열리므로 `|| true` 를 붙이라고 문서가 적음.

그리고 이 경로는 사용자가 만든 스킬만의 이야기가 아님. 훅 레퍼런스가 범위를 이렇게 적음. **"a shell command that runs while a skill or a command such as `/security-review` loads."** 훅 입력의 `command_source` 에 `builtin` 값이 있는 것도 같은 뜻임.

## 걸려 있던 검사와 안 걸려 있던 것

주입 명령에 대해 무엇이 돌고 무엇이 안 돌았는지를 갈라 보면 이렇게 됨.

| 층 | 주입 명령에 적용되나 |
|---|---|
| 권한 프롬프트 | **안 뜸.** 렌더링 중에는 물어보지 않음 |
| `deny` 룰 | 적용됨. 걸리면 `Shell command permission check failed for pattern "..."` 로 **스킬 호출이 중단**됨 |
| `ask` 룰 | 적용됨. auto 모드 밖에서는 중단 사유가 됨 |
| `allowed-tools` 사전 승인 | 적용됨. 중단을 막는 수단이 이것임 |
| `PreToolUse` 훅 | **안 걸림.** v2.1.332 이전에는 대체 수단도 없었음 |
| `PreSkillShellCommand` 훅 | **v2.1.332 부터.** 권한 검사를 통과한 명령만 여기 도달함 |
| `disableSkillShellExecution` 설정 | 적용됨. 각 명령이 `[shell command execution disabled by policy]` 로 치환됨 |

표의 셋째 칸부터가 이 릴리스의 의미임. `deny` 룰은 **문자열 패턴에 걸리는 것만** 막고, 그 룰이 셸 문법에 번번이 졌다는 것은 이 저장소가 [2.1.309 의 수정 13건](./2026-10-22-claude-code-permission-bypass-fixes.md)에서 이미 세 둔 결론임. 그 결론의 대안으로 쓰이던 것이 **훅으로 직접 판정하는 것**이었는데, 그 대안이 이 경로에만 없었음.

`disableSkillShellExecution` 은 성격이 다름. 판정하지 않고 **기능째로 끔.** 범위는 사용자·프로젝트·플러그인·추가 디렉터리 출처이고, **번들 스킬과 관리 설정으로 배포된 스킬은 영향을 받지 않음.** 관리 설정에서 `true` 로 박으면 다른 곳의 `false` 로 되돌릴 수 없음.

## 차단용으로 쓰려면 붙는 조건 네 개

문서가 조건을 네 줄로 못박아 뒀고, 하나라도 틀리면 차단이 안 되거나 반대로 전부 막힘.

| 항목 | 요구 |
|---|---|
| 매처 | `Bash\|PowerShell`. 주입 명령을 돌리는 툴이 둘이기 때문임 |
| 타입 | `command`·`http`·`mcp_tool` 중 하나이고 **async 가 아니어야** 함 |
| 위치 | **설정 파일 또는 플러그인.** 스킬 프론트매터는 안 됨 |
| 실패 처리 | `onFailure: "block"`. 안 걸면 훅이 죽거나 타임아웃일 때 **명령이 그냥 돎** |

타입 쪽에 함정이 하나 있음.

> "If you configure a `prompt` or `agent` hook on this event, Claude Code can't run it and blocks each command the hook matches."

즉 `prompt`·`agent` 훅을 걸면 **차단이 안 되는 게 아니라 전부 차단됨.** 매처에 걸린 명령이 모두 막히므로 스킬이 조용히 반쪽으로 로드됨. fail open 이 아니라 fail closed 인데, 의도한 fail closed 가 아니라 **설정 실수의 결과**라 증상만 보면 원인을 못 짚음.

반대로 `onFailure` 를 안 거는 쪽은 fail open 임. 이 저장소가 [mods 문서](../agents/2026-10-04-claude-code-mods.md)에서 정리한 것, 즉 차단용 훅이면 fail closed 를 **직접 만들어야 한다**는 것과 같은 자리임. 차이는 mods 쪽은 `.catch` 핸들러를 손으로 쓰는 일이고 여기는 키 하나라는 것뿐임.

결과는 세 갈래로 떨어짐.

| 훅 결과 | 일어나는 일 |
|---|---|
| exit 0, 또는 `permissionDecision: "allow"` | 다른 훅이 막지 않으면 실행됨 |
| exit 2, `permissionDecision: "deny"`, 또는 최상위 `decision: "block"` | 명령이 안 돎. **스킬은 그대로 로드되고** Claude 는 그 자리에서 `[not run: blocked by hook: <reason>]` 를 읽음 |
| `continue: false` | 스킬 호출 자체가 실패하고 **주입 명령이 하나도 안 돎** |

`"ask"`·`"defer"` 와 `updatedInput` 은 전부 `"deny"` 와 같이 처리됨. **승인하거나 명령을 고쳐 쓰는 것은 이 훅으로 못 함.** 막는 것만 됨.

그리고 차단 사유가 안 보이는 조건이 하나 있음. **Remote Control 이 붙어 있으면 차단은 되지만 사유가 사람에게도 Claude 에게도 표시되지 않고** 디버그 로그에만 남음.

## 스킬 자기 훅으로는 자기 명령을 못 막음

위 "위치" 조건이 실무에서 제일 쉽게 어긋나는 자리임. 문서가 이유까지 적어 둠.

> "A skill's injected commands run without the hooks in its own frontmatter on its first load in a session and on loads inside a subagent."

`context: fork` 로 포크되는 스킬은 더 나아가서 **자기 프론트매터의 훅도 서브에이전트 정의의 훅도 안 돎.** 순서를 생각하면 당연함. 훅은 스킬이 로드된 뒤에 등록되고 주입 명령은 로드되는 중에 돌기 때문임. 그래서 스킬을 배포하는 쪽이 "이 스킬은 자기 명령을 자기가 검사함" 이라고 주장하는 구조는 성립하지 않음.

[Agent Skills 표준 문서](../agents/2026-10-22-agent-skills-standard-aaif.md)에서 내린 판단이 여기서 한 칸 더 강해짐. 그 문서는 `allowed-tools` 가 명세상 실험적이므로 **권한은 스킬 바깥의 하네스 설정에서 잡아야 한다**고 적었음. 이번 릴리스가 확인해 주는 것은 그것이 권고가 아니라 **구조적 제약**이라는 것임. 스킬 안에서는 자기 로드 시점을 통제할 방법이 아예 없음.

## auto 모드는 구멍을 메우지 않고 옮김

auto 모드에서는 승인이 필요한 주입 명령이 스킬 호출을 중단시키지 않음. 대신 스킬이 **"먼저 이 명령을 돌려라"** 는 지시를 달고 로드되고, Claude 가 그것을 자기 툴 호출로 실행함. 훅 쪽 서술이 결과를 분명히 적음.

> "Claude can run it afterward as a tool call, which fires `PreToolUse` and not this event, so keep your script on both events."

즉 같은 명령이 **auto 모드에서는 `PreToolUse` 로, 그 밖에서는 `PreSkillShellCommand` 로** 떨어짐. 한쪽에만 스크립트를 걸어 두면 모드를 바꾸는 순간 기록이 끊김. 권한 모드가 훅 커버리지를 바꾼다는 것이 이 기능의 가장 반직관적인 부분임.

예외도 둘 적혀 있음. `agent` 를 설정한 **포크 스킬**과 주입 명령을 돌릴 셸 툴이 없는 세션에서는 auto 모드에서도 호출이 중단됨.

그리고 훅이 도는 범위에 하나가 더 있음. **`bypassPermissions` 모드에서도 이 훅은 돎.** [작업 디렉터리 밖 쓰기 차단](./2026-10-30-working-directory-write-block.md)에서 본 것과 같은 성격임. 모드 위에 얹히는 통제가 모드 안의 룰보다 오래 버팀.

## 버전이 실무의 전부임

릴리스 노트가 호환성을 한 줄에 같이 적어 뒀음.

> "Added a `PreSkillShellCommand` hook event that can refuse a skill or command's inline shell commands as it loads; on 2.1.331 and older, a settings file naming it opens a dialog at start"

훅 레퍼런스는 더 구체적임. 요구 버전은 **v2.1.332 이상**이고 **데스크톱 앱과 VS Code 확장도 포함**임. 더 낮은 버전은 훅을 부르지 않고 주입 명령을 그냥 실행함. 그리고 설정 파일에 이름만 적어 두면 v2.1.331 같은 버전이 **대화형 터미널 세션마다** 이 대화상자를 띄움.

> `Unknown hook event "PreSkillShellCommand" was ignored`

여기에 채널 문제가 겹침. 2026-11-26 에 npm 레지스트리에서 받은 dist-tag 가 이럼.

| 태그 | 버전 | 등록 시각(UTC) |
|---|---|---|
| `latest` / `next` | 2.1.332 | 2026-11-24T22:06Z |
| `stable` | 2.1.326 | 2026-11-16T18:43Z |

설정 문서가 `stable` 을 **"a version that is typically about one week old and skips releases with major regressions"** 로 설명함. 그래서 이 훅은 **stable 채널에서 아직 못 씀.** 한 주쯤 기다리면 올라오지만, 그때까지 `autoUpdatesChannel: "stable"` 인 머신과 `latest` 인 머신이 같은 설정 파일을 공유하면 후자만 보호되고 전자는 세션마다 대화상자를 봄.

버전 하한을 설정으로 강제하는 `requiredMinimumVersion` 이 있지만 **스코프가 `Managed` 임.** 관리 설정 밖에서는 조용히 무시됨. 개인 머신에서는 쓸 수 있는 수단이 아니고, 이 저장소가 [mods 문서](../agents/2026-10-04-claude-code-mods.md)에서 센 것과 같은 패턴임. **통제 수단이 관리 설정 전제로 설계돼 있어서 혼자 쓰는 개발자에게는 반쪽만 옴.**

## 개인 개발자가 지금 할 것

iOS 앱 저장소에서 Claude Code 를 쓰는 규모로 좁히면 순서가 이렇게 됨.

1. **주입 명령을 쓰는 스킬이 몇 개인지부터 셈.** 자기가 쓴 것만이 아니라 플러그인으로 받은 것, 저장소에 따라온 `.claude/skills/` 까지. 저장소와 홈의 스킬 디렉터리에서 `!` 로 시작하는 줄을 grep 하는 수준으로 시작해도 됨
2. 받아 온 스킬이 있으면 **그 안의 명령을 눌러서 읽음.** 스킬을 설치하는 것이 코드를 설치하는 것이라는 [표준 문서](../agents/2026-10-22-agent-skills-standard-aaif.md)의 결론이 여기서 실행 시점으로 확정됨. 사람이 `/스킬이름` 을 타이핑하는 것만으로 돌아가는 명령임
3. 자기가 쓴 스킬만 있고 받아 온 것이 없으면 **훅보다 `disableSkillShellExecution` 이 싼 선택일 수 있음.** 판정 스크립트를 유지보수하지 않아도 되고, 켜면 치환 문구가 남아 어디서 막혔는지 보임
4. 훅을 걸기로 했으면 **`PreToolUse` 에 이미 쓰고 있는 스크립트를 양쪽 이벤트에 같이 등록함.** auto 모드가 경로를 옮기기 때문임. `hookEventName` 은 `PreToolUse` 로 두면 한 스크립트로 됨
5. 버전을 먼저 올림. **설정에 이름을 적는 것이 먼저가 되면 세션마다 대화상자를 보게 됨**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Agent Skills 표준과 명세가 정하지 않는 것](../agents/2026-10-22-agent-skills-standard-aaif.md) | 스킬 설치가 코드 설치라는 결론의 실행 시점. `allowed-tools` 가 실험적이라 권한을 하네스에서 잡아야 한다는 판단이 구조적 제약으로 확정됨 |
| [Claude Code Mods](../agents/2026-10-04-claude-code-mods.md) | 차단용 훅은 fail closed 를 직접 만들어야 한다는 것, 그리고 통제 수단이 관리 설정 전제라는 것 |
| [2.1.309 의 권한 우회 수정 묶음](./2026-10-22-claude-code-permission-bypass-fixes.md) | `deny` 룰이 문자열 패턴에만 걸린다는 한계. 그 대안인 훅이 이 경로에만 없었음 |
| [작업 디렉터리 밖 쓰기 차단](./2026-10-30-working-directory-write-block.md) | 모드 위에 얹히는 통제가 모드 안의 룰보다 오래 버티는 구조. 이 훅도 `bypassPermissions` 에서 돎 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 주입 명령은 애초에 프롬프트를 띄우지 않음. 승인 정확도로 메울 수 없는 자리임 |

## 짚어야 할 것

- **이 문서는 훅을 실제로 걸어 보지 않았음.** 문서가 제시한 테스트 절차(아무것도 지우지 않는 한 줄짜리 스킬을 만들고 `/hook-test` 를 타이핑)와 기대 출력 `Not run in /hook-test: ...` 는 레퍼런스에 적힌 것을 옮긴 것이고, 실행해 확인하지 않았음
- **mods 의 `tool.check` 가 주입 명령에도 도는지는 확인할 수 없었음.** mods 문서에 주입 명령이나 스킬 로드 시점을 언급하는 서술이 없음. `PreToolUse` 가 안 걸린다는 서술이 mods 에도 적용되는지는 1차 문서로 확정되지 않으므로, mods 로 차단하고 있다면 직접 측정해야 함
- **v2.1.332 이전에 이 경로로 실제 사고가 있었다는 보고는 찾지 못했음.** 이 릴리스는 보안 권고가 아니라 기능 추가로 적혀 있음. 커버리지 공백이 있었다는 것과 그것이 악용됐다는 것은 다른 주장임
- **`stable` 이 2.1.326 이라는 것은 2026-11-26 한 시점의 값임.** 채널 설명이 "약 한 주"라고 적었으니 이 문서를 읽는 시점에는 이미 올라와 있을 수 있음
- 릴리스 노트 본문은 날짜를 적지 않음. **2026-11-25 는 `gh api` 가 준 `published_at`** 이고 npm 등록은 그보다 약 3시간 반 앞섬. 둘 중 무엇을 발행일로 볼지는 이 저장소가 앞선 문서들에서 쓴 기준(`published_at`)을 따랐음
- 훅 입력의 `invoked_by` 가 `"user"`·`"claude"`·`"other"` 로 갈리는데, **사람이 타이핑한 경우와 Claude 가 발동한 경우를 스크립트가 다르게 다룰 근거가 될 수 있음.** 다만 어느 쪽이 더 위험한지는 문서가 말하지 않고 이 문서도 판단하지 않음

## 유효기간

**2026-11-26 확인 기준이고 최신 릴리스는 2.1.332** 임. 다시 볼 때 확인할 것 셋. **`stable` 태그가 2.1.332 이상으로 올라왔는지**(그러면 위 채널 주의가 사라짐), **`PreSkillShellCommand` 가 승인이나 `updatedInput` 을 받게 됐는지**(지금은 차단 전용이고, 이것이 열리면 주입 명령에 사람 승인을 붙일 수 있게 됨), 그리고 **auto 모드에서 경로가 `PreToolUse` 로 넘어가는 동작이 유지되는지.** 셋째가 가장 크게 바뀔 자리인데, 지금 구조는 훅을 두 이벤트에 중복 등록하게 만들기 때문임. 버전 번호는 하루에 한두 개씩 올라가므로 읽는 시점에 이미 낡았다고 보는 게 맞음.
