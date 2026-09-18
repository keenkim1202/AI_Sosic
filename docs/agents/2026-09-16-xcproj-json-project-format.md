---
title: Xcode 27.2의 .xcproj JSON 포맷, 에이전트에게 프로젝트 파일을 열어줌
source: https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes
author: Apple
published: 2026-09-16
collected: 2026-09-18
tags: [apple, xcode, coding-agent, swift, source-control, project-format]
---

출처: [Xcode 27.2 Beta Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes) (항목 184661114, **2026-09-16**) · [Updating your Xcode project configuration file format](https://developer.apple.com/documentation/xcode/updating-your-xcode-project-configuration-file-format) · [apple/xcode-project-format](https://github.com/apple/xcode-project-format) (Apache-2.0, GitHub API 2026-09-18 확인) · [Get ready with the latest beta releases](https://developer.apple.com/news/?id=rfb1rooi) (2026-09-16)

## 요약

`project.pbxproj` 가 **JSON5 기반 `project.xcproj`** 로 갈아탈 수 있게 됨. Xcode 27.2 베타(**27B5019j**, 2026-09-16)의 프로젝트 포맷 새 기능이고, 릴리스 노트가 목적 셋을 직접 적음. 읽기 쉬움, 머지 친화적, 그리고 **코딩 에이전트가 편집하기 쉬움**. 같이 나온 것이 Apple 이 연 오픈소스 라이브러리 [apple/xcode-project-format](https://github.com/apple/xcode-project-format) 인데, **Xcode 자신의 인코딩·디코딩을 구현하는 바로 그 코드**라고 밝힘. 이 저장소 관점에서 값어치 있는 대목은 포맷이 예뻐졌다는 것이 아니라, **머지 충돌이 줄어드는 이유가 구조 설계에 명시돼 있다**는 것임. 타깃이 파일 목록을 들고 있던 관계를 뒤집어 **파일이 자기 타깃 멤버십을 들게** 했고, 그래서 파일 하나 추가가 diff hunk **둘이 아니라 하나**가 됨. 다만 UUID 가 사라진 것은 아니고, 포맷이 **strict JSON 이 아니라 JSON5** 라는 점이 도구를 붙일 때 제일 먼저 걸림.

## 기본 정보

| 항목 | 값 |
|---|---|
| 저장소 | `apple/xcode-project-format` |
| 라이선스 | Apache-2.0 |
| 언어 | Swift |
| 생성일 | **2026-09-15** |
| 태그 | `0.1.0` (GitHub Releases 항목은 없음) |
| stars / forks / open issues | **322 / 8 / 10** (2026-09-18 `gh api` 로 받음) |
| 요구사항 | Swift 6.1 이상, macOS 14 이상 |
| 부산물 | `xcprojformatter` CLI |

## 릴리스 노트가 말하는 것

> "Xcode now supports a JSON-based project format (.xcproj) that's more readable, merge-friendly, and easier for coding agents to edit. Enable it in the file inspector. Projects using .xcproj also open in earlier versions of Xcode 27."

읽을 것이 셋임.

- **에이전트가 1급 사용자로 문장에 들어와 있음.** 사람이 읽기 쉬운 것과 나란히 적혀 있지 별도 각주가 아님
- **켜는 것임.** File inspector 의 Project Format 팝업에서 JSON 을 고르면 `.xcodeproj` 번들 안의 파일이 `project.pbxproj` 에서 `project.xcproj` 로 바뀜. 기본값이 아님
- **바꾼 프로젝트가 Xcode 27 의 이전 버전에서도 열림.** 즉 변환하려면 27.2 베타가 필요하지만, 팀 전원이 베타로 올라갈 필요는 없다는 뜻으로 읽힘. ⚠️ 다만 **27.0 정식이 이 포맷을 쓰는 것을 넘어 편집·저장까지 하는지는 노트 문언으로 확정되지 않음.** 노트는 "open" 이라고만 씀

## 머지 충돌이 줄어드는 이유가 적혀 있음

라이브러리의 DocC 문서 `File Format Goals` 가 설계 목표 다섯을 열거하는데, 그중 둘이 이 저장소에 직접 걸림. **diff 를 이해 가능하게** 만드는 것과 **머지 충돌을 줄이고, 충돌이 나면 그 충돌이 납득되게** 만드는 것임.

핵심은 관계의 방향을 뒤집은 것임.

> "The file references the target it's been inserted into, rather than having the target reference the file that is now a member. Either of these encodings work, but having the file reference the target results in one diff hunk instead of two."

`pbxproj` 에서 파일 하나를 타깃에 넣으면 파일 참조가 생기고 타깃의 빌드 파일 목록도 바뀌어서 diff 가 두 군데에 남았음. 새 포맷은 파일 쪽에 `"target-membership"` 배열을 두고 거기서 끝냄. Apple 라이브러리의 `FileReference` 인코딩이 실제로 그렇게 돼 있음.

```swift
try container.encode(buildFiles, for: "target-membership", defaultValue: [])
```

문서가 덧붙이는 근거가 더 설득력 있음. 타깃이 파일 목록을 들면 그 **목록에 순서가 생기는데 그 순서에 의미가 없음.** 의미 없는 순서 때문에 두 번째 diff 가 나고, 거기서 나는 충돌은 납득되지 않는 충돌이 됨. 그래서 방향을 뒤집었다는 것임.

그리고 `build-file` 같은 내부 타입 이름을 JSON 키에서 없앴음. 사용자가 Xcode UI 에서 한 번도 본 적 없는 개념이라는 이유임. 키 이름과 enum 값을 Xcode UI 의 용어로 맞춘 것도 같은 목적임.

## 그래도 UUID 는 남아 있음

여기가 과장하기 쉬운 지점임. `ObjectID` 의 문서 주석이 범위를 정확히 그어 둠.

> "The `project.xcproj` prefers to minimize the usage of `ObjectID` for forming references, since they're difficult to read compared to name and path based references. But when names aren't unique enough to form precise references, ObjectIDs are used."

즉 **최소화지 제거가 아님.** 다른 프로젝트가 이 프로젝트의 타깃을 참조하는 것처럼 파일 바깥에서 들어오는 참조는 여전히 ID 기반이라고 같은 주석이 적음. `Project` 타입에도 `objectID`, `rootGroupDebugID`, `configurationListDebugID` 가 남아 있음.

같은 맥락에서 이름 기반 참조의 대가도 있음. `Project` 의 디코딩 경로가 **타깃 이름 중복을 오류로 막음**. 이름으로 참조를 만드는 포맷이니 당연한 제약인데, 이름이 겹치는 레거시 프로젝트를 변환하면 여기서 걸림.

## 에이전트 쪽에서 실제로 달라지는 것

이 저장소가 [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)에 정리한 그림은 에이전트가 빌드 설정·엔타이틀먼트·스킴을 **MCP 툴을 거쳐** 만지는 구조였음. 프로젝트 파일 자체는 에이전트가 직접 텍스트로 편집하기에 너무 나쁜 포맷이라 툴이 필요했던 것이기도 함. 이번 변경은 그 전제 한쪽을 바꿈.

- **직접 편집이 현실적인 선택지가 됨.** 툴이 없는 서드파티 에이전트도 파일을 읽고 고칠 수 있음
- **리뷰 비용이 내려감.** 파일 추가가 hunk 하나면 사람이 diff 를 실제로 읽게 됨. AI 생성 코드의 검증률이 낮다는 [InfoQ Culture & Methods Trends 2026](../industry/2026-08-07-infoq-culture-trends-2026.md)의 문제의식에서 보면, 읽히지 않는 diff 를 읽히는 diff 로 바꾸는 쪽이 리뷰 규칙을 하나 더 붙이는 것보다 값이 큼
- **병렬 워크트리의 마찰이 줄어듦.** 에이전트 여럿을 워크트리로 갈라 굴리면 `pbxproj` 가 늘 충돌 지점이었음. [Graph Engineering](../practices/2026-07-20-graph-engineering.md)이 말한 병렬화의 실질 비용 하나가 여기서 깎임

반대로 **줄어들지 않는 것**도 분명함. 에이전트가 프로젝트 구조를 잘못 바꾸는 사고의 빈도는 포맷과 무관함. 바뀐 것은 그 사고를 사람이 알아볼 확률임.

## xcprojformatter, 정규화를 훅으로 걸 자리

패키지가 같이 내는 CLI 가 있음. 하는 일은 하나, `project.xcproj` 를 **정규 포맷으로 다시 씀**.

```bash
# Project.xcodeproj/project.xcproj 를 제자리에서 갱신
xcprojformatter --update Project.xcodeproj

# 표준 입력으로 받아 표준 출력으로 내보냄
xcprojformatter
```

에이전트가 프로젝트 파일을 직접 편집하게 두면 공백과 키 순서가 제각각이 되고, 그 순간 "hunk 하나" 라는 설계 목표가 무너짐. pre-commit 훅이나 CI 단계에서 `--update` 를 한 번 돌려 정규형으로 고정하는 것이 이 도구의 쓸 자리임. 에이전트에게 포맷 규칙을 프롬프트로 가르치는 것보다 싸고 확실함. [팀 공유 AI 하네스 만들기](./2026-08-10-hq-team-ai-harness.md)가 말한 "환경의 어느 부분이 이 실수를 허용했나" 를 그대로 적용하면 답이 훅임.

## 저장소를 어떻게 다룰 것인가

`CONTRIBUTING.md` 가 범위를 좁게 그어 둠. 버그 수정, `xcprojformatter` 변경, 문서, 테스트는 받고, **새 스키마 필드나 타입 추가는 지금은 직접 PR 대상이 아님**. Xcode 의 실제 동작과 대조해 검증해야 하므로 먼저 feature request 로 논의하라고 적음. 유지보수 시간도 best effort 이고 응답 SLA 가 없다고 밝힘.

AI 에 대한 항목이 따로 있는데, 이 저장소에 적어둘 값어치가 있음.

> "Low-quality pull requests and issues, including those that appear to be generated without understanding, validation, or genuine human review, will be treated as spam and moderated accordingly."

도구를 쓰는 것은 허용하되 제출물에 대한 책임은 제출자에게 있고 설명할 수 있어야 한다는 것임. Apple 이 연 저장소의 기여 규칙에 이 문장이 들어간 것 자체가 신호임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md) | 에이전트가 빌드 설정·엔타이틀먼트를 MCP 툴로 만지던 구조. 프로젝트 파일 직접 편집이 선택지로 추가됨 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 정식 출시 직후의 27.x 흐름. macOS 26.6 요구사항이 27.2 베타에서도 그대로임 |
| [팀 공유 AI 하네스 만들기](./2026-08-10-hq-team-ai-harness.md) | 포맷 규칙을 프롬프트가 아니라 환경(훅)으로 강제하는 문제 |
| [Graph Engineering](../practices/2026-07-20-graph-engineering.md) | 병렬 워크트리의 충돌 비용 |
| [InfoQ Culture & Methods Trends 2026](../industry/2026-08-07-infoq-culture-trends-2026.md) | 사람 규모로 설계된 리뷰가 무의미해지는 문제. 읽히는 diff 가 그 완화책 하나 |

## 짚어야 할 것

- **strict JSON 이 아님.** 설계 문서가 "Being JSON5" 라고 적고, 라이브러리 직렬화 계층에 줄 주석과 블록 주석을 다루는 타입이 들어 있음. 파일 확장자가 `.xcproj` 라 파이프라인에서 JSON 으로 보이지만, **표준 JSON 파서에 그대로 물리면 깨질 수 있음.** 에이전트나 스크립트가 이 파일을 파싱하게 할 계획이면 이것부터 확인할 것
- **베타 기능임.** Xcode 27.2 베타에서 들어왔고 정식 27.2 가 아직 안 나왔음. 릴리스 노트가 정식에서 바뀌지 않는다고 약속하지 않음
- **되돌리는 공식 절차가 소스 컨트롤 의존임.** Apple 문서가 제시하는 복구 방법은 `.pbxproj` 삭제와 `.xcproj` 추가를 되돌리는 것, 즉 **변경을 discard 하는 것**임. 커밋한 뒤에 마음을 바꾸는 경로는 문서에 없음. 변환은 되돌릴 준비를 하고 별도 커밋으로 할 것
- **`0.1.0` 태그 하나뿐이고 저장소가 사흘 됐음.** 이 라이브러리에 도구를 얹을 계획이면 API 가 굳지 않았다는 전제로 볼 것. `CONTRIBUTING.md` 에 공개 전 정리되지 않은 `[TODO: link before public release]` 문구가 남아 있는 것도 같은 신호임
- **"머지 충돌이 사라진다" 는 말은 어디에도 없음.** 설계 목표의 문언은 줄이는 것과 납득되게 만드는 것임. 같은 객체를 두 사람이 만지면 충돌은 그대로 남
- **이 문서는 릴리스 노트, Apple 문서 페이지, 저장소 소스를 읽은 것이고 실제로 포맷을 켜서 프로젝트를 변환해 보지는 않았음.** 변환 소요 시간, 대형 프로젝트에서의 파일 크기 변화, CI 도구 호환성은 **확인할 수 없었음**
- 서드파티 도구(XcodeProj 등)의 지원 상황은 이 문서의 범위 밖이고 확인하지 않았음

## 유효기간

**2026-09-18 확인 기준, Xcode 27.2 베타(27B5019j)**임. 포맷 자체의 설계 목표는 문서로 고정돼 있어 잘 안 낡지만, 베타 딱지와 라이브러리 API 는 움직임. 다시 볼 때 확인할 것 셋. **27.2 정식에서 이 기능이 그대로 나왔는지**, **`0.1.0` 다음 태그에서 스키마 타입이 깨졌는지**, 그리고 **Xcode 27.0 정식이 이 포맷을 편집·저장까지 하는지**. 별·포크·이슈 수는 GitHub API 에서 2026-09-18 에 받은 값이고 저장소가 새것이라 빠르게 움직임.
