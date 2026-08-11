---
title: Agent Orchestrator (AO) — 병렬 코딩 에이전트 감독 IDE
source: https://github.com/Untrivial-ai/agent-orchestrator
author: Untrivial AI
collected: 2026-08-10
tags: [coding-agent, orchestration, worktree, ci, code-review, opensource]
---

출처: [Untrivial-ai/agent-orchestrator](https://github.com/Untrivial-ai/agent-orchestrator) · [aoagents.dev](https://aoagents.dev) (수치는 2026-08-10 GitHub API 기준)

## 요약

병렬 코딩 에이전트를 격리된 워크스페이스에서 굴리는 "메타 하네스" IDE임. 다른 오케스트레이터랑 갈리는 지점은 피드백 루프 자동화. CI 실패, 리뷰 코멘트, 머지 충돌을 알아서 담당 세션으로 되돌려 보냄. Go로 쓰였고 Apache-2.0, 별 9,098개.

## 기본 정보

| | |
|---|---|
| 저장소 | `Untrivial-ai/agent-orchestrator` |
| Stars / Forks | 9,098 / 1,323 |
| 언어 | Go |
| 라이선스 | Apache-2.0 |
| 생성 | 2026-02-13 |
| 플랫폼 | macOS(arm64/x64), Windows, Linux(AppImage/deb/rpm) |

![AO 대시보드](../assets/agent-orchestrator/dashboard.png)

## 무엇을 하는가

에이전트가 코딩하는 건 그대로임. AO는 그 주변 하네스를 담당함. 격리 워크스페이스, 라이브 터미널 접근, 세션 상태, PR 인식, 그리고 CI 실패랑 리뷰 코멘트, 머지 충돌을 알맞은 에이전트한테 되돌리는 자동 루프.

흐름은 단순함.

1. 에이전트가 작업할 프로젝트 등록
2. 데스크톱 앱이나 CLI에서 세션을 하나 이상 시작
3. AO가 세션마다 격리된 git worktree 생성
4. 세션이 고른 인터페이스에 따라 에이전트의 터미널 UI 또는 구조화된 Chat 컨트롤러를 띄움
5. 로컬 데몬이 세션 상태, 컨트롤러 활동, PR, CI, 리뷰 피드백을 감시
6. 앱이랑 CLI가 현재 상태를 보여주고 알맞은 세션에 후속 지시를 보냄

## 주요 기능

- 병렬 세션. 같은 프로젝트에서 여러 에이전트를 띄워도 파일, 브랜치, 터미널, PR 상태가 안 섞임
- 라이브 터미널 제어. 세션 요약이랑 PR 상태, 후속 액션을 보면서 워커 터미널에 붙음
- 리뷰 피드백 루프. 리뷰어 에이전트를 돌리고 요청된 변경사항을 담당 워커 세션으로 라우팅
- 인앱 브라우저 프리뷰. 세션의 로컬 앱을 터미널 옆에서 미리 봄

## 지원 에이전트

워커 하네스 25종의 어댑터를 제공함. claude-code, codex, aider, opencode, grok, droid, amp, auggie, autohand, agy, crush, cursor, qwen, copilot, goose, continue, devin, cline, kimi, muse, kiro, kilocode, vibe, pi, kimchi.

리뷰어 하네스는 별도로 설정하고 24종이 등록돼 있음. 여기 설계가 꽤 세밀해서 읽어볼 만함.

- 매 패스마다 새 리뷰어 프로세스를 열어서 그 하네스의 현재 태스크 컨텍스트랑 권한, 환경을 적용함
- Pi 리뷰어는 프로젝트/사용자 리소스 탐색이랑 내장 도구를 끈 채로 돌고, AO 자체 데이터 디렉터리에서 읽기 전용 체크아웃 검사와 구조화된 GitHub 리뷰 게시, `ao review submit`만 노출하는 확장을 로드함
- Agy, Continue, Devin, Droid, Goose, Kimchi, Kimi, Qwen, Vibe는 실험적 host-trusted 리뷰어임. 네이티브 모드가 OS 격리를 제공하지 않고, 일부는 리뷰를 끝내려고 자율 설정을 받음
- Grok, Crush, Auggie, Cline, Autohand는 실험적 user-approved 리뷰어임. 강화된 리뷰어 역할을 주되 네이티브 권한 프롬프트를 켠 채로 둠

## 설치

데스크톱 빌드 받아서 실행하고 관리할 저장소를 지정하면 끝임. 앱이 데몬을 대신 돌려서 CLI가 필요 없음.

npm 설치는 레거시임. `0.10.0`이 npm에 올라간 마지막 버전이고 `@aoagents/ao` 패키지는 동결됐음. 기존 사용자용으로 남아 있을 뿐이고 `ao start`도 결국 같은 데스크톱 빌드를 받아서 염.

## 다른 오케스트레이터랑 비교

[Paseo vs Orca](./2026-08-10-paseo-vs-orca-agent-orchestrators.md)와 같은 범주임. 셋을 갈라 보면:

| | Paseo | Orca | AO |
|---|---|---|---|
| 성격 | 얇은 컨트롤 플레인 | 풀스택 ADE | 감독 + 피드백 루프 |
| 언어 | TypeScript | TypeScript | Go |
| 라이선스 | AGPL-3.0 | MIT | Apache-2.0 |
| Stars | 12,968 | 40,844 | 9,098 |
| CI/리뷰 자동 라우팅 | 없음 | diff 주석 수동 | 자동 |
| 리뷰어 에이전트 분리 | 없음 | 없음 | 24종 별도 설정 |
| 모바일 | 있음 | 있음 | 없음 |
| 텔레메트리 | 없음 | 익명 수집 | 익명 수집 |

AO를 고를 상황은 명확함. PR을 많이 뽑아내는 팀에서 CI 실패랑 리뷰 코멘트를 사람이 일일이 에이전트한테 물어 나르는 게 병목일 때. 반대로 모바일이나 웹 접근, 통합 에디터가 필요하면 다른 쪽이 나음.

## 참고

- 문서: [architecture.md](https://github.com/Untrivial-ai/agent-orchestrator/blob/main/docs/architecture.md), [STATUS.md](https://github.com/Untrivial-ai/agent-orchestrator/blob/main/docs/STATUS.md)
- 익명 텔레메트리를 수집함. PII랑 프로젝트 내용은 제외한다고 명시. [telemetry.md](https://github.com/Untrivial-ai/agent-orchestrator/blob/main/docs/telemetry.md)
