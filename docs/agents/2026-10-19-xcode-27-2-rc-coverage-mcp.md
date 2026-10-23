---
title: Xcode 27.2 RC, 에이전트가 커버리지를 읽게 됨
source: https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes
author: Apple
published: 2026-10-19
collected: 2026-10-23
tags: [apple, xcode, coding-agent, mcp, agent-eval, tooling]
---

출처: [Xcode 27.2 RC Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes) · [Xcode 27.1 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_1-release-notes) · [Xcode 27 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes) · [개발자 릴리스 목록](https://developer.apple.com/news/releases/) (2026-10-23 확인)

## 요약

**Xcode 27.2 RC(27B48, 2026-10-19)의 Coding Assistant 절에 항목이 딱 하나 있고, 그게 `GetCodeCoverage` MCP 툴임.** 최근 테스트 결과의 코드 커버리지를 돌려주고 타깃이나 파일로 필터링할 수 있음 (181141715). Xcode 27 정식 노트의 Coding Intelligence 절에는 커버리지나 테스트 결과를 읽는 MCP 툴이 없었고 27.1 노트에는 Coding Assistant 절 자체가 없으니, **에이전트가 테스트 결과물을 수치로 읽는 경로는 27.2 에서 처음 열림.** 같은 노트에서 Device Hub 가 **Simulate Shake · Simulate Fall · Simulate Memory Pressure**를 더했고 (189236369), `.rotateDigitalCrown` 이 기기에 등록되지 않던 테스트 실패가 고쳐졌음 (188422137). 다만 **커버리지는 이 저장소가 반복해서 경고해 온 "얕은 지표"의 교과서적 사례**라, 이 툴을 에이전트 루프의 판정 기준으로 올리면 점수만 오르는 결과가 나옴.

## `GetCodeCoverage`, 에이전트가 처음으로 테스트 결과를 읽음

27.2 RC 노트의 Coding Assistant 절은 New Features 항목 하나로 끝남.

> "Added a GetCodeCoverage MCP tool that reports code coverage from the most recent test result, optionally filtered by target or file." (181141715)

문언에서 짚을 게 셋임.

- **"most recent test result"**: 에이전트가 테스트를 돌리는 툴이 아니고 **이미 나와 있는 결과를 읽는 툴**임. 즉 빌드·테스트 실행은 기존 build/test MCP 툴이 하고, 이 툴은 그 산출물에서 숫자를 뽑는 자리에 붙음
- **"optionally filtered by target or file"**: 전체 수치뿐 아니라 **파일 단위로 좁힐 수 있음.** 에이전트가 자기가 방금 고친 파일만 보고 판단하는 사용법이 가능해짐
- 절 이름이 **Coding Assistant** 로 바뀌었음. Xcode 27 정식 노트는 같은 영역을 **Coding Intelligence** 로 불렀음. 같은 것을 가리키는 것으로 읽었고 Apple 이 개칭을 공지한 문서는 확인하지 못했음

[Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)에 정리된 MCP 툴 표에 **테스트·커버리지 칸이 없었음.** 그 표의 영역은 디버깅, 빌드 구성, 프리뷰, 로컬라이제이션, 실행, 디버거 여섯이고, 에이전트가 자기 변경을 검증하는 수단은 시뮬레이터 스크린샷과 합성 터치 이벤트였음. **스크린샷은 사람이 봐야 판정이 되는 산출물**이고 커버리지는 에이전트가 혼자 비교할 수 있는 숫자임. 그 차이가 이 항목이 작아 보이는데 작지 않은 이유임.

## 그런데 커버리지를 루프 지표로 올리면 안 됨

이 저장소에 이미 답이 적혀 있는 문제임.

[자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)의 출발점이 **얕은 지표를 주면 점수는 오르고 동작은 나빠진 에이전트가 나오는데 루프는 성공했다고 보고한다**는 것임. 커버리지는 그 함정의 가장 흔한 모양임. 줄이 실행됐다는 것만 세고 **그 줄이 옳게 동작했는지는 세지 않으니**, 단정문을 하나도 안 넣은 테스트로도 수치가 올라감. 에이전트에게 "커버리지를 올려라"를 주면 정확히 그 테스트가 나옴.

반대로 쓰는 법은 있음. **게이트가 아니라 탐색 도구로** 쓰는 것임.

- **방향을 거꾸로 쓰기**: "커버리지를 올려라"가 아니라 "내가 고친 파일에서 **한 번도 실행되지 않은 분기**를 찾아 거기에 해당하는 테스트가 없는 이유를 말해라". 숫자를 목표로 주지 않고 **빈 곳을 가리키는 색인**으로만 씀
- **파일 필터를 쓰는 이유가 여기 있음**: 리포지토리 전체 수치는 목표로 삼기 좋게 생겼고, 방금 만진 파일의 미실행 분기 목록은 목표로 삼을 수 없음. 후자가 이 툴에서 건질 쪽임
- **판정은 밖에 두기**: [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md)의 결론대로 머지 게이트는 "틀리면 얼마나 아픈가"로 여는 것이고, 커버리지 퍼센트는 그 질문에 답하지 않음

두 문서가 서로 충돌하는 쌍이라는 점도 여기서 유용함. 경로를 채점할 것인가 동작을 채점할 것인가인데, **커버리지는 경로 채점의 극단**임. 그래서 둘 중 어느 쪽 편에서 봐도 단독 게이트로는 쓸 수 없음.

## Device Hub, 상태를 주입하는 축이 늘어남

Device Hub 의 New Features 가 한 항목(189236369)에 묶여 나왔고 시뮬레이터 전용 표시가 붙은 것이 셋임.

| 추가된 것 | 쓸모 |
|---|---|
| Slow Motion Animation (시뮬레이터 전용) | 애니메이션 중간 프레임을 스크린샷으로 잡을 수 있게 됨 |
| Simulate Shake (시뮬레이터 전용) | 흔들기 제스처로 들어가는 경로(undo 등) 재현 |
| Simulate Fall (시뮬레이터 전용) | 낙하 감지에 반응하는 코드 경로 재현 |
| Simulate Memory Pressure (시뮬레이터 전용) | 메모리 경고 처리와 캐시 해제 경로 재현 |
| Context Menu for Screenshots, 베젤 숨기기 옵션 | 스크린샷 산출물 다루기 |
| App Settings (종료 동작, 키보드 단축키 동작, 보안 키보드 입력, 하드웨어 키보드 시뮬레이션, 화면 캡처 저장 위치) | 반복 실행 환경 고정 |

**여기서 조심할 것.** 이 항목들은 Device Hub 의 기능으로 적혀 있고 **릴리스 노트는 이것들이 MCP 툴로 노출된다고 말하지 않음.** 에이전트가 직접 흔들기나 메모리 압박을 주입할 수 있다는 서술은 노트에 없고, 확인하지 못했음. 사람이 Device Hub 에서 상태를 만들어 두고 에이전트에게 검증을 맡기는 조합까지는 지금 문언으로 성립함.

Simulate Memory Pressure 는 그중에서도 값어치가 다름. **메모리 경고 처리는 사람이 수동으로 재현하기 가장 귀찮은 경로**인데, 같은 노트에 `Device->Simulate Memory Warning` 메뉴가 동작하지 않던 버그가 고쳐졌다고 적혀 있음 (188326513). 즉 **이 경로는 그동안 고장나 있었음.**

## watchOS 검증 쪽 수정 하나

XCTest UI Automation 의 해결 항목임.

> "Fixed: Tests utilizing `.rotateDigitalCrown` may fail due to the device not registering the action." (188422137)

[Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)이 watchOS 검증 능력으로 적은 것이 **Digital Crown 회전과 누름**이었음. 그 축이 테스트에서 간헐적으로 먹히지 않던 상태였다는 뜻이고, 에이전트가 자기 변경을 watchOS 에서 검증하게 해 뒀다면 **실패가 코드 탓이 아니었을 수 있는 구간**임. 27.2 이전 결과를 믿고 있었으면 다시 돌려볼 자리.

## 프로젝트 포맷 문언은 그대로

[.xcproj JSON 프로젝트 포맷](./2026-09-16-xcproj-json-project-format.md)이 유효기간에 "27.2가 정식으로 나왔을 때 기본값과 호환성 문언이 그대로인지"를 재확인 항목으로 적어 뒀음. RC 기준으로 **둘 다 그대로임.**

- 릴리스 노트의 Project Format 항목은 베타 때와 같은 문장이고 **"Enable it in the File inspector"**, **"Projects using .xcproj also open in earlier versions of Xcode 27"** 가 유지됨 (184661114)
- 프레임워크 문서도 **"In Xcode 27.2 and later, the default project configuration file is a smaller, hierarchical, self-describing JSON file with a `.xcproj` extension"** 와 **"The JSON file format `.xcproj` is compatible with Xcode 27 and later"** 를 그대로 둠

즉 그쪽 문서의 "팀에 Xcode 26 쓰는 사람이 있으면 지금은 아님" 판단은 **27.2 RC 에서도 유효함.**

## 27.1 에는 에이전트 항목이 없음

하나 건너뛰기 쉬운 사실임. **Xcode 27.1(27A9275, 2026-10-15) 노트에는 Coding Assistant 절도 Project Format 절도 없음.** 들어 있는 것은 Mac Catalyst 알려진 문제, Previews 의 대체 디스플레이 미리보기 피커, Simulator 알려진 문제뿐임.

그래서 **27.1 에 올라간 것만으로는 이 문서의 어느 항목도 안 들어옴.** 커버리지 툴을 쓰려면 27.2 를 기다려야 함. 27.1 과 27.2 RC 모두 **macOS Tahoe 26.6 이상**을 요구하고 Swift 6.4 를 포함함.

같은 날 나온 **iOS 27.2 RC(24B115)** 노트에는 Foundation Models, Core AI, App Intents, Apple Intelligence, Neural Engine 관련 항목이 하나도 없음. 즉 이번 사이클의 변화는 **온디바이스 모델 쪽이 아니라 IDE 쪽에만** 있음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md) | 그 문서의 MCP 툴 표에 테스트·커버리지 칸이 없었음. 이 문서가 그 칸을 채움. watchOS Digital Crown 검증 축의 수정도 그쪽에 직접 붙음 |
| [.xcproj JSON 프로젝트 포맷](./2026-09-16-xcproj-json-project-format.md) | 같은 27.2 릴리스. 그 문서가 재확인 항목으로 남긴 기본값·호환성 문언을 RC 로 대조함 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | 커버리지를 에이전트 지표로 주면 생기는 문제. 얕은 지표가 점수만 올리는 전형 |
| [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md) | 커버리지를 머지 게이트로 쓸 수 있는가. 게이트를 여는 기준이 퍼센트가 아니라는 쪽 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 정식 출시 이후 27.1, 27.2 로 이어지는 갱신 줄기. 그 문서가 27.1 이후를 먼저 확인하라고 적어 뒀음 |

