---
title: /code-review 의 기본 대상이 브랜치 전체가 됨, 보고 범위가 버그로 좁아지고 심각도 셋이 붙은 자리
source: https://code.claude.com/docs/en/code-review-local
author: Anthropic
published: 2026-11-19
collected: 2026-11-23
tags: [claude-code, code-review, coding-agent, subagent, effort, human-in-the-loop, guardrail]
---

출처: [Review locally with /code-review, Claude Code Docs](https://code.claude.com/docs/en/code-review-local) · [Commands, Claude Code Docs](https://code.claude.com/docs/en/commands) · [Claude Code v2.1.328 릴리스 노트](https://github.com/anthropics/claude-code/releases/tag/v2.1.328) (`gh api` 의 `published_at` 이 **2026-11-19T01:14:04Z**) · [v2.1.329](https://github.com/anthropics/claude-code/releases/tag/v2.1.329) (**2026-11-19T22:14:41Z**) · [v2.1.330](https://github.com/anthropics/claude-code/releases/tag/v2.1.330) (**2026-11-20T22:32:56Z**, `gh api` 기준 최신) · 문서 확인 **2026-11-23**

## 요약

하루 사이에 나온 두 릴리스가 `/code-review` 의 **기본 대상**, **보고 범위**, **결과 형식**을 한꺼번에 바꿨음. 셋 다 명령 이름은 그대로라 모르고 쓰면 전과 다른 것을 받게 됨. 첫째, **대부분 모델에서 버그와 CLAUDE.md 위반만 보고**하게 됐고 재사용·단순화·효율은 `/simplify` 로 넘어갔음. 둘째, 각 지적에 **Important / Pre-existing / Nit** 라벨이 붙고 요약 끝에 수정안이 붙음. 셋째, 타깃 없이 치면 **브랜치 전체를 기본 브랜치에 대고** 봄. 그리고 운영상 가장 밟기 쉬운 것은 이 셋이 아니라 `--fix` 임. 리뷰가 **백그라운드 서브에이전트로 돌기 때문에 그 수정은 세션 체크포인트 밖에 있고, `/rewind` 로 못 되돌림.** 셋 중 심각도 라벨은 **릴리스 노트에만 있고 문서 페이지에는 아직 없음.**

## 무엇이 언제 바뀌었나

| 바뀐 것 | 릴리스 | 릴리스 노트 문언 |
|---|---|---|
| 보고 범위 | v2.1.328 | "Changed `/code-review` on most models to report only bugs and CLAUDE.md violations; run `/simplify` for reuse, simplification and efficiency cleanups" |
| 심각도 라벨 | v2.1.328 | "Changed `/code-review` to mark each finding Important, Pre-existing or Nit on most models, and to end each summary with a suggested fix; a clean review's list now reads \"Code review found no issues.\"" |
| 기본 대상 | v2.1.329 | "Changed `/code-review` with no target on most models to review the whole branch against the default branch, and fixed it losing the diff on multi-commit branches" |
| PR 타깃 처리 | v2.1.329 | "Changed `/code-review <pr number> --fix`, and `/code-review` given a pull request URL, to stop without reviewing when `gh pr diff` is refused or fails" |

네 줄 중 셋에 **"on most models"** 가 붙어 있음. 어느 모델이 해당하는지는 릴리스 노트도 문서도 적지 않았고 **확인할 수 없었음.** 즉 같은 저장소에서 모델만 바꿔 치면 범위와 라벨이 달라질 수 있다는 뜻이고, 리뷰 결과를 CI 게이트에 물려 둔 쪽에서는 이게 조용한 변경임.

## 버그만 본다는 것의 의미

문서 페이지는 범위를 이렇게 적음.

> "Claude reviews the changes on your branch, looks for correctness bugs, and reports what it found in your conversation. Depending on your model and effort level, the review also covers reuse, simplification, and efficiency cleanups."

릴리스 노트는 "most models 에서 버그와 CLAUDE.md 위반만", 문서는 "모델과 effort 에 따라 정리 항목도 포함". **두 문언이 같은 것을 다르게 적고 있음.** 실무에서는 문서 쪽을 따라 읽는 편이 안전함. 정리 항목이 올 수도 있고 안 올 수도 있다고 보고, 정리를 반드시 받아야 하면 `/simplify` 를 따로 치는 것.

`/simplify` 쪽 명령 레퍼런스 문언이 그 분업을 못 박아 둠.

> "Review the changed code for cleanup opportunities and apply the fixes. Four review agents run in parallel, covering reuse of existing helpers, simplification, efficiency, and whether the change is at the right level of abstraction. The review doesn't look for correctness bugs. Use `/code-review` to find bugs."

즉 **`/simplify` 는 네 개를 병렬로 돌리고 기본으로 고침**, `/code-review` 는 보고만 하고 `--fix` 를 줘야 고침. 비용이 다른 두 명령임. 그리고 레퍼런스가 이름 이력도 적어 둠. v2.1.147 전까지 지금의 `/code-review` 가 `/simplify` 였고 기본으로 고쳤음. 스크립트에 옛 이름이 박혀 있으면 지금은 다른 일을 함.

## 타깃 문법

문서가 표로 적어 둔 것임. 효력 순서는 `/code-review [effort] [flags] [target]` 이고 **effort 와 플래그 뒤의 나머지 전부가 타깃 문자열**임.

| 보고 싶은 것 | 치는 것 |
|---|---|
| 현재 브랜치와 커밋 안 한 변경 | `/code-review` |
| 최근 15 커밋 | `/code-review HEAD~15..HEAD` |
| 브랜치 대 브랜치 | `/code-review main...my-feature` |
| 커밋 하나 | `/code-review abc1234^..abc1234` |
| PR 번호 | `/code-review 1234` |
| 경로 | `/code-review src/auth` |

타깃이 없을 때의 동작을 문서는 이렇게 적음.

> "With no target, `/code-review` reviews your branch's commits ahead of its upstream branch plus your uncommitted changes. If the branch has no upstream, Claude compares against `main`, or against the previous commit."

**여기가 v2.1.329 의 변경이 내려앉은 자리임.** 전에는 "current diff" 로 적혀 있었고 멀티커밋 브랜치에서 diff 를 잃는 버그가 같이 고쳐졌음. 그래서 **커밋 세 개짜리 브랜치에서 마지막 커밋만 보던 리뷰가 이제 셋 다 봄.** 토큰이 늘어나는 변경이기도 함.

⚠️ **명령 레퍼런스는 아직 "the current diff" 로 적혀 있음.** 같은 사이트의 두 페이지가 어긋나 있고, 상세 페이지 쪽이 새 동작을 반영한 것으로 읽었음. Apple 릴리스 노트 계열에서 자주 겪는 것과 같은 종류의 어긋남이라 **인용할 때는 상세 페이지를 쓰는 편이 맞음.**

그리고 `ultra` 에서는 **같은 줄을 다르게 읽음.**

- `ultra` 있음: 한 단어면 base 브랜치나 PR 번호로 읽고, 브랜치도 PR 도 아닌 긴 문장은 **리뷰에 붙는 메모**가 됨. `/code-review ultra check my auth changes` 는 현재 브랜치를 보고 그 메모에 결과를 맞춰 줌
- `ultra` 없음: 나머지 전부가 타깃임. **다른 명령 이름으로 시작해도 그럼.** `/code-review /fix-issue 123` 은 `/fix-issue` 를 스킬로 로드하지 않고 `/fix-issue 123` 이라는 타깃 문자열로 받음

## 결과가 두 가지 형태로 나옴

| 실행 자리 | 형태 |
|---|---|
| 터미널 세션 | 포크된 서브에이전트가 돌고 결과가 **대화에 텍스트**로 들어옴 |
| `-p` 실행 (text·JSON) | 같은 텍스트가 응답에 들어옴 |
| 호스트 앱이 리스트를 요청할 때 (데스크톱 앱 등) | **`ReportFindings` 툴**로 보고. 파일 위치, 한 문장 요약, `correctness` 같은 카테고리 태그 |

`ReportFindings` 경로에는 상한이 있음. 문서가 "Where the review reports through the findings tool, the limit is 32" 라고 적음. `--max-findings n` / `all` / `default` 로 바꾸는데, **한 번 준 값이 다음 리뷰에도 계속 쓰임**(`default` 를 주기 전까지). 단 비대화형 `-p` 에서 준 값은 기억되지 않음. 이 플래그는 **v2.1.288 이상**이 필요함.

여기서 실무 함정 하나. **같은 명령이 호스트에 따라 다른 형태로 보고하는데, 상한 32 는 리스트 경로에만 걸림.** 터미널에서 돌려 보고 "지적이 다 나왔다"고 판단한 뒤 데스크톱 앱에서 같은 리뷰를 돌리면 잘릴 수 있음.

고친 뒤의 재보고도 문서에 있음. "Claude Code then marks each finding in the updated findings list as fixed, skipped, or no change needed." 즉 **수정 후 상태가 세 가지**로 갈림. 라벨 셋(Important / Pre-existing / Nit)과는 다른 축임.

## effort 가 범위를 정함

> "At `low`, the review reports the findings it's most confident in, so you see fewer false positives. From `medium` through `max`, the review broadens coverage."

그리고 레벨을 안 치면 **이전에 친 레벨을 세션을 넘어 재사용**함. 문서가 규칙을 네 줄로 적어 둠.

- 레벨을 치면 그게 이후 재사용 값이 됨
- 비대화형 `-p` 에서 준 레벨은 재사용 값을 **갱신하지 않음**
- `ultra` 는 재사용 값을 **쓰지도 갱신하지도 않음**
- 한 번도 친 적 없으면 세션의 현재 effort 를 씀

[Claude Code 의 effort 해석 순서](./2026-10-27-claude-code-effort-resolution.md)에 정리한 다섯 단계와는 **별개의 기억 장치**임. 그쪽은 세션의 effort 를 정하는 경로고, 이쪽은 `/code-review` 전용으로 따로 남는 값임. 그래서 `Reusing high effort, the level you typed last time` 같은 공지가 뜨는데, **몇 주 전 다른 저장소에서 친 레벨이 오늘 리뷰 비용을 정할 수 있음.** 비용을 재고 있다면 매번 레벨을 명시하는 편이 낫고, v2.1.329 에서 "리뷰가 medium 으로 돌았는데 카드에 high 로 표시되던" 버그가 고쳐졌다는 것도 같이 알아야 함. 그 버그가 있던 동안의 스크린샷은 근거로 못 씀.

## --fix 가 체크포인트 밖에 있음

이 문서에서 가장 조심해서 볼 부분임. 문서가 직접 적음.

> "a background review applies its `--fix` edits outside your session's checkpoints. After one, `/rewind` restores only the files Claude had already edited in your conversation. The review's edits to other files stay, so use git to revert them."

정리하면 이럼.

| 실행 방식 | `/rewind` 로 되돌아감? |
|---|---|
| 백그라운드(터미널 기본) | **아님.** 대화에서 이미 고쳤던 파일만 복구되고 나머지는 남음 |
| 포그라운드 | 맞음. 내 턴 안에서 고치므로 평소대로 복구됨 |

포그라운드로 돌리는 방법은 셋임. `-p` 나 Agent SDK 로 돌리거나, `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` 을 켜거나(다른 백그라운드 기능도 같이 꺼짐), **이미 리뷰가 도는 중에 `/code-review` 를 또 치는 것**.

그러니까 `--fix` 를 쓸 거면 **커밋해 두고 돌리는 게 맞음.** `/rewind` 를 안전망으로 믿고 있으면 그 믿음이 여기서 깨짐. 이것은 [Claude Code 의 작업 디렉터리 바깥 쓰기 차단](../security/2026-10-30-working-directory-write-block.md)이 다루는 울타리와 다른 층의 문제임. 울타리 안쪽의, 승인까지 받은 수정이 복구 경로 밖에 있는 것임.

읽기·쓰기 범위에 대해 문서가 적어 둔 나머지 둘.

- 리뷰는 **`CLAUDE.md` 를 따름.** 비대화형 `-p` 에서도 그럼
- **`REVIEW.md` 는 읽지 않음.** 그 파일은 GitHub PR 을 보는 관리형 Code Review 서비스 쪽 설정임

앞의 것이 v2.1.328 의 "버그와 CLAUDE.md 위반만" 과 맞물림. **보고 범위가 좁아진 만큼 `CLAUDE.md` 가 리뷰 기준의 거의 전부가 됐음.** [CLAUDE.md 를 프롬프트로 감사하기](../prompting/2026-09-25-claude-md-prompt-audit.md)가 다룬 것, 즉 지시 파일에 든 모순과 사문화된 규칙을 터는 작업의 값어치가 여기서 한 단계 올라감. 리뷰가 지적하는 "위반" 의 모수가 그 파일이기 때문임. v2.1.329 에서 **"없는 `~/.claude/CLAUDE.md` 를 찾느라 권한 프롬프트에서 멈추던"** 버그가 고쳐졌다는 것도 같은 자리의 항목임.

## ultra 로 올릴 때

`/code-review ultra` 는 클라우드에서 도는 다른 물건임. 범위도 다름.

- 비교 대상: **현재 브랜치 대 저장소의 기본 브랜치**, 그리고 워킹 트리의 커밋 안 한 변경과 스테이지된 변경
- base 를 바꾸려면 브랜치 이름을 줌 (`/code-review ultra develop`)
- `.env`, `*.tfvars` 처럼 **자격증명 이름을 한 커밋 안 한 파일**은 로컬 저장소를 클라우드 세션에 올릴 때의 규칙을 따름

그리고 안 되는 자리가 문서에 박혀 있음. **claude.ai 계정 인증이 필요하고**, Amazon Bedrock·Google Cloud Agent Platform·Microsoft Foundry 에서는 안 되며, **Zero Data Retention 이나 HIPAA 구성 조직**에서도 안 됨. 이 경우들에서는 `/code-review ultra` 가 조용히 로컬 리뷰로 돌아감. **CMEK 를 쓰는 조직은 아예 못 돌림.** 비용 쪽은 명령 레퍼런스가 "Includes 3 free runs on Pro, Max, and Ultra, then requires usage credits" 라고 적음. v2.1.330 은 Team·Enterprise 플랜에서 **남은 월 지출 한도가 리뷰 1회 평균 비용보다 적을 때 그 사실을 확인 창에 표시**하도록 고쳤음.

세션 스코프 예약 작업으로 `/code-review` 를 걸 수 있는데, **예약 작업은 클라우드 리뷰를 절대 띄우지 않음.** 그래서 예약에는 `ultra` 를 빼고 걸라고 문서가 적음.

## Claude 가 스스로 리뷰를 시작하는 것을 막기

v2.1.246 부터 Claude 가 **말로 부탁만 해도** 이 스킬을 돌릴 수 있게 됐음. 내가 칠 때만 돌게 하려면 설정 파일에 이렇게 둠.

```json
{
  "skillOverrides": {
    "code-review": "user-invocable-only"
  }
}
```

이러면 Claude 도 예약 작업도 리뷰를 시작하지 못하고 `/code-review` 는 그대로 쓸 수 있음. **비용을 내가 쥐고 싶을 때 쓰는 스위치임.** 번들 스킬이라 디스크에 `SKILL.md` 가 없어서 지시문을 열어 고칠 수는 없고, 프로젝트나 개인 스킬에 `code-review` 라는 이름을 두면 그게 번들 것을 대신함. 단 **`/review` 별칭은 계속 번들 리뷰를 돌림.** 내 스킬로 갈아 끼웠다고 생각하고 `/review` 를 치면 다른 게 돎.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Code 의 effort 해석 순서](./2026-10-27-claude-code-effort-resolution.md) | effort 가 리뷰 범위를 정하는 손잡이임. 단 `/code-review` 는 세션과 별개로 마지막 레벨을 기억함 |
| [CLAUDE.md 를 프롬프트로 감사하기](../prompting/2026-09-25-claude-md-prompt-audit.md) | 보고 범위가 버그와 CLAUDE.md 위반으로 좁아져 그 파일이 리뷰 기준의 모수가 됨 |
| [Eval Engineering, 머지 게이트](./2026-08-01-eval-engineering-merge-gate.md) | 리뷰를 머지 조건으로 쓸 때의 문제. "on most models" 라는 조건부 동작이 게이트의 재현성을 깎음 |
| [Claude Sonnet 5.5](../models/2026-09-28-claude-sonnet-5-5.md) | 그 문서가 인용한 Cognition 의 관찰, 즉 max effort 에서 code-review 스킬이 서브에이전트를 많이 쪼개 범위 밖 수정과 타임아웃을 냈다는 것. 지금의 effort 규칙이 그 관찰과 바로 붙음 |
| [Claude Code 작업 디렉터리 바깥 쓰기 차단](../security/2026-10-30-working-directory-write-block.md) | 울타리 바깥 쓰기와는 다른 층. 이쪽은 울타리 안쪽 수정이 복구 경로 밖에 있는 문제 |
| [Claude Code 권한 우회 수정 13건](../security/2026-10-22-claude-code-permission-bypass-fixes.md) | `claude ultrareview` 가 workspace trust 를 요구하게 된 변경. 같은 명령의 신뢰 경계 쪽 |

