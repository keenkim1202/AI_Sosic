---
title: .xcproj JSON 프로젝트 포맷, Apple이 pbxproj를 갈아엎은 이유에 코딩 에이전트가 적혀 있음
source: https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes
author: Apple
published: 2026-09-16
collected: 2026-09-19
tags: [apple, xcode, coding-agent, swift, opensource, tooling, migration]
---

출처: [Xcode 27.2 Beta Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes) · [Updating your Xcode project configuration file format](https://developer.apple.com/documentation/xcode/updating-your-xcode-project-configuration-file-format) · [apple/xcode-project-format](https://github.com/apple/xcode-project-format) · [File Format Goals](https://github.com/apple/xcode-project-format/blob/main/Sources/Library/XcodeProjectFormat.docc/Docs/file-format-goals.md) · [Schema Overview](https://github.com/apple/xcode-project-format/blob/main/Sources/Library/XcodeProjectFormat.docc/Docs/schema-overview.md) (2026-09-19 확인)

## 요약

**Xcode 27.2 beta(2026-09-16)가 `project.pbxproj`를 대체하는 JSON 포맷 `project.xcproj`를 넣었음.** 릴리스 노트가 이유를 셋으로 적는데 그중 하나가 **"easier for coding agents to edit"**이고, 프레임워크 문서 쪽 부제는 아예 **"editable by coding intelligence agents"**임. 즉 이것은 소스 관리 편의 개선이면서 동시에 **에이전트가 프로젝트 파일을 직접 고치는 것을 전제로 한 포맷 변경**임. 포맷 모델과 도구는 **Apache-2.0으로 공개**됐고(`apple/xcode-project-format`, 2026-09-15 생성, **2026-09-19 기준 별 363**), 그 라이브러리가 **Xcode 자신의 인코딩·디코딩 구현**이라고 문서가 밝힘. 데이터 모델은 그대로 두고 **표기만 바꾼 것**이라 마이그레이션 비용이 낮지만, **`.xcproj`는 Xcode 27 이상에서만 열림.**

## 무엇이 바뀌는가

`.xcodeproj` 번들 **안의 설정 파일**이 바뀌는 것임. 번들 확장자는 그대로고 그 안의 `project.pbxproj`가 `project.xcproj`가 됨.

| | pbxproj | xcproj |
|---|---|---|
| 표기 | 프로퍼티 리스트 | **JSON5** |
| 도입 | 기존 | **Xcode 27.2 이상에서 새 프로젝트의 기본값** |
| 읽을 수 있는 Xcode | 전 버전 | **Xcode 27 이상만** |
| 데이터 모델 | 동일 | 동일 |

프레임워크 문서 문언임.

> "In Xcode 27.2 and later, the default project configuration file is a smaller, hierarchical, self-describing JSON file with a `.xcproj` extension. Xcode 27 and later supports both file formats so you can choose the format you prefer."

**기존 프로젝트는 자동으로 안 바뀜.** File inspector의 Project Document 아래 Project Format 팝업에서 JSON을 고르는 수동 전환이고, 소스 관리를 쓰면 `.pbxproj` 삭제와 `.xcproj` 추가를 되돌려서 취소할 수 있다고 문서가 적음.

핵심은 마지막 줄임. 포맷 목표 문서가 **"a reimagining of the text used to encode the project format, but not the data held by the project format"**이라고 씀. **모델이 아니라 표기만 바뀐 것**이라 전환 자체는 정보 손실이 없는 작업임.

## 왜 에이전트 얘기가 나오는가

릴리스 노트의 한 줄이 이 문서의 출발점임 (184661114).

> "Xcode now supports a JSON-based project format (.xcproj) that's more readable, merge-friendly, and easier for coding agents to edit. Enable it in the file inspector. Projects using .xcproj also open in earlier versions of Xcode 27."

pbxproj가 에이전트에게 나빴던 이유는 JSON이 아니어서가 아니라 **객체 사이의 참조가 불투명 ID로 돼 있고 한 의도가 여러 자리에 흩어져 기록**되기 때문임. 파일 하나를 타깃에 넣으려면 서로 떨어진 여러 자리를 동시에 맞춰야 하고, 그 편집이 맞았는지는 Xcode로 열어보기 전까지 모름. 에이전트에게는 **검증 없이 여러 곳을 동시에 고치는 작업**이라 실패 확률이 높은 형태였음.

포맷 목표 문서가 든 예가 정확히 그 지점임. 타깃 멤버십을 **타깃 쪽이 아니라 파일 쪽에** 적기로 한 결정.

> "The file references the target it's been inserted into, rather than having the target reference the file that is now a member. Either of these encodings work, but having the file reference the target results in one diff hunk instead of two."

결과로 파일 정의 옆에 `"target-membership": [ ... ]` 같은 키가 붙고, 헤더 공개 여부는 `"header-role": "public"`으로 같은 자리에 들어감. 문서는 **`build-file`이라는 용어가 파일 어디에도 안 나온다**는 것을 성과로 적음. Xcode UI가 사용자에게 보여주지 않는 개념은 파일에도 안 넣는다는 원칙임.

**판단.** 에이전트 관점에서 이 변경의 값어치는 "JSON이라 파싱이 쉽다"가 아님. **한 의도가 한 자리의 편집으로 끝난다**는 것임. 에이전트가 만든 diff를 사람이 리뷰할 때 "이 hunk가 그 의도와 대응하는가"를 눈으로 판정할 수 있게 됨. 리뷰 가능한 diff를 만드는 것이 자동 편집을 허용하는 전제라는 점에서, [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)이 말한 "판정 기준을 루프 바깥에 둘 것"과 같은 방향임.

**다만 이것이 에이전트의 프로젝트 편집을 안전하게 만든다는 뜻은 아님.** 포맷이 검증해 주는 것은 문법과 스키마까지고, 타깃 멤버십을 잘못 바꾼 것은 여전히 빌드로만 잡힘.

## 다른 목표 넷

포맷 목표 문서가 나열한 것이고, 순서도 그쪽 순서임.

- **Xcode 프로젝트와 상호운용하는 도구 생태계를 만드는 것.** 이것이 첫 번째로 적혀 있음
- **diff가 이해되게 만드는 것.** "파일 하나를 타깃에 추가하면 diff hunk 하나"가 기준
- **머지 충돌을 줄이고, 나더라도 납득되게 만드는 것.** 충돌이 났다면 다른 사람이 같은 객체를 만졌을 때뿐이게
- **사람이 읽을 수 있게 만드는 것.** 키 이름을 Xcode UI 용어로 쓰고, 기본값과 다른 값만 기록하고, 단순한 객체는 한 줄로 접히게 씀

생태계 목표가 첫 줄인 것은 립서비스가 아님. 라이브러리 README가 **"generators, linters, verifiers, and validators"**가 각자 포맷을 리버스 엔지니어링하지 않게 하는 것이 목적이라고 적음. XcodeGen·Tuist 계열이 지금까지 해 온 일이 그것임.

## 포맷 모델이 오픈소스로 나왔음

`apple/xcode-project-format`. **2026-09-19 `gh api`로 받은 값**임.

| 항목 | 값 |
|---|---|
| 라이선스 | **Apache-2.0** |
| 언어 | Swift |
| 별 / 포크 / 열린 이슈 | **363** / 11 / 10 |
| 생성 | 2026-09-15 |
| 최근 푸시 | 2026-09-16 |
| 태그 | `0.1.0` (GitHub Releases 항목은 없음) |
| 요구사항 | Swift 6.1 이상, macOS 14 이상 (비 Darwin 플랫폼에서도 빌드됨) |

이 저장소에서 제일 중요한 문장은 이것임.

> "It's used to implement Xcode's native coding and decoding of project files, so it's complete, and clients should have all of the resources they need to interoperate with this format."

**Xcode가 쓰는 바로 그 구현이 공개된 것**임. 서드파티 도구가 Xcode와 다르게 해석할 여지가 원리적으로 없다는 뜻이고, pbxproj 시절의 상황과 갈리는 지점이 여기임.

`XCSchema` 한 네임스페이스 아래에 값 타입으로 모델이 들어 있음. `Project`의 루트 필드는 `topLevelReferences`, `packages`, `configurations`, `buildSettings`, `targets` 등임. 참조 타입은 `FileReference`, `Group`, `Folder`, `VariantGroup`, `VersionGroup` 다섯 가지로 갈림.

같은 패키지가 CLI `xcprojformatter`를 함께 냄. 도움말 원문 기준으로 하는 일은 **정규 형식으로 pretty print** 하는 것 하나임.

```bash
# Project.xcodeproj/project.xcproj 를 제자리에서 갱신
xcprojformatter --update Project.xcodeproj

# 표준입력으로 받아 정규 형식으로 표준출력에 씀
xcprojformatter
```

에이전트가 프로젝트 파일을 고치게 할 거면 **편집 직후 `xcprojformatter --update`를 돌리는 훅**이 형식 흔들림을 없애는 가장 싼 방법임. 에이전트가 만든 diff에서 형식 노이즈를 걷어내면 리뷰가 내용만 보게 됨.

## 기여 범위가 좁게 잠겨 있음

`CONTRIBUTING.md`가 지금 받지 않는 것을 명시함. **새 스키마 필드와 타입 추가, API 확장은 직접 PR 대상이 아님.** 이유도 적혀 있는데, Xcode의 실제 동작·포맷과 대조해 검증해야 책임 있게 추가할 수 있다는 것임. 받는 것은 버그 수정, `xcprojformatter` 변경, 문서, 테스트임.

같은 문서가 AI 도구 사용에 대해 한 절을 따로 둠. 쓰는 것은 허용하되 제출자가 전적으로 책임지고 설명할 수 있어야 하며, **이해·검증·리뷰 없이 생성된 것으로 보이는 PR과 이슈는 스팸으로 취급**한다고 적음. 유지보수는 **전담 로테이션 없는 best effort**이고 응답에 며칠에서 몇 주가 걸릴 수 있다고 밝힘.

즉 **포맷은 열었지만 포맷의 진화 권한은 Apple이 쥐고 있음.** 이 저장소를 "Apple이 프로젝트 포맷을 커뮤니티에 넘겼다"로 읽으면 안 됨. 읽는 쪽은 완전히 열렸고 쓰는 쪽은 아님.

## 도입 판단

- **팀에 Xcode 26을 쓰는 사람이 한 명이라도 있으면 지금은 아님.** `.xcproj`는 Xcode 27 이상만 열고, 이건 기능 저하가 아니라 파일을 못 여는 문제임
- **CI 러너의 Xcode 버전을 먼저 확인할 것.** Xcode 27.2 beta는 macOS Tahoe 26.6 이상을 요구함
- **프로젝트 파일을 생성·수정하는 서드파티 도구를 쓰고 있으면 그 도구의 지원 여부가 선행 조건임.** 이 저장소 문서 기준으로는 확인하지 않았음
- 반대로 **새로 만드는 프로젝트라면 27.2 이상에서 기본값이 이미 JSON**이라 고를 일이 없음
- 전환 자체는 되돌릴 수 있음. 소스 관리에서 `.pbxproj` 삭제와 `.xcproj` 추가를 되돌리면 됨

## 같은 릴리스에서 같이 본 것

Xcode 27.2 beta 릴리스 노트에서 이 저장소 다른 문서에 걸리는 항목들임.

| 이슈 | 내용 |
|---|---|
| 186442676 | **`RenderPreview` MCP 툴이 사용 가능한 렌더 대상 목록을 반환**하고, 호출자가 렌더에 쓸 대상을 지정할 수 있게 됨 (해결 항목) |
| 186939138 | macOS 27.2 beta에서 **코드 완성 사용 중 Xcode가 크래시**할 수 있음. 회피법은 `defaults write com.apple.dt.Xcode CodeCompletionAssetsToLoad /dev/null`로 향상된 코드 완성 랭킹을 끄는 것 |
| 187160501 | macOS·watchOS·tvOS·visionOS SDK가 **27.1을 유효한 배포 대상으로 잘못 보고**함. 이 값을 쓰면 빌드가 예상과 다르게 동작할 수 있음 |

첫 항목이 [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)에 직접 붙음. 그 문서가 정리한 프리뷰 MCP 툴이 플랫폼·기기 타입·OS 버전을 **결과에 반환**하는 데서 한 칸 더 가서, 이제 **어디에 렌더할지를 호출자가 고를 수 있음.** 에이전트가 특정 기기에서만 나는 레이아웃 문제를 재현하려 할 때 필요한 축이 이것임.

두 번째 항목은 AI 기능 자체는 아니지만 **끄는 스위치가 "enhanced code completion ranking"** 이라는 점이 볼거리임. 코드 완성 랭킹이 별도 에셋을 로드하는 구조라는 것이 회피법에서 드러남.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md) | 같은 릴리스 계열. 에이전트가 Xcode를 통해 만지는 표면의 목록이 그쪽에 있고, 프로젝트 파일이라는 표면이 여기서 추가됨 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 정식 출시 직후 첫 베타에서 나온 변경임. macOS 26.6 요건은 27.2 beta에서도 같음 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | 에이전트 편집을 사람이 판정할 수 있게 만드는 문제. diff 하나가 의도 하나에 대응하는 것이 그 판정의 전제 |
| [팀 공유 AI 하네스 만들기](./2026-08-10-hq-team-ai-harness.md) | `xcprojformatter --update`를 훅으로 거는 것이 팀 기본값으로 배포할 종류의 규칙임 |

