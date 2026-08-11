---
title: 에이전트 생태계 레포 지형도 — 스킬, 하네스, 메모리
source: https://github.com/search?q=topic%3Aagent-skills&type=repositories
collected: 2026-08-10
tags: [claude-code, codex, agent-skills, plugin, harness, memory, ecosystem, github]
---

출처: GitHub Search API로 `topic:claude-code`, `topic:agent-skills`, `topic:claude-skills`, `topic:claude-plugin`, `topic:mcp-server`를 훑고 2026-08-10 기준 메타데이터를 직접 받아 정리함

## 요약

Claude Code와 Codex 주변에 붙는 스킬·플러그인·하네스 생태계를 한 장으로 정리한 스냅샷임. 개별 레포는 몇 달이면 순위가 뒤집히니 **유형별 지형과 판단 기준**에 무게를 뒀음. 지금 가장 뜨거운 카테고리는 **토큰 절감**이고, 밈으로 시작한 스킬이 별 10만을 받는 걸 보면 비용이 그만큼 아픈 지점임.

## 측정 방법과 한계

GitHub API에는 트렌딩 페이지가 없음. 그래서 **생성일 대비 별 증가 속도(★/일)** 를 대리 지표로 썼음.

- 최근 급등한 레포를 잡아내지만, 초반에 몰아서 오른 뒤 식은 레포도 위로 올라옴
- 즉 "지금 뜨는 중"이 아니라 "생애 평균 상승률"임. 진짜 최근 모멘텀은 별 히스토리를 봐야 알 수 있음
- ⚠️ **이 생태계의 별 수 자체를 품질 지표로 보면 안 됨.** 2026-01 생성 레포가 23만 별을 받는 규모는 역사적 기준으로 이례적임. 프로모션이 섞였을 가능성을 감안하고, 별보다 최근 커밋과 이슈 대응을 봐야 함

아래 표는 그렇게 뽑은 후보 풀에서 **손으로 걸러낸 부분집합**임. 기계적인 top-N이 아님. 제외 기준은 둘.

- **독립 에이전트 본체**는 뺐음. `NousResearch/hermes-agent`(★/일 594)나 `gemini-cli`처럼 스킬·플러그인이 아니라 에이전트 자체인 것들
- **주제가 벗어난 것**은 뺐음. `worldmonitor`(뉴스 대시보드)나 `n8n`처럼 `mcp-server` 토픽에 걸려 들어오지만 코딩 에이전트 생태계가 아닌 것들

후보 풀에 있었지만 이번에 다루지 않은 것도 있음. `Egonex-AI/Understand-Anything`(531), `santifer/career-ops`(495), `Leonxlnx/taste-skill`(434), `Panniantong/Agent-Reach`(419). 판단해서 뺀 게 아니라 지면 때문에 뺐음.

## 스냅샷 (2026-08-10)

