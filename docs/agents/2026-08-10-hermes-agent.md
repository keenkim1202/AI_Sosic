---
title: Hermes Agent, 자기개선 루프를 실제로 돌리는 에이전트
source: https://github.com/NousResearch/hermes-agent
author: Nous Research
collected: 2026-08-10
tags: [hermes-agent, nous-research, agent-memory, self-improving, skills, gateway, serverless]
---

출처: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) · [문서](https://hermes-agent.nousresearch.com/docs/) · [v0.20.0 릴리스 노트](https://github.com/NousResearch/hermes-agent/releases) (2026-08-10 기준 스냅샷)

## 요약

Nous Research가 만든 MIT 라이선스 에이전트인데, "또 하나의 코딩 에이전트"로 보면 놓치는 게 있음. 차별점은 둘임. **경험에서 스킬을 만들고 사용 중에 개선하는 학습 루프를 실제로 돌린다는 것**, 그리고 **노트북에 묶이지 않는다는 것**. Telegram으로 말을 걸어두면 클라우드 VM에서 일하고, 유휴 시 환경이 동면했다 깨어남. 여기 [Memory Engineer](../practices/2026-08-02-memory-engineer.md) 문서의 이론을 대볼 구현체로서 값어치가 있음.

## 기본 정보

| | |
|---|---|
| 저장소 | `NousResearch/hermes-agent` |
| Stars / Forks | 228,053 / 44,804 |
| 언어 | Python (3.11) |
| 라이선스 | MIT |
| 생성 | 2025-07-22 |
| 최신 릴리스 | v0.20.0 (2026-08-03), "The Herald Release" |
| 설치 | `curl -fsSL .../install.sh \| bash`, Windows는 PowerShell 원라이너 |

## 자기개선 학습 루프

README가 "학습 루프가 내장된 유일한 에이전트"라고 주장하는 부분임. 구성 요소가 이렇게 나뉨.

- **경험에서 스킬 생성.** 복잡한 작업을 끝내면 스스로 스킬을 만듦
- **사용 중 스킬 개선.** 만든 스킬이 쓰이면서 다듬어짐
- **자기 넛지.** 지식을 남기라고 주기적으로 스스로를 찌름
- **세션 검색.** FTS5 전문검색 + LLM 요약으로 과거 대화를 뒤져 회상
- **사용자 모델링.** [Honcho](https://github.com/plastic-labs/honcho) dialectic user modeling으로 세션을 넘어 "당신이 누구인지"에 대한 모델을 쌓음
- **표준 호환.** [agentskills.io](https://agentskills.io) 오픈 표준을 따름

### Memory Engineer 관점에서 볼 지점

[Memory Engineer](../practices/2026-08-02-memory-engineer.md)의 핵심 지적이 "비용은 조회가 아니라 **쓰기 경로**에서 나온다"였음. Hermes는 그 쓰기 경로를 상시로 돌리는 설계임. 그래서 검증할 질문이 명확함.

- Stanford 결과대로 **정답당 에너지**를 재면 어디에 서는지. 정확도가 같은 두 시스템이 47배 갈렸던 그 지표
- 스킬을 계속 만들고 개선하면 저장소가 자란다는 뜻인데, **망각 정책이 있는지**. Stanford가 테스트한 시스템 중 기본으로 가지치기하는 게 하나도 없었음
- Microsoft PlugMem의 지적처럼 **원시 메모리를 더 주면 오히려 나빠질 수 있음.** Hermes가 로그를 쌓는지 사실과 스킬로 증류하는지가 갈림길
- 모순되는 기억을 자동 병합하는지, 드러내고 사람에게 맡기는지

README와 릴리스 노트만으로는 이 답이 안 나옴. 직접 돌려보고 `/usage`와 `/insights`로 재봐야 할 부분임.

## 노트북에 묶이지 않는 구조

이쪽이 다른 에이전트와 실질적으로 갈리는 지점임.

**메시징 게이트웨이.** Telegram, Discord, Slack, WhatsApp, Signal, 이메일을 **게이트웨이 프로세스 하나로** 처리함. 음성 메모 전사와 플랫폼 간 대화 연속성이 붙음. `hermes gateway`로 띄우고 봇에 메시지를 보내면 시작됨.

**터미널 백엔드 7종.** local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox.

**서버리스 영속성.** Daytona와 Modal 백엔드에서는 에이전트 환경이 **유휴 시 동면하고 요청 시 깨어남.** 세션 사이 비용이 거의 없음. $5 VPS에서도, GPU 클러스터에서도 돌린다고 주장함.

**cron 스케줄러 내장.** 자연어로 일간 리포트, 야간 백업, 주간 감사를 무인 실행하고 결과를 아무 플랫폼으로 배달함.

**위임과 병렬화.** 격리된 서브에이전트를 띄워 병렬 작업을 돌리고, Python 스크립트가 RPC로 툴을 호출해 멀티스텝 파이프라인을 **zero-context-cost turn**으로 접음. [Graph Engineering](../practices/2026-07-20-graph-engineering-with-claude.md)의 "조율은 코드라서 토큰이 안 든다"와 같은 발상임.

## 다른 코딩 에이전트를 조종하는 계층

조사하다 나온 것 중 이게 제일 눈에 걸림. 번들 스킬 디렉터리 `skills/autonomous-ai-agents/` 안에 이런 게 들어 있음.

```
skills/autonomous-ai-agents/
  claude-code
  codex
  computer-use
  hermes-agent
  opencode
```

즉 Hermes가 Claude Code와 Codex, OpenCode를 **자기 스킬로 호출하는 메타 레이어**를 지향함. Paseo나 Orca가 여러 에이전트를 나란히 놓고 사람이 감독하는 구조라면, Hermes는 자기가 위에 서서 부리는 구조임. 층위가 다름.

번들 스킬 카테고리는 apple, autonomous-ai-agents, creative, email, github, index-cache, media, mlops, note-taking, productivity, research, smart-home, social-media, software-development임.

## v0.20.0 The Herald Release

2026-08-03. 이름값 하려고 넣은 기능들이 이렇게 정리됨.

- **음성.** 실시간 대화형 음성. 스트리밍 TTS, barge-in, 온디바이스 웨이크워드, CLI·데스크톱·오디오 지원 게이트웨이 전반에서 핸즈프리
- **A2A v1.0.** 다른 에이전트에게 말을 전달하는 프로토콜
- **서명된 아웃바운드 웹훅.** 이벤트를 외부 시스템에 알림
- **근거 있는 리서치.** 검증 가능한 인용과 팩트체킹
- **데스크톱 앱이 플랫폼으로.** 라이브 프리뷰가 붙은 artifacts, 플러그인 SDK, 어디서든 빠른 입력, 다중 창
- **CLI 파워 명령.** `!` 셸 모드, `/init`, `/diff`, `/context`, `/focus`
- **툴이 자기 실패에서 복구함.** 모델에게 추측을 떠넘기지 않음

## 개발 속도와 이슈 규모

먼저 규모를 보면 놀라움.

| 항목 | 값 |
|---|---|
| open 이슈 | 10,064 |
| **open PR** | **20,193** |
| 컨트리뷰터 | 650명 이상 |

open PR 2만 개는 이례적임. 다만 릴리스 노트를 보면 해석이 달라짐. v0.19.0에서 v0.20.0 사이에 **커밋 약 3,650개, 머지된 PR 약 1,400개, 파일 약 5,200개 변경, 이슈 약 1,200개 종료**가 있었음. 즉 처리 속도 자체는 빠름. 유입이 그보다 더 빠른 상황으로 추정되는데, 봇 생성 PR이나 중복 제출 같은 다른 설명도 가능하고 이 수치만으로는 가릴 수 없음.

이 속도의 반대편도 있음. 삽입 약 559,000줄에 삭제 약 405,000줄이 한 릴리스 사이클에 일어났음. 다만 이 숫자만으로 구조 변경이라고 단정할 수는 없음. 생성물, 벤더 파일, 일괄 포맷 변경이 섞였을 수 있음. **변화 폭이 크다는 것까지가 이 수치로 말할 수 있는 전부**이고, 프로덕션에 얹으려면 어떤 파일이 바뀌는지 직접 봐야 함.

## 짚어야 할 것

**설치가 `curl | bash`임.** Windows는 PowerShell `iex (irm ...)`. 신뢰 모델을 따져볼 지점임.

**번들한 `uv.exe`가 백신에 오탐됨.** README가 폴더를 화이트리스트하라고 안내하는데, 파일 해시가 아니라 폴더를 등록하라고 함. uv 버전마다 해시가 바뀌기 때문. Astral의 업스트림 이슈 세 건을 근거로 대고 `gh attestation verify`로 정품 확인하는 절차도 문서화해뒀음. 대응은 성실한 편이지만, 백신 예외를 폴더 단위로 등록하는 건 그 자체로 위험을 만드는 행위라 기록해둠.

**HN 논란 두 건은 확인된 것과 확인 못 한 것을 나눠야 함.**

- 2026-05에 "Nous Research가 표절 주장을 담은 GitHub 이슈를 편집해 지웠다"는 HN 스레드가 있었음(issue #10232). 확인해보니 그 이슈는 **닫혀 있고 제목이 `.`, 본문이 1글자**임. 내용이 비워진 건 사실임. 다만 **원래 무엇이 적혀 있었는지 확인할 수 없었고**, 표절 주장의 진위는 판단할 근거가 없음
- 2026-06에 "공동창업자가 Polymarket 스킬의 기본 설치를 옹호했다"는 스레드가 있었음. 지금 확인하면 **Polymarket 스킬은 `optional-skills/finance/`에 있고 기본 번들이 아님.** 그 사이 옮겨졌을 수도 있고 당시 상황이 달랐을 수도 있음. 현재 상태는 optional임

**Nous Portal 업셀이 있음.** 모델, 웹 검색(Firecrawl), 이미지 생성(FAL), TTS(OpenAI), 클라우드 브라우저(Browser Use)를 구독 하나로 묶어 팜. 300개 이상 모델. 다만 BYOK가 백엔드별로 가능하다고 명시하고 전부 아니면 전무가 아님

**OpenClaw에서의 이주 경로가 있음.** `hermes claw migrate`로 SOUL.md, 메모리, 스킬, 명령 허용목록, 메시징 설정, 허용된 API 키, TTS 자산, AGENTS.md를 가져옴. 토픽에 `clawdbot`, `moltbot`, `openclaw`가 걸려 있는 걸 보면 이 계보에 사연이 있어 보이는데, 이번에는 파지 않았음

## 이 저장소의 다른 문서와 겹치는 지점

| Hermes의 요소 | 관련 문서 |
|---|---|
| 학습 루프, 세션 검색, 사용자 모델링 | [Memory Engineer](../practices/2026-08-02-memory-engineer.md) |
| 서브에이전트 + RPC로 컨텍스트 비용 없는 조율 | [Graph Engineering with Claude](../practices/2026-07-20-graph-engineering-with-claude.md) |
| 스킬 로스터가 prefix에 들어가는 문제 | [에이전트 생태계 레포 지형도](./2026-08-10-agent-ecosystem-repos.md), [Prompt Caching In Agents](./2026-07-22-prompt-caching-in-agents.md) |
| 다른 에이전트를 조종하는 계층 | [Paseo vs Orca](./2026-08-10-paseo-vs-orca-agent-orchestrators.md), [Agent Orchestrator](./2026-08-10-agent-orchestrator-ao.md) |
| `curl \| bash`, 백신 예외, 명령 승인 | [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md) |

## 유효기간

**2026-08-10 스냅샷**임. 릴리스가 주 단위로 나오고 한 사이클에 50만 줄 이상이 바뀌는 프로젝트라, 기능 목록은 빠르게 낡음. 위 링크로 릴리스 노트를 직접 확인하는 편이 나음.
