---
title: Opus 4.8 fast mode 폐기, 모델은 2027년까지 사는데 기능만 11월 30일에 사라짐
source: https://platform.claude.com/docs/en/release-notes/api
author: Anthropic
published: 2026-10-30
collected: 2026-11-02
tags: [claude, model-deprecation, migration, llm-cost, inference, latency]
---

출처: [API 릴리스 노트 2026-10-30 항목](https://platform.claude.com/docs/en/release-notes/api) · [Fast mode](https://platform.claude.com/docs/en/build-with-claude/fast-mode) · [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations) · [Opus 5.5 마이그레이션 가이드](https://platform.claude.com/docs/en/models/opus-5-5/migration-guide) · [Fast mode pricing](https://platform.claude.com/docs/en/about-claude/pricing#fast-mode-pricing) · [Get Token Usage By Dimension](https://platform.claude.com/docs/en/api/beta/organization/analytics/usage/list) (전부 **2026-11-02 확인**)

## 요약

**2026-10-30 에 Claude Opus 4.8 의 fast mode 가 폐기 예고됐고 제거일은 2026-11-30 임.** 그 뒤로 `claude-opus-4-8` 에 `speed: "fast"` 를 보내면 에러가 남. 중요한 것은 폐기 대상의 범위임. **모델 자체는 Active 로 남아 있고 은퇴 예정일이 2027-05-28 보다 빠르지 않음.** 즉 모델 ID 를 그대로 둔 코드는 계속 돌고, **`speed` 한 줄만 있는 요청이 7개월 넘게 남은 모델 위에서 먼저 죽음.**

그래서 이 건은 이 저장소가 지금까지 추적해 온 폐기와 모양이 다름. [모델 은퇴 일정](../models/2026-09-30-claude-model-retirement-schedule.md)이 대조 기준으로 쓰는 **Model deprecations 페이지의 세 목록 어디에도 이 폐기가 없음.** 상태 표에도, 폐기 이력에도, API 파라미터 폐기 표에도 없고 적힌 곳은 **릴리스 노트 한 줄과 fast mode 문서의 Warning 블록 둘뿐임.** 모델 은퇴에 걸린 **최소 60일 공지** 약속도 이 건에는 적용되지 않음. 실제 창은 **31일**임.

## 폐기된 것은 모델이 아니라 기능임

| 축 | `claude-opus-4-8` 모델 | `claude-opus-4-8` + `speed: "fast"` |
|---|---|---|
| 상태 | **Active** | **Deprecated**(2026-10-30 예고) |
| 끝나는 날 | 은퇴 예정일이 **2027-05-28 보다 빠르지 않음** | **2026-11-30 제거** |
| 그 뒤 동작 | 계속 동작 | 요청이 **에러** |
| 적힌 곳 | Model deprecations 상태 표 | 릴리스 노트, fast mode 문서 Warning |

릴리스 노트 문언은 이렇게 적혀 있음.

> "We've deprecated fast mode for Claude Opus 4.8, with removal on November 30, 2026. After removal, requests to `claude-opus-4-8` with `speed: "fast"` will return an error."

fast mode 문서의 Warning 도 같은 내용인데, **같은 페이지의 지원 표는 아직 Opus 4.8 을 research preview 로 그대로 싣고 있음.** 베타 헤더 표에도, 페이지 메타데이터의 `supportedModels` 에도 `claude-opus-4-8` 이 남아 있음. 지원 표만 읽으면 폐기를 모르고 지나감. 표와 경고가 어긋난 상태라서, **이 페이지에서 믿을 것은 Availability 절의 Warning 쪽임.**

## 어느 폐기 목록에도 없다는 것

Model deprecations 페이지에 목록이 셋 있는데 셋 다 비어 있음.

| 목록 | 이 건이 있는가 |
|---|---|
| Model status 표 | 없음. `claude-opus-4-8` 은 Active, 은퇴 예정 2027-05-28 이후 |
| Deprecation history | 없음. 최신 항목이 **2026-09-30 Claude Sonnet 4.5** 임 |
| API parameter deprecations | 없음. `temperature`·`top_p`·`top_k` 세 개뿐임 |

파라미터 표에 없는 것은 어떻게 보면 맞음. 폐기된 것은 `speed` 파라미터가 아니라 **한 모델에서의 `fast` 값 하나**라서 그 표의 단위에 안 맞음. 문제는 **그러면 이 종류의 변경이 들어갈 자리가 아예 없다는 것**임. 페이지 설명 자신이 범위를 "models and API parameters" 로 적어 두었고, 모델과 파라미터 사이에 있는 조합은 범위 밖임.

운영상 결론 하나가 여기서 나옴. **폐기 페이지를 주기적으로 대조하는 것만으로는 이 계열을 못 잡음.** [모델 은퇴 일정](../models/2026-09-30-claude-model-retirement-schedule.md)이 재확인 방법으로 적어둔 "상태 표와 deprecation history 첫 항목만 대조하면 됨" 이 이 건에서는 통하지 않음. 릴리스 노트를 같이 봐야 함.

공지 창도 같은 이유로 짧음. 같은 페이지가 모델 은퇴에 대해서는 약속을 적어 둠.

> "Anthropic notifies customers with active deployments for models with upcoming retirements, providing at least 60 days’ notice before model retirement for publicly released models."

문언이 **모델 은퇴**에 한정돼 있음. 이 건은 모델 은퇴가 아니라 기능 제거이므로 그 약속의 대상이 아니고, 2026-10-30 에서 2026-11-30 까지 **31일**임. ⚠️ 이메일 공지가 실제로 나갔는지는 **확인할 수 없었음.**

## 내가 걸리는지 한 번에 확인하는 방법

4.8 fast 트래픽이 있는지는 추측하지 않고 조회할 수 있음. [Usage and Cost Analytics API](2026-10-29-usage-cost-analytics-api.md)에 `speeds[]` 필터와 `speed` 그룹화 차원이 둘 다 있음. 레퍼런스에서 확인한 값임.

```
GET /v1/organizations/analytics/usage
  ?starting_at=2026-10-02T00:00:00Z
  &models[]=claude-opus-4-8
  &speeds[]=fast
  &group_by[]=product
  &group_by[]=api_key_id
```

- 헤더는 `anthropic-beta: analytics-usage-cost-2026-08-24` 이고 키는 `read:analytics` 스코프 또는 Admin API 키임
- 한 번에 조회할 수 있는 범위가 **최대 31일**이므로 preview 를 쓰기 시작한 시점까지 거슬러 보려면 호출을 나눠야 함
- `group_by[]=product` 를 넣는 이유가 있음. 제거 경고가 Messages API 와 Managed Agents 를 갈라 적지 않는데, `products[]` 값에 `messages_api` 와 `managed_agents_api` 가 따로 있어서 **어느 표면에서 나온 트래픽인지는 이쪽으로 가름**
- 행이 비면 끝임. 행이 있으면 `api_key_id` 가 고칠 코드를 가리킴
- 신선도가 보통 **4시간**이라 방금 보낸 요청은 안 보임. 11-30 직전에 "0 이니 괜찮다"로 판단하지 말 것

## 옮길 곳은 셋이고 성격이 다름

| 선택 | 상태 | 요구 | 걸리는 점 |
|---|---|---|---|
| Opus 5.5 + fast | **정식** | 베타 헤더 불필요, 접근 신청 불필요 | 마이그레이션이 필요함(아래 절) |
| Opus 5 + fast | research preview | `fast-mode-2026-02-01` 헤더 + 접근 신청 | preview 에 남아 있어 같은 일이 또 올 수 있음 |
| Opus 4.8 유지 + `speed` 제거 | 모델은 Active | 없음 | 코드 한 줄 삭제. 출력 속도를 포기함 |

단가는 이렇게 갈림. 모두 per MTok 이고 가격 페이지에서 받은 값임.

| 항목 | 4.8 표준 | 4.8 fast | 5.5 표준 | 5.5 fast |
|---|---|---|---|---|
| base input | $5 | **$10** | $4 | **$8** |
| output | $25 | **$50** | $20 | **$40** |
| 캐시 적중 | $0.50 (0.1배) | 표에 행이 없음 | $0.20 (0.05배) | 표에 행이 없음 |

세 번째 선택지를 먼저 보는 편이 맞는 경우가 있음. [Fast mode 정식 공개](2026-10-15-fast-mode-opus-5-5-ga.md)가 정리한 대로 fast mode 가 줄이는 것은 **OTPS 뿐이고 TTFT 가 아니므로**, 체감 지연이 첫 글자 대기에서 오던 경로라면 `speed` 를 떼는 것이 기능 손실이 아니라 **단가 절반 환원**임. 11-30 이 강제하는 선택을 비용 재검토의 계기로 쓰는 쪽이 낫고, 이건 코드 한 줄 삭제로 끝남.

Opus 5 로 피하는 선택은 권할 만하지 않음. 가격이 4.8 fast 와 같은 $10 / $50 이고, **fast mode 가 여전히 research preview 라서 헤더와 접근 신청이 그대로 필요함.** 이번 폐기가 preview 구간에서 일어났다는 것 자체가 그 구간의 수명을 말해 줌.

## Opus 5.5 로 옮기는 값은 `speed` 한 줄이 아님

릴리스 노트는 "migrate to Claude Opus 5.5" 로 끝나지만 모델 교체 쪽 요구사항이 따로 있음. 마이그레이션 가이드의 4.8 절과 공통 절에서 확인한 것임.

| 바뀌는 것 | 4.8 | 5.5 |
|---|---|---|
| thinking | 요청하지 않으면 **꺼짐** | `thinking` 필드가 없어도 **돌고, 끌 수 없음** |
| `thinking: {"type": "disabled"}` | 받아짐 | **제거해야 함** |
| effort 기본값 | `high` | **`medium`** |
| `tool_choice` 의 `any`·`tool` | 받아짐 | `auto` + strict tool use 또는 structured outputs 로 교체 |
| computer use | `computer_20251124` | Claude API 에서 **`computer_toolset_20260801`** |
| 캐시 최소 길이 | 1,024 토큰 | **512 토큰** |
| Priority Tier | 지원 | **미지원** |

여기서 fast mode 쓰는 쪽에 실제로 아픈 줄은 **첫 줄 하나**임.

4.8 에서 `thinking` 을 안 보내던 코드는 5.5 에서 thinking 과 함께 돌고, 가이드가 과금을 못 박음.

> "Thinking tokens are billed as output tokens, so these workloads can produce more output tokens per request."

**fast mode 의 output 단가가 $40 / MTok 인 자리에서 output 토큰 수가 늘어남.** `max_tokens` 는 thinking 과 응답 텍스트를 합친 하드 리밋이라 같이 다시 잡아야 하고, `xhigh`·`max` effort 를 쓰면 가이드가 **64k 부터 시작**하라고 적음. effort 기본값이 `high` 에서 `medium` 으로 내려가는 것까지 겹치므로, **`effort` 를 명시하지 않으면 생각은 조용히 줄고 출력 토큰은 늘어나는 조합**이 됨. effort 해석 순서는 [Claude Code 의 effort 결정 순서](2026-10-27-claude-code-effort-resolution.md)에 정리돼 있음.

반대로 **Priority Tier 미지원은 fast mode 사용자에게는 애초에 해당하지 않음.** fast mode 문서의 Considerations 가 **Priority Tier 약정과 같이 쓸 수 없다**고 적어 두었으므로, 4.8 fast 를 쓰고 있었다면 그 약정이 없는 상태임. 마이그레이션 가이드의 경고 셋 중 하나는 이 경로에서 자동으로 지워짐.

캐시 최소 길이가 1,024 에서 512 로 내려가는 것은 **코드 변경 없이 유리해지는 쪽**임. 짧아서 캐시가 안 걸리던 프롬프트가 항목을 만들게 됨. 캐시 적중 배수도 0.1배에서 0.05배로 내려가므로 prefix 를 길게 들고 도는 루프는 모델 교체만으로 단가가 내려감. ⚠️ 다만 fast 단가 위의 캐시 가격은 여전히 가격표에 행이 없음. [Fast mode 정식 공개](2026-10-15-fast-mode-opus-5-5-ga.md)가 같은 공백을 적어둔 자리임.

## 릴리스 노트가 가리킨 문서에 fast mode 가 없음

릴리스 노트는 이전 안내로 [Migrating from Claude Opus 4.8](https://platform.claude.com/docs/en/models/opus-5-5/migration-guide#migrating-from-claude-opus-4-8) 를 지목함. 그런데 **그 절에는 `speed` 도 fast mode 도 나오지 않음.** 가이드 전체에서 fast mode 를 언급하는 곳은 **Opus 4.7 절**이고, 거기서는 4.7 이 fast mode 를 거절한다는 맥락임.

> "If you want fast mode, which Claude Opus 4.7 rejects, set `speed: "fast"` on your Claude API requests. Claude Opus 5.5 doesn’t need a beta header."

4.8 에서 올라오는 사람이 링크를 따라가면 **자기가 왜 왔는지에 대한 안내는 못 만남.** 실제로 필요한 두 줄은 다른 데 흩어져 있음. `speed: "fast"` 는 그대로 두면 되고, `fast-mode-2026-02-01` 헤더는 5.5 에 필요 없지만 **계속 보내도 동작함**(fast mode 문서 기준). 즉 헤더를 떼는 작업은 선택이고, 모델 ID 교체와 thinking 처리가 필수임.

## 2026-10-16 이후 fast mode 문서에서 바뀐 것

이 저장소가 fast mode 를 2026-10-16 에 정리했으므로 그 사이 차이를 같이 대조함. 그쪽 문서가 미해결로 남긴 항목 하나가 풀렸음.

| 항목 | 10-16 문서의 상태 | 2026-11-02 문서 |
|---|---|---|
| Managed Agents 에서 `speed` 지정 위치 | **확인하지 못했음**으로 남김 | 에이전트의 model 객체에 둠. 예시가 `{"id": "claude-opus-5-5", "speed": "fast"}` 임 |
| Opus 5.5 의 플랫폼 | Claude API, Microsoft Foundry(Azure 호스팅) | **Claude Models on Google Cloud (Anthropic managed)** 가 추가됨 |
| Claude Platform on AWS | 불가 | 여전히 목록에 없음 |
| Opus 4.6 예외 | 성공하지만 standard 로 돎 | Google Cloud (Anthropic managed) 에서는 헤더를 보내도 **400** |

첫 줄은 그쪽 문서의 `## 짚어야 할 것` 마지막 항목이 풀린 것임. 이 문서에서 고치지 않았고, 사람이 그쪽에 반영할 자리임.

## Swift 쪽에서는 걸리지 않음

이 저장소 주인 기준으로 앱 코드가 직접 걸릴 가능성은 낮음. [ClaudeForFoundationModels](../agents/2026-10-25-claude-for-foundation-models.md) 경로는 fast mode 를 노출하지 않음. 2026-11-02 에 `gh api search/code` 로 패키지 전체에서 `speed` 를 찾았을 때 걸린 파일이 `Sources/ClaudeAPI/JSONValue.swift` 하나이고, 그것도 **요청 파라미터가 아니라 주석 문장**("clarity wins over speed")임. 그 문서가 정리한 "빠지는 것" 표의 베타 헤더 항목과도 맞음. 4.8 fast 는 헤더가 필요했으므로 이 경로에서는 처음부터 켤 수 없었음.

다만 **`.opus4_8` 은 이 경로에 남아 있음.** 패키지 README 의 폴백 예시가 `ClaudeLanguageModel(name: .opus5_5, auth: auth, fallbacks: [.opus4_8])` 임. 모델이 2027-05-28 이후까지 Active 이므로 이 폴백 설정은 이번 폐기에 영향받지 않음. **기능 폐기와 모델 폐기를 갈라 봐야 하는 이유가 여기서도 같음.**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Fast mode 정식 공개](2026-10-15-fast-mode-opus-5-5-ga.md) | 같은 기능의 10-15 상태표. 그 문서가 재확인 과제로 적은 "Opus 5 와 Opus 4.8 이 preview 를 벗어났는지" 의 답이 **4.8 쪽은 폐기**임 |
| [모델 은퇴 일정](../models/2026-09-30-claude-model-retirement-schedule.md) | 그 문서가 대조하는 폐기 페이지에 이 건이 없음. 재확인 방법에 릴리스 노트가 더 필요해짐 |
| [Claude Opus 5.5](../models/2026-09-22-claude-opus-5-5.md) | 지원 모델을 Opus 5.5·5·4.8 셋으로 적은 자리. 넷 중 하나가 11-30 에 빠짐 |
| [Usage and Cost Analytics API](2026-10-29-usage-cost-analytics-api.md) | `speeds[]` 필터와 `speed` 차원. 영향 범위를 추측 없이 조회하는 경로 |
| [API 버전 2026-10-21 이전](2026-10-21-claude-api-version-migration.md) | 파괴 변경이 버전 축과 모델 축 둘로 들어온다는 이야기. 이번 건은 **셋째 축**(모델과 버전 모두 그대로인데 조합이 사라짐) |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 모델 ID 를 바꾸면 prefix 가 통째로 무효. 11-30 전 전환은 캐시 재작성 비용을 한 번 치름 |

## 짚어야 할 것

- **제거 후 에러의 종류가 적혀 있지 않음.** 릴리스 노트와 Warning 모두 "will return an error" 까지고 상태 코드가 없음. 400 인지 다른 것인지 **확인할 수 없었음.** 에러 분기를 상태 코드로 거는 코드는 11-30 에 한 번 밟아 보는 수밖에 없음
- **Managed Agents 쪽까지인지 갈라 적지 않음.** Warning 은 `claude-opus-4-8` 요청 전체로 적혀 있고, 같은 페이지의 Managed Agents 지원 표에도 4.8 이 남아 있음. 세션의 model 객체에 `speed: "fast"` 를 둔 에이전트가 같은 날 멈추는지는 **확인할 수 없었음**
- **이메일 공지 여부를 확인할 수 없었음.** 폐기 페이지의 통지 약속은 모델 은퇴에 한정된 문언이고, 기능 폐기에 대한 통지 정책은 문서에서 찾지 못했음
- **`usage.speed` 가 제거 후 무엇으로 오는지 모름.** Opus 4.6 처럼 조용히 standard 로 떨어지는 전례가 있는데, 4.8 은 에러로 적혀 있음. 두 선례가 갈리는 이유는 서술되지 않음
- **fast 단가 위의 캐시 가격 행이 여전히 없음.** 전환 비용을 계산할 때 캐시 쓰기·적중 쪽은 추정이 됨. 10-16 문서가 적은 공백이 그대로임
- fast mode 문서의 지원 표와 Warning 이 **서로 어긋난 상태**임. 표만 읽으면 4.8 이 아직 쓸 수 있는 선택지로 보임. 표가 갱신되기 전까지는 Warning 이 최신임

## 유효기간

**2026-11-02 확인 기준이고 제거일은 2026-11-30 임.** 이 문서에서 가장 먼저 낡는 것은 제거 전후의 서술임. 다시 볼 때 확인할 것 셋. **11-30 이 지난 뒤 fast mode 문서의 지원 표와 베타 헤더 표에서 Opus 4.8 줄이 실제로 빠졌는지**, **Opus 5 의 fast mode 에도 같은 예고가 붙었는지**(preview 가 남은 쪽이라 다음 차례로 보는 편이 맞음), 그리고 **이 종류의 기능 폐기가 Model deprecations 페이지에 자리를 얻었는지.** 셋째가 생기면 [모델 은퇴 일정](../models/2026-09-30-claude-model-retirement-schedule.md)의 대조 방법이 다시 한 페이지로 돌아옴. 출발점은 [릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서 fast mode 항목만 훑는 것이고, 단가는 가격 페이지가 1차 출처임.