| ★/일 | Stars | Forks | 생성 | 레포 | 라이선스 |
|---:|---:|---:|---|---|---|
| 1,688 | 99,566 | 5,478 | 2026-06-12 | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | MIT |
| 1,172 | 239,059 | 36,314 | 2026-01-18 | [affaan-m/ECC](https://github.com/affaan-m/ECC) | MIT |
| 815 | 84,788 | 9,907 | 2026-04-28 | [nexu-io/open-design](https://github.com/nexu-io/open-design) | Apache-2.0 |
| 812 | 104,704 | 10,181 | 2026-04-03 | [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify) | Apache-2.0 |
| 759 | 97,114 | 5,598 | 2026-04-04 | [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) | MIT |
| 520 | 167,280 | 19,956 | 2025-09-22 | [anthropics/skills](https://github.com/anthropics/skills) | 미표기 |
| 485 | 85,303 | 9,179 | 2026-02-15 | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) | MIT |
| 455 | 115,096 | 12,333 | 2025-11-30 | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | MIT |
| 340 | 126,020 | 8,576 | 2025-08-04 | [farion1231/cc-switch](https://github.com/farion1231/cc-switch) | MIT |
| 270 | 67,580 | 5,511 | 2025-12-03 | [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) | NOASSERTION |
| 262 | 90,240 | 7,858 | 2025-08-31 | [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) | Apache-2.0 |

## 유형별로 보기

### 토큰 절감

지금 생태계에서 가장 뜨거운 카테고리임.

- **[caveman](https://github.com/JuliusBrussee/caveman)** (97,114★). 원시인 말투로 프롬프트를 줄여 **토큰 65%를 깎는** Claude Code 스킬. "why use many token when few token do trick." 밈으로 시작했는데 별이 10만에 육박함
- **[alexgreensh/token-optimizer](https://github.com/alexgreensh/token-optimizer)** (1,835★). 규모는 작지만 목적이 정확함. 유령 토큰을 찾아 고치고, compaction에서 살아남고, 컨텍스트 품질 저하를 피하는 것
- **[oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)** (67,580★). "tokenmaxxers를 위한 코딩 에이전트"를 자칭하는 하네스. 복잡한 코드베이스를 노림. 한국 개발자 프로젝트

### 에이전트 사고방식을 바꾸는 스킬

- **[ponytail](https://github.com/DietrichGebert/ponytail)** (99,566★, ★/일 1위). "방 안에서 가장 게으른 시니어 개발자처럼 생각하게" 만듦. 안 쓴 코드가 최선의 코드라는 YAGNI를 에이전트에 강제. Cursor rules도 지원
- **[affaan-m/ECC](https://github.com/affaan-m/ECC)** (239,059★, 포크 36,314). 하네스 성능 최적화 시스템. 스킬, instincts, 메모리, 보안, research-first 개발을 묶음. Claude Code, Codex, OpenCode, Cursor 지원

### 도메인 스킬

- **[anthropics/skills](https://github.com/anthropics/skills)** (167,280★). Anthropic 공식 저장소. 스킬을 만들기 전에 규약을 확인할 기준점
- **[addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)** (85,303★). 프로덕션급 엔지니어링 스킬. Chrome 팀 출신이라 프론트엔드 성능 쪽 신뢰도가 있음
- **[ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)** (115,096★). UI/UX 설계 지능. 토픽에 React, Tailwind, mobile-ui, UIKit이 걸려 있음
- **[K-Dense-AI/scientific-agent-skills](https://github.com/K-Dense-AI/scientific-agent-skills)** (33,081★). 과학 연구용 스킬 라이브러리

### 하네스와 워크스페이스

- **[cc-switch](https://github.com/farion1231/cc-switch)** (126,020★, Rust/Tauri). Claude Code, Codex, OpenCode, OpenClaw, Grok Build, Hermes Agent를 한 데스크톱 앱에서 전환. 프로바이더 관리, 스킬 관리, WSL 지원
- **[NanmiCoder/cc-haha](https://github.com/NanmiCoder/cc-haha)** (14,016★, 포크 8,531). 멀티 에이전트와 Git 워크트리를 다루는 로컬 우선 워크스페이스. 별 대비 포크 비율이 특이하게 높음
- **[nexu-io/open-design](https://github.com/nexu-io/open-design)** (84,788★). Claude Design의 오픈소스 대안. 코딩 에이전트를 디자인 엔진으로 쓰고 HTML/PDF/PPTX/MP4로 실제 파일을 내보냄. BYOK로 CLI 20종 이상 연결

### 메모리와 컨텍스트

- **[claude-mem](https://github.com/thedotmack/claude-mem)** (90,240★). 세션 중 에이전트가 한 일을 포착해 AI로 압축하고 다음 세션에 관련 컨텍스트만 주입. SQLite + ChromaDB
- **[OthmanAdi/planning-with-files](https://github.com/OthmanAdi/planning-with-files)** (26,072★). 파일 기반 영속 계획. 크래시에 견디는 마크다운 구조

### 코드 이해

- **[graphify](https://github.com/Graphify-Labs/graphify)** (104,704★). 코드베이스와 문서, SQL 스키마, 설정, PDF를 질의 가능한 지식 그래프로. **벡터 스토어 없이 로컬 결정론적 AST 파싱**을 쓰고 모든 엣지에 설명이 붙음

### 디렉터리

개별 도구를 찾을 때 출발점.

- [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) 72,162★
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) 52,019★
- [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) 29,913★, 스킬 1,000개 이상 큐레이션
- [VoltAgent/awesome-openclaw-skills](https://github.com/VoltAgent/awesome-openclaw-skills) 51,855★, 5,400개 이상 필터링

## 도입할 때 판단 기준

**토큰 절감 스킬은 출력 품질을 함께 재야 함.** caveman처럼 프롬프트 문체를 바꾸는 방식은 65%를 깎는 대가로 무엇을 잃는지 측정하지 않으면 의미가 없음. [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md)이 말하는 원칙이 여기 그대로 적용됨. 품질과 정답당 비용을 항상 같이 인용할 것.

**사고방식을 바꾸는 대형 스킬은 CLAUDE.md와 충돌할 수 있음.** ECC나 ponytail은 에이전트에 상당한 지시를 주입함. 도입 전에 무엇을 주입하는지 읽어봐야 하고, 기존 프로젝트 지침과 어긋나면 둘 중 하나가 무력화됨.

**스킬 설치·제거와 스킬 호출을 구분해야 함.** 캐시가 깨지는 건 프롬프트 prefix 앞쪽이 바뀔 때임. 무엇이 prefix에 들어가는지에 따라 갈림.

| 행위 | 캐시 영향 |
|---|---|
| 스킬을 설치·제거해서 **사용 가능 목록이 바뀜** | 목록이 prefix에 들어가므로 그 뒤 대화가 무효화됨 |
| MCP 서버나 플러그인이 **툴 정의를 추가·변경** | 툴 정의는 prefix 앞쪽이라 무효화됨 |
| 이미 설치된 스킬을 **세션 중간에 호출** | 트랜스크립트에 append되므로 무효화되지 않음 |

즉 "스킬을 쓰면 캐시가 깨진다"가 아니라 **"사용 가능한 것의 목록을 바꾸면 깨진다"**임. 실행 중에 스킬을 붙였다 떼는 것을 피하고 세션 시작 시 고정하라는 결론은 그대로인데, 이유는 툴 스키마 변경이 아니라 prefix 변경임. 자세한 원리는 [Prompt Caching In Agents](./2026-07-22-prompt-caching-in-agents.md)의 툴 로드아웃 항목에 있음.

**메모리 도구는 쓰기 경로 비용을 봐야 함.** [Memory Engineer](../practices/2026-08-02-memory-engineer.md)의 지적대로 비용은 조회가 아니라 구축에서 나옴. claude-mem처럼 "세션 중 전부 포착해서 AI로 압축"하는 방식은 그 압축이 곧 쓰기 경로임. 정확도가 같은 두 시스템이 정답당 에너지로 47배 갈렸다는 Stanford 결과를 기억할 것.

**라이선스가 미표기이거나 NOASSERTION인 레포가 섞여 있음.** `anthropics/skills`가 라이선스 미표기이고, `oh-my-openagent`와 일부 awesome 목록이 NOASSERTION임. 사내 도입이면 LICENSE 파일을 직접 확인해야 함.

## 이 저장소의 관련 문서와 겹치는 지점

| 이 지형도의 항목 | 이미 정리된 문서 |
|---|---|
| graphify | [Semantica](../infra/2026-08-10-semantica-context-graph.md) — 같은 문제의식인데 graphify가 훨씬 가벼움 |
| claude-mem, planning-with-files | [Memory Engineer](../practices/2026-08-02-memory-engineer.md) |
| caveman, token-optimizer | [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) 레버 4 |
| cc-haha, cc-switch | [Paseo vs Orca](./2026-08-10-paseo-vs-orca-agent-orchestrators.md), [Agent Orchestrator](./2026-08-10-agent-orchestrator-ao.md) |

## 유효기간

이 문서는 **2026-08-10 스냅샷**임. 이 생태계는 주 단위로 바뀌니 순위는 몇 달 안에 무의미해짐. 다시 볼 때는 표를 갱신하지 말고 유형별 분류와 판단 기준만 남기고 새로 조사하는 편이 나음.
