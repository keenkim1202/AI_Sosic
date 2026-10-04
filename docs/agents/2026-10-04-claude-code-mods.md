---
title: Claude Code Mods, 플러그인이 하네스 내부를 바꾸게 됨
source: https://code.claude.com/docs/en/plugins/mods
author: Anthropic
collected: 2026-10-04
tags: [claude-code, plugins, harness, permissions, agent-security, prompt-caching, tooling]
---

출처: [Mods overview](https://code.claude.com/docs/en/plugins/mods) · [Mods reference](https://code.claude.com/docs/en/plugins/mods/reference) · [React to events with a mod](https://code.claude.com/docs/en/plugins/mods/events) · [Configure permissions](https://code.claude.com/docs/en/permissions) · [CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) (2026-10-04 확인)

## 요약

**mod** 는 Claude Code 안에서 도는 JavaScript·TypeScript 이벤트 핸들러를 담은 플러그인임. 설정 파일에 셸 명령을 등록하는 기존 훅(공식 문서가 이제 **settings hook** 으로 갈라 부름)과 달리 **Claude Code 프로세스 안에서 함수로 실행**되고, 그래서 툴 호출을 보류·거부·대체하고, 시스템 프롬프트의 절을 다시 쓰고, 요청 하나를 다른 모델로 보내고, 화면을 직접 그림. **v2.1.287 이상에서 기본으로 켜져 있음.** 값어치는 분명한데 대가가 하나 있고 그게 이 문서에서 제일 중요한 줄임. **관리 설정이 없는 개인 머신에서는 내가 설치한 mod 가 내 `deny` 룰이 거부한 툴 호출을 승인할 수 있음.** 즉 권한 룰이 더 이상 최종 결정이 아니게 됨.

## 기본 정보

| | |
|---|---|
| 최소 버전 | **v2.1.287** 이상. 레퍼런스 문서는 **v2.1.289 기준**으로 쓰여 있음 |
| 기본 상태 | **켜짐.** 초기 접근 때 쓰던 `CLAUDE_CODE_ENABLE_FUNCTION_HOOKS` 는 이제 무시되므로 `0` 으로 꺼둔 설정은 효력이 없음 |
| 최소 파일 | 셋. `.claude-plugin/plugin.json`, `hooks/hooks.json`, 훅 모듈(`hooks/register.js` 등) |
| 훅 모듈 | ES 모듈이고 `register(on, options)` 를 export 함. `.js`·`.mjs`·`.cjs`·`.jsx`·`.ts`·`.mts`·`.cts`·`.tsx` |
| 설치 | 플러그인과 같음. `/plugin install <이름>@<마켓플레이스>`. 셸에서 설치했으면 세션에서 `/reload-plugins` |
| 개발 루프 | `claude --plugin-dir <경로>` 로 한 세션만 로드하고 저장할 때마다 훅 모듈이 리로드됨 |
| 전부 끄기 | 세션 하나는 `--safe-mode`, 영구적으로는 `~/.claude/settings.json` 의 `disableAllHooks` |

## 무엇과 다른가

공식 문서가 직접 비교표를 두고 있고, 갈림길은 **"프로세스 안인가 밖인가"** 하나임.

| | mod | settings hook | 스킬 | MCP 서버 |
|---|---|---|---|---|
| 실행 위치 | Claude Code 프로세스 안의 함수 | 밖에서 도는 셸 명령·HTTP 요청·프롬프트 | `SKILL.md` 지시문 | 외부 프로세스·서비스 |
| 바꿀 수 있는 것 | 툴 호출, 프롬프트, 커맨드, 턴, **화면에 그려지는 것** | 툴 호출·프롬프트의 통과 여부, 인자와 결과, 추가 컨텍스트 | Claude 가 아는 것 | Claude 가 가진 툴 |
| 화면을 그릴 수 있나 | 됨 | 안 됨 | 안 됨 | 안 됨 |
| 쓰는 언어 | JavaScript·TypeScript | 스크립트와 `settings.json` 항목 | 마크다운 | 아무 언어 |

플러그인 하나가 넷을 다 담을 수 있으므로 **택일이 아님.** 이미 있는 셸 스크립트로 차단·기록만 하면 settings hook 이 맞고, 창을 그리거나 이벤트를 다시 쓰려면 mod 임.

## 가로챌 수 있는 것

이벤트가 많은데(세션, 서브에이전트, 커맨드, `/config` 행, 텔레메트리까지) 이 저장소 맥락에서 값어치 있는 것만 고르면 이렇게 됨.

| 이벤트 | 언제 | 훅이 할 수 있는 것 |
|---|---|---|
| `tool.call` | 툴이 돌기 직전. **서브에이전트 호출과 MCP 툴 호출도 포함** | 통과, 인자 교체, `{ deny: 이유 }`, 또는 툴을 안 돌리고 `{ result }` 로 직접 답 |
| `tool.check` | 권한 룰과 `PreToolUse` 훅이 결정을 낸 **뒤** | `allow`·`ask`·`deny` 중 하나로 그 결정을 덮어씀 |
| `tool.describe` | 툴 설명이 Claude 에게 처음 보내질 때 | 설명 교체, `isDeferred` 로 툴 서치 뒤로 밀거나 앞으로 당김 |
| `prompt.submit` | 프롬프트가 제출될 때 | 텍스트 재작성, Claude 만 읽는 `context` 추가, `{ drop: 이유 }` |
| `prompt.section` | 시스템 프롬프트의 **각 절마다** | 절 텍스트 교체, `{ text: null }` 로 절 자체를 빼기 |
| `prompt.attachment` | Claude Code 가 리마인더 같은 자기 메시지를 끼울 때 | 문구 교체 또는 제거 |
| `turn.step` | 모델에 요청 하나가 나가기 직전 | `next({ ...e, model })` 로 모델 교체, `effort` 교체, 토큰 사용량 읽기 |
| `session.compact` | 대화가 compaction 되기 직전 | `{ skip: 이유 }` |
| `plugin.register` | **다른 mod 의 훅 모듈이 로드되기 직전** | `{ refuse: 이유 }` 로 거절 |

여기에 더해 **모든 mods API 메서드가 그 자체로 이벤트**임. `fs.read`, `model.complete`, `ui.open` 같은 이름으로 훅을 걸면 **뒤에 도는 mod 들의 호출을 가로채서** `{ deny }` 하거나 값을 바꿔 돌려줄 수 있음. mod 가 mod 를 감시하는 구조가 설계에 들어가 있음.

가드 하나의 모양은 이렇게 짧음. 원문 예제임.

```javascript
// The matcher limits the hook to Bash calls, so e.command is the shell command
on('tool.call', { tool: 'Bash' }, async ($, e, next) => {
  if (/git push .*--force/.test(e.command)) {
    // Returning without calling next answers the event, so the command never runs
    return { deny: 'Force pushes are not allowed in this repository. Push to a new branch instead.' }
  }
  // Every other command goes on to the permission check and then to Bash
  return next(e)
})
```

`next` 를 안 부르면 **권한 프롬프트조차 뜨지 않고** Claude 는 `deny` 문구를 툴 결과로 읽음. 반대로 보류는 `$.ui.ask` 로 함. 사람이 답할 때까지 툴 호출이 pending 으로 멈추고, **그 대기 시간은 훅의 시간 제한에 안 들어감.** 자기가 만든 프라미스를 `await` 하는 시간은 들어가므로, 거기서 타임아웃이 나면 **훅이 건너뛰어지고 보류하려던 명령이 그냥 실행됨.** 차단용 훅이라면 `.catch` 핸들러로 fail closed 를 직접 만들어야 함.

## 비용과 캐시에 닿는 지점

이 저장소가 여러 문서에서 쫓아온 주제가 여기서 하나의 API 로 노출됨.

- **`turn.step` 이 요청 단위로 모델과 effort 를 바꿈.** [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md)의 스마트 라우팅을 하네스 설정이 아니라 내 코드로 하는 자리임. [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)이 적어둔 "대화 중간 effort 변경"이 API 쪽 베타 기능이고, 이쪽은 그걸 세션에서 걸 지점임
- **같은 훅에서 `result.usage` 를 읽을 수 있음.** `input_tokens`, `output_tokens`, `cache_read_input_tokens`, `cache_creation_input_tokens`. [Prompt Caching In Agents](./2026-07-22-prompt-caching-in-agents.md)의 적중률 문제를 세션 안에서 바로 계측하게 됨. 문서 예제가 그 로그를 찍는 것임
- **반대 방향의 함정도 문서가 직접 적어 둠.** `prompt.section`·`prompt.context`·`skill.prompt` 에서 **요청마다 달라지는 텍스트를 넣으면 프롬프트 캐시가 무효화됨.** 시스템 프롬프트에 시계나 토큰 잔량 같은 걸 끼우는 mod 가 가장 쉽게 밟는 지뢰고, 효과는 [Prompt Caching In Agents](./2026-07-22-prompt-caching-in-agents.md)에 정리된 것과 같음
- **`session.compact` 를 `{ skip }` 으로 막을 수 있음.** [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md)이 비용 계측을 조용히 비틀어 놓는다고 적어둔 그 동작을, 끌 수 있다는 뜻이기도 하고 **실수로 끄면 컨텍스트가 터진다**는 뜻이기도 함

## 권한 경계, 가장 조심해서 볼 부분

mod 는 샌드박스 안에 없음. 문서가 세는 것만 옮기면, 로드된 mod 는 내 계정 권한으로 **파일을 읽고 쓰고, 프로그램을 띄우고, 네트워크를 쓰고**, 환경변수와 설정 파일의 **API 키를 읽고**, 모든 프롬프트와 툴 호출을 보고, 프롬프트를 다시 쓰거나 **내가 입력한 것처럼 제출하고**(`$.prompt.submit({ text, asUser: true })` 가 발신자 표기 없이 보냄), 내 플랜이나 키로 모델을 호출함. [sandboxing](https://code.claude.com/docs/en/sandboxing)을 켜도 격리되는 건 Claude 가 돌리는 Bash 명령이고 **mod 가 띄운 프로세스는 그 밖에서 돎.**

그중 권한 쪽 결론만 원문 그대로 둠.

> "**Deny rules**: on a machine with managed settings, or when you're signed in with a Team or Enterprise plan, deny rules hold over the mod by default, and your organization can change that. Anywhere else, the mod can approve a call that a deny rule refuses."

개인 플랜으로 혼자 쓰는 머신이 바로 그 "anywhere else" 임. 정리하면 `tool.check` 를 다루는 mod 는 이렇게 됨.

| 무엇을 덮을 수 있나 | 결과 |
|---|---|
| `ask` 룰 | 물어볼 호출을 **승인해 버릴 수 있음** |
| 관리 설정 밖의 `PreToolUse` 훅이 낸 차단 | 뒤집을 수 있음 |
| auto 모드의 분류기 | mod 가 승인한 호출은 **분류기 검사 없이** 실행됨 |
| `deny` 룰 | 관리 설정이 있거나 Team·Enterprise 로그인이면 룰이 이김. **그 밖에서는 mod 가 이김** |
| 관리 설정의 `PreToolUse` 차단 | 못 뒤집음. 그 차단이 최종이고 **mod 는 호출 자체를 못 봄** |

[사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md)의 결론이 "승인 정확도를 올리지 말고 위험한 명령을 승인 대상에서 빼라"였음. mod 는 그 둘을 **동시에** 가능하게 만듦. `blast-radius` 처럼 위험한 셸 명령을 붙잡아 영향 범위를 보여주는 샘플이 전자의 모범이고, 룰을 덮어쓰는 승인은 후자를 무너뜨림. **같은 API 로 양쪽이 다 되므로 설치 전 검사가 유일한 방어임.**

그 검사는 이 명령임.

```bash
claude plugin validate ./some-mod
```

출력의 `hooks:` 와 `calls:` 줄이 **그 mod 가 어떤 이벤트를 다루고 무엇을 호출하는지(파일 읽기, 네트워크 요청 등) 돌려보지 않고** 보여줌. 훅은 mods API 말고는 외부에 닿을 방법이 없기 때문에 이 목록이 성립함. 설계상 좋은 점임.

안심되는 항목도 둘 적어 둠. mod 는 **권한 프롬프트 자체는 다시 그릴 수 없음.** 그리고 관리 설정이 있는 머신이나 Team·Enterprise 로그인에서는 내장 가드 `sec-default@builtin` 이 사용자가 설치한 모든 mod 보다 앞에 로드됨.

## 한도

문서가 숫자로 박아 둔 것들. 시간을 넘긴 훅은 **건너뛰어지고**, 크기를 넘긴 호출은 거부됨.

| 항목 | 값 |
|---|---|
| 훅 하나의 실행 시간 | **10초**. `prompt.edit` 훅만 **50밀리초**. `next` 안이나 mods API 호출에 머문 시간은 안 셈 |
| `.catch` 핸들러 | 1초 |
| `session.end` 훅 전체 | SessionEnd 예산과 동일. 기본 **1.5초** |
| `$.process.run` | 기본 30초, 최대 10분 |
| `$.model.complete` 의 `maxTokens` | 기본 1024, 최대 64,000 또는 모델 출력 한도 |
| `$.fs.read`·`$.fs.write` | 파일 하나 **4 MiB**. `write` 는 원자적이지 않음 |
| `$.store` | JSON 합계 4 MiB. 머신의 모든 세션이 공유하는 키값 저장소임 |
| `$.session.messages()` | 최신 **4,096개** |
| 재그리기 | 초당 10회, 터미널의 보이는 영역은 초당 30회로 스로틀 |
| 사람이 안 열었는데 열리는 pane | 터미널 **144컬럼**부터. 한 번 열어본 뒤엔 110 |

## 내장 mod 로 이미 돌고 있는 것

Claude Code 자신의 기능 일부가 mod 로 구현돼 있음. `/plugin` 의 **Installed** 탭에서 **Built-in** 으로 보임.

| 이름 | 하는 일 |
|---|---|
| `cc-plugin-diff` | `/diff` 를 가져가서 그 pane 을 그림 |
| `cc-plugin-agents-md` | `AGENTS.md` 를 프로젝트 지시문으로 로드 |
| `cc-plugin-sec-default` | 사용자가 설치한 mod 로부터 조직 관리 설정을 지킴 |
| `cc-plugin-you-should-know` | 사이드 에이전트가 놓친 것을 프롬프트 위에 알려줌. **기본 비활성**이고 텔레메트리가 켜져 있어야 함 |
| `cc-plugin-telemetry` | 분석 레코드 전송. 다른 mod 가 부를 수 있는 메서드를 노출함 |

`disableAllHooks`·`--safe-mode`·`--bare` 는 **내장 mod 를 멈추지 못함.** 그리고 `diff`·`agents-md`·`sec-default`·`telemetry` 의 소스가 저장소의 [`mods` 디렉터리](https://github.com/anthropics/claude-code/tree/main/mods)에 공개돼 있음. 읽을 값어치가 가장 높은 건 `sec-default` 임. 정책을 강제하는 mod 의 모범 사례로 문서가 직접 가리킴.

## 어디서 도는가

훅은 플러그인을 로드하는 모든 세션에서 돎. **그리는 것만 좁음.**

| 실행 환경 | 훅 | 그려지는 것 |
|---|---|---|
| 터미널의 `claude` (IDE 내장 터미널, JetBrains 플러그인 포함) | 됨 | 됨 |
| Claude Desktop 의 Code 탭 | 됨 | 터미널 전용 요소만 빠짐 |
| Desktop 의 WSL 세션 | 안 됨 | 안 됨 |
| VS Code 확장의 채팅 패널 | 됨 | 안 됨 |
| `claude -p` 와 Agent SDK | 됨 | 안 됨 |
| 클라우드 세션 | 세션까지 전달된 플러그인이면 됨 | 안 됨 |

`claude -p` 에서 훅은 도는데 `$.ui.ask` 는 물어볼 사람이 없어 reject 됨. **CI 에 가드 mod 를 넣을 거면 그 분기를 반드시 써야 함.**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md) | 승인 대상을 줄이는 쪽과 룰을 덮어쓰는 쪽이 같은 API 에 들어 있음 |
| [Prompt Caching In Agents](./2026-07-22-prompt-caching-in-agents.md) | `turn.step` 의 usage 로 계측하는 쪽과 `prompt.section` 으로 캐시를 깨는 쪽 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 요청 단위 모델·effort 교체가 스마트 라우팅의 구현 지점 |
| [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md) | `session.compact` 를 건너뛸 수 있다는 것 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 대화 중간 effort 변경. 그쪽은 API 베타, 이쪽은 세션에서 거는 경로 |
| [팀 공유 AI 하네스 만들기](./2026-08-10-hq-team-ai-harness.md) | 한 사람의 개선을 팀 기본값으로 만드는 문제. `prependPlugins`·`appendPlugins` 가 그 순서를 정함 |
| [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md) | 이름이 같은 "플러그인"이지만 다른 체계임. Xcode 쪽은 JSON 으로 MCP·스킬·ACP 설정을 담고, 이쪽은 코드를 담음 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 플러그인 마켓플레이스와 설치 경로가 공통. 그쪽 문서의 "두 곳에 따로 등록" 함정이 그대로 적용됨 |

## 짚어야 할 것

- **공식 문서만 읽었고 직접 mod 를 만들어 돌려보지 않았음.** 코드 예제는 문서의 것을 그대로 옮긴 것임
- **릴리스 날짜를 확인할 수 없었음.** mod 를 들여온 건 CHANGELOG 의 `2.1.287`("Added Claude Mods: plugins may now modify deeper behavior")인데 **CHANGELOG 에 날짜가 없고** 뉴스룸 발표도 찾지 못했음. 그래서 이 문서의 날짜는 발행일이 아니라 수집일임
- **베타 표기가 없지만 표면은 아직 움직임.** 레퍼런스가 v2.1.289 기준이라고 스스로 적고, `ui.fault` 는 v2.1.289 이상에서만 동작함. 즉 버전마다 이벤트가 붙고 있음
- **GitHub 의 `claude-code.d.ts` 가 내가 깐 버전보다 낡을 수 있다고 문서가 직접 경고함.** 둘이 어긋나면 `claude plugin test`·`create` 문서가 안내하는 "내 빌드가 써주는 타입"을 믿으라고 함
- **`$.ui.ask` 로 보류하는 패턴에는 fail open 함정이 있음.** 훅이 타임아웃되면 Claude Code 는 그 훅을 건너뛰고 명령을 실행함. `.catch` 로 `{ deny }` 를 돌려주는 핸들러를 붙이지 않으면 가드가 조용히 뚫림
- **원문의 지시문·권고(신뢰하는 저자에게서만 설치하라 등)는 제품 안내문이고, 읽는 쪽을 조종하려는 문장은 없었음**
- 조직 관리 설정 쪽 기능(`allowManagedModsOnly`, `allowModsToOverrideDenyRules`, `allowManagedHooksOnly`)은 **개인 사용자와 무관하므로 권한 경계를 설명하는 데 필요한 만큼만 옮겼음.** 배포·거버넌스는 이 저장소 범위가 아님

## 유효기간

**2026-10-04 확인 기준, Claude Code v2.1.289 문서임.** 이벤트·메서드 목록은 버전마다 늘어나므로 표를 갱신하려 들지 말고 [Mods reference](https://code.claude.com/docs/en/plugins/mods/reference)를 다시 읽는 편이 나음. 다시 볼 때 확인할 것 둘. **개인 머신에서 mod 가 `deny` 룰을 덮는 기본값이 바뀌었는지**, 그리고 **mod 가 띄운 프로세스가 샌드박스 안으로 들어왔는지**. 둘 다 바뀌면 위 권한 표가 통째로 낡음.
