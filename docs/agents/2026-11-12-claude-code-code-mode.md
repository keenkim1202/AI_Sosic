---
title: Claude Code 의 Code Mode, 툴 호출이 JavaScript 한 덩어리로 묶임
source: https://code.claude.com/docs/en/code-mode
author: Anthropic
published: 2026-11-12
collected: 2026-11-14
tags: [claude-code, permissions, agent-security, harness, human-in-the-loop, tooling]
---

출처: [Run tool calls from code with Code Mode](https://code.claude.com/docs/en/code-mode) · [v2.1.323 릴리스 노트](https://github.com/anthropics/claude-code/releases/tag/v2.1.323) (2026-11-12) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Environment variables](https://code.claude.com/docs/en/env-vars) · [Tools reference](https://code.claude.com/docs/en/tools-reference) (문서 네 건은 **2026-11-14 확인**. 릴리스 날짜는 `gh api` 의 `published_at`(UTC) 기준임)

## 요약

**v2.1.323 에 Code Mode 가 붙었음.** Claude 가 Bash·Read·Edit 를 하나씩 호출하고 결과를 읽어 다음을 정하는 대신, **툴 호출을 하는 JavaScript 를 한 덩어리로 써서 세션 안에서 돌림.** 한 덩어리 안에서 명령을 병렬로 돌리고, 앞 결과를 뒤 호출에 넣고, Claude 가 읽어야 할 부분만 골라 반환함. 실험 기능이고 기본은 꺼져 있고 **어느 릴리스에서든 바뀌거나 사라질 수 있다고 문서가 직접 적음.**

이 저장소에서 중요한 줄은 기능 설명이 아니라 권한 쪽임. **auto 모드의 분류기가 검사하는 대상이 툴 호출 하나하나에서 코드 덩어리로 올라감.** 그리고 문서가 그 한계를 스스로 적어 둠. **분류기는 실행 시점에 계산되는 값을 못 봄.** 검색 출력에서 뽑아낸 경로 같은 것이 그렇다고 예시까지 들어 있음. `deny`·`ask` 룰과 `PreToolUse` 훅은 개별 호출에 그대로 걸리므로, **통제를 룰과 훅으로 내려둔 사람은 그대로 가고 분류기에 기대고 있던 사람만 검사 해상도를 잃음.**

## 무엇이 달라지는가

문서의 예시 코드가 변화를 가장 짧게 보여줌. 재시도 한도를 올려 달라는 프롬프트 하나에 대해 Claude 가 쓰는 것은 툴 호출 세 번이 아니라 이 네 줄임.

```javascript
const files = await $`rg -l "function retryPolicy" src`.text();
const target = files.trim().split("\n")[0];
await edit(target, "maxAttempts: 3", "maxAttempts: 5");
({ edited: target, tests: $`npm test -- retryPolicy`.nothrow() });
```

대화창에는 코드가 아니라 **툴 호출이 보임.** edit 의 diff 는 그대로 뜸. 코드를 보려면 `Ctrl+O` 로 트랜스크립트 뷰어를 열고 접혀 있는 **Code** 행을 펼쳐야 함. 즉 기본 화면에서 보는 것과 실제로 실행된 것 사이에 한 층이 생김.

문서가 권하는 쓸 자리는 셋임. 여러 파일에 같은 변경이 반복되는 작업, 결과를 걸러낸 다음에 Claude 가 읽어야 하는 검색, 동시에 돌려도 되는 독립 명령 여러 개.

## 권한: 검사 대상이 코드로 올라감

갈라 보면 이렇게 됨.

| 층 | Code Mode 에서 | 비고 |
|---|---|---|
| `deny`·`ask` 권한 룰 | **개별 툴 호출에 그대로 걸림** | `Bash` 같은 툴의 룰이나 matcher 를 고칠 필요 없다고 문서가 명시 |
| `PreToolUse` 훅 | **개별 툴 호출에 그대로 걸림** | 코드 자체를 보려면 `REPL` 을 matcher 로 걸고 `tool_input.code` 를 읽음 |
| auto 모드 분류기 | **코드 덩어리 단위로 올라감** | 실행 전 1회. 호출 하나하나를 보지 않음 |
| 그 밖의 권한 모드 | 검토 없이 코드가 시작되고, **그 모드에서 물어볼 호출마다 그때 물음** | |

문서가 적어 둔 분류기의 한계가 핵심임. **실행 시점에 계산되는 값은 분류기 시야에 없음.** 위 예시에서 `target` 은 `rg` 출력에서 뽑힌 값이라 분류기가 코드를 읽는 시점에는 존재하지 않음. 문서의 결론도 같음. "반드시 통제해야 하는 호출에는 룰이나 훅을 유지하라."

이 저장소가 같은 모양을 이미 두 번 적어 뒀음. [2.1.309 의 권한 우회 수정 묶음](../security/2026-10-22-claude-code-permission-bypass-fixes.md)은 **명령 문자열을 파싱해 허용·거부를 정하는 층이 실제 실행 결과에 번번이 졌다**는 이야기고, [작업 디렉터리 밖 쓰기 차단](../security/2026-10-30-working-directory-write-block.md)은 그래서 **문자열 검사 대신 경로 울타리로 옮겨 가는 중**이라는 이야기임. Code Mode 의 분류기는 그 축에서 반대쪽으로 한 칸 간 것임. 검사 단위가 커지면 정적으로 못 보는 부분도 같이 커짐.

문서 본문에 경고가 하나 더 붙어 있고, 이건 Code Mode 탓이 아니라고 문서가 먼저 밝힘. **Bash 룰은 명령 텍스트를 매칭하므로 Claude 는 거부된 명령을 룰이 매칭하지 않는 형태로 실행할 수 있음.** "with or without Code Mode" 라고 적혀 있음. 명령 텍스트에 의존하지 않는 통제는 샌드박싱 쪽으로 넘김.

## 훅이 실패했을 때의 규칙이 달라짐

여기가 조용히 깨질 수 있는 자리임. `PreToolUse` 훅이 결정을 못 내리면 호출이 거부되는데, **`onFailure` 를 설정하지 않은 경우에도 거부됨.** 두 경우로 갈림.

| 에러 문구 | 조건 |
|---|---|
| `the hook failed` | Claude 의 코드에서 나온 호출에 대해 훅이 시작되지 못했거나, 죽었거나, 126·127 또는 128 초과 코드로 종료했거나, HTTP 에러나 잘못된 출력을 반환함 |
| `the hook did not finish` | Claude 의 코드에서 나온 호출에서 훅이 타임아웃했거나, **세션에서 Claude 가 코드를 한 번 돌린 뒤의 다른 모든 툴 호출**에서 타임아웃함 |

두 번째 줄이 중요함. 코드를 한 번 돌리면 그 뒤로는 **일반 툴 호출에서도** 훅 타임아웃이 거부로 바뀜. 훅이 느린 검사(원격 호출, 린트 전체 실행)를 하고 있으면 Code Mode 를 켠 세션에서 처음 보는 거부가 나옴. 고치는 쪽은 훅의 `timeout` 을 올리거나 `command`·`http` 훅에 `"onFailure": "continue"` 를 주는 것임.

`UserPromptSubmit` 훅에도 새 경로가 생김. **코드가 백그라운드로 돌 수 있는 상태면 `command`·`http`·`mcp_tool` 훅이 코드 출력에도 돌아감.** 출력이 통째로 막히는 모양으로 나타나고, 입력의 `source` 가 `poll_event` 일 때 막지 않도록 훅을 고치라고 적혀 있음.

## 시간 제한과 모드 의존

| 상태 | 수명 | 조건 |
|---|---|---|
| 대기 | **10분**. 권한 프롬프트 앞에서 보낸 시간도 포함 | 기본 |
| 백그라운드 | **최대 2시간**. 코드가 도는 동안 Claude 와 계속 대화 가능 | 세션을 **auto 또는 `bypassPermissions` 모드로 시작**해야 함 |

"권한 프롬프트 앞에서 보낸 시간도 10분에 포함" 이 실무에서 먼저 걸림. 프롬프트를 띄우는 모드에서 긴 빌드를 돌리면 사람이 자리를 비운 사이 코드가 정지함.

백그라운드 조건이 **시작 시점의 모드**라는 것도 함정임. `Shift+Tab` 으로 모드를 옮겼거나 `claude --resume` 목록에서 세션을 골랐으면 안 됨. 문서는 `claude --continue --permission-mode auto` 로 다시 들어가라고 안내함. `--tools` 로 툴 이름을 넘긴 경우도 안 됨.

auto 모드에는 카운트다운 프롬프트가 하나 더 있음. **앞선 코드가 도는 중에 Claude 가 새 코드를 시작하면 앞선 코드가 분류기 검토를 잃음.** 그러면 남은 호출에 대해 사람에게 물음. IDE 가 연결돼 있지 않으면 `1:59` 카운트다운이 붙고, **답하지 않으면 그 호출이 거부되고 이후 카운트다운이 붙을 호출은 프롬프트 없이 거부됨.** 다시 물어보게 하려면 Claude 에게 메시지를 보내야 함. 무인 세션에서 조용히 전부 거부로 수렴할 수 있는 모양임.

## 켜고 끄기, 그리고 설정 파일별 규칙

환경변수 하나임. `CLAUDE_CODE_EXPERIMENTAL_CODE_MODE=1`. 첫 세션 전에 확인 프롬프트가 뜨고(**Yes, turn on Code Mode**), Claude Code 가 재시작되면서 세션 상단에 `Code Mode (experimental) is on · this session can be resumed only with Code Mode on` 이 붙음.

설정 파일별 규칙은 두 문서의 서술이 갈리므로 더 구체적인 쪽(`settings-reference`)을 기준으로 적음.

| 위치 | `1` 로 켜짐 | `0` 으로 꺼짐 |
|---|---|---|
| 셸 환경변수 | 켜짐. 단 **관리 설정에 `1` 이 있으면 셸의 `1` 은 무시됨** | 꺼짐 |
| 사용자 설정(`~/.claude/settings.json`) | 켜짐 | 꺼짐 |
| `--settings` 파일 | 켜짐 | 꺼짐 |
| 관리 설정(managed) | **안 켜짐.** 관리 설정으로 사용자에게 켜 줄 수는 없음 | **꺼짐.** 조직 전체 차단 수단 |
| 프로젝트·로컬 설정 | **무시됨** | 무시됨 |

프로젝트 설정이 무시되는 이유를 문서가 밝혀 둠. **클론한 저장소가 나를 위해 Code Mode 를 켤 수 없게 하려는 것.** 이 변수는 저장소가 제어해서는 안 되는 변수 목록에 들어가 있고, 그 목록에는 `CLAUDE_CONFIG_DIR`, `HOME`, OTel exporter 변수, claude.ai 로그인 토큰 변수 같은 것들이 같이 있음.

끄려면 셸과 사용자 설정에서 변수를 빼고 새 세션을 시작하거나, 값을 `0` 으로 바꿈. **한쪽에 `0` 이 있으면 다른 쪽의 `1` 과 무관하게 꺼짐.**

## 못 쓰는 조건이 넓음

실험 기능이라 배제 목록이 기능 설명보다 길다고 봐도 됨. 이 저장소 주인에게 걸리는 것만 골라 적음.

| 축 | 조건 |
|---|---|
| 버전 | **v2.1.323 이상**, `stable`·`latest` 어느 채널이든. 설치는 **네이티브 빌드** |
| 표면 | **터미널의 대화형 `claude` 세션만.** `-p`, Agent SDK, **VS Code 패널**, **데스크톱 앱**, 클라우드 세션은 안 됨 |
| 계정 | Claude 구독 또는 Console 계정. **게이트웨이, Amazon Bedrock, Claude Platform on AWS, Google Cloud Agent Platform, Microsoft Foundry 는 안 됨** |
| 조직 | HIPAA 구성이나 zero data retention 을 쓰면 안 됨 |
| 모델 | **Opus 5.5, Sonnet 5.5, Haiku 5.5, Fable 5.2** |
| 같이 못 켜는 것 | `apiKeyHelper`, agent teams(`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`), **OpenTelemetry export**, `DISABLE_TELEMETRY`, 툴 호출을 검사하는 mod, `--restricted`, `--disallowedTools` 나 `deny` 룰의 `REPL` |
| Windows | Git for Windows 필요 |

세 줄이 특히 걸림. 첫째, **VS Code 패널과 데스크톱 앱이 빠져 있음.** Xcode 로 작업하면서 터미널에서 `claude` 를 쓰는 사람에게는 문제가 없지만, 데스크톱 앱 Code 탭으로 옮겨 간 사람은 지금 쓸 수 없음. 둘째, **OpenTelemetry export 와 상호 배타임.** Claude Code 쪽 계측을 켜 둔 상태로는 시작되지 않으므로 둘 중 하나를 골라야 함. 셋째, **툴 호출을 검사하는 mod 와 같이 못 씀.** [Claude Code Mods](./2026-10-04-claude-code-mods.md) 로 승인 경로를 직접 짜 둔 경우가 정확히 그 경우임.

되돌릴 수 없는 쪽도 하나 있음. **Code Mode 세션은 Code Mode 를 켠 터미널에서만 재개됨.** 끈 상태로 `--continue`·`--resume` 하면 거부됨(반대로 끄고 시작한 세션은 변수를 켜 둬도 끈 상태로 재개됨). 데스크톱 앱으로 옮기거나 백그라운드 세션으로 만드는 것, `/fork` 로 백그라운드 사본을 만드는 것도 안 됨. 실험 기능을 켠 세션이 특정 실행 경로에 묶이는 셈이라, 길게 가져갈 작업에는 지금 붙일 자리가 아님.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Code Mods](./2026-10-04-claude-code-mods.md) | 둘 다 하네스 안에서 JavaScript 를 돌리지만 방향이 반대임. mod 는 **사람이 쓴 코드가 툴 호출을 검사**하고, Code Mode 는 **모델이 쓴 코드가 툴 호출을 함.** 그래서 같이 못 켬 |
| [2.1.309 의 권한 우회 수정 묶음](../security/2026-10-22-claude-code-permission-bypass-fixes.md) | 정적 검사가 실행 시점의 실제 모양에 지는 패턴. 분류기가 실행 시점 계산값을 못 보는 것이 같은 축의 새 사례 |
| [작업 디렉터리 밖 쓰기 차단](../security/2026-10-30-working-directory-write-block.md) | 문자열 검사에서 경로 울타리로 옮겨 가는 흐름. 검사 단위가 코드로 커지면 울타리 쪽 통제의 값어치가 더 커짐 |
| [사람이 승인에서 위협 3건 중 1건을 놓침](../security/2026-08-05-agent-approval-miss-rates.md) | 사람이 보는 화면에는 코드가 아니라 툴 호출만 뜨고 코드는 `Ctrl+O` 뒤에 접혀 있음. 승인 UI 가 보여주는 것과 실제 실행되는 것의 거리가 이 측정의 전제임 |
| [Claude Code effort 해석 순서](../practices/2026-10-27-claude-code-effort-resolution.md) | 같은 종류의 설정 해석 문제. Code Mode 는 "관리 설정에 `1` 이 있으면 셸의 `1` 이 무시된다" 는 역전까지 있어 더 꼬임 |

