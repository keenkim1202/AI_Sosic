---
title: Prompting Claude Opus 5 — 공식 프롬프팅 가이드
source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5
author: Anthropic
collected: 2026-08-10
tags: [claude, opus-5, prompt-engineering, effort, subagent, thinking]
---

출처: [Prompting Claude Opus 5 — Claude Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5)

## 요약

Opus 5는 기존 Opus 4.8 프롬프트로도 잘 돔. 다만 더 길게 말하고, 시키지도 않았는데 검증하고, 쉽게 위임하고, 자기 발언을 자주 정정함. 그래서 튜닝 방향이 **지시를 더하는 쪽이 아니라 예전에 넣어둔 지시를 빼는 쪽임.**

## 빼야 할 것

| 제거 대상 | 이유 |
|---|---|
| "마지막에 검증 단계를 넣어라" | 알아서 검증함. 과잉 검증으로 토큰만 태움 |
| "서브에이전트로 검증해라" | 위임 비용만 배가됨. 자기 검증에 서브에이전트 쓰지 말 것 |
| "답변 전에 다시 확인해라" | 자체 자기교정이랑 중첩돼서 비용만 오름 |
| (코드리뷰) "심각도 높은 것만", "보수적으로" | 문자 그대로 따라서 **덜 보고함** |
| "생각하지 마라", "추론하지 마라" | `<thinking>` 태그 유출을 오히려 늘림 |
| 이전 모델용 비전 우회책 | 이제 필요 없을 수 있으니 재검증 |
| 레거시 하네스의 별도 검증 스캐폴딩 | 위랑 같은 이유 |

## 넣어야 할 것

| 추가 대상 | 이유 |
|---|---|
| 간결성 지시 | effort는 생각의 양만 조절함. **응답 길이는 안 줄어듦** |
| 내레이션 카덴스 지시 | 에이전틱 작업 중 진행 상황을 많이 떠듦 |
| 문서 길이 보정 | 디스크에 쓰는 리포트가 이전보다 길어졌음 |
| 스코프 제약 | 요청 안 한 단계를 추가하는 경향이 있음 |
| 서브에이전트 상한 | 이전 모델보다 쉽게 위임함 |
| 정정 내레이션 억제 | 이전 발언을 정정한다고 더 많이 말함 |

## 능력 변화

멀티파일 작업이랑 대규모 리팩터에서 제일 강함. stub이나 placeholder를 안 남김. 문서가 권하는 최적 사용법은 **완전한 작업 명세를 처음에 다 주고 그대로 돌게 두는 것**임.

코드 리뷰는 precision이랑 recall이 둘 다 높고, 낮은 effort에서도 정확도가 유지됨. 덕분에 리뷰 시점에 빠른 패스 돌리고 나중에 정밀 패스를 한 번 더 도는 2단 구성이 가능함.

effort는 `low`랑 `medium`이 훨씬 적은 토큰이랑 지연시간으로 강한 품질을 냄. 기본값 `high`에서 시작해서 자체 eval로 조정하고, 어려운 작업만 `xhigh`로 올리라는 게 권장 방향임. 이전 모델 기본값을 물려받았으면 effort 스윕을 다시 돌려야 함.

나머지는 짧게:

- 비전. 차트랑 문서, 다이어그램, UI 복제에 강함. **도구를 쥐여주는 게 thinking을 늘리는 것보다 비용 효율이 좋음**
- 롱컨텍스트. 1M 토큰이 기본값이자 최대값이고, 전 구간에서 지시 따르기랑 툴 호출이 일관되게 유지됨
- Office 작업. 다중 시트 스프레드시트랑 슬라이드 덱을 만듦. 스타일이랑 템플릿은 프롬프트로 지정해야 함
- 멀티에이전트. writer-verifier 패턴이 잘 먹고 서로 덮어쓰는 경우가 드묾

## 프롬프트 스니펫

간결성. 사용자 대면 멀티턴 제품용:

