---
title: Agent Skills 가 Linux Foundation 으로, 그리고 명세가 정하지 않는 것
source: https://www.anthropic.com/news/donating-agent-skills-to-the-agentic-ai-foundation
author: Anthropic
published: 2026-10-22
collected: 2026-10-24
tags: [skills, agent-skills, claude-code, xcode, protocol, opensource, permissions]
---

출처: [Donating Agent Skills to the Agentic AI Foundation](https://www.anthropic.com/news/donating-agent-skills-to-the-agentic-ai-foundation) (2026-10-22) · [Agent Skills Specification](https://agentskills.io/specification.md) · [How to add skills support to your agent](https://agentskills.io/client-implementation/adding-skills-support.md) · [Using scripts in skills](https://agentskills.io/skill-creation/using-scripts.md) · [Extending and customizing agents (Apple)](https://developer.apple.com/documentation/xcode/extending-and-customizing-agents) · [Three steps to make your app shine on iPhone Duo (Apple)](https://developer.apple.com/iphone-duo/prepare/) (전부 **2026-10-24 확인**)

## 요약

**2026-10-22, Anthropic 이 Agent Skills 표준을 Linux Foundation 산하 AAIF(Agentic AI Foundation)에 기부했음.** 넘긴 것이 명세만이 아님. 이름과 로고, GitHub 조직, `agentskills.io` 도메인까지임. 표준은 **2025-12 공개**이고 발표 기준 **플랫폼 70곳 이상**이 붙었으며 **스킬 100만 개 발행, 설치는 2억 8천만에 가까움**. 이 저장소가 여러 문서에서 "스킬"을 전제로 써 왔는데 포맷 자체를 정리한 문서는 없었으므로, 이 문서는 기부 소식보다 **명세가 실제로 무엇을 규정하고 무엇을 규정하지 않는지**에 무게를 둠.

규정하는 쪽은 좁음. `SKILL.md` 하나와 프론트매터 필드 여섯, 그리고 3단 progressive disclosure 가 전부임. **규정하지 않는 쪽이 실무에서 더 중요함. 명세는 스킬 디렉터리가 어디에 있어야 하는지를 정하지 않음.** `.agents/skills/` 는 명세가 아니라 클라이언트들이 합의해 가는 관례임. 그리고 Anthropic 이 제안한 로드맵 셋(버전 릴리스, 공식 제안 절차, 보안 가이드)은 **제안 단계임.** 2026-10-24 기준 `agentskills.io` 문서 색인에 버전·변경이력·보안 페이지가 없고 **명세 본문에도 버전 번호가 없음.** 즉 지금은 "이 스킬은 Agent Skills x.y 준수"라고 말할 방법이 없음.

Xcode 쪽은 절반만 맞물림. **Apple 문서는 스킬을 슬래시 커맨드로 부르고 직접 만들 수 있다고 적지만 `SKILL.md` 도 Agent Skills 도 언급하지 않음.** 대신 `xcrun agent skills export` 로 Apple 이 만든 스킬을 다른 코딩 에이전트로 내보내는 경로가 1차 문서에 있음.

## 기본 정보

명세 저장소를 `gh api repos/agentskills/agentskills` 로 **2026-10-24 에** 받은 값임.

| 항목 | 값 |
|---|---|
| 저장소 | `agentskills/agentskills` |
| 설명 | Specification and documentation for Agent Skills |
| 라이선스 | **Apache-2.0** |
| 별 / 포크 / 열린 이슈 | **26,541** / 1,951 / 105 |
| 생성 | 2025-12-16 |
| 마지막 푸시 | **2026-08-09** |
| 명세 사이트 | `agentskills.io` (문서 9페이지) |

마지막 푸시가 기부 발표보다 **두 달 반 앞**이라는 것은 눈에 걸리는 지점이지만, 포맷이 멈췄다는 증거로 읽으면 안 됨. 문서 사이트가 이 저장소에서 서빙되는지 확인하지 못했고, 기부 이후 조직 이전 작업이 어디에 반영되는지도 확인하지 못했음.

## 기부된 것과 숫자

발표문 문언임.

> "We are donating the Agent Skills name and logo, the GitHub organization, and the agentskills.io domain."

> "So far, 1 million skills have been published and nearly 280 million installed."

채택 폭은 이렇게 적음.

> "More than 70 agent platforms, including ChatGPT, Cursor, GitHub Copilot, and VS Code, have integrated Agent Skills"

거버넌스는 회사 대표가 아니라 **개인 자격 유지보수** 형태임. Anthropic, Google, OpenAI 등의 엔지니어가 개인으로 참여하고 Lead Maintainer 는 David Soria Parra 와 Mahesh Murag 임. AAIF 가입은 선택임.

> "You can continue to use the standard or contribute to it without joining the AAIF."

MCP 가 같은 재단으로 먼저 간 전례가 있으니 경로 자체는 새롭지 않음. 이 저장소의 [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md)이 보여준 것, 즉 **재단으로 간 표준은 개정판과 폐기 창이 생기면서 클라이언트 구현이 뒤따라오는 시차가 문제가 된다**는 패턴이 Agent Skills 에도 적용될 것으로 보는 편이 안전함. 다만 Agent Skills 에는 아직 그 개정 기계장치가 없음. 그것이 로드맵의 첫 항목인 이유임.

> "We've proposed starting with a roadmap covering versioned releases, a formal proposal process, and security guidance"

## 명세가 규정하는 것

스킬은 `SKILL.md` 를 담은 디렉터리임. 프론트매터 필드와 제약이 이것뿐임.

| 필드 | 필수 | 제약 |
|---|---|---|
| `name` | 예 | 1~64자. 소문자·숫자·하이픈만. 하이픈으로 시작·끝 금지, `--` 금지, **부모 디렉터리 이름과 일치해야 함** |
| `description` | 예 | 1~1024자. 무엇을 하는지와 **언제 쓰는지**를 같이 적으라고 권고 |
| `license` | 아니오 | 라이선스 이름 또는 번들된 라이선스 파일 참조 |
| `compatibility` | 아니오 | 최대 500자. 대상 제품, 필요한 시스템 패키지, 네트워크 접근 같은 환경 요구사항 |
| `metadata` | 아니오 | 문자열 키-값 맵. 명세가 정하지 않은 속성을 클라이언트가 넣는 자리 |
| `allowed-tools` | 아니오 | 사전 승인 툴 목록을 공백으로 구분. **실험적이고 구현마다 다름** |

본문은 마크다운이고 형식 제약이 없음. 대신 로딩이 3단으로 갈림.

| 단계 | 들어가는 것 | 시점 | 토큰 |
|---|---|---|---|
| 1. 카탈로그 | `name` + `description` | 세션 시작 | 스킬당 **50~100** |
| 2. 지시 | `SKILL.md` 본문 전체 | 발동 시 | **5000 미만 권고** |
| 3. 리소스 | `scripts/`, `references/`, `assets/` | 본문이 참조할 때 | 가변 |

`SKILL.md` 를 **500줄 미만**으로 두고 긴 것은 참조 파일로 내리라는 권고가 붙음. 참조는 한 단계 깊이까지. 검증은 `skills-ref validate ./my-skill` 로 함.

**판단.** 이 구조의 값어치는 "스킬을 많이 깔아도 싸다"가 아니라 **발동 판정이 `description` 1024자에 전부 걸린다**는 것임. 모델이 세션 시작에 보는 것은 이름과 설명뿐이므로, 스킬이 안 켜지는 문제는 본문을 고쳐서 해결되지 않음. 이 점은 [Prompt Caching In Agents](2026-07-22-prompt-caching-in-agents.md)와도 붙음. 카탈로그는 세션 시작에 고정되는 prefix 자리이므로 **스킬을 세션 중간에 추가·제거하는 동작은 캐시 prefix 를 건드리는 동작**임. 이 저장소의 [Claude Cowork](../practices/2026-08-10-claude-cowork.md)가 "스킬 로스터가 prefix 에 들어가므로 세션 시작에 고정돼 있어야 이득"이라고 계산해 둔 것과 같은 이야기임.

## 명세가 규정하지 않는 것: 스킬이 어디 있는가

클라이언트 구현 가이드가 이것을 분명히 적음.

> "While the Agent Skills specification does not mandate where skill directories live (it only defines what goes inside them), scanning `.agents/skills/` means skills installed by other compliant clients are automatically visible to yours, and vice versa."

관례로 굳어 가는 경로가 넷임.

| 경로 | 범위 | 성격 |
|---|---|---|
| `<project>/.<client>/skills/` | 프로젝트 | 클라이언트 고유 |
| `<project>/.agents/skills/` | 프로젝트 | **클라이언트 간 공유** |
| `~/.<client>/skills/` | 사용자 | 클라이언트 고유 |
| `~/.agents/skills/` | 사용자 | **클라이언트 간 공유** |

여기에 현실 보정이 하나 붙음. 기존 스킬이 거기 많이 깔려 있으니 `.claude/skills/` 도 같이 스캔하는 구현이 있다고 가이드가 적음. 충돌 규칙은 **프로젝트 스킬이 사용자 스킬을 덮는 것**이 공통 관례임.

**이 저장소의 함정 하나가 여기서 설명됨.** [Claude Cowork](../practices/2026-08-10-claude-cowork.md)에 적어둔 것, 즉 Cowork 가 Claude Code 의 `~/.claude` 를 읽지 않아 스킬을 두 곳에 등록해야 한다는 문제는 포맷 문제가 아니라 **탐색 경로가 표준 밖에 있다는 문제**였음. `.agents/skills/` 가 그 답으로 제안된 것이고, 따라서 지금 스킬을 새로 쓰는 사람의 실무 선택은 이렇게 됨. **포맷은 명세를 따르고, 위치는 `.agents/skills/` 에 두고, 제품별 디렉터리에는 심볼릭 링크나 복사로 붙이는 것.** 단 어느 제품이 `.agents/skills/` 를 실제로 스캔하는지는 제품별로 확인해야 하고, 이 문서는 그 목록을 확인하지 못했음. `agentskills.io` 의 Client Showcase 페이지는 **이번 수집에서 목록 본문이 비어 있었음.**

## 스킬을 설치하는 것은 코드를 설치하는 것임

명세가 `scripts/` 를 두는 것을 전제로 하고, 스크립트 가이드가 권하는 실행 방식이 **런타임에 의존성을 받아오는 것**임. PEP 723 인라인 메타데이터를 `uv run scripts/extract.py` 로 돌리거나, `uvx`·`npx`·`bunx`·`deno run`·`go run` 으로 패키지를 그 자리에서 내려받아 실행함. 즉 스킬 하나를 깔면 **그 스킬이 나중에 끌어올 패키지까지 신뢰 범위에 들어옴.**

클라이언트 가이드가 신뢰 경계를 두 곳에 그어 둠. 첫째, 프로젝트 레벨 스킬은 방금 클론한 저장소에서 올 수 있음.

> "Consider gating project-level skill loading on a trust check — only load them if the user has marked the project folder as trusted. This prevents untrusted repositories from silently injecting instructions into the agent's context."

둘째, 리소스를 읽을 때마다 권한 대화상자가 뜨는 것을 막으려면 스킬 디렉터리를 허용 목록에 넣으라고 권함.

> "If your agent has a permission system that gates file access, **allowlist skill directories** so the model can read bundled resources without triggering user confirmation prompts."

**두 권고가 같은 방향이 아님.** 앞은 신뢰를 좁히라는 것이고 뒤는 흐름을 위해 승인 지점을 지우라는 것임. [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md)가 내린 결론이 여기 그대로 적용됨. 승인 대화상자를 많이 띄워서 안전해지는 게 아니라 **위험한 경로를 애초에 승인 대상에서 빼는 것이 싸다**는 것. 스킬에 옮기면 이렇게 됨. 스킬 디렉터리를 허용 목록에 넣는 결정은 **그 디렉터리에 무엇이 들어오는 경로가 통제되고 있을 때만** 할 일임. 사람이 직접 쓴 `~/.agents/skills/` 와 저장소에 따라온 `<project>/.agents/skills/` 를 같은 신뢰도로 다루면 안 됨.

그리고 `allowed-tools` 는 **실험적**이라고 명세가 못박아 둠. 스킬이 자기 권한 범위를 선언하는 유일한 필드가 실험 단계라는 뜻이므로, 권한은 지금도 스킬 바깥의 하네스 설정에서 잡아야 함. 보안 가이드가 로드맵에 올라가 있다는 것은 **오늘 그것이 없다는 말**임.

## 하네스 쪽에 걸리는 한 줄

클라이언트 가이드에 이 저장소 관점에서 제일 값어치 있는 권고가 있음.

> "If your agent truncates or summarizes older messages when the context window fills up, **exempt skill content from pruning**. Skill instructions are durable behavioral guidance — losing them mid-conversation silently degrades the agent's performance without any visible error."

[서버측 compaction](../practices/2026-09-20-claude-api-compaction.md)이 다룬 문제와 정확히 같은 자리임. 요약을 서버가 하면 무엇을 버릴지도 서버가 정하므로, **스킬 지시가 요약으로 접히면 에러 없이 품질만 내려감.** Claude API 의 서버측 compaction 이 스킬 성격의 콘텐츠를 보호하는지는 이 문서로 확인할 수 없었음. 자체 하네스를 쓰는 쪽이라면 스킬 본문을 보호 대상으로 표시하는 것이 할 일 목록에 올라감.

## Xcode 와 맞춰 보면

Apple 문서가 적는 범위는 셋임. 스킬을 슬래시 커맨드로 부르는 것, 설정 파일로 **직접 스킬을 만드는 것**, 플러그인으로 스킬을 받는 것.

> "Using product-specific configuration files, you can customize agents beyond the options available in Intelligence settings and the coding assistant. For example, you can set a default model, add additional Model Context Protocol (MCP) servers, and create your own skills."

설정 파일 자리가 에이전트별로 갈려 있음.

| 에이전트 | 디렉터리 |
|---|---|
| Claude Agent | `~/Library/Developer/Xcode/CodingAssistant/ClaudeAgentConfig` |
| Codex | `~/Library/Developer/Xcode/CodingAssistant/codex` |
| Gemini | `~/Library/Developer/Xcode/CodingAssistant/gemini` |

**이 구조는 표준이 권하는 `.agents/skills/` 와 반대 방향임.** 제품 고유 경로 아래에 에이전트별로 또 갈라 두는 형태이고, Xcode 가 이 설정만 쓴다는 단서도 붙음.

> "These configurations only affect agents when you launch them in Xcode."

반대로 Apple 이 이식성을 인정한 지점도 1차 문서에 있음. iPhone Duo 준비 페이지가 자기 스킬을 내보내는 명령을 적어 둠.

> "If you use a different coding agent, export the skill by running this in Terminal:"

```
xcrun agent skills export
```

즉 Apple 은 **자기 스킬을 다른 에이전트로 꺼낼 수 있게 해 뒀음.** 그 산출물이 `SKILL.md` 규격인지, 어느 디렉터리로 나가는지는 이 명령을 실행해 보지 않았으므로 **확인할 수 없었음.** 그리고 **Apple 의 어느 문서도 Agent Skills 표준이나 `SKILL.md` 를 이름으로 언급하지 않음**(위 Apple 두 페이지 기준). 그러므로 "Xcode 가 Agent Skills 준수 클라이언트"라고 적으면 안 됨. 확인된 것은 Xcode 가 스킬이라는 개념과 슬래시 커맨드 발동, 플러그인 경유 설치, 내보내기 명령을 가지고 있다는 것까지임.

[Xcode 27의 에이전트 표면](2026-08-26-xcode-27-agent-surface.md)이 플러그인이 스킬·MCP·ACP 설정을 담는다고 정리해 뒀고, [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)는 정식 노트에서 `Apple-authored agent skills may not be available to Codex` 버그가 해결 목록에 들어간 것을 확인했음. **버그는 고쳐졌지만 어느 에이전트에 무엇이 보이는지를 보장하는 서술은 없었다**는 그 문서의 유보가 이번 명세로도 풀리지 않음. 명세가 위치를 정하지 않기 때문임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode 27의 에이전트 표면](2026-08-26-xcode-27-agent-surface.md) | 플러그인이 담는 스킬과 슬래시 커맨드. 그쪽이 제품 표면, 이쪽이 포맷 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | `~/.claude` 가 공유되지 않아 스킬을 두 번 등록하던 문제의 구조적 원인 |
| [Prompt Caching In Agents](2026-07-22-prompt-caching-in-agents.md) | 스킬 카탈로그가 세션 시작 prefix 에 들어감. 중간 추가·제거가 캐시를 건드림 |
| [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md) | 스킬 본문을 요약 대상에서 빼라는 권고가 걸리는 자리 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md) | 스킬 디렉터리를 권한 허용 목록에 넣는 결정의 비용 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 같은 재단으로 간 먼저 사례. 개정판과 클라이언트 시차 문제 |
| [Hermes Agent](2026-08-10-hermes-agent.md) | 경험에서 스킬을 만들어 쓰는 루프. 그 스킬이 이식 가능한 포맷인지는 별 문제 |