## 짚어야 할 것

- **릴리스 노트만 읽었고 직접 써보지 않았음.** `GetCodeCoverage` 의 실제 입출력 스키마, 반환되는 수치의 단위(줄 커버리지인지 분기 커버리지인지), 테스트 결과가 없을 때의 동작은 **노트에 없고 확인하지 못했음**
- **`GetCodeCoverage` 가 27.2 의 어느 빌드에서 처음 들어왔는지는 확인할 수 없었음.** 확인한 것은 27.2 RC 노트에 New Features 로 있다는 것, Xcode 27 정식 노트의 Coding Intelligence 절에 커버리지·테스트 결과 MCP 툴이 없다는 것, 27.1 노트에 해당 절이 없다는 것까지임. 9월 베타 노트는 페이지가 RC 로 갱신돼 당시 문언을 다시 열 수 없었음
- **Device Hub 의 Simulate Shake · Fall · Memory Pressure 가 에이전트에게 노출되는지는 노트에 없음.** 이 문서는 사람이 쓰는 기능으로만 서술했음
- 이슈 번호는 Apple 이 붙인 것을 그대로 옮겼음
- **27.2 는 RC 임.** 정식에서 항목이 빠지거나 문언이 바뀔 수 있음
- 커버리지에 대한 이 문서의 판단은 **원문에 없는 것**임. 릴리스 노트는 툴의 존재만 적고 쓰는 법을 권고하지 않음

## 유효기간

**2026-10-23 확인 기준이고 Xcode 27.2 RC(27B48) 시점임.** 다시 볼 때 확인할 것 셋. **27.2 정식 노트에서 Coding Assistant 절이 늘었는지**, **`GetCodeCoverage` 의 파라미터와 반환 형식이 문서화됐는지**(릴리스 노트 말고 Xcode 문서 쪽에 MCP 툴 레퍼런스가 생기는지), 그리고 **Device Hub 의 상태 주입 기능이 MCP 툴로 올라오는지**. 마지막 항목이 올라오면 에이전트가 메모리 압박 같은 상태를 스스로 만들어 검증하게 되므로 [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)의 권한 모델 절을 같이 다시 봐야 함.
