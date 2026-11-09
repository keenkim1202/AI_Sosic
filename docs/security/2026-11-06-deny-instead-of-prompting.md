---
title: 프롬프트 대신 거부, auto 모드의 분류기 폴백이 사라진 자리
source: https://github.com/anthropics/claude-code/releases/tag/v2.1.319
author: Anthropic
published: 2026-11-06
collected: 2026-11-09
tags: [agent-security, claude-code, permissions, guardrail, human-in-the-loop, gateway, ci]
---

출처: [Claude Code v2.1.319 릴리스 노트](https://github.com/anthropics/claude-code/releases/tag/v2.1.319) (2026-11-06) · [v2.1.318](https://github.com/anthropics/claude-code/releases/tag/v2.1.318) (2026-11-04) · [Permission modes](https://code.claude.com/docs/en/permission-modes) · [All settings](https://code.claude.com/docs/en/settings-reference) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [Environment variables](https://code.claude.com/docs/en/env-vars) · [Non-interactive mode](https://code.claude.com/docs/en/headless) (문서 다섯 건은 **2026-11-09 확인**. 릴리스 날짜는 `gh api` 의 `published_at`(UTC) 기준임)

## 요약

**2026-11-06 v2.1.319 에 권한 판정의 기본값을 거부 쪽으로 미는 변경이 둘 들어왔음.** 하나는 설정이고 하나는 auto 모드의 동작임.

첫째, **`permissions.denyInsteadOfPrompting`.** 노트 문언은 "a tool call that would wait on a permission prompt is denied and the turn continues, in terminal sessions, subagents and teammates" 임. 즉 **프롬프트에서 기다리던 호출이 거부되고 턴은 계속됨.** 비대화형 실행에는 `--permission-prompts none` 이 **2026-09-02 v2.1.259** 부터 있었는데, 같은 모양이 **사람이 앉아 있는 터미널 세션과 서브에이전트·팀메이트**로 내려온 것이 이번임.

둘째, **auto 모드에서 내장 분류기로 떨어지는 폴백이 일부 세션에서 없어졌음.** 대상은 **텔레메트리나 피처 플래그를 끈 세션**이고 LLM 게이트웨이 뒤도 포함임. 서버가 검사할 수 없는 호출은 이제 **묻거나 거부**되고, 내장 분류기로 내려가지 않음. 되살리려면 `CLAUDE_CODE_AUTO_MODE_SERVER=0` 임. Bedrock·Vertex·Foundry 는 그대로임.

덤으로 셋. **`claude -p` 가 안전 검사가 없어 행동이 거부된 경우에 0 이 아니라 1 로 끝나게 됐고**, **Bash `deny`·`ask` 룰이 따옴표로 적힌 명령을 놓치던 것이 고쳐졌으며**, [2026-10-30 문서](./2026-10-30-working-directory-write-block.md)가 "문서가 없다"로 남겨둔 `permissions.blockWritesOutsideWorkingDirectories` 가 **설정 레퍼런스에 올라왔음.** 그 문서의 미확인 넷이 여기서 풀림.

## 들어온 것

v2.1.319 노트에서 권한에 걸리는 줄만 뽑은 것임. 괄호 안은 노트의 분류임.

| 항목 | 내용 |
|---|---|
| `permissions.denyInsteadOfPrompting` (Added) | 프롬프트에서 기다릴 호출을 거부하고 턴을 계속함. 터미널 세션·서브에이전트·팀메이트 |
| auto 모드 폴백 제거 (Changed) | 텔레메트리·피처 플래그를 끈 세션에서 서버가 못 보는 호출에 **내장 분류기 폴백 없음.** `CLAUDE_CODE_AUTO_MODE_SERVER=0` 으로 되살림 |
| 게이트웨이 뒤 서버 검사 (Changed) | `CLAUDE_CODE_AUTO_MODE_SERVER=1` 이 `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` 가 있어도 서버 검사를 켬. 관리 설정이 후자를 걸면 `1` 도 같이 걸어야 함 |
| `CLAUDE_CODE_DISABLE_FEATURE_FLAGS` (Added) | 텔레메트리를 끄지 않고 피처 플래그 조회만 멈춤. 모든 플래그가 내장 기본값이 됨 |
| `claude -p` 종료 코드 (Fixed) | 안전 검사가 없어 행동이 거부됐는데 0 으로 끝나던 것. text·JSON 출력이 **1** 로 끝남 |
| Bash 룰과 따옴표 (Fixed) | `deny`·`ask` 룰이 따옴표로 적힌 단어를 담은 명령을 놓치던 경우 |
| `/status` 표시 (Fixed) | 서버 검사가 세션에 안 닿는데 `Auto mode server: Enabled` 로 보이던 것. 예로 결과를 떨어뜨리는 LLM 게이트웨이 |
| 큰 룰셋의 거부 이유 (Fixed) | 권한 룰이나 `autoMode` 설정이 아주 클 때 이유 없이 거부되던 것. 이제 설명하고, `autoMode` deny 룰이 없으면 터미널에서 물음 |

## 프롬프트가 기다리던 자리가 거부가 됨

이 설정의 값어치는 **이미 있던 것과 무엇이 다른가**에 있음. 프롬프트를 없애는 경로가 전부터 둘 있었음.

| 경로 | 버전 | 적용 자리 |
|---|---|---|
| `--permission-prompts none` | 2.1.259 (2026-09-02) | `-p` 실행과 Agent SDK. 권한 호스트(`canUseTool`, `--permission-prompt-tool`)를 아예 안 부름 |
| `dontAsk` 모드 | 모드 | 프롬프트가 뜰 모든 호출을 거부. **모드 자체를 쓰는 것**이라 다른 모드와 같이 못 씀 |
| `permissions.denyInsteadOfPrompting` | 2.1.319 (2026-11-06) | 터미널 세션·서브에이전트·팀메이트 |

차이가 둘임. 첫째, **자리가 터미널까지 내려왔음.** 지금까지 "사람이 못 답하는 자리"를 전제로 설계된 플래그였던 것이, 사람이 앉아 있는 세션에도 걸리게 됨. 둘째, **모드가 아니라 설정임.** `dontAsk` 는 모드를 바꿔야 하므로 auto 모드와 같이 쓸 수 없는데, 설정이면 auto 모드 위에 얹히는 것으로 읽힘. auto 모드에서 분류기가 판단을 못 해 프롬프트로 떨어지는 자리가 정확히 이 설정의 대상 후보임.

**읽히는 용도는 백그라운드 세션과 서브에이전트임.** 서브에이전트와 팀메이트가 명시적으로 적혀 있다는 점이 그 신호임. 서브에이전트가 프롬프트에서 멈춰 있으면 부모 쪽에서는 그냥 느린 것처럼 보이고, 터미널에 떠 있는 프롬프트가 어느 에이전트의 것인지도 세션이 여럿이면 금방 헷갈림. 거부하고 턴을 계속하면 적어도 **멈춤이 결과로 드러남.**

⚠️ 다만 이 설정은 **릴리스 노트 한 줄이 유일한 출처임.** 2026-11-09 확인 기준으로 설정 레퍼런스의 설정 색인(모든 키를 나열하는 표), Configure permissions, Permission modes 어디에도 이 키가 없음. 그래서 아래가 전부 미확인임.

- **scope.** 저장소 설정으로도 켜지는지, 관리 설정이 끌 수 있는지 모름. 쓰기 블록 쪽 전례(`Any file` 이고 어느 파일에서 온 `true` 든 이김)가 있지만 전례는 근거가 아님
- **어느 모드에서 사는지.** auto 모드 위에 얹히는지, Manual 모드의 모든 프롬프트까지 덮는지 적혀 있지 않음
- **거부를 모델에게 어떻게 알리는지.** `--permission-prompts none` 은 "nobody can approve the request and not to retry it" 를 모델에게 알린다고 문서가 적는데, 이 설정의 서술에는 그 부분이 없음
- **`PermissionRequest` 훅과의 순서.** `--permission-prompts none` 에서는 훅이 허용하면 통과함. 이 설정도 같은지 모름

## auto 모드: 서버가 못 보는 호출의 착지점이 바뀜

auto 모드의 검사는 두 경로임. Claude Code 가 **분류기 요청을 따로 보내는** 경로와, **세션의 모델 요청에 얹어 서버가 보는** 경로. 후자가 server-side classifier review 이고 **2.1.271 부터** 세션 종류별로 순차 적용돼 왔음. 지금 문서가 적는 실패 경로가 둘임.

> "**The server doesn't review the session**: a response completes with no review results, or the server answers that it doesn't review this session. The most common causes are an LLM gateway or proxy that drops the request for review or the results, and a platform, region, or credential that doesn't have server-side checks yet. Claude Code falls back to its own classifier requests."

> "**The server gives no verdict for an action**: Claude Code denies the action rather than run it unreviewed."

v2.1.319 가 바꾼 것은 **앞쪽**임. 노트 문언이 "calls the server can't check are asked about or denied, with no fallback to the built-in classifier" 이고, 대상이 **텔레메트리나 피처 플래그를 끈 세션**임. 그러니까 이제 그 세션에서는 검사 없는 호출이 분류기로 내려가지 않고 **사람에게 묻거나(터미널) 거부**됨.

**여기가 위 설정과 맞물리는 자리임.** auto 모드를 켜 둔 이유가 프롬프트를 없애는 것이었는데, 서버 검사가 안 닿는 세션에서는 그 프롬프트가 돌아옴. 그리고 `permissions.denyInsteadOfPrompting` 을 켜면 돌아온 프롬프트가 다시 거부로 바뀜. 둘을 같이 쓰면 **서버가 못 보는 호출은 조용히 안 돌음.** 조합이 명시된 것은 아니지만 방향은 하나임.

⚠️ 문서와 노트가 **2026-11-09 기준으로 어긋나 있음.** 위에 인용한 "falls back to its own classifier requests" 에는 텔레메트리·피처 플래그 조건이 붙어 있지 않음. 노트가 바꾼 범위가 문서에 아직 안 내려온 것으로 보이지만, **문서가 늦은 것인지 노트가 더 좁은 경우를 말한 것인지는 갈라낼 근거가 없음.** 그래서 이 절은 두 출처를 나란히 적어 두는 데까지임.

### 끈 적이 없다고 생각하는 쪽이 대상일 수 있음

"텔레메트리나 피처 플래그를 끈 세션"이 생각보다 넓음. 문서가 적어 둔 것이 이럼.

- **2.1.282 부터** 피처 플래그를 조회하지 않는 세션은 세션 종류와 무관하게 기본으로 서버에 물음
- 조회를 멈추는 경로에 **`DISABLE_TELEMETRY`** 가 들어감. 즉 텔레메트리를 끈 것이 권한 판정 경로를 바꿈
- v2.1.319 가 **`CLAUDE_CODE_DISABLE_FEATURE_FLAGS`** 를 새로 더함. 텔레메트리는 켜 두고 플래그 조회만 멈추는 변수이고, 모든 플래그가 내장 기본값이 됨

**프라이버시나 사내 정책 때문에 텔레메트리를 끈 설정이 권한 폴백을 같이 끄게 됨.** 이 저장소가 [에이전트 웹 egress 통제 3층](./2026-10-07-managed-agents-web-egress.md)에서 본 것과 같은 모양임. 안 적은 값이 기본값으로 떨어지고, 그 기본값이 사람이 생각하던 것과 다름. 다만 방향은 반대임. 그쪽은 안 적으면 열리는 쪽이었고, 이쪽은 끄면 닫히는 쪽임.

`CLAUDE_CODE_DISABLE_FEATURE_FLAGS` 도 **2026-11-09 확인 기준으로 환경 변수 문서에 없음.** 같은 페이지에 `CLAUDE_CODE_AUTO_MODE_SERVER` 와 `CLAUDE_CODE_DISABLE_MCP_TASK_BACKGROUND` 는 있음. 즉 같은 릴리스에서 온 변수 중 일부만 문서에 반영된 상태임.

### 비용 쪽도 같이 움직임

분류기 요청은 공짜가 아님. 문서 문언이 이럼.

> "On Enterprise plans and on accounts that use the Claude API, Claude Platform on AWS, Amazon Bedrock, Google Cloud's Agent Platform, or Microsoft Foundry, classifier calls count toward your token usage."

그리고 서버가 보는 경로에서는 **따로 셀 분류기 호출이 없음.** 기존 폴백은 세션 내내 유지되면 과금 안내를 띄우는 종류였는데, 그 경로가 없어졌으니 **텔레메트리를 끈 세션에서 조용히 쌓이던 분류기 토큰도 같이 없어짐.** 안전 쪽 변경으로 적힌 줄이지만 청구서 쪽에도 걸림. 토큰을 어디서 쓰는지 세는 쪽은 [사용량·비용 분석 API](../practices/2026-10-29-usage-cost-analytics-api.md)임.

## `-p` 가 1 로 끝나게 된 것

작아 보이지만 CI 에 직접 걸리는 줄임. 노트 문언이 "Fixed `claude -p` exiting 0 in auto mode when its actions were refused because no safety check was available: text and JSON output now exit 1" 임.

고쳐지기 전의 모양을 그려 보면 이럼. CI 잡이 `claude -p --permission-mode auto` 로 돌고, 게이트웨이가 리뷰 결과를 떨어뜨려서 행동이 전부 거부되고, **아무것도 안 했는데 종료 코드 0 으로 성공 보고가 나감.** 종료 코드로 분기하는 파이프라인은 이걸 초록으로 읽음. [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md)의 "초록은 증거지 증명이 아님"이 가장 싼 형태로 실현된 자리임.

그러니 **이 버전 이전의 auto 모드 CI 잡은 종료 코드를 믿을 수 없었음.** 거부를 세고 싶으면 `--output-format stream-json` 의 `permission_denials` 를 보는 편이 안전함. 결과 메시지가 거부를 목록으로 들고 있고, 이건 종료 코드 수정과 무관하게 전부터 있던 경로임.

## 셸 파서 계열이 또 한 건

"Fixed a case where Bash deny and ask rules missed commands written with quoted words" 한 줄임. 세부는 노트에 없음.

이 저장소가 [2026-10-22 릴리스의 권한 우회 수정 13건](./2026-10-22-claude-code-permission-bypass-fixes.md)에서 센 계열이 넷이었고 그중 **셸 문법 8건**이 전부 같은 모양이었음. 명령 문자열을 파싱해서 룰과 맞춰 보는 층이, 셸이 실제로 읽는 방식과 갈리는 자리. 감싼 `cd`, 서브셸 리다이렉트, 열리지 않을 수 있는 리다이렉트가 설정한 변수. **따옴표가 거기 하나 더 붙음.**

[2026-10-30 문서](./2026-10-30-working-directory-write-block.md)가 내린 결론이 여기서 한 번 더 확인됨. 경로 울타리처럼 **문자열을 해석하지 않는 통제**와, `deny` 룰처럼 **문자열을 해석해야 하는 통제**는 신뢰 등급이 다름. 후자는 계속 수정이 쌓이고 있고, 쌓이는 속도가 느려지는 징후는 아직 없음.

## 쓰기 블록 문서가 내려왔음

[2026-10-30 문서](./2026-10-30-working-directory-write-block.md)는 `permissions.blockWritesOutsideWorkingDirectories` 를 **릴리스 노트 한 줄만 보고** 쓴 것이고, 유효기간에 "설정 레퍼런스에 항목이 생기면 미확인 넷이 한 번에 풀린다"고 적어 뒀음. 2026-11-09 확인 기준으로 **생겼음.**

| 그 문서가 미확인으로 둔 것 | 설정 레퍼런스 항목 (2026-11-09 확인) |
|---|---|
| scope | **`Any file`**. "a repository can turn the block on for itself but can't lift yours" |
| 기본값 | **unset.** unset 이면 바깥 쓰기가 권한 모드를 따름 |
| 관리 설정이 `false` 로 덮을 수 있는지 | **못 덮음.** "Exceptions to managed settings precedence" 표에 올라와 있고 `true` from any scope 임. v2.1.314 이상 필요 |
| Bash 가 걸리는지 | **안 걸림.** "The key doesn't limit shell commands, MCP tools, or the programs hooks run. To limit where shell commands write, turn on sandboxing" |

**그 문서의 `xcodebuild` 추론이 맞았음.** 울타리는 파일 툴 셋에만 서고, 빌드 도구가 DerivedData 에 쓰는 것은 이 설정으로 안 막힘. 막으려면 샌드박스 쪽 일이라는 것까지 문서가 명시함.

항목에 **그 문서에 없던 것이 둘** 더 있음.

- **worktree 툴이 사라짐.** 블록을 켜면 Claude Code 가 `EnterWorktree`·`ExitWorktree` 툴을 제거하거나 호출을 거부함. 워크트리를 띄워 에이전트를 병렬로 돌리는 구성이면 이게 먼저 걸림
- **관리 설정을 못 읽으면 블록이 켜짐.** "The block is also on while Claude Code can't read the `permissions` object in your managed settings." 즉 이 키 자체가 fail closed 쪽 동작을 가짐

세 번째 재확인 항목인 **auto 모드의 첫 바깥 쓰기 1회 프롬프트는 아직 없음.** 항목에 그런 서술이 없고, 읽기 쪽에만 있는 구조 그대로임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [작업 디렉터리 밖 쓰기 차단](./2026-10-30-working-directory-write-block.md) | 그 문서가 미확인으로 남긴 넷이 설정 레퍼런스 항목으로 풀림. 같은 fail closed 방향의 다음 한 걸음 |
| [Claude Code 2.1.309의 권한 우회 수정 묶음](./2026-10-22-claude-code-permission-bypass-fixes.md) | 셸 문법 계열에 따옴표 한 건이 더 붙음. 문자열을 해석하는 층의 신뢰 등급 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 착지점이 프롬프트일 때의 실측 미탐율. `denyInsteadOfPrompting` 은 그 착지점을 아예 없애는 쪽 |
| [Claude를 제품마다 어떻게 격리하는가](./2026-05-25-containing-claude-across-products.md) | 그 문서가 재확인 과제로 적은 "auto mode 분류기 수치가 다시 공개됐는지". 이번 확인 범위(Permission modes 문서와 v2.1.319 노트)에는 수치가 없었고, 대신 **분류기가 서버로 옮겨간 경로**가 드러났음 |
| [Managed Agents 의 `auto` 권한 정책](./2026-09-10-managed-agents-auto-permission.md) | 서버가 판정을 들고 `evaluation` 객체로 돌려주는 구조. Claude Code 쪽 server-side review 와 같은 계열로 읽힘 |
| [에이전트 웹 egress 통제 3층](./2026-10-07-managed-agents-web-egress.md) | 안 적은 값이 기본값으로 떨어지는 모양. 방향은 반대임 |
| [Claude Code 의 effort 해석 순서](../practices/2026-10-27-claude-code-effort-resolution.md) | 같은 릴리스 계열에서 문서와 노트가 갈리는 자리를 다루는 방법 |

## 짚어야 할 것

- **`permissions.denyInsteadOfPrompting` 의 근거는 릴리스 노트 한 줄뿐임.** 2026-11-09 확인 기준으로 설정 색인·Configure permissions·Permission modes 에 없음. scope, 적용 모드, 모델에게 알리는 문언, `PermissionRequest` 훅과의 순서는 **확인할 수 없었음**
- **`CLAUDE_CODE_DISABLE_FEATURE_FLAGS` 도 환경 변수 문서에 없음.** 같은 릴리스의 `CLAUDE_CODE_DISABLE_MCP_TASK_BACKGROUND` 는 올라와 있어서, 문서 반영이 변수별로 갈린 상태임
- **auto 모드 폴백 제거의 범위를 좁히지 못했음.** 노트는 "텔레메트리나 피처 플래그를 끈 세션"이라고 적고 문서의 폴백 서술에는 그 조건이 없음. 문서가 늦은 것인지 노트가 더 좁은 경우를 말한 것인지 **갈라낼 근거가 없음**
- **재현하지 않았음.** 설정을 켜거나 게이트웨이 뒤에서 돌려 본 것이 아니라 릴리스 노트와 문서 문언을 읽은 것임. 종료 코드 1 도 확인하지 않았음
- **따옴표 룰 우회의 세부가 없음.** 어떤 형태의 따옴표가 룰을 비켜 갔는지, 노출 기간이 얼마인지 노트에 없고 보안 권고로도 아직 안 나왔음. [2026-10-22 문서](./2026-10-22-claude-code-permission-bypass-fixes.md)가 센 대로 권고는 수정보다 몇 달 늦게 붙음
- **릴리스 노트에 날짜가 없음.** 저장소 `CHANGELOG.md` 도 같아서, 이 문서의 날짜는 `gh api` 로 받은 `published_at`(UTC) 기준임
- 이 문서가 "착지점"이라고 쓴 말은 정리자의 표현임. Anthropic 문서는 denied, refused, asked about 를 섞어 씀

## 유효기간

**2026-11-09 확인 기준이고 최신 릴리스는 2.1.319** 임. 이 릴리스는 하루에 한두 개씩 나오므로 버전 번호는 읽는 시점에 이미 낡았다고 보는 게 맞음. 다시 볼 때 확인할 것 셋. **설정 레퍼런스의 설정 색인에 `permissions.denyInsteadOfPrompting` 이 생겼는지**(생기면 위 미확인 넷이 한 번에 풀림), **Permission modes 의 "The server doesn't review the session" 항목에 텔레메트리·피처 플래그 조건이 붙었는지**(붙으면 문서와 노트의 어긋남이 해소됨), 그리고 **`CLAUDE_CODE_DISABLE_FEATURE_FLAGS` 가 환경 변수 표에 올라왔는지.** 셋 다 문서 쪽 반영 여부의 문제이고, 동작 쪽에서 가장 큰 변경이 될 자리는 **auto 모드의 서버 검사가 폴백 없이 전체 세션으로 확대되는지**임. 그렇게 되면 내장 분류기 요청이라는 경로 자체가 없어지고, 분류기 토큰 과금 서술도 같이 낡음.
