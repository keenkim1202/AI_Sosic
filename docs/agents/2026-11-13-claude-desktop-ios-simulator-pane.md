---
title: Claude Code Desktop 의 iOS Simulator 창, 동의 단위가 세션이 아니라 기기로 바뀜
source: https://code.claude.com/docs/en/desktop-ios-simulator
author: Anthropic
collected: 2026-11-13
tags: [claude-code, ios, xcode, coding-agent, permissions, testing, agent-security]
---

출처: [Test iOS apps in the simulator](https://code.claude.com/docs/en/desktop-ios-simulator) · [Let Claude use your computer from the CLI](https://code.claude.com/docs/en/computer-use) · [Claude Code Desktop](https://code.claude.com/docs/en/desktop) (managed settings 표와 pane 서술) (셋 다 **2026-11-13 확인**)

## 요약

Claude Code Desktop 이 **iOS Simulator 를 대화 옆 창에 띄우고 에이전트가 그 기기를 직접 몰게 하는 경로**를 공개 베타로 열어둠. macOS 전용이고 **Claude Desktop v1.24012.0 이상**, Xcode 와 iOS 플랫폼이 깔려 있어야 함. 실무에서 값어치가 큰 것은 기능이 아니라 **권한 모델 둘**임. 첫째, 이 창은 시뮬레이터 도구를 직접 부르므로 **computer use 가 필요 없고 macOS 손쉬운 사용·화면 기록 권한도 요구하지 않음.** CLI 에서 같은 일을 시키면 computer use 로 가고 그쪽은 두 권한을 다 받아야 함. 둘째, 기기 제어 동의가 **세션당이 아니라 기기당 1회**이고, 한 번 허용하면 탭·입력·스크린샷이 프롬프트 없이 돎. 그 대가로 문서가 직접 적는 경고가 이것임. **스크린샷은 Anthropic 으로 전송되고 일반 대화 보관 설정을 따르므로, Claude 가 쓰는 기기에 실제 계정으로 로그인하지 말 것.** 그리고 **빌드·설치·실행·URL 열기 세 가지는 동의 뒤에도 권한 모드를 따름.**

## 기본 정보

| 항목 | 값 |
|---|---|
| 상태 | **공개 베타.** Claude Code Desktop, macOS 한정 |
| 플랜 | Pro, Max, Ultra, Team, Enterprise. **HIPAA 구성이 켜진 Enterprise 조직은 제외** |
| 서드파티 프로바이더 | **사용 불가** |
| 요구 사항 | Claude Desktop **v1.24012.0** 이상, Mac, Xcode + iOS 플랫폼 |
| 권장 Xcode | **26.x 또는 27.** 둘 다 동작함 |
| 세션 종류 | **로컬 세션만.** cloud·SSH 세션은 내 Mac 의 시뮬레이터에 닿지 못함 |
| 대상 기기 | 시뮬레이터만. **실물 iPhone·iPad 제어는 불가** |

Xcode 가 여럿 깔려 있으면 `xcode-select` 가 가리키는 것을 씀. 바꾸려면 `sudo xcode-select -s /Applications/Xcode-26.4.app` 한 뒤 **Desktop 을 종료하고 다시 열어야** 적용됨. 현재 선택은 `xcode-select -p` 로 확인함. 시뮬레이터가 하나도 없으면 `xcodebuild -downloadPlatform iOS` 를 안내함.

## 창을 여는 조작이 없음

켜는 설정도 슬래시 커맨드도 없음. **Claude 가 앱을 시뮬레이터에서 돌리면 창이 열림.** 문서가 드는 프롬프트 모양이 "Build the app and run it in the simulator to check the onboarding flow." 임. 기기를 고르려면 프롬프트에 이름을 적음("run it on the iPhone SE simulator").

사람이 직접 열 수도 있음. 세션에 시뮬레이터가 붙어 있거나 Claude Code 가 iOS 앱 작업을 감지하면 제목줄에 **iOS Simulator** 버튼이 생기고, **Attach simulator** 또는 기기 메뉴로 붙임. 종료 상태 기기를 고르면 부팅됨.

창은 뷰어가 아니라 조작 가능한 기기임. Apple Simulator 앱과 같은 단축키를 씀.

| 조작 | 키 |
|---|---|
| 홈 | **Cmd+Shift+H** |
| 잠금 | **Cmd+L** |
| 볼륨 | **Cmd+Up / Cmd+Down** |
| 시계방향 90도 회전 | **Cmd+Right** |
| 스크린샷 / 화면 녹화 | **Cmd+S / Cmd+R** (파일은 데스크톱에 저장됨) |

**Display** 메뉴의 프레임레이트·해상도는 **스트림 표시만 바꾸고 앱이 도는 방식은 바꾸지 않음.** Mac 이 버거우면 내리는 쪽으로 쓰는 값임.

## 동의가 기기에 붙는다는 것

이 문서에서 제일 조심해서 볼 절임. 처음 쓰는 기기에서 Desktop 이 허용을 물음. 그 동의가 덮는 범위가 **그 기기의 제어와 스크린샷**이고, **세션당이 아니라 기기당 1회**임.

> "Claude's screenshots of the device are sent to Anthropic and kept under your normal conversation retention settings, so don't sign in to real accounts on a device Claude uses."

iOS 개발자에게 이 한 줄이 실무 규칙을 하나 만듦. **로그인·결제·푸시 플로를 에이전트에게 검증시키려면 그 기기는 더미 계정 전용 기기여야 함.** 온보딩을 보라고 시킨 다음 사람이 자기 Apple 계정으로 들어가 이어서 테스트하면, 그 화면도 같은 대화 보관 설정 안으로 들어감.

허용을 거절해도 창은 쓸 수 있음. **Attach simulator → Continue → 다음 대화상자에서 Cancel** 을 누르면 기기는 붙고 사람의 탭만 동작하며 Claude 접근은 꺼진 상태로 남음. 나중에 **Let Claude use it** 으로 켬.

[사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md)가 내린 결론, 즉 승인 정확도를 올리는 것보다 위험한 동작을 승인 대상에서 빼는 게 싸다는 판단과 방향이 같음. [Claude를 제품마다 어떻게 격리하는가](../security/2026-05-25-containing-claude-across-products.md)의 **승인률 93%** 수치가 가리킨 곳도 여기임. 다만 프롬프트를 없앤 대가로 **신뢰 단위가 기기로 옮겨갔음.** 기기를 재사용하면 동의도 재사용되므로, 관리할 것이 승인 버튼이 아니라 시뮬레이터 목록이 됨.

## 동의 뒤에도 묻는 것 셋

허용한 기기에서도 권한 모드를 따르는 동작이 셋 있고, 이유가 각각 다름.

| 동작 | 문서가 드는 이유 |
|---|---|
| 기기에서 **URL 열기** | URL 이 데이터를 기기 밖으로 실어낼 수 있음. 딥링크 테스트나 Safari 로드가 여기 걸림 |
| 앱 **빌드** | `xcodebuild` 가 **내 Mac 에서 프로젝트의 빌드 스크립트를 돌림.** 진행 중인 빌드를 확인하는 것은 안 물음 |
| 앱 **설치·실행** | Claude 가 지금 만드는 앱만이 아니라 **이름을 적은 아무 앱이나** 실행할 수 있음 |

여기서 하나 짚어야 함. 셋 중 임의 코드 실행에 가장 가까운 것은 **빌드**이고, 동시에 루프를 돌리는 사람이 가장 먼저 자동 승인으로 넘길 동작도 빌드임. 고치고 빌드하고 확인하는 사이클에서 매번 묻는 프롬프트가 제일 거슬리기 때문임. 즉 **제일 중요한 프롬프트가 제일 먼저 꺼질 자리**에 있음. 빌드 스크립트가 있는 프로젝트라면 이 창을 켜는 것과 `xcodebuild` 를 상시 허용하는 것을 같은 결정으로 보는 편이 맞음.

## 기기를 공유한다는 함정

창은 사람과 Claude 가 **같은 기기**를 몰음. 사람의 탭이 Claude 가 보는 앱 상태를 바꿈. Claude 가 몰고 있을 때는 화면 위에 **Claude is using this device** 배지가 뜨고, 문서는 배지가 사라질 때까지 탭을 참으라고 적음. 특정 화면을 확인시키려면 사람이 그 화면까지 탭해 두고 물어보는 순서가 맞음.

세션 사이에서도 같은 문제가 생김.

- 프롬프트가 시뮬레이터를 지정하지 않으면 **이미 떠 있는 기기를 씀. 다른 병렬 세션이 띄운 것이어도 그럼**
- 세션마다 기기를 분리하려면 **각 세션 프롬프트에 다른 기기 이름을 적는 것이 유일한 방법**으로 적혀 있음
- Desktop 이 띄운 기기는 앱 종료 시, 세션 아카이브 시, **detach 후 10분**에 자동 종료됨
- 사람이 Simulator 앱이나 Device Hub 에서 띄운 기기는 **자동 종료되지 않음**

병렬 세션으로 검증 루프를 돌릴 계획이라면 이 네 줄이 설계 제약임. 기기 이름을 세션별로 박아두지 않으면 **A 세션의 탭이 B 세션의 검증 결과를 바꾼다.**

## CLI 와 Xcode 와 이 창, 경로가 셋임

같은 "에이전트가 시뮬레이터를 몬다"를 세 경로가 하고 있고 권한 표면이 전부 다름.

| 경로 | 무엇으로 미는가 | 받아야 하는 권한 |
|---|---|---|
| **Desktop iOS Simulator 창** | 시뮬레이터 도구를 직접 호출 | 기기당 동의 1회. **손쉬운 사용·화면 기록 불필요** |
| **CLI** | computer use (`/mcp` 에서 `computer-use` 활성화) | macOS **손쉬운 사용 + 화면 기록**, 세션당 앱 승인 |
| **Xcode 27 자체 에이전트** | Xcode 의 Coding Intelligence·MCP | [코드 서명된 에이전트에 디렉터리 트리 권한](./2026-08-26-xcode-27-agent-surface.md) |

CLI 쪽은 화면을 실제로 점거함. 다른 앱이 숨겨지고, 스크린샷이 다운스케일돼 올라가고, 한 번에 **한 세션만** 락을 잡음. Desktop 창은 그 반대로 **화면을 가져가지 않고 다른 창을 숨기지도 않음.** Desktop 문서도 툴 선택 순서에서 "iOS 앱을 돌리거나 테스트하는 작업이면 화면 제어를 쓰지 않는 iOS Simulator 창을 쓴다"고 적어둠.

여기서 [macOS Full Disk Access 제한 예고](../security/2026-10-02-macos-full-disk-access-controls.md)와 묘하게 어긋나는 지점이 생김. Apple 은 **AI 에이전트를 근거로 들어 OS 권한을 좁히는 중**인데, 이 경로는 OS 권한 화면을 아예 지나가지 않음. 시뮬레이터만 건드리므로 낮은 문턱이 부당하다고 할 수는 없음. 다만 **조직이 macOS 개인정보 보호 설정만 감사하면 이 경로는 보이지 않는다**는 것은 사실이고, 그래서 통제가 아래 managed settings 쪽에만 있음.

## 조직 차원 차단 둘

| 키 | 효과 |
|---|---|
| `disableMobileSimulatorTools` | **Claude 의 기기 제어·캡처 툴만** 차단. 창은 사람의 탭용으로 남고 앱 안에서 덮어쓸 수 없음. 값은 **JSON 불리언 `true`** 여야 하고 문자열 `"true"` 는 무시됨 |
| `requireCoworkFullVmSandbox` | Claude 의 툴을 격리된 VM 에서 돌리는 정책. 켜면 **창과 시뮬레이터 툴이 통째로 비활성**되고 기기를 붙일 수 없음 |

둘 중 어느 쪽이 걸렸는지는 Claude 가 알려준다고 적혀 있음. 문자열 `"true"` 가 무시된다는 단서는 managed settings 표가 같은 문장을 `disableBrowserExternalNavigation`·`disableRemotePreviewScripts` 에도 붙여둔 것으로 보아 **이 키 계열의 공통 함정**임. MDM 으로 내릴 때 따옴표 하나로 차단이 조용히 풀리는 자리라서, 켠 뒤에 실제로 막혔는지 확인하는 편이 맞음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md) | Xcode 자체 에이전트도 시뮬레이터를 부팅하고 앱을 설치·실행하고 터치 이벤트를 합성해 스크린샷으로 검증함. 같은 일을 하는 다른 경로이고 권한 모델이 다름 |
| [Xcode 27.2 RC, 에이전트가 커버리지를 읽게 됨](./2026-10-19-xcode-27-2-rc-coverage-mcp.md) | 스크린샷 검증과 커버리지 수치 검증의 대비. 이 창은 스크린샷 쪽을 싸게 만듦 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | 에이전트가 자기 변경을 자기가 확인하는 구조. 확인이 쉬워지면 함정도 같이 깊어짐 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md) | 기기당 1회 동의로 프롬프트를 없앤 설계의 근거와 대가 |
| [Claude를 제품마다 어떻게 격리하는가](../security/2026-05-25-containing-claude-across-products.md) | 승인률 93%와 샌드박스 우선 원칙. `requireCoworkFullVmSandbox` 가 그 VM 격리 쪽 키임 |
| [macOS Full Disk Access 제한 예고](../security/2026-10-02-macos-full-disk-access-controls.md) | OS 권한을 좁히는 흐름과, OS 권한을 지나가는 경로가 같은 시점에 생긴 것 |
| [App Intents Testing](../practices/2026-10-05-app-intents-testing-and-onscreen-context.md) | iOS 27 이 더한 코드 기반 검증 경로. 이쪽이 수치로 확인하는 자리면 시뮬레이터 창은 눈으로 확인하는 자리임 |

