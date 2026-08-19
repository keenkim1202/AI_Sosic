---
title: Paseo vs Orca, 멀티 코딩 에이전트 오케스트레이터 비교
source:
  - https://github.com/getpaseo/paseo
  - https://github.com/stablyai/orca
collected: 2026-08-10
tags: [coding-agent, orchestration, ade, worktree, opensource, tooling]
---

출처: [getpaseo/paseo](https://github.com/getpaseo/paseo) · [stablyai/orca](https://github.com/stablyai/orca) (수치는 2026-08-10 GitHub API 기준)

## 요약

둘 다 여러 코딩 에이전트를 병렬로 굴리는 TypeScript OSS임. 차이는 야심의 범위. Paseo는 기존 IDE를 안 건드리는 얇은 오케스트레이션 레이어고, Orca는 에디터랑 터미널, 브라우저까지 삼킨 개발 환경 자체를 노림.

## 비교표

| | Paseo | Orca |
|---|---|---|
| Stars | 12,968 | 40,844 |
| 라이선스 | AGPL-3.0 | MIT |
| 저장소 생성 | 2025-10 | 2026-03 |
| 성격 | 로컬 데몬 + 다중 클라이언트 | 데스크톱 ADE |
| 지원 에이전트 | 5종 | 29종 + 모든 CLI 에이전트 |
| 웹 UI / Docker | 둘 다 있음 | 없음 |
| 모바일 | 정식 클라이언트 | 컴패니언 앱 |
| 원격 실행 | E2E 릴레이, TCP, Tailscale | SSH 워크트리 |
| 에디터 내장 | 없음 | VS Code |
| 터미널 내장 | 없음 | Ghostty급, WebGL |
| 브라우저 내장 | 없음 | Design Mode |
| 병렬 워크트리 | `--worktree` 플래그 | 1급 기능 (팬아웃, 비교, 머지) |
| 음성 입력 | 있음 | 없음 |
| GitHub / Linear | 없음 | 네이티브 |
| 텔레메트리 | 없음 | 익명 수집, opt-out 가능 |
| MCP 서버 | 있음 | README 미언급 |

## Paseo

_(스크린샷 생략. 원 저장소 README 참고)_

> "Run agents in parallel on your own machines. Ship from your phone or your desk."

중심은 daemon임. 로컬에서 에이전트 프로세스를 관리하고 데스크톱·모바일·웹·CLI는 전부 그 클라이언트일 뿐임. IDE를 대체할 생각이 아예 없고 컨트롤 플레인 역할에 집중함. 텔레메트리랑 트래킹, 강제 로그인이 전부 없다는 걸 전면에 내세움.

앱에서 되는 건 터미널에서도 다 된다는 게 명시적 설계 원칙임.

```bash
paseo run --provider claude/opus-4.6 "implement user authentication"
paseo ls                            # 실행 중인 에이전트
paseo attach abc123                 # 라이브 출력 스트리밍
paseo send abc123 "also add tests"  # 후속 지시
paseo --host workstation.local:6767 run "run the full test suite"
```

제일 차별적인 건 Skills임. `npx skills add getpaseo/paseo`로 깔면 에이전트 대화 안에서 다른 에이전트를 부릴 수 있음.

- `/paseo-handoff` 에이전트 간 인계. 예를 들어 Claude로 계획하고 Codex로 구현
- `/paseo-loop` 수용 기준에 대해 루프를 돌림. 이른바 Ralph loop. verifier 붙일 수 있음
- `/paseo-advisor` 작업을 안 넘기고 second opinion만 받음
- `/paseo-committee` 성향 다른 두 에이전트로 위원회를 꾸려서 근본원인 분석이랑 계획을 뽑음

## Orca

_(스크린샷 생략. 원 저장소 README 참고)_

> "Run Codex, ClaudeCode, OpenCode or Pi side-by-side — each in its own worktree, tracked in one place."

스스로를 ADE(Application Development Environment)라고 부름. IDE를 대체하겠다는 의도가 아주 분명함.

- Parallel Worktrees. 프롬프트 하나를 다섯 에이전트에 팬아웃하고 각자 격리된 워크트리에서 일 시킨 뒤 결과 비교해서 승자를 머지함
- Design Mode. Chromium에서 UI 요소를 클릭하면 그 HTML, CSS, 크롭 스크린샷이 프롬프트로 바로 들어감
- Terminal Splits. Ghostty급 WebGL 렌더링에 무한 분할, 재시작해도 스크롤백이 살아남음
- SSH Worktrees. 원격 고사양 박스에서 편집이랑 git, 터미널을 그대로 씀. 자동 재접속이랑 포트 포워딩 포함
- Annotate AI Diffs. diff 라인에 코멘트 달아서 에이전트한테 되돌려줌
- GitHub, Linear 네이티브. PR이랑 이슈, 보드를 인앱에서 보고 태스크에서 바로 워크트리를 염
- 계정 스위처랑 사용량 추적. Claude, Codex의 rate-limit 리셋을 확인하고 재로그인 없이 계정을 바꿈
- 그 외 Quick open, Computer Use, VS Code 에디터, 파일 드래그앤드롭, `orca` CLI

README에 "we ship daily, so this list is perpetually behind"라고 적혀 있음.

## 어떤 걸 언제 쓸까

Paseo 쪽이 맞는 경우:

- Cursor나 VS Code, Neovim을 그대로 쓰고 싶을 때
- 웹 UI나 Docker 셀프호스팅이 필요할 때. Orca에는 둘 다 없음
- 팀 서버나 VPS에 올려서 브라우저로 접근하고 싶을 때
- 텔레메트리 0을 요구하는 보안 정책이 있을 때
- 음성으로 지시하고 싶을 때
- 에이전트가 에이전트를 부리는 패턴을 쓰고 싶을 때
- CLI랑 스크립트 중심으로 일할 때

Orca 쪽이 맞는 경우:

- 에이전트 병렬 실행이 주 워크플로일 때
- 개발 환경을 한 앱으로 통합하고 싶을 때
- 프론트엔드 작업 비중이 높을 때. Design Mode는 Orca에만 있음
- GitHub나 Linear 기반으로 일할 때
- AI가 만든 diff를 꼼꼼히 리뷰해야 할 때
- 원격 고사양 머신에서 돌릴 때
- Claude Code 말고도 온갖 에이전트를 쓸 때
- AGPL이 부담스러워서 MIT가 필요할 때

간단한 판단 순서:

```
에디터·터미널까지 통합된 환경을 원함?     YES → Orca
웹이나 Docker로 서버에 띄워야 함?         YES → Paseo
같은 일을 여러 에이전트에 시키고 비교?    YES → Orca
텔레메트리 0 또는 음성 입력이 중요?       YES → Paseo
나머지: 프론트엔드 비중 높으면 Orca, 백엔드·CLI 중심이면 Paseo
```

둘 다 기존 에이전트 CLI 구독을 그대로 쓰고 로컬 git 위에서 도니까 병행해도 됨. 데스크톱은 Orca로 쓰고 원격 VPS에는 Paseo를 Docker로 올려서 웹으로 접근하는 조합이 가능함.

## 주의할 점

- Paseo 라이선스가 애매함. README에는 AGPL-3.0인데 GitHub API는 `NOASSERTION`을 반환함. 상업적으로 도입할 거면 LICENSE 파일을 직접 확인해야 함
- 두 프로젝트 다 변화가 빠름. 위 비교는 2026-08-10 스냅샷이고, 없다고 적은 기능이 이미 생겼을 수 있음
- 별 수는 인기 지표지 적합도 지표가 아님. Orca는 5개월 만에 40.8k를 찍었음

## 관련 문서

- [Prompt Caching In Agents](./2026-07-22-prompt-caching-in-agents.md): 에이전트를 병렬로 굴릴 때 실제 비용이 어디서 나오는지
- [Agent Orchestrator (AO)](./2026-08-10-agent-orchestrator-ao.md): 같은 범주의 세 번째 선택지