```text
Keep responses focused, brief, and concise. Keep disclaimers and caveats short, and spend most of the response on the main answer. When asked to explain something, give a high-level summary unless an in-depth explanation is specifically requested.
```

시스템 프롬프트가 길면 끝 근처에 리마인더를 하나 더 둠:

```text
<tone_preference>
Keep outputs reasonably concise.
</tone_preference>
```

내레이션 카덴스:

```text
Before your first tool call, say in one sentence what you're about to do. While working, give a brief update only when you find something important or change direction. When you finish, lead with the outcome: your first sentence should answer "what happened" or "what did you find," with supporting detail after it for readers who want it.
```

문서에 따르면 "하지 마라"는 지시보다 원하는 스타일의 긍정 예시가 더 잘 먹음.

문서 길이 보정:

```text
Match the length of written documents to what the task needs: cover the substance, but do not pad with filler sections, redundant summaries, or boilerplate.
```

스코프 제약:

```text
Deliver what was asked, at the scope intended. Make routine judgment calls yourself, and check in only when different readings of the request would lead to materially different work. If the request seems mistaken or a better approach exists, say so in a sentence and continue with the task as asked rather than quietly narrowing, widening, or transforming it. Finish the whole task, and stop short of actions that are clearly beyond what was asked.
```

서브에이전트 통제:

```text
Delegate to a subagent only for large tasks that are genuinely independent and parallelizable, such as a wide multi-file investigation. Do not delegate work you can finish yourself in a handful of tool calls, and do not use subagents to verify or double-check your own work. If one subagent can complete the task, use one rather than several, and keep spawn counts low.
```

정정 내레이션 억제:

```text
Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it.
```

## Thinking을 끄고 쓸 때

Opus 5는 thinking이 기본 ON이고, effort `high` 이하에서만 끌 수 있음. 끄면 두 가지 문제가 간헐적으로 생김.

하나는 **툴 호출이 텍스트로 새는 것.** 구조화된 `tool_use` 대신 본문에 써버려서 호출이 실행되지 않음. 에이전틱 루프에선 그 텍스트가 대화 기록에 남아서 이후 턴까지 오염시킴. 검색처럼 툴 많이 쓰는 워크로드에서 흔함.

다른 하나는 `<thinking>` 같은 내부 XML 태그가 가시 응답에 섞이는 것.

문서가 제시하는 1차 완화책은 둘 다 **thinking을 켜 두고 effort를 낮추는 것**임. 비슷한 비용이면 thinking 끄고 effort 높이는 것보다, thinking 켜고 `low`로 두는 쪽이 나음.

그래도 꺼야 하면 결합 지시 하나로 두 문제를 같이 완화함:

```text
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

thinking 태그를 이름으로 콕 집어 언급하면 오히려 효과가 떨어진다고 함.

## 4.8 → 5 마이그레이션 체크리스트

- [ ] 검증 지시 제거 ("final verification step", "use a subagent to verify")
- [ ] 재확인 지시 제거 ("double-check", "re-verify before responding")
- [ ] 레거시 하네스 검증 스캐폴딩 제거
- [ ] "생각하지 마라" 류 규칙 제거
- [ ] 코드리뷰의 "보수적으로", "심각도 높은 것만" 제거. 전부 보고시키고 별도 패스에서 필터
- [ ] 간결성, 내레이션, 문서 길이, 스코프, 서브에이전트, 정정 억제 지시 추가
- [ ] 자체 eval로 effort 스윕 재실행
- [ ] 이전 모델용 비전 우회책 재검증 후 제거
- [ ] API 변경 확인. thinking 기본 ON, 비활성화는 effort `high` 이하에서만
- [ ] 1M 컨텍스트가 기본값임을 전제로 컨텍스트 전략 재검토

## 관련 문서

- [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) — reasoning level 변경은 툴 정의 변경이랑 마찬가지로 프롬프트 캐시를 무효화함. 실서비스에서 effort를 동적으로 바꾸면 캐시 비용을 물게 됨