## 짚어야 할 것

- **문서만 읽었고 직접 돌려보지 않았음.** 단축키와 동작 서술은 공식 문서 문언을 옮긴 것임
- **페이지에 발행일이 없어서 이 문서의 날짜는 수집일임.** Claude Code CLI 의 CHANGELOG 에는 simulator 라는 단어가 한 번도 안 나옴(2026-11-13 확인). Desktop 앱 쪽 기능이라 CLI 릴리스 노트로는 시점을 잡을 수 없고, **이 창이 언제 들어왔는지는 확인할 수 없었음**
- **Desktop 버전 번호를 기능 간에 비교할 수 없음.** 이 창은 `v1.24012.0` 이상인데 같은 managed settings 표가 다른 키에 `v5.1231.0`, `v3.6556.0`, `v1.37937.0`, `v1.49585.0` 을 적고 있음. 어느 체계가 더 최신인지는 **확인할 수 없었음.** 요구 버전은 숫자로 비교하지 말고 **Claude > Check for Updates…** 로 올리는 쪽이 맞음
- **`requireCoworkFullVmSandbox` 는 managed settings 표에 없음.** 시뮬레이터 페이지와 Cowork 관련 한 줄에만 나오고, **타입·허용값·적용 범위는 확인할 수 없었음**
- **스크린샷을 보관에서 빼는 설정이 있는지 확인할 수 없었음.** 문서가 적는 것은 "일반 대화 보관 설정을 따른다"까지임. 그래서 완화책이 설정이 아니라 **기기를 분리하는 운영**이 됨
- **플랜 조건이 CLI 쪽과 다름.** 이 창은 Team·Enterprise 에서도 쓸 수 있는데(HIPAA 구성 제외) CLI 의 computer use 는 Pro·Max·Ultra 전용이고 Team·Enterprise 에서 안 됨. 같은 "시뮬레이터를 몬다"가 플랜에 따라 **가능한 경로가 갈림**
- 베타 표기는 **공개 베타(public beta)** 이고 리서치 프리뷰가 아님. CLI 의 computer use 는 리서치 프리뷰임

## 유효기간

**2026-11-13 확인 기준이고 공개 베타 시점**임. 요구 버전 `v1.24012.0` 은 읽는 시점에 이미 낡았다고 보는 게 맞음. 다시 볼 때 확인할 것 셋. **창이 베타를 벗었는지**, **`requireCoworkFullVmSandbox` 가 managed settings 표에 항목으로 올라와 타입과 적용 범위가 적혔는지**(올라오면 위 미확인 하나가 풀림), 그리고 **실물 기기 지원이 생겼는지**. 셋째가 가장 큰 변경이 될 자리인데, 실물 기기는 스크린샷의 성격이 통째로 달라지므로 동의 모델도 같이 바뀌어야 함. 당장은 기대하지 않는 편이 맞음.
