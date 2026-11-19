---
title: Xcode 27.3 beta 2 의 에이전트 표면, String Catalog 편집 권한과 Instruments 전문가 둘
source: https://developer.apple.com/documentation/xcode-release-notes/xcode-27_3-release-notes
author: Apple
published: 2026-11-16
collected: 2026-11-19
tags: [apple, xcode, coding-agent, mcp, skills, tooling, localization]
---

출처: [Xcode 27.3 Beta 2 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_3-release-notes) · [개발자 릴리스 목록](https://developer.apple.com/news/releases/) · [releases.rss](https://developer.apple.com/news/releases/rss/releases.rss) · [Localizing your app using agents](https://developer.apple.com/documentation/xcode/localizing-your-app-using-agents) · [Extending and customizing agents](https://developer.apple.com/documentation/xcode/extending-and-customizing-agents) · [Xcode updates](https://developer.apple.com/documentation/updates/xcode) (전부 **2026-11-19 확인**. 릴리스 노트와 Xcode 문서는 자바스크립트로 렌더링되므로 같은 경로의 `/tutorials/data/...json` 으로 본문을 받았음)

## 요약

**Xcode 27.3 beta 2(27C5035i)가 2026-11-16 에 나왔고, 이 문서는 beta 2 에서 새로 붙은 절만 본 것임.** 에이전트 쪽에 걸리는 항목이 셋임.

첫째, **로컬라이제이션 MCP 툴의 권한이 넓어졌음.** [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md)이 적어 둔 27 시점의 툴은 "번역을 읽고 계획하고 편집"까지였는데, beta 2 는 **"String Catalog Editor 가 하는 것과 같은 방식으로" 문자열을 추가·제거하고 state 를 바꾸고 Swift 심볼로 문자열을 지목**하는 툴을 더함 (184511781). 여기서 **state 가 핵심임.** Apple 자신의 문서가 에이전트 번역과 사람 번역을 가르는 표식으로 `machine_translated` 를 쓰라고 적어 뒀는데, **그 표식을 바꿀 수 있는 툴을 같은 에이전트에게 준 것**이 이번 변경임.

둘째, **Instruments 산출물이 에이전트가 읽을 수 있는 텍스트가 됐음.** 새 `xctrace triage` 가 트레이스 안의 행(hang)을 하나의 보고서로 묶어 주고 (182217728), `xctrace` 가 콜 트리·요약·리스트·비용 주석 소스를 **plain text 나 Markdown 으로 내보냄** (184951123). 그리고 Xcode 가 **"Apple specialists" 둘(`xctrace`, hangs)** 을 에이전트용으로 탑재함 (187385971).

셋째, **같은 툴을 Xcode 밖에서 쓰려면 OS 버전이 따로 걸림.** `xcstringstool sync` 의 키 순서가 IDE 와 달랐던 버그 수정이 "String Catalog MCP 툴을 Xcode 밖에서 쓸 때의 키 순서도 고친다"면서 **macOS 27.3 Beta 2 를 요구한다**고 명시함 (190849755).

## String Catalog, 읽기·편집에서 구조 변경으로

beta 2 의 Localization New Features 는 항목 셋임.

> "New tools allow coding agents to manipulate String Catalogs in the same ways that the String Catalog Editor does. For example, agents can add and remove strings, change state, and reference strings by Swift symbol." (184511781)

> "A new skill in Xcode aids coding agents in writing localizable code and detecting common localization problems." (184951217)

> "Translating projects with coding agents now uses fewer tokens." (190440819)

27 시점과 무엇이 달라졌는지가 이 문장에 다 들어 있음.

| 축 | Xcode 27 (176376425) | 27.3 beta 2 (184511781) |
|---|---|---|
| 대상 | 번역 값 | **문자열 항목 자체**(추가·제거) |
| state | 서술 없음 | **에이전트가 바꿈** |
| 지목 방식 | 카탈로그 파일 안의 키 | **Swift 심볼** |

- **추가·제거가 붙은 것**: 이전까지 에이전트가 할 수 있는 일은 이미 있는 키에 번역을 채우는 쪽이었음. 이제 키 자체를 지울 수 있으므로, **번역이 사라지는 종류의 회귀가 에이전트 쪽에서 나올 수 있는 경로**가 처음 열림. 카탈로그를 리뷰 대상에서 빼 두고 있었으면 지금 다시 넣을 자리임
- **Swift 심볼로 지목하는 것**: 코드의 `String(localized:)` 호출 지점과 카탈로그 항목을 에이전트가 같은 이름으로 다룬다는 뜻임. "이 뷰에 쓰이는 문자열만 고쳐라" 같은 지시가 성립하는 조건이고, 거꾸로 **심볼을 리네임하면 에이전트가 짚는 대상도 같이 움직임**
- **새 스킬 하나**: 번역을 만드는 스킬이 아니라 **로컬라이즈 가능한 코드를 쓰게 하고 흔한 문제를 잡아내는 쪽**임. Apple 문서가 "SwiftUI 밖의 Swift 코드는 `String(localized:)` 를 명시적으로 써야 카탈로그에 잡힌다"고 적어 둔 그 함정을 겨냥한 것으로 읽힘. 다만 스킬 이름과 발동 조건은 노트에 없고 **확인하지 못했음**
- **토큰이 줄었다는 항목에는 수치가 없음** (190440819). 얼마나 줄었는지, 어느 구성 기준인지 노트에 없음. 비용 계획의 근거로 쓸 수 있는 문장이 아님

## `machine_translated` 를 바꿀 수 있다는 것

Apple 문서가 에이전트 번역을 사람 번역과 가르는 방법을 직접 적어 둠.

> "To differentiate agentic translations from those that you provide, Xcode sets the state of translations to `machine_translated` in the string catalog file. If you export your localizations to XML Localization Interchange File Format (XLIFF), Xcode also sets the `state-qualifier` property to `leveraged-mt`."

즉 **state 는 번역 품질을 관리하는 쪽에서 보면 감사 표식임.** 어느 문자열이 기계 번역이고 어느 것이 사람 손을 탔는지가 파일 안에 남고, XLIFF 로 내보내도 따라감. 외주 검수를 쓰는 팀이라면 이 값으로 작업 범위를 뽑음.

beta 2 가 더한 것은 그 값을 **에이전트가 쓰는 툴로 바꿀 수 있게 한 것**임. 노트는 `change state` 라고만 적고 어느 값으로 바꿀 수 있는지, 제한이 있는지 말하지 않음. 그래서 지금 문언으로 성립하는 최악은 이것임. **번역을 에이전트가 만들고, 같은 에이전트가 그 항목의 state 를 번역 완료로 올리는 것.** 그러면 위 문단의 표식이 무의미해짐.

악의를 가정할 필요도 없음. "미번역 상태를 정리해라" 같은 지시는 사람이 자연스럽게 쓰고, 그 지시를 가장 싸게 만족시키는 행동이 정확히 state 를 바꾸는 것임. [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)이 든 얕은 지표 문제와 같은 모양이고, 여기서는 지표가 아니라 **지표를 기록하는 필드 자체**를 에이전트가 쥐었다는 점이 다름.

실무에서 할 수 있는 것은 둘임.

- **state 변경을 diff 에서 본다.** `.xcstrings` 는 JSON 이므로 state 필드 변경은 리뷰에서 보이는 줄임. 번역 값은 그대로인데 state 만 바뀐 hunk 가 있으면 그게 신호임
- **번역 가이드를 설정 파일에 둔다.** Apple 문서가 `AGENTS.md` 나 `CLAUDE.md` 에 번역 지침을 두고 `TRANSLATION.md` 같은 별도 파일을 참조시키는 방식을 적어 뒀음. 번역하지 말 문자열 목록과 용어집이 여기 들어가는 자리이고, **state 를 손대지 말라는 문장도 여기 들어감**. 다만 설정 파일의 지시는 강제가 아니라 프롬프트임

## Xcode 밖에서 쓸 때 OS 버전이 따로 걸림

Resolved Issues 에 있는 한 줄인데 운영에 걸림.

> "Fixed an issue where running `xcstringstool sync` on the command line would use a different key order than the IDE. This also fixes key order when using String Catalog MCP tools outside of Xcode. This requires macOS 27.3 Beta 2." (190849755)

- **키 순서가 달랐다는 것은 머지 충돌과 가짜 diff 를 만들었다는 뜻임.** 사람은 IDE 에서 고치고 CI 는 `xcstringstool sync` 를 돌리는 구성이면, 내용이 같아도 파일이 계속 흔들림. [.xcproj JSON 프로젝트 포맷](./2026-09-16-xcproj-json-project-format.md)이 pbxproj 에서 풀었던 것과 같은 종류의 문제가 카탈로그 쪽에도 있었던 것임
- **"String Catalog MCP 툴을 Xcode 밖에서 쓸 때"** 라는 문구가 중요함. Xcode MCP 서버를 워크스페이스 없이 돌려 외부 에이전트가 붙는 구성이 27 베타 5 부터 있었고, 그 경로에서도 이 툴이 보인다는 뜻임
- **그런데 고침이 Xcode 가 아니라 macOS 27.3 Beta 2 에 들어 있음.** Xcode 만 올리고 OS 를 안 올린 머신에서는 안 고쳐짐. CI 러너와 개발자 머신의 OS 버전이 갈리면 **같은 Xcode 로도 키 순서가 갈릴 수 있는 구간**임

## Instruments 에 "전문가" 둘

Xcode 절의 New Features 가 beta 2 에서 한 항목임.

> "Xcode now includes two Apple specialists for coding agents. The `xctrace` specialists helps agents use the tool’s new export and filtering options to navigate trace data. The hangs specialist helps agents find hangs in your traces and fix them." (187385971)

그 "new export and filtering options" 이 같은 노트의 Instruments 절에 있음.

| 항목 | 내용 |
|---|---|
| `xctrace triage` (182217728) | 트레이스 안의 행들을 보고서로 만듦. **여러 instrument 의 데이터를 묶어** 행마다 개요와 조사용 맥락을 냄 |
| `xctrace export` 확장 (184951123) | 콜 트리·요약·리스트·비용 주석 소스를 **plain text 또는 Markdown** 으로 내보냄. 필터·정렬·데이터 마이닝 옵션으로 범위를 좁힘 |
| `xctrace list --verbose` (188404842) | 템플릿·instrument 목록에 설명을 같이 출력 |
| Target CPU Cores (188637563) | Recording Information 에 대상 기기의 코어 구성이 붙고 `xctrace export --toc` 에 `cpu-cores` 엘리먼트로 나옴 |

**방향이 [Xcode 27.2 RC 의 `GetCodeCoverage`](./2026-10-19-xcode-27-2-rc-coverage-mcp.md) 와 같음.** 그 문서가 짚은 것은 에이전트가 스크린샷 같은 "사람이 봐야 판정되는 산출물" 대신 **혼자 비교할 수 있는 숫자**를 처음 갖게 됐다는 것이었음. 트레이스는 그보다 한 단계 더 어려운 산출물이었음. `.trace` 는 GUI 로 여는 바이너리고, 에이전트에게 줄 방법이 없었음. Markdown 내보내기와 `triage` 보고서가 그 벽을 치움.

다만 **커버리지 때의 경고를 그대로 옮기면 안 됨.** 행 개수는 커버리지와 성격이 다름. 커버리지는 "줄이 실행됐다"만 세어 단정문 없는 테스트로도 올라가는 반면, 행은 **실제로 관측된 사용자 체감 증상**임. 그래서 지표로서는 덜 얕음. 대신 다른 한계가 있음. **트레이스는 그 세션에서 무엇을 조작했는지에 묶임.** 에이전트가 자기가 녹화한 짧은 트레이스에서 행이 0 이라고 보고하면 그건 앱이 안 멈춘다는 뜻이 아니라 **그 경로를 안 밟았다는 뜻**임. 녹화 시나리오를 사람이 고정해 두지 않으면 `triage` 보고서는 쉬운 시나리오를 고르는 쪽으로 최적화됨.

그리고 **"specialist" 라는 단어가 이번에 처음 나옴.** Apple 의 Xcode 에이전트 문서([Extending and customizing agents](https://developer.apple.com/documentation/xcode/extending-and-customizing-agents), [Xcode updates](https://developer.apple.com/documentation/updates/xcode))는 같은 영역을 `skills`, `subagents`, `expertise` 로 부르고 **`specialist` 라는 말을 쓰지 않음**(2026-11-19 확인). 이것이 스킬인지 서브에이전트인지, `/` 슬래시 커맨드로 직접 부를 수 있는지는 **확인할 수 없었음.** [Agent Skills 가 Linux Foundation 으로](./2026-10-22-agent-skills-standard-aaif.md)가 짚은 문제, 즉 같은 것을 제품마다 다르게 부르는 상태가 Apple 안에서도 정리되지 않았다는 신호로 읽는 편이 안전함.

## 에이전트 루프에 같이 걸리는 줄 셋

beta 2 의 다른 절에 있는데 자동화 환경에서 먼저 터질 항목들임.

- **시뮬레이터 런타임 검증이 강화됐음** (187122954). 노트가 직접 적은 부작용이 이것임. `/Library`, `/Library/Developer`, `/Library/Developer/CoreSimulator` 의 **파일시스템 권한을 완화해 둔 사용자는 시뮬레이터가 아예 발견되지 않을 수 있고**, 해결은 `root:wheel 755` 로 되돌리는 것임. CI 러너나 에이전트 전용 머신에서 sudo 를 피하려고 이 경로의 권한을 풀어 두는 구성이 드물지 않음. **그런 머신이 27.3 beta 2 에서 조용히 "기기 없음"이 됨**
- **`RenderPreview` MCP 툴 스냅샷의 해상도 버그가 고쳐짐** (190043246). 이미지가 다운스케일될 때 스냅샷 해상도가 틀릴 수 있었음. 에이전트가 프리뷰 스냅샷으로 UI 를 판정하게 해 뒀다면 **그 판정의 입력이 틀려 있던 구간**임
- **View Body 와 View Properties instrument 가 제거됐음** (188227249). SwiftUI 프로파일링은 SwiftUI instrument 로 가야 하고, 그쪽은 26 계열 이상만 지원함. 템플릿 이름을 박아 둔 스크립트나 에이전트 지시문이 있으면 그 줄이 깨짐

이 문서가 본 것은 **beta 2 에서 새로 붙은 절까지임.** 같은 페이지 아래쪽의 "Updates in Xcode 27.3 Beta" 절은 beta 1 내용이라 범위 밖으로 뒀음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode 27의 에이전트 표면](./2026-08-26-xcode-27-agent-surface.md) | 그 문서의 MCP 툴 표에서 로컬라이제이션 칸이 "읽고 계획하고 편집"이었음. 이 문서가 그 칸의 범위가 구조 변경과 state 까지 넓어진 것을 대조함 |
| [Xcode 27.2 RC, 에이전트가 커버리지를 읽게 됨](./2026-10-19-xcode-27-2-rc-coverage-mcp.md) | 같은 방향의 다음 걸음. 에이전트가 읽을 수 있는 측정 산출물이 커버리지에서 트레이스로 넓어짐. 지표로 쓸 때의 주의는 성격이 다름 |
| [Agent Skills 가 Linux Foundation 으로](./2026-10-22-agent-skills-standard-aaif.md) | "specialist" 가 스킬인지 서브에이전트인지 Apple 문서가 정리하지 않음. 명칭이 제품마다 갈리는 그 문제 |
| [.xcproj JSON 프로젝트 포맷](./2026-09-16-xcproj-json-project-format.md) | 키 순서 비결정성이 만드는 diff 와 머지 충돌. pbxproj 에서 풀린 문제가 String Catalog 쪽에 남아 있었음 |
| [에이전트 생태계 레포 지형도](./2026-08-10-agent-ecosystem-repos.md) | 그 문서의 캐시 무효화 표. **MCP 툴 정의가 늘면 prefix 앞쪽이 바뀌어 그 뒤 대화가 무효화됨.** 툴이 추가되는 릴리스마다 걸림 |
| [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md) | 에이전트가 자기 작업을 기록하는 필드를 직접 쓸 수 있을 때 생기는 문제. `machine_translated` 가 그 자리 |

## 짚어야 할 것

- **릴리스 노트와 Xcode 문서만 읽었고 직접 써보지 않았음.** 새 String Catalog 툴의 이름, 파라미터, `change state` 가 허용하는 값의 범위는 **노트에 없고 확인할 수 없었음**
- **새 로컬라이제이션 스킬의 이름과 발동 방식을 확인할 수 없었음.** 슬래시 커맨드로 노출되는지, 자동으로만 발동하는지 노트에 서술이 없음
- **"Apple specialists" 가 무엇인지 Apple 문서에 정의가 없음.** `specialist` 라는 단어가 Xcode 에이전트 문서 세 곳 어디에도 없음(2026-11-19 확인). 인용문의 `"The xctrace specialists helps"` 는 원문 그대로이고 Apple 쪽 오타로 보임
- **토큰 절감 수치가 없음** (190440819). "fewer tokens" 뿐이고 비교 기준도 없음
- **state 를 에이전트가 되돌릴 수 있다는 것은 이 문서의 판단임.** 노트는 `change state` 만 적고 그 위험을 말하지 않음. 제한이 걸려 있을 가능성도 열려 있음
- **beta 2 임.** 정식에서 항목이 빠지거나 문언이 바뀔 수 있고, beta 1 절과 beta 2 절이 한 페이지에 있으므로 다음 베타에서 페이지가 갱신되면 지금 문언을 다시 열 수 없음
- 이슈 번호는 Apple 이 붙인 것을 그대로 옮겼음

## 유효기간

**2026-11-19 확인 기준이고 Xcode 27.3 beta 2(27C5035i, 2026-11-16) 시점임.** Xcode 27.3 beta 2 는 Swift 6.4 를 포함하고 **macOS Tahoe 26.6 이상**을 요구함. 다시 볼 때 확인할 것 셋. **27.3 정식 노트에서 String Catalog 툴 항목의 문언이 그대로인지와 툴 레퍼런스가 Xcode 문서 쪽에 생겼는지**, **`specialist` 가 Apple 문서에 정의되는지 아니면 `skill`·`subagent` 로 흡수되는지**, 그리고 **`change state` 에 제한이 명시되는지**. 셋째가 가장 먼저 봐야 할 것이고, 제한이 없는 것으로 확인되면 위의 리뷰 권고를 팀 규칙으로 올릴 자리임. 릴리스 노트 페이지는 베타가 올라갈 때마다 덮어쓰이므로, 다시 볼 때는 [releases.rss](https://developer.apple.com/news/releases/rss/releases.rss) 로 어느 빌드 기준인지부터 확인할 것.
