---
title: 작업 디렉터리 밖 쓰기 차단, 문자열 검사 대신 경로 울타리로 가는 중
source: https://github.com/anthropics/claude-code/releases/tag/v2.1.314
author: Anthropic
published: 2026-10-30
collected: 2026-11-01
tags: [agent-security, claude-code, permissions, sandboxing, guardrail, human-in-the-loop]
---

출처: [Claude Code v2.1.314 릴리스 노트](https://github.com/anthropics/claude-code/releases/tag/v2.1.314) (2026-10-30) · [v2.1.316](https://github.com/anthropics/claude-code/releases/tag/v2.1.316) (2026-10-31) · [v2.1.257](https://github.com/anthropics/claude-code/releases/tag/v2.1.257) (2026-09-01) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [Permission modes](https://code.claude.com/docs/en/permission-modes) · [CLI reference](https://code.claude.com/docs/en/cli-reference) (문서 네 건은 **2026-11-01 확인**. 릴리스 날짜는 `gh api`의 `published_at`(UTC) 기준임)

## 요약

**2026-10-30 v2.1.314 에 `permissions.blockWritesOutsideWorkingDirectories` 가 붙었음.** Write·Edit·NotebookEdit 가 **모든 권한 모드에서** 작업 디렉터리 밖 경로를 거부함. 읽기 쪽 짝인 `permissions.blockReadsOutsideWorkingDirectories` 는 **2026-09-01 v2.1.257** 부터 있었고, `--restricted` 가 파일 툴을 작업 디렉터리에 묶는 것은 **v2.1.248**(2026-08-27) 부터임. 두 달 사이에 같은 모양의 통제가 셋 생긴 셈임.

이 저장소가 [2026-10-22 릴리스에서 센 권한 우회 수정 13건](./2026-10-22-claude-code-permission-bypass-fixes.md)에서 내린 결론은 **셸 문자열을 읽어 안전 여부를 판정하는 층이 번번이 진다**는 것이었음. 이 두 설정은 성격이 다름. 명령 문자열을 해석하지 않고 **경로가 울타리 안인지만 봄.** 그리고 allow·deny 룰과 같은 층이 아니라 모드 위에 얹히므로 `bypassPermissions` 에서도 유지됨.

다만 울타리는 파일 툴 쪽에만 서 있음. 읽기 쪽 문서가 **"Claude Code doesn't refuse shell commands the same way"** 라고 못 박아 뒀고, 셸 쪽은 거부가 아니라 **프롬프트**로 떨어짐. 쓰기 쪽 릴리스 노트는 세 툴만 적고 Bash 를 적지 않음. 그리고 **같은 울타리가 하루 뒤에 한 번 뚫린 것이 고쳐졌음**: v2.1.316 이 `--restricted` 세션에서 **Read allow 룰이 덮은 경로의 파일이 작업 디렉터리 밖에서 업로드되던 것**을 고침. 울타리도 결국 코드임.

## 셋이 생긴 순서

| 날짜 | 버전 | 들어온 것 |
|---|---|---|
| 2026-08-27 | 2.1.248 | `--restricted` 가 내장 파일 툴을 **작업 디렉터리에 묶음.** 평가 하네스가 공유 머신에서 `claude` 를 굴리는 경우를 위한 플래그로 들어옴 |
| 2026-09-01 | 2.1.257 | `permissions.blockReadsOutsideWorkingDirectories`. 같은 릴리스에 **auto 모드의 첫 바깥 읽기 1회 프롬프트**가 붙음 |
| 2026-10-30 | 2.1.314 | `permissions.blockWritesOutsideWorkingDirectories`. Write·Edit·NotebookEdit |
| 2026-10-31 | 2.1.316 | `--restricted` 가 **Read allow 룰이 덮은 바깥 경로의 파일을 업로드**하던 것을 고침 |

평가 하네스용 플래그로 먼저 나온 뒤에 설정으로 내려온 순서임. 즉 **처음부터 일반 사용자용으로 설계된 통제가 아니고**, 격리가 필요한 자리에서 먼저 쓰이던 것이 일반 설정이 된 것으로 읽힘.

## 읽기 쪽은 문서가 다 갖춰져 있음

쓰기 쪽은 아직 문서가 없으므로(아래 짚어야 할 것), 울타리가 실제로 어떻게 서는지는 읽기 쪽 항목이 유일한 1차 서술임.

- 거부되는 것은 **`Read`·`Grep`·`Glob`·`LSP`** 호출이고, 거부와 함께 모델에게 **`/add-dir` 로 디렉터리를 추가해 달라고 사용자에게 요청하라**고 알려 줌
- Claude Code 자신이 필요한 파일은 계속 읽힘. `~/.claude/` 아래의 스킬·플러그인·룰·에이전트·커맨드와 `CLAUDE.md`
- `bypassPermissions` 를 **포함한** 모든 모드에서 적용됨
- 기본값은 unset 이고, unset 이면 바깥 읽기는 권한 모드를 따름

scope 규칙이 비대칭이라는 점이 중요함. 문서 문언이 이럼.

> "A `true` in any file applies, so a repository can turn the block on for itself but can't lift yours."

즉 **저장소가 자기 자신에게 블록을 걸 수는 있고, 내가 건 블록을 풀 수는 없음.** 관리 설정도 못 품. 설정 문서의 "Exceptions to managed settings precedence" 표에 이 키가 올라가 있고, 관리 소스가 `false` 로 적어도 어느 파일에서든 온 `true` 가 이김.

**울타리에 들어가는 디렉터리 범위에 함정이 하나 있음.** `--add-dir`, `/add-dir`, 그리고 **사용자·관리 설정**의 `additionalDirectories` 는 작업 디렉터리로 셈. 반면 **저장소 설정에 적힌 것은 안 셈**: `.claude/settings.json`, 그리고 git 이 추적하는 `.claude/settings.local.json` 이 거기 해당함. git 저장소가 아니거나 git 이 그 파일을 추적하면 `.claude/settings.local.json` 도 저장소 설정 취급임.

실무에서 걸리는 모양은 이럼. 공유 패키지나 생성 산출물 디렉터리를 팀 `.claude/settings.json` 의 `additionalDirectories` 에 적어 두고 쓰던 저장소에서 블록을 켜면, **그 디렉터리가 울타리 밖으로 떨어짐.** 계속 읽혀야 하는 경로는 사용자 설정으로 옮겨야 함.

auto 모드의 1회 프롬프트는 선택지가 넷인데, 그중 하나가 **설정 파일을 직접 쓴다**는 점을 알아 둘 값어치가 있음. "No, and block reads outside the working directories from now on" 을 고르면 Claude Code 가 **사용자 설정에 `permissions.blockReadsOutsideWorkingDirectories: true` 를 기록**함. 이후 모든 세션과 모든 모드에 적용됨. 비대화형 `-p` 실행과 백그라운드 세션에서는 이 프롬프트가 안 뜨고 읽기가 그대로 돎.

## 셸은 울타리 밖에 있음

읽기 전용 Bash 명령 집합(`ls`, `cat`, `echo`, `pwd`, `head`, `tail`, `grep`, `find`, `wc`, `which`, `diff`, `stat`, `du`, `cd`, 그리고 읽기 전용 `git`)은 모든 모드에서 프롬프트 없이 도는데, 블록이 켜지면 작업 디렉터리 밖 경로에 대해서만 달라짐. 달라지는 방향이 거부가 아니라 프롬프트임.

문서의 "Actions no mode auto-approves" 절이 적는 것이 둘임.

- 블록이 켜져 있으면 파일을 읽는 Bash 명령은 **auto 모드와 `bypassPermissions` 모드에서도 프롬프트**가 뜸. 샌드박스 밖으로 나가려는 unsandboxed retry 도 같음
- **셸 파서가 추적하지 못하는 명령**은 바깥 경로를 하나도 적지 않아도 같은 프롬프트가 뜸. 문서가 드는 예가 **디렉터리를 두 번 이상 바꾸는 명령과 서브셸을 돌리는 명령**임. 단 그 명령이 샌드박스 안에서 돌고 샌드박스가 블록을 집행하면 이 프롬프트는 안 뜸

**여기가 이 변경의 실제 내용임.** 파서는 없어지지 않았고, **파서가 포기할 때의 기본값이 통과에서 프롬프트로 뒤집혔음.** [2026-10-22 문서](./2026-10-22-claude-code-permission-bypass-fixes.md)가 센 셸 문법 계열 8건은 전부 파서가 포기하거나 틀린 자리를 **통과시키던** 것이었음. 감싼 `cd`, 서브셸 리다이렉트, 열리지 않을 수 있는 리다이렉트가 설정한 변수. 블록을 켜면 그 자리들이 fail open 에서 fail closed 쪽으로 움직임.

그런데 fail closed 의 착지점이 **사람의 프롬프트**임. [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md)가 측정한 것이 바로 그 지점이고, [Claude를 제품마다 어떻게 격리하는가](./2026-05-25-containing-claude-across-products.md)의 텔레메트리는 승인 요청의 **93%가 그냥 승인**됐다고 적음. 그래서 이 설정을 켜는 것만으로 끝내면 **판정이 가장 약한 층으로 옮겨가는 것**이 됨. 셸 쪽을 실제로 닫으려면 샌드박스를 같이 켜야 하고, 그러면 위 프롬프트 자체가 사라짐.

## 샌드박스를 켜면 울타리가 OS 로 내려감

블록이 켜진 상태에서 샌드박싱도 켜져 있으면 샌드박스된 명령까지 덮임. 동작이 **"일단 전부 닫고 필요한 것만 다시 연다"** 순서임.

- 홈 디렉터리와 사용자 파일을 담는 다른 루트들의 읽기를 거부함: `/Users`, `/home`, `/root`, `/Volumes`, `/mnt`, `/media`, `/run/media`, `/srv`
- 그다음 작업 디렉터리, Claude Code 가 세션 중에 만든 worktree, 세션 temp 디렉터리, 그리고 `~/.claude` 에서 명령이 필요한 부분(스킬·플러그인 등)만 다시 엶
- 블록이 서 있는 동안 **저장소 설정의 `allowRead`·`allowWrite` 항목은 안 셈.** 위 `additionalDirectories` 규칙과 같은 방향임
- 세션의 작업 디렉터리가 링크된 git worktree 면 저장소의 공통 `.git` 디렉터리는 읽기·쓰기가 유지됨. git 이 거기서 계속 돌아야 하므로

전역 git 설정은 다시 열어 줌. `~/.gitconfig`, `$XDG_CONFIG_HOME/git`(기본 `~/.config/git`) 아래의 `config`·`ignore`·`attributes`, 그리고 전역 설정이 `[include]`·`[includeIf]`·`core.excludesFile`·`core.attributesFile` 로 가리키는 파일들임. 다시 여는 규칙에 조건이 하나 붙음.

> "When a file lies where a sandboxed command can write, directly or through a symlink, Claude Code doesn't re-open the files it names."

즉 **쓸 수 있는 자리에 있는 설정 파일이 가리키는 것은 다시 열지 않음.** 샌드박스 안에서 설정 파일을 고쳐 바깥 경로를 끌어오는 경로를 닫은 것임. `~/.git-credentials` 와 `$XDG_CONFIG_HOME/git/credentials` 는 계속 막혀 있고, 다시 열린 파일에 `http.extraHeader` 토큰 같은 비밀이 들어 있으면 `sandbox.filesystem.denyRead` 에 그 경로를 넣으라고 문서가 안내함. `denyRead` 는 이 재개방보다 우선함.

**블록이 샌드박스에 닿지 않는 경우가 셋 있음.** 이때도 Claude 의 파일 툴은 블록을 계속 집행하지만, 샌드박스된 명령은 울타리 밖으로 나갈 수 있음.

- `sandbox.filesystem.disabled` 로 파일시스템 격리를 끈 경우
- `allowManagedReadPathsOnly` 가 설정된 경우
- **Claude Code 를 시작한 디렉터리의 경로에 `*`·`?`·`[` 같은 글로브 문자가 들어 있는 경우**

마지막 것이 읽다가 멈추게 되는 항목임. 디렉터리 이름에 대괄호를 쓰는 습관이 있으면(날짜나 티켓 번호를 `[AB-123]` 처럼 묶는 식으로) 블록을 켜 뒀어도 샌드박스 쪽에서는 조용히 적용되지 않음. 설정은 켜져 있고 경고도 없음.

## iOS 쪽에서 울타리를 어디에 그을까

Xcode 작업은 저장소 폴더 밖에 산출물과 자재를 많이 둠. DerivedData, 프로비저닝 프로파일, 키체인, 시뮬레이터 상태가 전부 홈 디렉터리 아래임. 울타리를 켜면 그 경로를 파일 툴로 건드리는 작업은 `/add-dir` 로 명시적으로 열어야 하고, 그게 이 설정의 비용이자 값어치임. 무엇이 울타리 밖인지 한 번 세 보게 됨.

다만 **빌드는 Bash 로 돌음.** `xcodebuild` 가 DerivedData 에 쓰는 것은 파일 툴 호출이 아니므로 위 세 툴 목록에 걸리지 않을 것으로 읽히는데, **쓰기 쪽 설정에 문서가 없어 이 추론은 확인하지 못했음.** 읽기 쪽은 셸을 같은 방식으로 거부하지 않는다고 문서가 명시하므로 방향은 같을 가능성이 높음. 셸이 울타리 밖이라면 이 설정으로 막히는 것은 **에이전트가 파일 툴로 홈 디렉터리를 직접 고치는 경로**까지이고, 빌드 도구를 경유하는 쓰기는 샌드박스 쪽 일이 됨.

[macOS Full Disk Access 제한 예고](./2026-10-02-macos-full-disk-access-controls.md)가 다룬 것이 같은 울타리를 OS 쪽에서 좁히는 움직임이고, Apple 이 그 근거로 AI 에이전트를 댔음. 두 층이 같은 방향으로 움직이는 중이라고 보는 편이 맞음. 하네스 설정으로 긋는 울타리와 OS 권한으로 긋는 울타리는 서로를 대체하지 않음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Code 2.1.309의 권한 우회 수정 묶음](./2026-10-22-claude-code-permission-bypass-fixes.md) | 그 문서가 "로컬 권한 규칙은 경계가 아니라 완화 수단"으로 끝냈음. 경로 울타리는 룰이 아니라 모드 위에 얹히는 다른 종류이고, 그래도 셸은 밖에 있음 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 셸 쪽이 거부가 아니라 프롬프트로 떨어지는 부분. 판정을 가장 약한 층으로 옮기는 설계가 되기 쉬움 |
| [Claude를 제품마다 어떻게 격리하는가](./2026-05-25-containing-claude-across-products.md) | 샌드박스로 프롬프트를 84% 줄인 쪽과 같은 방향. 샌드박스가 블록을 집행하면 위 프롬프트가 사라짐 |
| [macOS Full Disk Access 제한 예고](./2026-10-02-macos-full-disk-access-controls.md) | 같은 울타리를 OS 권한 쪽에서 좁히는 움직임 |
| [팀 공유 AI 하네스 만들기](../agents/2026-08-10-hq-team-ai-harness.md) | 설정을 팀 기본값으로 배포하는 경로. **저장소 설정이 울타리를 넓히지 못한다**는 규칙이 그 경로에 직접 걸림 |