## 짚어야 할 것

- **토큰·비용 절감 수치가 문서에 없음.** 문서가 적는 것은 "Claude 가 읽어야 할 부분만 반환한다" 는 서술과 쓸 자리 세 가지뿐이고, 호출당 토큰이나 지연이 얼마나 줄어든다는 숫자는 한 줄도 없음. 절감 폭은 **확인할 수 없었음.** 다른 데서 본 배수를 이 기능에 붙여 인용하면 안 됨
- **`REPL` 툴이 [Tools reference](https://code.claude.com/docs/en/tools-reference) 에 없음**(2026-11-14 확인). 코드를 실행하는 툴 이름은 `PreToolUse` 훅 matcher 설명과 트러블슈팅 표에서만 나옴. 그래서 훅에서 `tool_input.code` 말고 무엇을 받는지, 룰 문법에서 `REPL` 에 인자를 붙일 수 있는지는 문서로 확인되지 않음
- **JavaScript 를 돌리는 런타임과 그 격리 수준이 문서에 없음.** 예시에서 `$` 템플릿 리터럴과 `.text()`·`.nothrow()`, `edit()` 이 보이지만 전체 API 목록이 없고, 코드 자체가 샌드박스 안에서 도는지(툴 호출이 아니라 `fetch` 나 파일 접근을 직접 하는 코드를 쓸 수 있는지)도 적혀 있지 않음. **확인할 수 없었음.** 네트워크·파일 접근이 툴 호출을 거치지 않고 가능하면 위 권한 표가 통째로 달라지므로 이것이 제일 먼저 확인해야 할 항목임
- **설정 위치 규칙이 두 문서에서 다르게 적혀 있음.** Code Mode 페이지는 "셸이나 사용자 설정" 이라고만 하고 프로젝트 설정이 무시된다는 것만 덧붙임. `settings-reference` 는 `--settings` 파일을 포함하고 관리 설정의 `1` 이 셸의 `1` 을 무시한다는 역전까지 적음. 후자가 더 구체적이라 위 표는 후자를 따랐음
- 실험 기능이고 **어느 릴리스에서든 바뀌거나 사라질 수 있다고 문서가 직접 적음.** `/feedback` 로 의견을 받는 단계임. 기능 자체를 전제로 워크플로를 짜는 단계가 아님
- `it isn't open to your account yet` 이라는 거부 이유가 있음. 버전이 맞아도 계정별로 열리지 않을 수 있고, 문서의 처방이 "다른 날 다시 시도" 임. 즉 **버전만으로 가용 여부가 결정되지 않음**

## 유효기간

**2026-11-14 확인 기준이고 기능은 2026-11-12 v2.1.323 에서 시작, 최신 릴리스는 v2.1.325(2026-11-13)임.** 실험 단계라 위 표들을 고치려 들지 말고 [Code Mode 문서](https://code.claude.com/docs/en/code-mode)를 다시 읽는 편이 나음. 다시 볼 때 확인할 것 넷. **실험 딱지가 떨어졌거나 반대로 기능이 사라졌는지**, **`REPL` 이 Tools reference 에 올라와 코드 런타임과 노출 API 가 문서화됐는지**(위 "짚어야 할 것" 의 둘째·셋째가 여기서 풀림), **배제 목록에서 VS Code 패널·데스크톱 앱·OpenTelemetry·mod 가 빠졌는지**, 그리고 **auto 모드 분류기가 실행 시점 값을 보게 됐는지.** 마지막 것이 바뀌면 권한 절이 통째로 낡음. 버전별 변경은 `gh api repos/anthropics/claude-code/releases` 로 받아 `Code Mode`·`REPL` 줄만 훑어도 대부분 잡힘.
