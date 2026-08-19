---
title: Xcode에 에이전틱 코딩이 정식 탑재됨, 채팅에서 에이전트로
source: https://www.apple.com/newsroom/2026/02/xcode-26-point-3-unlocks-the-power-of-agentic-coding/
author: Apple
published: 2026-02-03
collected: 2026-08-19
tags: [apple, ios, xcode, claude-agent-sdk, codex, tooling]
---

출처: [Xcode 26.3 unlocks the power of agentic coding](https://www.apple.com/newsroom/2026/02/xcode-26-point-3-unlocks-the-power-of-agentic-coding/) (2026-02-03) · [Apple aids app development with new intelligence frameworks and advanced tools](https://www.apple.com/newsroom/2026/06/apple-aids-app-development-with-new-intelligence-frameworks-and-advanced-tools/) (2026-06-08, Xcode 27) · [Xcode 26.3 Brings Integrated Agentic Coding (InfoQ)](https://www.infoq.com/news/2026/02/xcode-26-3-agentic-coding/)

## 요약

Xcode의 AI 통합이 채팅에서 에이전트로 넘어감. 2025년 Xcode 26은 Claude Sonnet 4를 붙여 코드 질문·설명·디버깅을 돕는 **채팅 수준**이었음. 2026-02의 Xcode 26.3이 **Claude Agent SDK**, 즉 Claude Code를 돌리는 그 하네스를 네이티브 통합하면서 성격이 바뀜. 서브에이전트, 백그라운드 태스크, 플러그인이 IDE 안에서 그대로 돔.

2026-06 발표된 Xcode 27은 Anthropic·Google·OpenAI 에이전트를 전부 받고 인터랙티브 플래닝과 자율 코드 검증까지 확장함. 별도 터미널에 코딩 에이전트를 띄워 두고 Xcode와 왔다 갔다 하던 워크플로를 유지할 이유가 줄어듦.

## Xcode 26.3, 이미 쓸 수 있는 쪽

Claude Agent SDK 네이티브 통합. **Claude Code의 기능 축소판이 아니라 같은 하네스**라는 게 요지임. OpenAI Codex도 같은 자리에서 선택 가능함.

에이전트가 교체 가능한 부품이 된 구조인데, 이건 [Foundation Models가 모델 백엔드에 한 것](../industry/2026-06-08-apple-ai-platform-for-ios.md)과 같은 패턴임. 올해 Apple 발표를 관통하는 형태가 "프로토콜을 열고 구현체는 고르게 한다"임.

## Xcode 27, 발표된 쪽

| 항목 | 내용 |
|---|---|
| 에이전트 | Anthropic · Google · OpenAI 전부 |
| 인터랙티브 플래닝 | 실행 전 계획을 사람이 보고 고침 |
| 멀티턴 대화 | 세션이 이어짐 |
| 자율 코드 검증 | 에이전트가 자기 변경을 스스로 돌려 봄 |
| Device Hub | 기기 관리 통합 |
| 앱 자체 | 30% 작아짐, **Apple Silicon 전용** |

## 지금 쓸 수 있나

| | 상태 |
|---|---|
| Xcode 26.3 에이전틱 코딩 | **출시됨** |
| Xcode 27 | 베타 (2026 가을) |

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md) | 같은 발표의 프레임워크 쪽. 이 문서는 그 위에서 코드를 짜는 도구 이야기 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | Xcode 27의 "자율 코드 검증"이 에이전트가 자기 답을 채점하는 구조임 |
| [팀 공유 AI 하네스 만들기](2026-08-10-hq-team-ai-harness.md) | 터미널 하네스를 IDE로 옮길 때 스킬·훅·설정이 따라오는지가 남는 문제임 |

## 짚어야 할 것

- Xcode 27은 **Apple Silicon 전용**임. Intel Mac 빌드 머신이 남아 있으면 지금 계획에 넣어야 함
- "자율 코드 검증"은 통과시키는 실패가 정확히 중요한 실패인 구조임. 검증 주체를 에이전트 밖에 두는 장치가 따로 필요함
- 기존 터미널 하네스를 IDE로 옮길 때 그대로 따라오는지는 발표에서 다루지 않았고, 직접 확인하지 못했음

## 유효기간

2026-08-19 기준 스냅샷임. Xcode 27 정식 출시 시점에 지원 에이전트 목록과 하드웨어 요구사항을 재확인해야 함.