## 짚어야 할 것

- **직접 전환해 보지 않았음.** 이 문서는 릴리스 노트, 프레임워크 문서, 공개된 라이브러리 소스와 DocC 문서로 확인한 범위임. 실제 프로젝트를 JSON으로 바꿨을 때 어떤 도구가 깨지는지는 확인하지 않았음
- **`xcprojformatter`가 Xcode 27.2와 함께 `/usr/bin`에 설치된다는 2차 보도가 있으나 1차 출처에서 확인하지 못했음.** 확인한 것은 오픈소스 패키지가 이 CLI를 함께 낸다는 것까지임
- **포맷의 실제 JSON 예시를 1차 출처에서 확보하지 못했음.** 저장소에 샘플 `.xcproj`가 없고 테스트는 외부 경로를 환경변수로 받게 돼 있음. 본문의 `"target-membership"`과 `"header-role"`은 포맷 목표 문서가 예로 든 키 이름을 그대로 옮긴 것이고, 전체 파일이 어떤 모양인지는 확인하지 않았음
- **"에이전트가 편집하기 쉽다"는 것은 Apple의 설계 의도이지 측정된 결과가 아님.** 에이전트의 프로젝트 파일 편집 성공률이 실제로 올랐다는 수치는 어디에도 없음
- 릴리스 노트의 이슈 번호는 Apple이 붙인 것을 그대로 옮겼음
- **27.2는 베타임.** 정식에서 기본값 정책이나 문언이 바뀔 수 있음

## 유효기간

**2026-09-19 확인 기준이고 Xcode 27.2 beta(27B5019j) 시점임.** 별 수와 이슈 수는 며칠 단위로 낡으므로 갱신하지 말고 `gh api repos/apple/xcode-project-format`로 다시 받을 것. 다시 볼 때 확인할 것 둘. **27.2가 정식으로 나왔을 때 기본값과 호환성 문언이 그대로인지**, 그리고 **`0.1.0` 이후 태그가 붙으면서 스키마에 필드가 추가됐는지**. 후자는 `CONTRIBUTING.md`가 "expect this scope to grow over time"이라고 예고해 둔 부분임.