## 짚어야 할 것

- **쓰기 쪽 설정은 문서가 없음.** 2026-11-01 확인 기준으로 설정 레퍼런스의 색인과 본문, Configure permissions, Permission modes 어디에도 `permissions.blockWritesOutsideWorkingDirectories` 가 없음. 근거는 v2.1.314 릴리스 노트 한 줄뿐임. **기본값, scope, 샌드박스 취급, 심링크 취급은 확인할 수 없었음**
- 따라서 **관리 설정이 쓰기 블록을 `false` 로 덮을 수 있는지 모름.** 읽기 쪽 키는 "Exceptions to managed settings precedence" 표에 올라 `true` 가 어느 파일에서든 이기는데, 쓰기 쪽 키는 그 표에 없음. 문서가 하루 늦은 것인지 실제로 규칙이 다른 것인지 **갈라낼 근거가 없음**
- **Bash 가 쓰기 블록에 걸리는지도 미확인.** 릴리스 노트가 적은 것은 Write·Edit·NotebookEdit 셋임. 위 iOS 절의 `xcodebuild` 추론이 여기 걸려 있음
- **재현하지 않았음.** 설정을 실제로 켜서 동작을 확인한 것이 아니라 릴리스 노트와 문서 문언을 읽은 것임
- **릴리스 노트에 날짜가 없음.** 저장소의 `CHANGELOG.md` 도 항목에 날짜가 없어서, 이 문서의 날짜는 전부 `gh api` 로 받은 릴리스 `published_at`(UTC) 기준임
- v2.1.316 이 고친 `--restricted` 업로드 구멍이 **언제부터 있었는지는 노트에 없음.** `--restricted` 자체가 v2.1.248(2026-08-27) 부터이므로 노출 기간의 상한이 약 두 달이라는 것까지만 말할 수 있음
- 이 문서가 쓰는 "울타리"라는 말은 정리자의 표현임. Anthropic 문서는 block, confine, fence 같은 단어를 섞어 쓰고 통일된 이름을 두지 않음

## 유효기간

**2026-11-01 확인 기준이고 최신 릴리스는 2.1.316** 임. 이 릴리스는 하루에 한두 개씩 나오므로 버전 번호는 읽는 시점에 이미 낡았다고 보는 게 맞음. 다시 볼 때 확인할 것 셋. **설정 레퍼런스에 `permissions.blockWritesOutsideWorkingDirectories` 항목이 생겼는지**(생기면 위 미확인 넷이 한 번에 풀림), **"Exceptions to managed settings precedence" 표에 쓰기 키가 올라갔는지**, 그리고 **auto 모드가 첫 바깥 쓰기에도 1회 프롬프트를 띄우게 됐는지**(읽기 쪽에 이미 있는 구조이고, 같은 방식으로 사용자 설정을 직접 쓰게 됨). 셸 쪽이 프롬프트 말고 거부로 바뀌는지가 가장 큰 변경이 될 자리인데, 그건 Bash 권한 검사 구조를 바꾸는 일이라 당장은 기대하지 않는 편이 맞음.
