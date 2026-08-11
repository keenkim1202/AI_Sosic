---
title: Human Review — HTML·Markdown을 눈으로 고쳐 에이전트에 넘기는 스킬
source: https://github.com/petergyang/human-review
author: Peter Yang
collected: 2026-08-10
tags: [claude-code, skill, human-in-the-loop, review, markdown, html]
---

출처: [petergyang/human-review](https://github.com/petergyang/human-review) · [런치 포스트](https://creatoreconomy.so/p/use-my-human-review-skill-to-edit-html-markdown-visually)

## 요약

에이전트가 만든 HTML·Markdown을 브라우저에서 직접 고치고 구글 독스처럼 코멘트 단 다음, 한꺼번에 에이전트로 보내는 스킬임. 채팅으로 "세 번째 문단의 X를 Y로 바꿔줘"라고 받아쓰는 대신 그냥 커서로 고침. MIT, 별 654개.

## 문제 인식

README가 짚는 통증이 아주 구체적임. 문장 하나 바꾸고 싶을 뿐인데 채팅에는 이렇게 쓰게 됨.

> In the third paragraph, change X to Y. Cut the third card because it repeats the first one. Also rewrite the CTA.

그리고 에이전트가 파일을 고친 뒤에는 지시를 전부 제대로 이해했는지 사람이 다시 검사해야 함. 긴 계획서나 랜딩 페이지, 여러 페이지짜리 사이트를 리뷰할 땐 이게 급격히 힘들어짐.

## 설치

에이전트한테 이 문장을 그대로 붙여넣는 게 제일 쉬움.

```text
Install the /human-review skill globally from https://github.com/petergyang/human-review
```

npx도 됨.

```sh
npx -y human-review setup --global
```

## 사용

```text
/human-review (파일 경로)
/human-review (localhost URL)
```

브라우저 열리면 직접 고치고 코멘트 달고 Send 누름. 에이전트가 피드백을 한 배치로 받아서 소스를 수정하고 페이지를 새로고침함.

HTML 파일은 직접 편집이랑 리사이즈가 자동 저장됨. Markdown이랑 localhost 페이지는 Send를 눌러야 에이전트가 소스에 반영함.

![Human Review 편집 화면](../assets/human-review/human-review.png)

## 할 수 있는 것

- 텍스트 직접 편집이랑 기본 서식(굵게, 기울임)
- 불릿·번호 목록. 줄 앞에 `- `나 `1. `을 치거나 ⌘⇧8 / ⌘⇧7. Tab이랑 Shift+Tab으로 들여쓰기
- 링크 추가. 텍스트 선택하고 ⌘K. 기존 링크 안에서 ⌘K 누르면 편집·삭제
- 이미지 리사이즈(모서리 드래그)랑 이동(드래그)
- 블록 재배치. 블록에 호버하면 왼쪽에 나오는 핸들을 잡아서 옮김
- 클립보드 이미지 붙여넣기. 파일 옆 `assets/` 폴더에 저장되고 커서 위치에 삽입됨
- 특정 문구를 선택해서 그 텍스트에 앵커된 코멘트 달기
- 이미지·차트·섹션을 클릭해서 코멘트
- 삭제한 이유를 채팅에 설명하지 않고 요소 제거
- 링크를 Command-클릭해서 여러 페이지를 피드백 잃지 않고 리뷰
- 모든 편집이랑 코멘트를 한 번에 전송

만든 사람은 AI가 만든 계획서 수정, 랜딩 페이지 업데이트, localhost 앱 리뷰, 그리고 "AI가 UX에 자꾸 덧붙이는 군더더기 카피 제거"에 쓴다고 적었음.

## 구성

| 파일 | 역할 |
|---|---|
| `src/cli.js` | `human-review`, `poll`, `status`, `setup` 명령 |
| `src/server.js` | 로컬 리뷰 세션 |
| `src/sdk.js` | 편집, 코멘트, 하이라이트, 피드백 처리 |
| `src/chrome-client.js` | 비주얼 리뷰 인터페이스 |
| `src/markdown.js` | Markdown 렌더링 |
| `src/SKILL.md` | Claude Code, Codex 등에 사용법을 가르치는 스킬 정의 |

전부 로컬에서 돔. 계정, 클라우드, DB, API 키가 필요 없음.

## 메모

- 에이전트가 뽑은 산출물이 문서나 웹페이지인 워크플로에서 바로 값이 나옴. 반대로 코드 diff 리뷰에는 안 맞음. 그쪽은 AO나 Orca의 diff 주석 기능이 담당함
- Claude Code 전용이 아니라 SKILL.md 규약을 읽는 에이전트면 붙음. README는 ChatGPT, Claude Code, Codex를 예로 듦
