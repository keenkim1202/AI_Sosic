---
title: Fast mode 정식 공개, 출력 속도를 사는 스위치가 캐시를 반납한다
source: https://platform.claude.com/docs/en/build-with-claude/fast-mode
author: Anthropic
published: 2026-10-15
collected: 2026-10-16
tags: [claude, llm-cost, prompt-caching, inference, latency, rate-limits]
---

출처: [Fast mode, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/fast-mode) · [Fast mode pricing](https://platform.claude.com/docs/en/about-claude/pricing#fast-mode-pricing) · [Fast mode rate limits](https://platform.claude.com/docs/en/api/rate-limits#fast-mode-rate-limits) · [릴리스 노트 2026-10-15 항목](https://platform.claude.com/docs/en/release-notes/api) (2026-10-16 확인)

## 요약

요청에 `speed: "fast"` 한 줄을 넣으면 출력이 **최대 2.5배 OTPS**로 나오는 기능임. **2026-10-15에 Claude Opus 5.5에 대해 research preview를 벗어났고**, 베타 헤더 `fast-mode-2026-02-01`도 접근 신청도 더 필요 없음. Opus 5와 Opus 4.8은 아직 preview고 헤더와 신청이 그대로 필요함.

값은 명시돼 있음. Opus 5.5 fast는 **$8 / $40 per MTok**이고 표준이 $4 / $20이니 **정확히 2배**임. 그런데 가격표보다 중요한 단서가 둘 더 있음. 첫째, **빨라지는 것은 OTPS뿐이고 TTFT는 아님.** 사용자가 기다리는 첫 글자까지의 시간은 그대로임. 둘째, **fast와 standard는 system·messages 캐시 prefix를 공유하지 않음**(tools 캐시는 유지됨). [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 정리한 캐시 깨짐 8가지에 9번째가 붙는 것이고, 모델 ID가 그대로라 제일 안 보이는 축임.

## 2026-10-15에 바뀐 것

| 모델 | 상태 | `fast-mode-2026-02-01` 헤더 | 플랫폼 |
|---|---|---|---|
| Opus 5.5 (`claude-opus-5-5`) | **정식** | 불필요. 보내도 동작함 | Claude API(Managed Agents 포함), Microsoft Foundry의 **Azure 호스팅** 배포 |
| Opus 5 (`claude-opus-5`) | research preview | 필요 + 접근 신청 | Claude API(Managed Agents 포함) |
| Opus 4.8 (`claude-opus-4-8`) | research preview | 필요 + 접근 신청 | Claude API(Managed Agents 포함) |

문이 열린 범위에 조건이 붙음.

- **Evaluation tier는 못 씀.** Start·Build·Scale로 올라가야 기본 fast rate limit이 생김
- **Claude Platform on AWS에서는 불가**
- Microsoft Foundry라도 **"hosted on Anthropic" 배포는 불가**. Azure 호스팅만 됨
- [Batch API](https://platform.claude.com/docs/en/build-with-claude/batch-processing)와 **같이 못 씀**. Batch 50% 할인과 fast 2배를 동시에 고를 수는 없음
- Priority Tier 약정과도 **같이 못 씀**

## 2.5배는 출력 토큰 속도고, 체감 지연이 아님

문서가 성격을 분명히 적어 둠.

> "Fast mode runs the same model with a faster inference configuration. There is no change to intelligence or capabilities."

같은 가중치, 같은 동작임. 모델이 바뀌는 게 아니라 추론 설정이 바뀜. 그리고 효과의 위치를 못 박음.

> "Speed benefits are focused on output tokens per second (OTPS), not time to first token (TTFT)"

앱에 붙이는 입장에서 이 문장이 구매 결정을 거의 다 결정함. 사용자가 "느리다"고 느끼는 지점이 **첫 글자가 안 나오는 구간**이면 fast mode는 돈만 두 배로 내고 끝임. 반대로 긴 코드 블록이나 문서 초안처럼 output이 수천 토큰 나가는 경로, 그중에서도 스트리밍으로 흘려보내는 화면에서는 효과가 바로 보임. 문서도 이득이 가장 잘 보이는 조건으로 스트리밍을 지목함.

즉 이건 모델 선택이 아니라 **경로별 스위치**임. 같은 앱 안에서도 요약 한 줄 받는 호출과 긴 생성 호출을 같은 설정으로 둘 이유가 없음.

## 가격은 정확히 2배, 캐시 배수는 그 위에 얹힘

| 항목 | Opus 5.5 표준 | Opus 5.5 fast |
|---|---|---|
| base input | $4 / MTok | **$8 / MTok** |
| output | $20 / MTok | **$40 / MTok** |

가격 페이지가 적용 범위와 중첩 규칙을 적음.

> "Fast mode pricing applies across the full context window, including requests over 200k input tokens."

그리고 배수가 쌓임. **프롬프트 캐싱 배수와 데이터 레지던시 배수가 fast 단가 위에 곱으로 얹힘.** 가격 페이지 각주 기준으로 Opus 5.5의 캐시 적중은 기본 input의 **0.05배**이므로, fast에서는 $8을 기준으로 계산됨. 5분 캐시 쓰기는 1.25배, 1시간 쓰기는 2배 관계가 그대로 유지됨.

⚠️ 다만 **가격 페이지에 fast 전용 캐시 가격 행은 없음.** 위 문언 둘을 곱하면 적중 $0.40, 5분 쓰기 $10, 1시간 쓰기 $16이 나오지만 **이것은 이 문서가 한 계산이고 표로 확인한 값이 아님.** 비용 모델에 넣기 전에 실제 청구 내역으로 한 번 맞춰 보는 편이 안전함.

`inference_geo: "us"`를 쓰고 있으면 1.1배가 또 위에 얹힘. fast + US 전용 + 캐시 쓰기를 동시에 켜면 배수가 셋 겹치는 자리가 됨.

## 캐시가 깨지는 9번째 원인

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 히트율 낮을 때 볼 것 8가지를 정리했는데, 그 목록에 speed가 없음. 지금 문서에 문언이 생겼음.

> "Switching between fast and standard speed invalidates the prompt cache. Requests at different speeds do not share cached prefixes."

목록 2번(모델·프로바이더 전환)과 같은 계열인데 성질이 더 나쁨. **모델 ID도 툴 정의도 시스템 프롬프트도 그대로인데 미스가 남.** 코드 diff로는 `speed` 한 줄만 보임.

⚠️ 다만 **"캐시 전체"는 아님.** 프롬프트 캐싱 문서의 무효화 표는 speed 행에서 **tools 캐시는 유지(✓)**, system과 messages 캐시만 무효(✘)로 적음. 그래서 손실은 prefix 전체가 아니라 **시스템 프롬프트와 메시지 구간**이고, 툴 카탈로그가 큰 구성이라면 그 부분은 그대로 적중함. 위 인용문은 이 구분 없이 `invalidates the prompt cache`로만 적혀 있어서 그대로 읽으면 손실을 과대평가하게 됨.

제일 밟기 쉬운 자리가 공식 문서가 권하는 fallback 패턴 그 자체임. fast rate limit을 넘기면 429가 오고, `retry-after`를 기다리기 싫으면 `speed`를 떼고 다시 보내라고 적혀 있음. 문서도 바로 Note를 붙임.

> "Falling back from fast to standard speed will result in a prompt cache miss. Requests at different speeds do not share cached prefixes."

10만 토큰 컨텍스트를 들고 도는 루프에서 이 fallback 한 번은 **그 요청에서 system·messages 구간을 캐시 쓰기 단가로 다시 처리한다**는 뜻임. 정상 input 가격에 쓰기 비용을 **따로 더하는 것이 아님.** `usage` 가 `input_tokens` 와 `cache_creation_input_tokens` 를 분리해서 세고, 쓰기 토큰은 쓰기 단가로만 청구됨. 표준 Opus 5.5 기준 5분 쓰기는 $5/MTok(기본 input $4의 1.25배)이므로 10만 토큰 재작성은 **$0.50**이고, $0.40 + $0.50 이 아님(출력과 나머지 input 은 별도). [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 "미스가 얼마나 비싼가" 절 산술이 그대로 적용됨.

그래서 실무 규칙은 단순해짐. **speed는 세션 단위로 고정하고, 요청마다 토글하지 않음.** 토글이 필요하다면 그 경로는 애초에 캐시를 재사용하지 않는 1회성 호출이어야 함.

## 무엇이 돌았는지 어떻게 아는가

응답 `usage`에 `speed` 필드가 붙고 값은 `"fast"` 또는 `"standard"`임. 지원하지 않는 모델에 `speed: "fast"`를 보내면 에러고, rate limit이나 용량을 넘기면 429나 529임. 즉 성공했으면 fast로 돈 것인데, **예외가 하나 있음.**

> "Claude Opus 4.6 is an exception: instead of returning an error, a request with `speed: "fast"` and the `fast-mode-2026-02-01` beta header succeeds, but runs at standard speed."

조용히 느린 쪽으로 도는 조합이 하나 있다는 뜻임. 마이그레이션 중에 모델 ID만 되돌려 놓고 `speed`를 그대로 둔 코드가 정확히 여기 걸림. `usage.speed`를 로그에 넣어야 하는 이유가 이것임. ⚠️ 이때 과금이 fast 단가인지 standard 단가인지는 문서에 없고 **확인할 수 없었음.**

rate limit 쪽은 헤더 여섯 개가 따로 붙음.

| 헤더 | 내용 |
|---|---|
| `anthropic-fast-input-tokens-limit` / `-remaining` / `-reset` | fast 전용 분당 input 토큰 한도, 잔량, 리셋 시각 |
| `anthropic-fast-output-tokens-limit` / `-remaining` / `-reset` | fast 전용 분당 output 토큰 한도, 잔량, 리셋 시각 |

**fast 버킷은 표준 Opus 버킷과 완전히 별개임.** 표준 Opus 5.5는 tier별 수치가 공개돼 있는데(Start에서 ITPM 2,000,000 / OTPM 400,000) **fast 기본 한도의 숫자는 어느 문서에도 없고 Console의 Rate limits 페이지에서만 보임.** 용량 계획을 세워야 하면 거기서 먼저 확인해야 함. 429는 `retry-after`를 들고 오고, 토큰 버킷이 연속 보충되는 방식이라 대기가 보통 짧다고 적혀 있음.

## 언제 켜고 언제 안 켜는 게 맞나

1. **output이 길고 스트리밍으로 보여주는 1회성 호출**: 켤 만함. prefix 재사용이 없으니 캐시 손실이 비용 계산에 안 들어옴
2. **캐시 히트율로 단가를 잡아 둔 에이전트 루프**: 루프 전체를 fast로 고정할 각오가 있을 때만. 중간에 섞으면 2배 단가에 미스까지 겹침
3. **체감 지연이 TTFT에서 오는 UI**: 효과 없음. 프롬프트 길이나 라우팅 쪽을 봐야 함
4. **Batch·Priority Tier·Claude Platform on AWS**: 선택지가 아예 없음

[대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md)의 레버 2가 라우팅이었는데, 거기에 축이 하나 늘어난 셈임. 모델을 바꾸는 라우팅은 캐시를 어차피 버린다는 걸 전제로 설계하게 되는데, speed 라우팅은 **같은 모델 안에서도 캐시를 버림.** 그 글이 강조한 "서버측 캐싱을 라우팅 계산에 넣어라"가 여기서도 그대로 유효하고, 오히려 더 숨어 있음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 히트율 점검 8가지에 speed 전환이 9번째로 붙음. 모델 ID가 그대로라 2번 항목보다 찾기 어려움 |
| [서버측 compaction](2026-09-20-claude-api-compaction.md) | `usage` 필드를 읽는 비용 계측 코드. `iterations`에 이어 `speed`도 로그 대상이 됨 |
| [대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md) | 레버 2의 라우팅. 캐시를 계산에 넣으라는 권고가 speed 축에도 적용됨 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 캐시 읽기 배수가 모델마다 다르다는 이야기. 그 문서는 Fable 5.1만 예외고 나머지는 0.1배라고 적었는데, 가격 페이지 각주는 지금 Opus 5.5와 Sonnet 5.5를 **0.05배**로 적고 있음. 그 문서는 이 PR에서 손대지 않았음 |

## 짚어야 할 것

- **"최대 2.5배"는 Anthropic 자체 수치임.** 측정 조건, 비교 기준, 분포가 문서에 없음. 독립적으로 재보지 못했음. 자기 프롬프트 길이와 output 길이로 직접 재는 것 말고는 방법이 없음
- **fast 전용 캐시 가격 행이 없음.** 위에 적은 $0.40 / $10 / $16은 "캐시 배수가 fast 단가 위에 얹힌다"는 문언과 Opus 5.5의 0.05배 각주를 곱한 **이 문서의 계산**임. 공식 표의 값이 아님
- **fast 기본 rate limit의 숫자가 공개돼 있지 않음.** tier별 표준 한도는 공개돼 있는데 fast 버킷은 Console에서만 보임. 문서로 확인할 수 없었음
- **Opus 4.6 예외의 과금을 확인할 수 없었음.** 요청은 성공하고 standard로 도는데 어느 단가로 청구되는지 문서에 서술이 없음
- **Managed Agents 안에서 `speed`를 어디에 지정하는지 확인하지 못했음.** 지원 표가 "Claude API, including Claude Managed Agents"까지만 적고 세션 생성 쪽 필드 이름을 보여주지 않음
- TTFT가 안 줄어든다는 문언은 명시적이지만 **얼마나 안 줄어드는지**는 수치가 없음. 같다는 뜻인지 거의 같다는 뜻인지 갈리지 않음

## 유효기간

**2026-10-16 확인 기준**이고 정식 전환은 **2026-10-15 릴리스 노트 항목**임. 다시 볼 때 확인할 것 셋. **Opus 5와 Opus 4.8이 research preview를 벗어났는지**, **가격 페이지에 fast 전용 캐시 행이 생겼는지**, 그리고 **Claude Platform on AWS 지원이 열렸는지**. 셋 다 [릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서 fast mode 항목만 훑으면 잡힘. 단가와 배수는 가격 페이지가 1차 출처임.
