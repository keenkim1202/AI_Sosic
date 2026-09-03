---
title: Xcode 27의 에이전트 표면, 플러그인·MCP·ACP
source:
  - https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes
  - https://www.apple.com/newsroom/2026/02/xcode-26-point-3-unlocks-the-power-of-agentic-coding/
author: Apple
collected: 2026-08-26
tags: [apple, xcode, coding-agent, mcp, acp, skills, plugins, agent-security]
---

출처: [Xcode 27 Beta 6 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes) (베타 1~6 항목 전체 확인, 2026-08-26) · [What's New in iOS](https://developer.apple.com/ios/whats-new/) · [Xcode 26.3 unlocks the power of agentic coding](https://www.apple.com/newsroom/2026/02/xcode-26-point-3-unlocks-the-power-of-agentic-coding/) (2026-02-03)

## 요약

2026-02의 Xcode 26.3이 에이전틱 코딩을 정식 탑재한 뒤 이 문서는 **27 베타 6까지 쌓인 릴리스 노트에서 실제로 무엇이 되는지** 확인한 것임. 요지는 셋임. 첫째, 터미널 하네스의 구조가 그대로 옮겨왔음. **플러그인이 skills·MCP 서버·ACP 에이전트 설정을 담고 스킬은 슬래시 커맨드로 호출**됨. 둘째, Xcode 자신이 **MCP 서버가 되어 빌드·디버거·스킴·빌드 설정·엔타이틀먼트를 툴로 노출**하고, 베타 5부터는 **워크스페이스를 안 열어도 돎**. 셋째, 그 대가로 **코드 서명된 에이전트에게 디렉터리 트리 권한을 장기간 주는 모델**이 들어왔고, 그래서 파일시스템 접근을 감시하는 보안 계층이 같이 붙었음.

## 앞 단계, Xcode 26.3에서 이미 되는 것

27은 아직 베타고, **오늘 쓸 수 있는 건 26.3**임. 둘을 섞어서 계획하면 안 됨.

| | 상태 |
|---|---|
| Xcode 26.3 에이전틱 코딩 | **출시됨** (2026-02) |
| Xcode 27 | 베타 6, 2026 가을 정식 |

26.3이 붙인 건 **Claude Agent SDK**, 즉 Claude Code를 돌리는 그 하네스임. 기능 축소판이 아니라 **같은 하네스**라는 게 요지고, 서브에이전트·백그라운드 태스크·플러그인이 IDE 안에서 그대로 돎. OpenAI Codex도 같은 자리에서 고름.

에이전트가 교체 가능한 부품이 된 구조인데, 이건 [Foundation Models가 모델 백엔드에 한 것](../industry/2026-06-08-apple-ai-platform-for-ios.md)과 같은 패턴임. 올해 Apple 발표를 관통하는 형태가 "프로토콜을 열고 구현체는 고르게 한다"임. 아래 27 항목들은 그 패턴이 어디까지 갔는지에 해당함.

## 플러그인, 하네스가 IDE로 들어옴

> "Agents in Xcode can now be extended with plugins that contain skills, MCP servers, and ACP agent configurations. Skills are invokable as slash commands with completion support." (178289210)

그리고 별도 항목으로 **"Xcode adds support for the Agent Client protocol."** (178294840)

플러그인 정의는 JSON이고 MCP 서버를 담음. `_meta`로 IDE에 표시될 아이콘과 툴 이름을 지정할 수 있음.

```json
{
  "name": "MyGreatPlugin",
  "description": "An awesome MCP server configuration.",
  "version": "1.0.0",
  "mcpServers": {
    "MyGreatMCP": {
      "type": "http",
      "url": "...",
      "tools": ["*"],
      "_meta": {
        "ideToolIconPath": "./icon.svg",
        "ideToolIconRendersAsTemplate": true,
        "ideToolTitles": {
          "whoami": "Who Am I",
          "get-current-email": "Get Current Email Message"
        }
      }
    }
  }
}
```

**Apple이 만든 specialist가 기본 탑재됨.** 로컬라이제이션, UIKit 리사이징, 접근성 같은 좁은 작업용임 (178289150). 에이전트 언어 번역을 위한 언어별 스타일 가이드도 추가됨 (183679292).

⚠️ **여기서 갈리는 게 하나 있음.** 베타 2 해결 항목에 "Apple-authored agent skills may not be available to Codex" (179171480)가 있음. Apple 제작 스킬이 특정 서드파티 에이전트에서 안 보이는 문제가 있었다는 뜻이고, **어느 에이전트에 무엇이 노출되는지가 균일하지 않을 수 있다**는 신호로 읽어야 함.

## Xcode가 노출하는 MCP 툴

Xcode MCP 서버가 에이전트에게 주는 것들.

| 영역 | 노출되는 것 |
|---|---|
| 디버깅 | 활성 run 상태 조작, 디버거 콘솔과 상호작용 및 내용 읽기 (176935844) |
| 빌드 구성 | 스킴·run destination 목록과 전환, **빌드 설정·컴파일러 플래그·엔타이틀먼트·Info.plist 키 조회 및 수정** (176935844) |
| 프리뷰 | `RenderPreview` / Preview Snapshot. 라이트·다크, 방향, 타입 크기 변형, **위젯 타임라인과 Live Activity 토글 상태**까지 렌더 (172961797). 플랫폼·기기 타입·OS 버전을 결과에 반환 (177076406), 프리뷰 표시 이름과 줄 번호도 반환 (182215198) |
| 로컬라이제이션 | **String Catalog 파일의 번역을 읽고 계획하고 편집**하는 툴 (176376425) |
| 실행 | `ExecuteSnippet` |
| 디버거 | **LLDB가 자체 MCP 서버(`lldb-mcp`)를 같이 배포**함 (176901842) |

빌드·테스트 MCP 툴이 외부 MCP 클라이언트에 정적 텍스트 대신 **동적 활동 상태**를 보냄 (181078347). 즉 Xcode 밖의 에이전트가 Xcode를 도구로 부리는 시나리오를 전제하고 있음.

**워크스페이스 없이 도는 MCP 서버**가 베타 5에 프리뷰로 들어옴.

```bash
sudo xcrun mcp-server enable
xcrun mcp-server status
```

## 에이전트가 앱을 직접 만져서 검증함

기존 코딩 에이전트와 갈리는 지점임. 코드를 고치는 데서 끝나지 않고 **실행해서 확인**함.

- 시뮬레이터를 부팅하고 앱을 설치·실행하고 **터치 이벤트를 합성하고 스크린샷을 찍어** UI 동작을 검증 (175179787)
- **watchOS**: Digital Crown 회전과 누름, side 버튼과 Action 버튼(Apple Watch Ultra) (181147968)
- **tvOS**: Siri Remote로 포커스 이동·선택·홈으로 가기, 포커스된 필드에 텍스트 입력 (183317784)
- 프로젝트 인사이트 접근. **크래시, 디스크 쓰기, 에너지, 행, 실행 문제** (177568662)

[자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)의 문제의식이 여기 그대로 걸림. 에이전트가 자기 변경을 자기가 검증하는 구조라, **통과시키는 실패가 정확히 중요한 실패**임. 시뮬레이터 스크린샷이 앵커가 되려면 기대 상태를 에이전트 밖에서 정의해야 함.

## 계획을 문서로 다루기

> "Planning with agents is now first class in Xcode. Plans appear as **editable Markdown artifacts** next to the conversation. You can use dedicated UI to review, annotate, discuss changes to the plan, and approve before the agent proceeds." (172857081)

코딩 어시스턴트가 내비게이터에서 **에디터 영역으로 이동**했고, 코드 diff·계획·SwiftUI 프리뷰 스냅샷 같은 산출물이 대화 옆에 나란히 붙음. 코드 스니펫과 계획 문서에 **인라인 주석**을 달아 맥락을 벗어나지 않고 피드백을 줌 (178288550). 사이드바는 대화 관리 전용이 됐고 실시간 상태·읽지 않음 표시·드래그앤드롭 그룹핑·아카이브를 지원함 (172926345).

[Human Review](./2026-08-10-human-review-skill.md)가 브라우저에서 하던 것을 Xcode가 IDE 안에서 하는 셈임.

⚠️ **알려진 버그가 하나 있음.** 에이전트가 아직 응답을 스트리밍하는 중에 계획 확인 바("Implement the plan?" Yes/No)가 뜨고, 그 상태에서 버튼을 누르면 진행 중인 턴 위에 새 턴이 얹혀 대화가 깨짐 (178673449). 회피법은 응답이 끝난 뒤 누르는 것.

## 권한 모델이 바뀐 지점

이 문서에서 가장 조심해서 볼 부분임.

워크스페이스 없이 도는 MCP 서버는 **코드 서명된 에이전트에게 디렉터리 트리 안의 프로젝트를 장기간 쓸 권한**을 주는 것을 함께 허용함. 매번 묻지 않는다는 뜻임. 그리고 무인 환경용으로 이런 플래그가 있음.

```bash
sudo xcrun mcp-server enable --unsafe-always-allow-all-agents
```

릴리스 노트가 직접 **"자리에 앉아 쓰는 용도로는 권장하지 않는 구성"**이라고 적음. 이름에 `unsafe`가 들어가 있는 것도 그래서임.

균형추로 들어온 게 이것임.

> "Coding Intelligence now includes a new security layer that monitors and controls **filesystem access by coding agents and any processes they spawn**. This can be enabled in Coding Intelligence settings." (178289431)

**에이전트가 띄운 자식 프로세스까지 감시 대상**이라는 게 핵심임. [OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](../security/2026-08-07-openai-hugging-face-agent-incident.md)에서 파일 쓰기 권한 하나로 에이전트끼리 채널을 만든 것과 정확히 같은 표면임. 그리고 [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md)의 결론, 즉 승인 정확도를 올리는 것보다 위험한 명령을 승인 대상에서 빼는 게 싸다는 판단이 이 설정 화면에서 그대로 필요함.

## 그 밖에

- **Foundation Models Instrument**: instructions, 프롬프트, 응답, 토큰 사용량, 추론 성능을 추적·디버그 (164223804). [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)의 컨텍스트 4K 문제를 실제로 계측하는 도구가 이것임
- 코딩 어시스턴트에 **Google Gemini**(171990272)에 이어 **Google Antigravity**(183074664)가 추가됨. 3사로 세는 것은 이제 맞지 않음
- 개발자 문서를 **자연어로 검색**함. 의미 기반 매칭 (165476491)
- Device Hub가 별도 앱으로 분리됨. 다만 **물리 Apple Vision Pro의 화면 전송과 입력 전달은 미지원**이고 두 손가락 터치 전송도 아직 안 됨 (142582218, 169537162)
- Game Porting Toolkit 4가 **Metal과 Apple 게임 개발 모범사례를 담은 오픈소스 에이전틱 코딩 스킬**을 제공함
- Xcode 27 beta 6은 **Swift 6.4**와 iOS·iPadOS·tvOS·watchOS·macOS·visionOS 27 SDK를 포함하고, **macOS Tahoe 26.4 이상**에서 돎

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md) | 같은 Apple 2026 발표의 프레임워크 쪽. 백엔드를 교체 가능하게 만드는 패턴이 에이전트에도 똑같이 적용됨 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | Xcode가 붙이는 규격. stateless 전환과 툴 목록 결정적 정렬이 여기 클라이언트에도 적용됨 |
| [팀 공유 AI 하네스 만들기](./2026-08-10-hq-team-ai-harness.md) | 플러그인·스킬을 팀 기본값으로 배포하는 문제. Xcode 플러그인이 그 배포 경로가 됨 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md) | 장기 권한 부여와 `--unsafe-always-allow-all-agents`가 만드는 표면 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | 에이전트가 자기 변경을 시뮬레이터로 검증하는 구조의 함정 |
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | Foundation Models Instrument가 그쪽 토큰 문제를 계측함 |

