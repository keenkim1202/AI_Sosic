---
title: macOS 27.3 beta 2 의 권한 쪼개기, FDA 예고가 폴더 단위 승인과 엔타이틀먼트로 내려옴
source: https://developer.apple.com/news/?id=oqng0zjk
author: Apple
published: 2026-11-16
collected: 2026-11-17
tags: [apple, macos, agent-security, permissions, privacy, coding-agent]
---

출처: [Updates to Device Control and Data Access, and Full Disk Access in macOS](https://developer.apple.com/news/?id=oqng0zjk) (2026-11-16) · [macOS 27.3 Golden Gate Beta 2 Release Notes](https://developer.apple.com/documentation/macos-release-notes/macos-27_3-release-notes) · [Releases](https://developer.apple.com/news/releases/) (전부 **2026-11-17 확인**. 릴리스 노트는 자바스크립트로 렌더링되므로 같은 경로의 `/tutorials/data/documentation/...json` 으로 본문을 받았음)

## 요약

[macOS Full Disk Access 제한 예고](./2026-10-02-macos-full-disk-access-controls.md)가 "통제의 형태와 적용 버전이 전부 안 적혀 있음"으로 남겨 둔 칸이 채워졌음. **2026-11-16 macOS 27.3 beta 2 에 들어갔고**, 같은 문언이 개발자 공지와 릴리스 노트의 Privacy & Security 절(192585326)에 동시에 올라왔음. 바뀌는 것은 여섯이고 핵심은 둘임. **Device Control and Data Access 가 System & App Protection Override 와 Accessibility 두 설정으로 쪼개졌고**, **Full Disk Access 는 워크플로에 필요한 폴더만 주도록 유도하는 형태로 좁혀졌음.** 거기에 부여 방법이 목록에서 앱을 고르는 하나로 통일되고, **매번 묻기** 옵션과 **앱이 다른 앱을 제어하는 중임을 알리는 투명성 표시기**가 붙음.

개발자 쪽 결론은 한 줄임. **일부 폴더나 일부 앱에만 접근이 허용된 상태를 정상 경로로 다뤄야 함.** 기존에 사용자가 준 권한은 그대로 유지되고, 디스크 백업용 FDA 와 보조기술용 Accessibility 는 **엔타이틀먼트를 요청해 27.3 이전과 같은 신규 설치 경험을 유지할 수 있다**고 적음. 다만 엔타이틀먼트 세부는 "available soon" 이고, "known apps" 는 당장 영향이 없다고만 적혀 있음.

**10월 공지와 가장 다른 점은 AI 가 한 번도 안 나온다는 것임.** 10-02 공지는 근거로 AI 에이전트를 직접 댔는데, 형태가 내려온 이번 문언에는 에이전트도 자동화도 안 나옴. 그런데 쪼개진 두 설정이 가리키는 동작은 **다른 앱을 제어하는 것**과 **기기의 모든 파일을 읽는 것**이고, 그것이 컴퓨터를 조작하는 에이전트가 Mac 에서 쓰는 표면 그대로임. 같은 릴리스 노트의 Testing 절에서 **`automationmodetool` 이 표준 입력으로 관리자 암호를 더 받지 않게 된 것**(173706149)을 같이 놓으면 방향이 더 분명해짐.

## 10월 예고와 대조

그 문서가 "안 적힌 것" 으로 세어 둔 항목에 이번 문언을 그대로 맞춰 본 것임.

| 10월에 비어 있던 칸 | 2026-11-16 에 채워진 것 |
|---|---|
| 적용 OS 버전과 시점 | **macOS 27.3 beta 2**. 사용자는 "when approving new app permission requests" 시점에 보게 됨 |
| "additional controls" 의 형태 | 설정 분리, 부여 방법 단일화, 범위 축소 유도, 범위 가시화, 매번 묻기, 제어 중 표시기. **UI 변경과 엔타이틀먼트가 같이** 들어옴 |
| 기존 부여분의 취급 | "All app permissions previously granted by users will remain in effect" |
| 문제가 된 사용 방식의 구체적인 예 | **여전히 없음** |
| 범위가 Xcode 의 코딩 에이전트나 `xcrun mcp-server` 까지 닿는지 | **여전히 안 적힘** |

10월 문서가 "설계 전제를 바꾸는 것까지가 지금 할 수 있는 것" 으로 멈춰 둔 상태는 이제 끝났음. 베타 2 에 들어간 변경이므로 **온보딩 코드를 고치는 단계**로 넘어감.

## 쪼개진 것과 좁혀진 것

공지와 릴리스 노트가 같은 목록을 씀. 여섯 항목 그대로임.

> "A split of Device Control and Data Access into two settings: (1) System & App Protection Override, and (2) Accessibility"
> "A single method for granting permissions by selecting the requesting app from a list"
> "Guidance to encourage users to grant access to just the specific apps (for System & App Protection Override) or folders (for Full Disk Access) that are required for their workflows"
> "Increased visibility into the wide range of data apps will receive access to or control over if full permissions are granted"
> "The option for users to configure apps to ask every time they require access"
> "A transparency indicator when a requesting app is controlling another app using System & App Protection Override"

읽을 때 갈라야 하는 지점이 셋임.

- **쪼개짐의 축은 "다른 앱을 제어하는 것" 대 "보조기술"임.** 하나의 권한이 두 목적을 함께 덮고 있었고, 이제 목적별로 다른 설정에서 승인됨. 보조기술로 쓰는 앱과 다른 앱을 조종하는 앱이 같은 스위치를 공유하지 않게 되는 것이 요지로 읽힘
- **FDA 는 폴더 단위로 내려갔음.** 다만 문언은 "guidance to encourage" 이고 강제가 아님. 사용자가 전체를 줄 수 있는 경로가 남아 있다는 뜻으로 읽는 게 맞음
- **매번 묻기는 사용자가 켜는 옵션임.** 앱이 거는 것도 Apple 이 기본값으로 켜는 것도 아님. 그래서 앱 입장에서는 **같은 세션 안에서 접근이 거부될 수 있는 상태**가 새로 생김

이 권한이 27.2 까지 시스템 설정 화면에서 정확히 어떤 이름으로 보였는지는 두 출처에 없어 **확인할 수 없었음.** 공지는 이것을 "two of the APIs" 중 하나로만 부르고, 쪼개진 뒤의 이름 둘만 줌.

## 제어하는 쪽에 걸린 알려진 문제 둘

릴리스 노트의 Known Issues 에 이번 변경과 직접 붙은 항목이 둘 있음. 공지에는 없고 노트에만 있음.

| 항목 | 내용 |
|---|---|
| 192510863 | 새 System & App Protection Override 로 다른 앱 제어 권한을 받은 앱이 **Safari 와 WebKit 을 쓰는 다른 앱과의 상호작용에서** 문제를 겪을 수 있음 |
| 192728601 | **CoreGraphics API 로 마우스·키보드 입력 이벤트를 주입하는 앱**이 live activity 를 제대로 표시하지 못할 수 있음. **Accessibility API 를 쓰는 앱과 listening event tap 을 설치하는 앱은 해당 없음** |

둘째 항목이 경계선을 보여 줌. **같은 "앱을 조작한다" 는 동작이 입력 이벤트를 어느 API 로 넣느냐에 따라 다른 쪽으로 분류되고 있음.** Mac 을 화면과 입력으로 조작하는 구현은 보통 합성 입력 이벤트를 쓰는데, 그 경로가 이번 분리에서 Accessibility 쪽이 아닌 자리에 놓인 것으로 읽힘. 다만 **노트가 적은 증상은 live activity 표시 문제뿐**이고 입력 주입 자체가 막힌다고는 적지 않았음. 거기까지 넘겨 읽으면 원문에 없는 말이 됨.

## 무인 자동화 쪽에서 닫힌 문 하나

같은 노트의 Testing 절 신규 항목임.

> "`automationmodetool enable-automationmode-without-authentication` no longer accepts administrator passwords from standard input. Instead, it always prompts for authentication in the UI with biometrics or a password. To enable this mode, log in to the macOS administrator account through the UI and run the command at least once. The setting persists across restarts." (173706149)

이것이 실무에서 걸리는 자리는 **CI 머신과 에이전트용 Mac 을 스크립트로 프로비저닝하는 흐름**임. 암호를 파이프로 넣어 자동화 모드를 켜던 경로가 막히고, **UI 로그인과 1회 수동 실행**이 선행 조건이 됨. 설정 자체는 재부팅을 넘어 유지되므로 한 번만 하면 되는 형태이기는 함.

권한 분리와 방향이 같음. **사람의 승인을 거치는 횟수를 줄이는 대신 그 승인이 반드시 사람 손을 통과하게 만드는 쪽**임. 변경 전 동작이 정확히 어땠는지는 이 문언에서 역추론한 것이고 이전 매뉴얼은 확인하지 않았음.

## 앱을 내는 쪽에서 지금 할 것

1. **부분 접근을 예외가 아니라 기본값으로 다룸.** 공지 문언이 "gracefully handle having access to only a limited set of folders or apps" 임. 폴더 한두 개만 받은 상태에서 기능이 어디까지 되는지가 설계 질문이 됨
2. **온보딩을 고침.** "설정에서 전체 디스크 접근을 켜 주세요" 한 줄로 끝나는 흐름은 깨짐. 부여 방법이 목록에서 앱을 고르는 하나로 통일됐으므로 안내 문구와 스크린샷도 같이 낡음
3. **백업이나 보조기술이면 엔타이틀먼트 경로를 전제로 계획함.** 다만 세부가 공개되지 않았으므로 지금 할 수 있는 것은 어느 기능이 그 경로에 의존하는지 목록화하는 것까지임
4. **매번 묻기가 켜진 상태를 테스트 케이스로 넣음.** 접근이 중간에 거부되는 경로가 코드에 없으면 그 상태에서 조용히 잘못된 결과를 내게 됨
5. **CI 가 `automationmodetool` 에 의존하면 이미지 프로비저닝을 손봄.** 암호를 넣는 단계가 사람 1회 실행으로 바뀜

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [macOS Full Disk Access 제한 예고](./2026-10-02-macos-full-disk-access-controls.md) | 같은 변경의 예고판. 그 문서가 재확인 항목으로 남긴 셋 중 둘(형태, OS 버전)이 이 문서에서 닫히고 하나(Xcode 코딩 에이전트까지 닿는지)는 여전히 열려 있음 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 코드 서명된 에이전트에게 디렉터리 트리 권한을 장기간 주는 모델. OS 권한 쪽이 폴더 단위로 좁아지는 동안 IDE 쪽은 그대로라, 같은 기기에서 두 기준선이 계속 어긋나 있음 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 승인 정확도를 올리는 대신 승인 대상을 줄인다는 결론. 폴더 단위 부여가 그 결론을 OS 설정에서 실행한 형태임 |
| [Anthropic 의도하지 않은 모델 행동 보고서](./2026-10-09-unintended-model-actions.md) | 막히면 우회로를 찾는 끈질김. 매번 묻기가 켜진 환경에서 에이전트가 거부를 받았을 때 어떤 경로를 시도하는지가 그 보고서의 범주와 겹침 |
| [작업 디렉터리 밖 쓰기 차단](./2026-10-30-working-directory-write-block.md) | 명령 문자열을 해석하지 않고 경로가 울타리 안인지만 보는 통제. FDA 를 폴더 단위로 내리는 것이 OS 층의 같은 패턴임 |
| [Claude를 제품마다 어떻게 격리하는가](./2026-05-25-containing-claude-across-products.md) | 승인 프롬프트를 샌드박스로 바꾸는 방향과 매번 묻기를 옵션으로 두는 방향의 차이. 후자는 프롬프트를 남겨 두는 선택임 |

## 짚어야 할 것

- **베타 2 항목임.** 설정 이름과 엔타이틀먼트 조건이 27.3 정식까지 바뀔 수 있음. 이 문서가 확인한 것은 2026-11-16 beta 2 문언까지임
- **AI 나 에이전트가 이번 문언에 한 번도 안 나옴.** 10-02 공지와 달라진 점이고, 에이전트와 연결한 것은 **이 문서의 판단**임. Apple 이 그렇게 적은 것이 아님
- **"known apps" 의 정의가 없음.** 어떤 기준으로 아는 앱인지, 그 상태가 언제까지 유지되는지는 "currently" 라는 단어 말고는 확인할 수 없었음
- **엔타이틀먼트의 이름과 심사 기준이 공개되지 않았음.** "Additional entitlement details will be available soon" 까지임. 요청 경로가 기존 엔타이틀먼트 요청 양식과 같은지도 확인하지 못했음
- **Xcode 의 코딩 에이전트가 이 변경에 걸리는지는 두 출처 어디에도 없음.** 10월 문서가 같은 항목을 열어 뒀고 이번에도 닫히지 않았음
- **`automationmodetool` 의 변경 전 동작은 노트 문언에서 역추론한 것임.** 이전 매뉴얼 페이지는 확인하지 않았음
- **원문에 지시문이나 명령문 형태의 문장은 개발자 권고뿐임.** 읽는 쪽의 행동을 바꾸려 심어 둔 문장은 없었음
- 이 변경은 macOS 에 대한 것임. **iOS 에 대응물이 있다는 서술은 공지에도 노트에도 없음**

## 유효기간

**2026-11-17 확인 기준, macOS 27.3 Golden Gate beta 2**임. 공지는 날짜가 박힌 기록이라 문언은 낡지 않고, 낡는 것은 "엔타이틀먼트 세부가 아직 없다" 는 이 문서의 상태임. 다시 볼 때 확인할 것 셋. **엔타이틀먼트 이름과 요청 경로가 공개됐는지**, **Known Issues 둘(192510863, 192728601)이 27.3 정식 노트에서 사라졌는지**, 그리고 **두 설정의 이름이 정식까지 유지됐는지**. 출발점은 [개발자 뉴스](https://developer.apple.com/news/)와 macOS 27.3 릴리스 노트의 Privacy & Security 절임.
