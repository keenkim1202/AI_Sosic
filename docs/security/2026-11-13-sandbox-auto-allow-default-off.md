---
title: 샌드박스가 프롬프트 면제를 잃었음, autoAllowBashIfSandboxed 기본값이 false 로 뒤집힘
source: https://github.com/anthropics/claude-code/releases/tag/v2.1.325
author: Anthropic
published: 2026-11-13
collected: 2026-11-15
tags: [agent-security, claude-code, sandboxing, permissions, human-in-the-loop, guardrail]
---

출처: [Claude Code v2.1.325 릴리스 노트](https://github.com/anthropics/claude-code/releases/tag/v2.1.325) (2026-11-13) · [CHANGELOG.md](https://raw.githubusercontent.com/anthropics/claude-code/main/CHANGELOG.md) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) · [Permission modes](https://code.claude.com/docs/en/permission-modes) · [Configure permissions](https://code.claude.com/docs/en/permissions) (문서 네 건은 **2026-11-15 확인**. 릴리스 날짜는 `gh api` 의 `published_at`(UTC) 기준임)

## 요약

**샌드박스를 켜면 Bash 프롬프트가 사라진다는 전제가 기본값에서 빠졌음.** v2.1.325 릴리스 노트가 "sandboxed Bash commands to follow the normal permission flow by default" 로 적고, 설정 레퍼런스가 `sandbox.autoAllowBashIfSandboxed` 의 **기본값을 `false`, 그 이전 기본값을 `true`** 로 명시함. 되돌리려면 키를 직접 `true` 로 써야 함. 이 저장소에 이게 걸리는 이유는 Anthropic 자신이 [반년 전 글](./2026-05-25-containing-claude-across-products.md)에서 **승인 프롬프트의 93% 가 그냥 승인되고 있어서 OS 샌드박스를 깔아 프롬프트를 84% 줄였다**고 적었기 때문임. 그 84% 를 만들던 장치가 이제 기본으로 꺼져 있음.

바꿔 말하면 **격리와 승인이 분리됐음.** 샌드박스는 전과 같은 파일시스템·네트워크 경계를 그대로 집행하고, 달라진 것은 "경계 안이니 물어보지 않는다"는 판단뿐임. 그래서 이 변경은 보안을 조이는 쪽이 아니라 **프롬프트 수를 되돌리는 쪽**이고, 그 프롬프트를 받는 사람의 승인 정확도는 이 저장소가 이미 두 번 측정해 둔 값임. 그리고 **`dontAsk` 모드와 `-p` 실행에서는 프롬프트가 거부로 떨어지므로**, 샌드박스에 기대 무인으로 돌던 설정은 기본값만 따라가면 조용히 멈출 수 있음.

## 기본값이 뒤집힌 지점

| 항목 | 내용 |
|---|---|
| 설정 키 | `sandbox.autoAllowBashIfSandboxed` |
| scope | `Any file`. 저장소 설정에서도 설정 가능 |
| 타입 | Boolean |
| `true` | 샌드박스된 Bash 명령을 프롬프트 없이 돌림. `/sandbox` 의 **Mode** 탭에서 auto-allow |
| `false` | 샌드박스된 명령도 정규 권한 흐름을 탐. Mode 탭에서 **regular permissions** |
| 기본값 | **`false`.** 문서 문언은 "Before v2.1.324, the default was `true`" |

설정 레퍼런스 본문이 적는 범위가 중요함.

> "Let Claude Code run sandboxed Bash commands without a permission prompt. Commands that can't run in the sandbox still go through the regular permission flow. `deny` rules and content-scoped `ask` rules such as `Bash(git push *)` still apply, but a bare `Bash` ask rule is skipped for sandboxed commands."

즉 `true` 로 되돌려도 **deny 룰과 내용이 박힌 ask 룰은 그대로 살아 있음.** 면제되는 것은 `Bash` 또는 `Bash(*)` 형태의 포괄 ask 룰 하나임. 그래서 이 키를 켜는 것과 `bypassPermissions` 로 가는 것은 전혀 다른 일인데, 기본값이 `true` 였을 때는 그 구분이 잘 보이지 않았음.

릴리스 노트의 짝 항목이 하나 더 있음. 관리 설정으로 샌드박스를 강제하는 조직은 **최상위 관리 소스의 `sandbox` 블록에 이 키를 `true` 로 적어야** 프롬프트 없는 동작이 유지됨. 샌드박스를 켜는 것만으로는 더 이상 따라오지 않음.

## v2.1.324 는 공개된 릴리스가 아님

문서가 경계로 지목한 버전이 **공개 릴리스로 존재하지 않음.** `gh api repos/anthropics/claude-code/releases` 로 받은 목록에서 v2.1.323(2026-11-12) 다음이 v2.1.325(2026-11-13) 이고, `git/matching-refs/tags/v2.1.32` 에도 `v2.1.324` 태그가 없음. 저장소의 `CHANGELOG.md` 도 `## 2.1.325` 바로 다음이 `## 2.1.323` 임.

그래서 실제로 이 변경을 받는 자리는 이럼.

- **사용자가 체감하는 경계는 v2.1.323 에서 v2.1.325 로 올라가는 구간**임. 중간 버전을 건너뛰는 것이 아니라 그 번호가 공개되지 않았음
- 기본값 변경을 알려 주는 문장은 **설정 레퍼런스 한 줄과 릴리스 노트 한 줄**뿐임. 샌드박싱 문서의 Sandbox modes 절은 두 모드를 나란히 설명하면서 **어느 쪽이 기본인지 적지 않음**
- 따라서 `/sandbox` 를 열어 Mode 탭을 직접 보는 것이 지금 자기 상태를 확인하는 가장 빠른 길임

## auto-allow 를 켜도 프롬프트가 뜨던 자리들

기본값을 되돌릴지 판단하려면 `true` 가 실제로 무엇을 면제했는지 알아야 함. 샌드박싱 문서가 세는 예외가 이것임.

- `Bash` **deny 룰에 걸리는 형태**는 그대로 차단됨
- `rm`·`rmdir` 이 **critical path** 를 가리키는 것으로 인식되면 정규 권한 흐름을 탐
- `Bash(git push *)` 처럼 **내용이 박힌 ask 룰**은 샌드박스된 명령에도 프롬프트를 띄움
- **plan 모드에서는 포괄 `Bash` ask 룰이 면제되지 않음.** 읽기 전용 명령까지 프롬프트가 뜸
- **Monitor 툴 명령은 자동 승인 대상이 아님.** 샌드박스 안에서 돌기는 함
- **앞에 변수 대입이 붙거나 실행 프로그램이 바뀔 수 있는 형태**는 자동 승인되지 않음. 문서가 드는 예가 `FOO=bar npm test`, `make CC=clang`, `eval "ls"` 임
- Linux 에서 `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` 이 설정돼 있으면 **auto-allow 자체가 꺼짐**

마지막 세 항목이 이 변경의 성격을 보여 줌. auto-allow 는 처음부터 완전한 면제가 아니었고 **"프로그램이 무엇인지 문자열로 확정할 수 있을 때만" 면제**였음. [2026-10-22 권한 우회 수정 묶음](./2026-10-22-claude-code-permission-bypass-fixes.md)이 센 셸 문법 계열 8건이 바로 그 확정이 실패하던 자리들이고, 그래서 면제의 경계가 계속 움직였음. 기본값을 끄는 것은 그 경계를 유지보수하는 일 자체를 기본 경로에서 뺀 것으로 읽힘.

## `-p` 와 `dontAsk` 에서는 프롬프트가 거부가 됨

여기가 이 변경에서 실무로 가장 먼저 닿는 지점임.

권한 모드 문서가 `dontAsk` 를 "auto-denies every tool call that would otherwise prompt you" 로 적고, 그 모드의 용도를 **CI 파이프라인과 제한된 환경**으로 안내함. 프롬프트를 받을 사람이 없는 실행에서 프롬프트는 거부로 착지함.

그래서 조합을 나눠 보면 이렇게 됨.

| 설정 | 기본값 변경 전 | 변경 후 |
|---|---|---|
| 샌드박스 on, `dontAsk` 또는 `-p` | 샌드박스된 Bash 가 프롬프트 없이 돎 | **포괄 `Bash` 허용 룰이나 명령별 allow 룰이 없으면 거부**로 떨어짐 |
| 샌드박스 on, Manual 모드 | 샌드박스된 Bash 가 자동 승인 | 명령마다 프롬프트 |
| 샌드박스 on, auto 모드 | 샌드박스 경계가 프롬프트를 대신함 | 분류기가 명령을 검토함 |

`dontAsk` 행은 **문서 두 곳의 문언을 이어 붙인 추론임.** 실제로 돌려서 확인한 것이 아님(아래 짚어야 할 것). 다만 방향은 확실히 이쪽이고, 무인으로 도는 설정이 있으면 기본값을 받기 전에 allow 룰을 먼저 세는 편이 안전함. 읽기 전용 명령 집합(`ls`, `cat`, `grep`, `find`, 읽기 전용 `git` 등)은 모드와 무관하게 프롬프트 없이 도므로, 끊기는 것은 **빌드·테스트·설치처럼 쓰기를 하는 명령**임.

## 같은 릴리스에서 프롬프트가 늘어난 곳이 셋 더 있음

기본값 하나만 바뀐 게 아니고 **방향이 같은 변경이 묶여 들어왔음.** 릴리스 노트 문언 그대로임.

- **읽기 전용 git 명령**이 "a repository that isn't the project's or one you trusted" 에서는 먼저 물어봄. 프롬프트 없이 돌리려면 `additionalDirectories` 가 아니라 `--add-dir` 로 그 저장소를 지목해야 함
- **스케줄된 태스크와 `/loop`** 가 `allowed-tools` 나 훅을 선언한 스킬을 돌리기 전에 물어봄. 무인으로 돌리려면 `permissions.allow` 에 `Skill(name)` 을 넣어야 함
- Desktop·IDE·SDK 호스트의 **"Always allow"** 가 룰만으로 다시 허용되는 경우에만 제시됨. 그래서 그 룰에 기대던 스케줄 태스크가 사람을 기다리게 될 수 있음

셋의 공통점은 **"한 번 허용해 두면 다음부터 조용히 통과"하는 경로를 줄인 것**임. `additionalDirectories` 가 더 이상 안 되고 `--add-dir` 로 지목해야 한다는 항목은 [작업 디렉터리 밖 쓰기 차단](./2026-10-30-working-directory-write-block.md)이 정리한 비대칭(저장소 설정은 울타리를 넓히지 못함)과 같은 모양임. 설정 파일에 적어 둔 영구 완화보다 **세션 시작 시의 명시적 지목**을 요구하는 쪽으로 가고 있음.

같은 릴리스에 샌드박스 경계 자체를 좁히는 항목도 하나 있음. 샌드박스가 **추가 디렉터리 안의 `.claude` 설정·스킬·에이전트·커맨드·훅에 대한 쓰기를 거부**하게 됐음. 시작 폴더에 이미 있던 규칙을 `--add-dir` 로 붙인 곳까지 넓힌 것임.

## 저장소가 되돌릴 수 있는 유일한 샌드박스 키

관리 설정으로 샌드박스를 강제하는 조직 입장에서 짚어야 할 비대칭이 있음.

샌드박싱 문서의 "Repository settings under an admin-required sandbox" 절은 admin-required 상태에서 저장소 파일(`.claude/settings.json`, `.claude/settings.local.json`)의 샌드박스 완화 설정을 **대부분 무시**한다고 적음. `excludedCommands`, `network.allowedDomains`, `filesystem.allowWrite` 가 전부 거기 들어감. 그런데 계속 적용되는 것으로 명시된 항목이 둘임.

> "**In a repository's files**: deny entries and the `autoAllowBashIfSandboxed` value. Set the key in managed settings to keep a repository from changing it"

즉 **저장소 파일이 바꿀 수 있는 거의 유일한 완화 스위치가 이 키임.** 그리고 `/sandbox` 의 Mode 탭에서 모드를 고르면 Claude Code 가 그 값을 **프로젝트의 `.claude/settings.local.json` 에 저장**함(문서는 그 파일을 전역 gitignore 에 넣어 준다고 적음). 기본값이 `true` 였을 때는 이 경로가 "원래 그런 동작을 적어 두는 자리" 였는데, 기본값이 `false` 가 된 다음에는 **같은 경로가 기본값을 뒤집는 자리**가 됨.

조직 쪽에서 프롬프트 없는 동작을 유지하려면 최상위 관리 소스에 `true` 를 적어야 하고, 반대로 저장소가 프롬프트를 끄지 못하게 막으려면 같은 자리에 `false` 를 적어 잠가야 함. **둘 다 같은 키를 관리 설정에 명시하는 일이고, 적지 않으면 저장소 쪽이 이김.**

## 84% 를 되돌리는 변경인가

[Claude를 제품마다 어떻게 격리하는가](./2026-05-25-containing-claude-across-products.md)가 인용한 자사 문언이 이것임.

> "Our telemetry showed users approved roughly 93% of permission prompts."

> "The result was an 84% reduction in permission prompts, and we open-sourced the runtime."

승인률 93% 를 근거로 **승인 정확도를 올리는 대신 승인 요청 자체를 없애는** 길을 골랐다는 서술이었고, [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md)의 처방도 같았음. 그 두 문서를 읽은 상태에서 이 기본값 변경을 보면 **반대 방향**으로 보임.

다만 "되돌렸다"고 단정하면 안 되는 이유가 둘 있음.

- **격리는 그대로임.** 샌드박스가 집행하는 파일시스템·네트워크 경계는 두 모드에서 동일하고, 문서가 그 점을 먼저 적음. 줄어든 것은 위험 노출이 아니라 **자동 승인 범위**임. 93% 승인률이 문제였던 이유는 프롬프트가 많아서였는데, 프롬프트를 늘리면 그 문제가 그만큼 돌아옴
- **면제의 경계가 반복해서 뚫렸음.** 위 예외 목록과 이 저장소의 [권한 우회 수정 묶음](./2026-10-22-claude-code-permission-bypass-fixes.md)이 같은 이야기를 함. "샌드박스 안이니 자동 승인" 이 성립하려면 명령이 정말 샌드박스 안에서 도는지를 매번 맞혀야 하는데, 그 판정이 틀린 사례가 계속 나왔음

그래서 읽는 쪽에서 할 판단은 **어느 쪽이 맞느냐가 아니라 자기 설정에서 둘 중 무엇을 고를지**임. 기준을 하나만 세우면 이것임. **프롬프트를 사람이 실제로 읽을 수 있는 횟수로 유지할 수 있으면 `false`, 그렇지 않으면 `true` 로 두고 deny·ask 룰로 좁히는 편이 나음.** 하루 수십 번 뜨는 프롬프트는 위 93% 쪽으로 수렴하고, 그 상태의 `false` 는 통제가 아니라 통제의 외형임.

## iOS 쪽에서 당장 걸리는 것

Xcode 작업은 Bash 명령 하나가 길고 오래 걸림. `xcodebuild`, `xcrun simctl`, `swift test`, 패키지 해석이 전부 쓰기를 하므로 읽기 전용 집합에 없음. 기본값을 그대로 받으면 **빌드 한 번에 프롬프트 한 번**이 됨.

- 명령별 allow 룰로 좁히는 편이 포괄 허용보다 나음. `Bash(xcodebuild *)`, `Bash(xcrun simctl *)`, `Bash(swift test *)` 처럼 **자기가 반복해서 돌리는 것만** 적는 방식임. 이러면 `autoAllowBashIfSandboxed` 를 `false` 로 둔 채 프롬프트가 사라지고, 목록에 없는 명령은 계속 물어봄
- 시뮬레이터를 샌드박스 안에서 돌릴 때 필요한 키가 따로 있음. 설정 레퍼런스가 `sandbox.network.allowMachLookup` 을 **"Let macOS sandboxed tools like the iOS Simulator or Playwright reach their XPC services"** 로 적음. 이 키는 프롬프트와 무관하게 샌드박스 경계 쪽 설정임
- DerivedData 와 시뮬레이터 상태가 홈 디렉터리 아래에 있다는 문제는 그대로 남음. 그쪽은 [작업 디렉터리 밖 쓰기 차단](./2026-10-30-working-directory-write-block.md)의 울타리와 `sandbox.filesystem.allowWrite` 가 다루는 자리이고, 이번 변경과는 층이 다름

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude를 제품마다 어떻게 격리하는가](./2026-05-25-containing-claude-across-products.md) | 승인률 **93%**, 샌드박스로 프롬프트 **84% 감소**. 이번 변경이 그 84% 를 기본에서 빼는 쪽임 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 늘어난 프롬프트를 받는 사람의 판정 정확도. npm 계열 **52.5%** 가 정확히 빌드·설치 명령 자리임 |
| [Claude Code 2.1.309의 권한 우회 수정 묶음](./2026-10-22-claude-code-permission-bypass-fixes.md) | "샌드박스 안이니 자동 승인" 판정이 반복해서 틀렸던 기록. 면제를 기본에서 빼는 근거로 읽을 수 있는 쪽 |
| [작업 디렉터리 밖 쓰기 차단](./2026-10-30-working-directory-write-block.md) | 저장소 설정이 울타리를 넓히지 못하는 비대칭. 이번 `--add-dir` 요구와 같은 모양이고, `autoAllowBashIfSandboxed` 는 그 비대칭의 예외임 |
| [Claude Code mods](../agents/2026-10-04-claude-code-mods.md) | mod 가 띄운 프로세스는 샌드박스 밖이라는 점. 이번 변경은 Bash 쪽 승인만 건드리고 그 표면은 그대로임 |
| [Managed Agents 의 자동 권한 판정](./2026-09-10-managed-agents-auto-permission.md) | 사람 없는 실행에서 프롬프트가 무엇으로 대체되는가. `dontAsk` 가 거부로 착지하는 것과 같은 문제 |