## 짚어야 할 것

- **Xcode 27은 Apple Silicon 전용임.** Intel 빌드 머신이 남아 있으면 지금 계획에 넣어야 함
- **릴리스 노트만 읽었고 직접 써보지 않았음.** 항목 번호는 Apple의 이슈 번호를 그대로 옮긴 것임
- 베타 6 기준이라 **정식 출시 때 빠지거나 바뀌는 항목이 있을 수 있음.** 특히 워크스페이스 없는 MCP 서버는 릴리스 노트 본인이 "early preview"라고 적었고, `xcrun mcp-server` 유틸리티가 모든 구성에서 동작하지 않을 수 있으며 설정 적용에 Xcode 재실행이나 재부팅이 필요할 수 있다고 명시함 (181836944)
- ACP(Agent Client Protocol) 자체의 명세는 확인하지 못했음. Xcode가 지원한다는 사실과 플러그인이 ACP 에이전트 설정을 담는다는 것까지만 확인함
- 보안 계층은 **기본값이 아니라 Coding Intelligence 설정에서 켜는 것**임. 켜지 않으면 감시가 없음

## 유효기간

**2026-08-26 확인 기준, Xcode 27 beta 6**임. iOS 27과 함께 2026 가을 정식 출시 예정이고 그 시점에 베타 항목을 다시 대조해야 함. 다시 볼 때는 릴리스 노트의 Coding Intelligence·Code Intelligence 절만 훑어도 대부분 잡힘.
