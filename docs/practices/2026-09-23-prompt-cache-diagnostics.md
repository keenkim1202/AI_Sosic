---
title: Cache diagnostics, 캐시 미스의 원인을 API가 지목해 줌
source: https://platform.claude.com/docs/en/build-with-claude/cache-diagnostics
author: Anthropic
published: 2026-09-23
collected: 2026-09-26
tags: [claude, prompt-caching, kv-cache, llm-cost, coding-agent, swift]
---

출처: [Cache diagnostics, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/cache-diagnostics) · [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [Claude Platform release notes](https://platform.claude.com/docs/en/release-notes/api) · [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) (2026-09-26 확인)

## 요약

캐시가 왜 깨졌는지를 API에 물어보는 경로임. 릴리스 노트 기준 **2026-09-23에 베타를 벗어났고 `cache-diagnosis-2026-04-07` 헤더가 더 필요하지 않음.** 요청에 `diagnostics` 객체를 넣고 직전 응답의 `id`를 `previous_message_id`로 주면, API가 두 요청의 지문을 비교해서 **prefix가 처음 갈라진 지점을 모델·시스템·툴·메시지 중 하나로 지목**함. 이 저장소에 걸리는 지점은 셋임. 첫째, [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 "히트율 낮을 때 확인할 8가지"가 이제 추측이 아니게 되는데, **8가지 중 이름이 붙는 것은 4개뿐**이고 나머지는 `null`이나 `unavailable`로 떨어짐. 둘째, 캐시를 깨기 가장 쉬운 파라미터들(`tool_choice`, thinking, `output_config`)이 정확히 **이름이 안 붙는 쪽**임. 셋째, 프롬프트 캐싱 문서의 점검 목록에 **Swift와 Go가 JSON 직렬화에서 키 순서를 무작위화해 캐시를 깬다**는 항목이 있음. 이 저장소 주인이 쓰는 언어가 그 예시로 박혀 있음.

옵트인의 단위가 요청 하나라는 것도 먼저 알아야 함. **`diagnostics` 객체를 보낸 요청만 지문이 저장됨.** 한 턴이라도 빼먹으면 다음 턴이 `previous_message_not_found`로 돌아옴.

## 어떻게 도는가

`diagnostics` 객체가 든 요청마다 API가 응답 `id`를 키로 가벼운 지문을 저장함. 다음 요청에서 그 `id`를 `previous_message_id`로 넘기면 새 요청의 지문을 다시 만들어 비교하고, 응답에 `diagnostics` 객체를 붙여 **첫 divergence 지점**을 적어 줌.

- 첫 턴은 `"previous_message_id": null`로 보냄. 비교 대상이 없어도 지문은 저장됨
- 스트리밍에서는 `message_start` 이벤트에 실려 옴
- 지문에는 **해시와 토큰 수 추정값만** 들어가고 원문은 안 들어감. ZDR 적격이라고 문서가 명시함
- 조직·워크스페이스 단위로 격리됨. 두 응답의 `anthropic-workspace-id` 헤더를 비교해 확인할 수 있음

비교 결과는 캐시가 실제로 맞았는지와 별개임. 문서가 이 점을 먼저 못박음.

> "The comparison is about request structure, independent of whether the cache actually hit."

## 응답의 세 가지 값

| 값 | 의미 |
|---|---|
| `null` | 세 가지가 겹쳐 있음. 요청에 `diagnostics` 객체가 없었거나, `previous_message_id`가 `null`인 첫 턴이거나, **비교를 돌렸고 divergence가 없었음** |
| `{"cache_miss_reason": null}` | 응답을 직렬화하는 시점에 비교가 아직 도는 중이었음. 응답이 아주 빨리 시작되면 생김. 결론 없음으로 보고 다음 턴을 봄 |
| `{"cache_miss_reason": {...}}` | 원인이 붙음. `*_changed`면 첫 divergence 지점이고, `previous_message_not_found`와 `unavailable`은 비교가 안 나온 경우임 |

**`null`이 세 갈래로 겹쳐 있는 게 실무에서 제일 걸리는 지점임.** "정상"과 "옵트인을 빼먹음"이 같은 값으로 오기 때문에, 로그에서 `null`을 보고 안심하려면 그 요청이 `diagnostics`를 보냈는지를 클라이언트 쪽에서 따로 알고 있어야 함. 매 턴 무조건 넣는 코드로 만드는 게 그래서 싸게 끝남.

## 미스 원인 6종

`cache_miss_reason`은 `type`으로 갈리는 union임. **가장 앞선 divergence 하나만 보고하므로 뒤에 숨은 것이 있을 수 있음.**

| type | 무슨 뜻인가 | 무엇을 고치나 |
|---|---|---|
| `model_changed` | `model`이 직전 요청과 다름. 라우터·A/B·폴백이 다른 모델을 고른 경우. 캐시는 모델별임 | 캐시된 대화 안에서는 모델을 고정 |
| `system_changed` | `system`이 다름. 타임스탬프나 요청 ID 같은 값이 시스템 프롬프트에 끼어든 경우 | 시스템 프롬프트를 바이트 단위로 고정하고, 변하는 값은 중단점 뒤 첫 `user` 메시지로 내림 |
| `tools_changed` | `tools` 배열이 다름. 추가·제거·재정렬, 또는 `input_schema` JSON이 비결정적으로 직렬화된 경우 | 매 턴 같은 목록을 **같은 순서로**, 스키마는 결정적으로 직렬화(예: 키 정렬) |
| `messages_changed` | 모델·시스템·툴은 같은데 앞선 `messages` 항목이 바뀌거나 재정렬·삭제됨. 히스토리를 자르거나 편집했거나, assistant 턴과 `tool_result`가 재전송에서 다르게 직렬화된 경우 | 히스토리를 append 전용으로 다루고, assistant `content`와 tool result를 **그대로** 되돌려 보냄 |
| `previous_message_not_found` | 준 `previous_message_id`에 해당하는 지문이 없음. **요청이 바뀌었다는 증거가 아님.** 직전 요청이 `diagnostics`를 안 보냈거나, 다른 워크스페이스였거나, 시간이 너무 지난 경우 | 매 턴 `diagnostics`를 넣고 연속 턴의 간격을 좁힘 |
| `unavailable` | 진단 정보를 못 만든 경우. 모델·시스템·툴은 같은데 **다른 프롬프트 영향 파라미터가 다른 경우**와, divergence가 비교 지평선 밖인 아주 긴 대화가 여기 들어감 | 캐시된 대화의 수명 동안 프롬프트 영향 파라미터를 고정 |

네 개의 `*_changed`에는 `cache_missed_input_tokens` 정수가 같이 붙음. divergence 지점 뒤로 몇 토큰이 떨어졌는지의 추정값임. **다만 토크나이즈 전 바이트 길이에서 유도한 값이라 과금 숫자가 아니고, `usage.input_tokens`를 넘길 수도 있다고 문서가 직접 적음.** 대시보드에 비용으로 올리면 안 되는 값이고 규모 지표로만 씀.

## 이 저장소의 8가지 점검표와 대조

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 정리한 8가지를 이 기능의 출력에 그대로 대 봄. 이 문서를 쓰는 이유가 이 표임.

| 8가지 점검 항목 | diagnostics가 뭐라고 하나 |
|---|---|
| 1. 유휴. 명령 실행이나 리뷰로 보관 윈도우를 넘김 | **이름이 안 붙음.** `null` + 캐시 읽기 0으로 나옴 |
| 2. 모델 전환 | `model_changed` |
| 2b. 프로바이더 전환 | **범위 밖.** 이 기능은 Claude API 전용임 |
| 3. 브랜치 이동(rewind·fork) | `messages_changed` |
| 4. compaction이나 수동 히스토리 재작성 | `messages_changed`로 나올 자리인데, 서버측 compaction을 켠 경우의 동작은 **확인할 수 없었음** |
| 5. 툴 변경 | `tools_changed` |
| 5b. reasoning level(effort) 변경 | **이름이 안 붙음.** 최상위 `output_config`가 달라진 경우 `unavailable`로 떨어짐 |
| 6. 동적 시스템 프롬프트 | `system_changed` |
| 7. 오래된 메시지나 payload를 고치는 익스텐션 | `messages_changed` |
| 8. 프로바이더 라우팅과 eviction | **이름이 안 붙음.** 1번과 같은 모양(`null` + 캐시 읽기 0)으로 나와 구분되지 않음 |

읽는 방식이 이렇게 갈림. **원인이 내 요청의 모양이면 이름이 붙고, 원인이 내 요청 밖(시간, 인프라)이면 안 붙음.** 8가지 목록은 폐기되는 게 아니라 절반으로 줄어듦. 이름이 안 붙는 세 줄(1, 5b, 8)은 여전히 사람이 판단해야 함.

## diagnostics와 usage를 같이 읽는 표

`diagnostics`는 "내 요청이 바뀌었나"에, `usage.cache_read_input_tokens`는 "캐시가 맞았나"에 답함. 둘을 곱해야 어디를 볼지가 나옴. 문서가 준 조합임.

| diagnostics | 캐시 읽기 토큰 | 해석 |
|---|---|---|
| `null` | 많음 | 정상. prefix가 안정적이고 캐시가 맞았음 |
| `null` | 0이거나 적음 | 요청은 같은데 **캐시 항목이 이미 없었음.** 턴 간격을 좁히거나 1시간 TTL을 검토 |
| `*_changed` | 0이거나 적음 | 내 버그. `type`이 지목한 원인을 고침 |
| `*_changed` | 많음 | 드묾. 프롬프트 뒤쪽에서 변경이 났지만 앞선 `cache_control` 중단점이 맞은 경우. 고칠 값은 있으나 영향은 작음 |

이 표는 **실제 `previous_message_id`를 넘긴 턴에만** 적용됨. 첫 턴은 캐시를 쓰는 게 아니라 만드는 중이라 읽기가 0인 것이 정상이고, `cache_miss_reason`이 `null`이거나 `previous_message_not_found`·`unavailable`인 경우도 이 표 밖임.

## Swift에서 먼저 밟는 것

프롬프트 캐싱 문서의 점검 목록에 이 항목이 있음.

> "Verify that the keys in your `tool_use` content blocks have stable ordering as some languages (for example, Swift, Go) randomize key order during JSON conversion, breaking caches"

`tool_use` 블록은 assistant 메시지 안에 있으므로 여기서 깨지면 `messages_changed`로 나옴. `tools` 배열의 `input_schema`가 같은 이유로 흔들리면 `tools_changed`임. **둘 다 코드를 한 글자도 안 바꿨는데 턴마다 미스가 나는 형태**라 원인을 찾기가 특히 어려운 부류고, 이 기능이 값어치를 내는 자리가 정확히 여기임.

원문이 준 처방은 "키를 정렬하라"까지임. Swift에서 바로 걸 수 있는 것은 `JSONEncoder`의 `outputFormatting`에 `.sortedKeys`를 넣는 것인데, **이 한 줄은 원문에 없고 이 저장소가 붙인 것임.** 직렬화 경로가 여러 개면(직접 만든 dict, 서버에서 받아 되돌려 보내는 블록) 전부 같은 규칙을 통과하는지 확인해야 함.

## 이름을 못 붙이는 쪽이 왜 문제인가

`unavailable`의 설명에 프롬프트 영향 파라미터 목록이 들어 있음. `tool_choice`, `thinking`, `context_management`, `output_config`, `output_format`, 그리고 **활성 `anthropic-beta` 헤더의 집합**임.

프롬프트 캐싱 문서의 무효화 표를 나란히 놓으면 어긋남이 보임. 그 표는 `tool_choice` 변경, 이미지 추가·제거, thinking 설정 변경, `output_config.effort` 변경이 **메시지 캐시를 항상 무효화한다**고 적음. 즉 **캐시를 깨는 것이 확실한 항목들이 진단에서는 이름 없이 `unavailable`로 떨어짐.** 베타 헤더 집합이 여기 들어가 있는 것도 실무에서 걸림. 베타 기능을 켜고 끄는 플래그가 배포마다 달라지는 하네스면 미스의 원인이 헤더인데 진단은 그 말을 안 해 줌.

그러니 순서는 이렇게 됨.

1. `*_changed`가 오면 그것부터 고침. 가장 앞선 것 하나만 보이므로 고친 뒤 다시 봄
2. `unavailable`이 반복되면 진단을 더 기다리지 말고 **파라미터 6종과 베타 헤더를 손으로 대조**함
3. `null` + 읽기 0이면 요청 모양은 무죄임. 시간과 인프라 쪽, 즉 TTL과 eviction을 봄

## 켤 때 재 볼 것

- **Claude API 전용임.** Amazon Bedrock, Google Cloud, Microsoft Foundry에서는 안 됨. 그쪽으로 붙인 하네스는 8가지 점검표가 그대로 남음
- 지문 보관 기간은 "짧은 기간"이라고만 적혀 있음. **구체적인 값은 확인할 수 없었음.** 연속 턴을 가깝게 두라는 권고만 있음
- 가격표에 **cache diagnostics 항목이 없음.** Pricing 페이지를 훑어 확인했고 별도 과금 서술을 찾지 못했음. 다만 "무료"라고 적힌 문장도 없어서, 확인된 것은 과금 항목이 문서에 없다는 것까지임
- 프로덕션 전체에 매 턴 켜는 것이 기본 사용법임. 옵트인이 요청 단위라 샘플링해서 켜면 `previous_message_not_found`만 쌓임. **샘플링하려면 대화 단위로 갈라야 함**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | "히트율 낮을 때 확인할 8가지"의 절반에 API가 이름을 붙여 줌. 나머지 절반이 왜 안 붙는지가 이 문서의 추가분 |
| [서버측 compaction](2026-09-20-claude-api-compaction.md) | 그 문서가 8가지 중 4번을 둘로 갈랐음. 갈라진 두 경로가 진단에서 어떻게 보이는지는 확인하지 못했음 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 앞 턴 편집이 400이 되는 검사와 같은 버그를 다른 방식으로 잡음. 한쪽은 막고 한쪽은 이름을 붙임 |
| [대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md) | 레버 3의 가시성. `cache_missed_input_tokens`를 비용으로 읽으면 안 되는 이유가 여기 걸림 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 툴 목록을 결정적 순서로 내보내라는 권고. `tools_changed`가 그 권고를 안 지켰을 때 보이는 신호임 |

## 짚어야 할 것

- **이 기능은 원인을 지목하는 것이지 캐시를 살려 주는 것이 아님.** 미스는 이미 일어났고 그 요청은 정상 요금으로 청구됨. 값어치는 두 번째 미스를 막는 데 있음
- **`null`은 "문제 없음"이 아님.** 옵트인을 안 한 요청도 `null`임. 로그에서 이 둘을 갈라 보려면 클라이언트가 자기 요청에 객체를 넣었는지를 같이 남겨야 함
- **`cache_missed_input_tokens`는 과금 숫자가 아님.** 문서가 `usage.input_tokens`를 넘을 수 있다고 직접 적음
- **가장 앞선 divergence 하나만 옴.** 고쳤는데 히트율이 안 오르면 두 번째가 뒤에 있었던 것이지 진단이 틀린 게 아님
- **서버측 compaction을 켠 대화에서 진단이 무엇을 반환하는지 확인할 수 없었음.** compaction은 prefix를 의도적으로 바꾸는 지원 경로인데, 그 변경이 `messages_changed`로 잡히는지 아니면 예외로 처리되는지를 두 문서 어느 쪽도 적지 않음. 켜 둔 상태로 진단을 붙이면 잡음이 섞일 가능성이 있음
- **`previous_message_not_found`를 원인으로 오해하기 쉬움.** 문서가 "요청이 바뀌었다는 증거가 아니다"라고 따로 못박은 유일한 항목임
- 기능 자체가 best effort임. 요청을 막거나 실패시키지 않고, 정보가 없으면 `unavailable`이나 `cache_miss_reason: null`로 돌아옴

## 유효기간

**2026-09-26 확인 기준**임. 릴리스 노트 기준 GA 날짜는 **2026-09-23**이고, 그 직전 2주 사이에 동작이 두 번 바뀌었음(09-09에 지문 저장이 `diagnostics` 객체가 있는 요청으로 한정됐고, 09-18에 `diagnostics` 필드가 항상 응답에 실리게 됨). 즉 **필드 존재 여부에 기대는 코드가 그 사이 두 번 깨졌을 수 있는 자리임.** 다시 볼 때 확인할 것 둘. **`cache_miss_reason` 타입이 늘었는지**(`unavailable`로 뭉쳐 있는 파라미터 변경에 이름이 붙으면 위 표가 바뀜), 그리고 **Bedrock·Google Cloud로 확대됐는지**. 지문 보관 기간이 문서에 숫자로 적히는지도 같이 볼 것.