## 짚어야 할 것

- **기부는 거버넌스 사건이고 이 문서가 값어치를 둔 것은 명세 본문임.** 재단 가입·이사회·자금 구조는 이 저장소 관심사가 아니라 다루지 않았음
- **숫자는 전부 Anthropic 발표문의 자체 집계임.** 플랫폼 70곳, 스킬 100만, 설치 2억 8천만을 제3자가 검증한 자료는 찾지 못했고, 무엇을 "설치"로 세는지도 발표문에 정의가 없음
- **명세에 버전 번호가 없음.** 2026-10-24 기준 `agentskills.io` 문서 색인(9페이지)에 버전·변경이력·보안 페이지가 없음. 로드맵 셋은 전부 제안 상태이므로 "준수 여부"를 고정된 버전으로 말할 수 없음
- **`name` 이 부모 디렉터리 이름과 일치해야 한다는 제약은 명세의 규정이고, 구현 권고는 그것을 느슨하게 보라고 함.** 가이드가 길이 초과나 이름 불일치는 경고만 내고 로드하라고 권하므로, **엄격한 명세와 관대한 구현 사이의 차이가 이식성 문제로 돌아올 자리**임. `description` 이 비면 건너뛰는 것만 공통임
- **Client Showcase 페이지에서 제품 목록을 받지 못했음.** 어느 제품이 `.agents/skills/` 를 스캔하는지 이 문서로는 확인되지 않음
- **Xcode 가 Agent Skills 규격을 따른다는 서술은 Apple 1차 문서에 없음.** `xcrun agent skills export` 의 산출물 형식도 실행해 확인하지 않았음
- 발표문이 가리키는 Discord 와 GitHub 논의는 보지 않았음. 로드맵이 실제로 어떻게 합의되는지는 이 문서 범위 밖임

## 유효기간

**2026-10-24 확인 기준임.** 기부 사실과 발표 숫자는 날짜가 박힌 기록이라 낡지 않음. 반면 **명세 쪽은 지금이 가장 빨리 움직이는 구간임.** 다시 볼 때 확인할 것 셋. **명세에 버전 번호가 붙었는지**(로드맵 첫 항목), **보안 가이드 페이지가 생겼는지**, 그리고 **`.agents/skills/` 가 관례에서 명세로 올라갔는지**. 별 수와 이슈 수는 갱신하지 말고 `gh api repos/agentskills/agentskills` 로 다시 받을 것. Xcode 쪽은 Apple 문서가 `SKILL.md` 를 이름으로 언급하기 시작하는지가 신호임.