## 짚어야 할 것

- **재현하지 않았음.** 설정을 켜고 끄며 동작을 확인한 것이 아니라 릴리스 노트와 공식 문서 문언을 읽은 것임
- **`dontAsk`·`-p` 에서 샌드박스된 쓰기 명령이 실제로 거부되는지는 확인할 수 없었음.** 위 표의 그 행은 권한 모드 문서의 "auto-denies every tool call that would otherwise prompt you" 와 샌드박스 모드 설명을 이어 붙인 추론임. 두 문서 어디에도 이 조합을 직접 적은 문장이 없음
- **기본값 변경의 이유가 어디에도 없음.** 릴리스 노트는 변경만 적고 근거를 적지 않음. 위 "84% 를 되돌리는 변경인가" 절의 해석은 정리자의 것이고 Anthropic 이 그렇게 설명한 바 없음
- **v2.1.324 가 왜 공개되지 않았는지 확인할 수 없었음.** 태그도 릴리스도 CHANGELOG 항목도 없고, 설정 레퍼런스만 그 번호를 경계로 적음. 내부 빌드로 보이지만 근거는 없음
- **샌드박싱 문서가 기본값을 적지 않음.** 2026-11-15 확인 기준으로 Sandbox modes 절은 두 모드를 설명하면서 어느 쪽이 기본인지 말하지 않고, Get started 는 "select auto-allow or regular permissions" 로만 안내함. 기본값 문언은 설정 레퍼런스 쪽에만 있음
- **읽기 전용 git 명령의 새 프롬프트가 문서에 없음.** 2026-11-15 확인 기준으로 Configure permissions 의 읽기 전용 명령 절에 "저장소가 프로젝트의 것이 아니거나 신뢰하지 않은 경우" 라는 조건이 없음. 근거는 릴리스 노트 한 줄뿐임
- 릴리스 노트에 날짜가 없어서 이 문서의 날짜는 전부 `gh api` 로 받은 릴리스 `published_at`(UTC) 기준임

## 유효기간

**2026-11-15 확인 기준이고 최신 릴리스는 2.1.325(2026-11-13)** 임. 이 릴리스는 하루에 한두 개씩 나오므로 버전 번호는 읽는 시점에 이미 낡았다고 보는 게 맞음. 기본값 자체는 설정 레퍼런스에 박힌 사실이라 되돌려지면 같은 자리에 적힘. 다시 볼 때 확인할 것 셋. **샌드박싱 문서의 Sandbox modes 절이 기본값을 적게 됐는지**(적히면 위 미확인 하나가 풀림), **읽기 전용 git 명령의 저장소 신뢰 조건이 Configure permissions 에 올라갔는지**, 그리고 **`dontAsk`·`-p` 조합에 대한 서술이 생겼는지**. 셋 다 지금은 릴리스 노트 한 줄에만 근거가 있는 항목임.