## 짚어야 할 것

- **심각도 라벨 셋(Important / Pre-existing / Nit)은 릴리스 노트에만 있음.** 문서 페이지와 `ReportFindings` 툴 레퍼런스 어디에도 없고, 둘 다 `correctness` 같은 **카테고리** 태그만 적음. 라벨과 카테고리가 같은 필드인지 별개인지는 **확인할 수 없었음.** 라벨로 필터링하는 스크립트를 짜기 전에 실제 출력을 한 번 보고 할 것
- **"on most models" 의 모델 목록이 어디에도 없음.** 범위·라벨·기본 대상 셋이 전부 이 조건부라서, 모델을 바꾸면 같은 명령이 다른 것을 돌려줄 수 있음. 어느 모델이 예외인지는 **확인할 수 없었음**
- **명령 레퍼런스와 상세 페이지가 기본 대상에서 어긋남.** 전자는 "the current diff", 후자는 "your branch's commits ahead of its upstream branch". 릴리스 노트가 후자 쪽 변경을 적었으므로 후자를 따라 읽었지만, Anthropic 이 전자를 갱신한 공지는 확인하지 못했음
- **문서만 읽었고 직접 돌려보지 않았음.** 타깃 표와 플래그는 문서 표를 옮긴 것이고, 실제 출력 형식은 대조하지 않았음
- `/simplify` 가 "네 개의 리뷰 에이전트를 병렬로" 돌린다는 것은 명령 레퍼런스 한 줄이 출처임. **그 네 에이전트가 각각 무슨 모델과 effort 로 도는지는 적혀 있지 않고 확인하지 못했음**
- 릴리스 노트의 **"a clean review's list now reads \"Code review found no issues.\""** 는 문자열이 바뀐 것임. 이 문자열로 CI 가 성공을 판정하고 있었다면 그 자리는 이번에 깨졌음

## 유효기간

**2026-11-23 확인 기준이고 변경 자체는 v2.1.328·v2.1.329(둘 다 2026-11-19), 최신 릴리스는 v2.1.330(2026-11-20)** 임. 이 릴리스는 하루에 한두 개씩 나오므로 버전 번호는 읽는 시점에 이미 낡았다고 보는 게 맞음. 다시 볼 때 확인할 것 셋. **심각도 라벨이 문서 페이지나 `ReportFindings` 레퍼런스에 적혔는지**(적히면 위 첫 유보가 풀림), **"on most models" 의 대상 목록이 공개됐는지**, 그리고 **명령 레퍼런스의 "current diff" 문언이 갱신됐는지.** 버전별 변경은 `gh api repos/anthropics/claude-code/releases` 로 받아 `code-review`·`simplify`·`ultrareview` 줄만 훑어도 대부분 잡힘.
