---
title: Claude Opus 4.5 폐기 공지, 하한 하루 전에 공지가 오고 통지 기간이 정확히 60일임
source: https://platform.claude.com/docs/en/about-claude/model-deprecations
author: Anthropic
published: 2026-11-23
collected: 2026-11-24
tags: [claude, model-deprecation, model-lifecycle, api, claude-code, ios]
---

출처: [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations) · [Claude Platform release notes](https://platform.claude.com/docs/en/release-notes/api) (2026-11-23 항목) · [Claude Opus 5.5 migration guide](https://platform.claude.com/docs/en/models/opus-5-5/migration-guide) · [Models overview](https://platform.claude.com/docs/en/models/overview) (전부 **2026-11-24 확인**)

## 요약

**2026-11-23** 에 **Claude Opus 4.5(`claude-opus-4-5-20251101`)가 폐기 공지**됐고 은퇴일이 **2027-01-22**, 권장 대체는 `claude-opus-5-5` 임. 날짜 없는 `claude-opus-4-5` 도 같이 죽음. 이 문서를 쓰는 **2026-11-24 기준 59일** 남았음.

여기서 읽어야 할 것은 공지 자체가 아니라 날짜가 놓인 자리임. 이 저장소의 [Claude 모델 은퇴 일정](./2026-09-30-claude-model-retirement-schedule.md)이 2026-10-01 에 받아 적은 표에서 `claude-opus-4-5-20251101` 의 은퇴 하한은 **2026-11-24**, 즉 오늘이었음. 공지는 그 **하루 전**에 왔고 은퇴일은 **공지로부터 정확히 60일** 뒤임. 그 문서가 "계획을 '출시일 + 1년 + 60일'로 잡으면 60일을 더 버는 것으로 착각함"이라고 경고한 자리가 그대로 실현됨. 하한 + 60일은 2027-01-23 인데 실제 은퇴일은 **그보다 하루 이른 2027-01-22** 임.

통지 기간 60일은 Anthropic 이 약속한 하한값과 **같음**. 공지 이력에서 지금까지 가장 짧았던 값(Haiku 3, 60일)과 동률이고, 2026년에 들어온 뒤 61·61·62·60·62 사이에서 움직이던 값이 다시 바닥을 찍음. **60일은 하한이 아니라 기본값으로 굴러간다**는 그 문서의 판단은 유지됨.

그리고 같은 페이지에 그사이 두 가지가 더 생겼음. **Claude Code 가 이 페이지의 날짜를 따른다는 절이 명시적으로 들어왔고**, 상태 표와 Models overview 양쪽에서 **`Legacy` 로 표시된 모델이 하나도 남지 않음.** 후자는 위 문서가 운영 감시 신호로 쓰라고 한 것이 지금은 가리킬 대상이 없다는 뜻임.

## 공지 내용 자체

deprecation history 의 첫 항목임.

| 항목 | 값 |
|---|---|
| 공지일 | **2026-11-23** |
| 은퇴일 | **2027-01-22** (Claude API) |
| 폐기 모델 | `claude-opus-4-5-20251101` |
| 같이 죽는 ID | `claude-opus-4-5` (날짜 없는 쪽) |
| 권장 대체 | `claude-opus-5-5` |
| 상태 표 표기 | Deprecated |

> "This retirement includes `claude-opus-4-5`, the same model's ID without a date."

4.6 이전 세대는 날짜 없는 ID 가 별칭이고 날짜 박힌 쪽을 가리키므로, 둘 중 어느 쪽을 쓰고 있어도 같은 날 깨짐. 코드에서 `claude-opus-4-5` 를 쓰고 있다고 해서 날짜 박힌 공지와 무관하다고 읽으면 안 되는 자리임.

## 같은 주에 겹치는 날짜 셋

2026-11-24 기준 상태 표에서 가까운 순으로 뽑은 것임.

| 모델 ID | 상태 | 날짜 | 오늘 기준 |
|---|---|---|---|
| `claude-sonnet-4-5-20250929` | Deprecated | 은퇴 **2026-11-30** | **6일 뒤.** 아직 Retired 로 넘어가지 않았음 |
| `claude-opus-4-5-20251101` | Deprecated | 은퇴 **2027-01-22** | 59일 뒤 |
| `claude-haiku-4-5-20251001` | Active | 하한 2026-10-15 | **하한이 40일 전에 지났고 폐기 공지는 아직 없음** |
| `claude-mythos-preview` | Deprecated | 은퇴일 **To be announced** | 공지로부터 5개월 반 |
| `claude-opus-4-6` | Active | 하한 2027-02-05 | 다음 차례로 가장 유력한 자리 |

위 문서가 유효기간에 적어 둔 재확인 항목 둘이 여기서 풀림. **Sonnet 4.5 는 아직 은퇴하지 않았음**(6일 남음). **Haiku 4.5 는 하한이 지났는데도 Active 로 남아 있고 공지가 없음.** 즉 하한 통과가 공지를 당기지는 않는다는 것이 실측으로 확인됨. 반대로 Opus 4.5 는 하한 전날에 공지가 왔으니, **하한 전후 며칠이 공지가 뜰 수 있는 가장 가까운 구간**으로 보는 편이 맞음.

`claude-mythos-preview` 는 2026-06-09 공지에 은퇴일이 여전히 미정임. 공개 릴리스 모델에 걸리는 60일 약속 바깥에 있는 것으로 읽히지만 페이지가 이유를 적지는 않았음.

## Legacy 가 사라졌음

페이지의 수명주기 정의는 그대로 네 단계(Active / Legacy / Deprecated / Retired)임. 그런데 **2026-11-24 기준 상태 표에 Legacy 로 적힌 줄이 하나도 없음.** [Models overview](https://platform.claude.com/docs/en/models/overview) 쪽도 Legacy 목록 절 자체가 없어졌고 "Latest models" 와 "All models" 두 표만 남아 있음.

이게 운영에 걸림. [Claude 모델 은퇴 일정](./2026-09-30-claude-model-retirement-schedule.md)이 내린 결론 하나가 **"감시할 신호는 Deprecated 가 아니라 Legacy 로 내려갔는가"** 였음. Deprecated 를 기다리면 이미 60일 시계가 돌고 있으니 그 앞단을 보자는 것이었는데, **지금은 그 앞단에 아무것도 표시되지 않음.** 두 페이지가 어긋난다고 그 문서가 적어 둔 항목은 어긋남이 사라지는 방향으로 풀렸지만, 풀린 방식이 "둘 다 Legacy 를 쓰지 않는 것"이었음.

그래서 남는 신호는 둘뿐임. **상태 표의 하한 날짜**, 그리고 **릴리스 노트에 폐기 항목이 뜨는 것.** 전자는 Opus 4.5 사례처럼 하루 단위로는 못 쓰지만 월 단위로는 쓸 수 있고, 후자는 이미 60일 시계가 시작된 시점임. Legacy 를 기다리는 설계는 지금 기준으로 동작하지 않음.

## Claude Code 가 이 페이지 안으로 들어옴

새로 붙은 절임. 2026-10-01 시점 이 저장소의 문서에는 없던 서술임.

> "Claude Code uses the Claude API, including when you sign in with a Claude plan, so this page's dates apply. If Claude Code is set up for another platform or for a gateway, the dates are those of the platform that serves its requests."

요금제 로그인으로 쓰고 있어도 같은 일정이 적용된다는 것을 명시했음. 그리고 할 일 순서를 적어 뒀는데 순서 자체가 요점임.

1. **Claude Code 를 먼저 올림.** 새 모델이 새 버전을 요구할 수 있음
2. 모델을 안 지정했거나 `sonnet` 같은 **별칭을 쓰고 있어도 현재 모델을 확인함.** 별칭이 폐기 모델로 해석되고 있을 수 있음
3. 모델을 **지정한 모든 자리**에서 바꿈. 페이지가 **서브에이전트 정의**를 명시적으로 지목함
4. 관리자가 설정한 모델이 있으면 관리자에게 변경을 요청함

3번이 밟기 쉬운 자리임. 최상위 모델만 바꾸고 서브에이전트 정의에 남은 모델 이름을 놓치면, 은퇴 후에 메인 루프는 돌고 **서브에이전트만 죽음.** 이 저장소의 [Claude Code 의 effort 해석 순서](../practices/2026-10-27-claude-code-effort-resolution.md)가 정리한 대로 모델과 effort 가 여러 층에서 각각 해석되므로, 실제로 어느 층에 어떤 이름이 박혀 있는지는 설정 파일을 직접 훑어야 나옴.

Sonnet 4.5 항목에는 은퇴 뒤의 동작까지 적혀 있음. Claude API 로 붙은 Claude Code 는 `There's an issue with the selected model` 로 시작하는 에러를 띄우고, **fallback model chain 을 설정해 두면 다음 모델로 넘어감.** 그런데 한 줄이 더 붙음.

> "It tries Claude Sonnet 4.5 again on your next message."

**fallback 이 끈적하지 않음.** 메시지마다 죽은 모델을 다시 시도하고 실패한 뒤 넘어감. 즉 fallback chain 은 사고를 막는 장치이지 이주를 미루는 장치가 아님. 매 메시지에 한 번씩 왕복을 버리는 상태로 계속 쓰게 됨.

## 플랫폼 구분이 다시 그려짐

이 페이지의 날짜가 어디에 적용되는지가 2026-10-01 시점과 달라졌음.

| 구분 | 2026-11-24 기준 |
|---|---|
| 이 페이지의 날짜를 따름 | Claude API, Claude Platform on AWS, **Claude Models on Google Cloud (Anthropic managed)**, Microsoft Foundry |
| 자체 일정 | **Claude Models on Google Cloud (Google managed)**, Amazon Bedrock |

이 저장소의 문서는 2026-10-01 에 "Google Cloud (Vertex AI) 자체 일정"으로 적었음. 지금은 Google Cloud 가 **Anthropic managed 와 Google managed 로 갈려서 전자는 Anthropic 일정, 후자는 자체 일정**임. GCP 를 백엔드로 두고 있으면 **어느 쪽 경로로 붙어 있는지에 따라 캘린더가 달라짐.** Bedrock 쪽도 `bedrock-mantle` 과 `bedrock-runtime` 두 엔드포인트의 모델 표를 따로 보라고 적혀 있음.

Models overview 의 각주가 같은 내용을 반대 방향에서 적어 둠. Foundry 는 "follows the Claude API lifecycle schedule", Claude Platform on AWS 는 "follows Anthropic's first-party model lifecycle" 임. Bedrock 과 Google managed 만 "sets its own lifecycle dates" 로 적혀 있음.

## 4.5 에서 5.5 로 갈 때 쌓여 있는 것

Opus 4.5 는 4.6, 4.7, 4.8, 5, 5.5 를 건너뛰는 거리임. 마이그레이션 가이드가 **"이 페이지를 위에서부터 순서대로 읽고 이 절까지 적용하라"** 는 누적 구조로 쓰여 있음. 4.5 에서 출발하면 다섯 묶음이 전부 걸림.

`claude-opus-5-5` 로 가는 요청이 **반드시** 만족해야 하는 것부터임. 어기면 400 임.

| 항목 | 요구 |
|---|---|
| thinking | `thinking` 필드를 안 보내거나 `{"type": "adaptive"}`. **`disabled` 와 `budget_tokens` 는 거부** |
| effort | 사고 깊이를 정하는 **유일한** 파라미터. 다섯 단계, 기본 `medium` |
| tool_choice | `auto` 또는 `none` 만. **`any` 와 `{"type": "tool", ...}` 거부** |
| 샘플링 | `temperature`·`top_p`·`top_k` 생략 |
| prefill | assistant 턴으로 끝나는 `messages` **거부** |
| computer use | `computer_toolset_20260801` 툴셋. Bedrock 에서만 `computer_20251124` 도 동작 |
| 컨텍스트 | 1M 이 기본. 옛 베타 헤더는 **무효** |

4.5 에서 출발할 때 추가로 걸리는 것이 가이드의 "Claude Opus 4.5 or earlier" 묶음임.

- **prefill 제거.** Opus 4.6 부터 이미 400 이었으므로 4.5 에서 올라오는 경우에만 새 변경임
- **툴 호출 JSON 을 표준 파서로 읽을 것.** 4.6 이후 모델이 유니코드 이스케이프나 슬래시 이스케이프를 다르게 낼 수 있음. `JSON.parse()` 류를 쓰면 자동으로 처리되고, 원시 문자열을 직접 자르는 코드만 깨짐
- **`client.beta.messages.create()` 에서 `client.messages.create()` 로 이동.** adaptive thinking 과 effort 는 베타 네임스페이스가 필요 없음
- **베타 헤더 셋 제거**: `effort-2025-11-24`, `fine-grained-tool-streaming-2025-05-14`, `interleaved-thinking-2025-05-14`
- **`output_format` 을 `output_config.format` 으로.** 안 바꾸고 쓰려면 `structured-outputs-2025-11-13` 베타 헤더가 필요하고, 없으면 400 임

여기에 4.6 묶음이 더해지는데 그쪽에 조용한 변경이 둘 있음. **thinking 블록이 응답에 계속 오지만 `thinking` 필드가 비어 있음**(4.6 의 요약 사고 기본 반환에서 바뀜. 명시적으로 켜야 내용이 옴), 그리고 **고해상도 이미지가 자동으로 적용됨**(장변 2,576px, 이미지당 최대 약 4,784 토큰. 이전 상한 약 1,600 토큰의 약 3배). 좌표는 실제 픽셀과 1:1 이므로 **스케일 변환 코드를 제거**해야 함.

Managed Agents 를 쓰고 있으면 가이드가 모델 이름 교체 외에는 변경이 필요 없다고 적어 뒀음.

## 가격은 내려가는데 토큰이 늘어남

Models overview 의 두 표에서 뽑은 것임.

| | Claude Opus 4.5 | Claude Opus 5.5 |
|---|---|---|
| 입력 | $5 / MTok | **$4 / MTok** |
| 출력 | $25 / MTok | **$20 / MTok** |
| 컨텍스트 | 200K | **1M** |
| 최대 출력 | 64K | **128K** |

**단가만 보면 이주가 20% 싸짐.** 그런데 가이드가 같은 자리에 토크나이저 경고를 붙여 둠. Opus 4.7 에서 들어온 새 토크나이저를 5.5 도 쓰고, 텍스트 처리에 **같은 내용이 약 1배에서 1.35배 토큰**을 쓸 수 있음(내용에 따라 최대 약 35% 증가). `/v1/messages/count_tokens` 가 4.6 과 다른 값을 돌려줌.

그래서 **"단가가 내려갔으니 비용도 내려간다"로 읽으면 안 됨.** 입력 20% 인하와 토큰 최대 35% 증가가 방향이 반대고, 워크로드 모양에 따라 어느 쪽이 이기는지가 갈림. 그리고 토큰이 늘어나는 것은 비용만의 문제가 아님. **`max_tokens` 와 compaction 트리거, 클라이언트측 토큰 추정 코드가 같이 틀어짐.** 문자 수에 고정 비율을 곱해 토큰을 추정하는 코드가 있으면 그 자리가 먼저 깨짐.

이미지를 보내는 앱은 반대 방향으로 더 큼. 이미지당 토큰이 약 3배까지 늘 수 있으므로 **전체 해상도가 필요 없으면 보내기 전에 줄이는 쪽**이 가이드 권고임.

## 앱 쪽에서 할 일

- **Opus 4.5 를 쓰고 있으면 59일이 남았고, 앱 심사가 그 안에 들어가야 함.** 단말에서 직접 Claude API 를 때리는 구조면 모델 교체가 앱 릴리스임. 서버 경유면 배포 한 번임. 이 차이가 59일을 여유로 만들거나 비상으로 만듦
- **Sonnet 4.5 는 6일 남았음.** 이쪽이 먼저임. 보조 작업이나 분류용으로 Sonnet 4.5 를 쓰고 있었다면 이번 주에 끝내야 함
- **Claude Code 설정에서 서브에이전트 정의까지 훑음.** 최상위 모델만 바꾸면 서브에이전트가 조용히 죽는 자리임
- **fallback model chain 을 이주 유예로 쓰지 않음.** 메시지마다 죽은 모델을 다시 때림
- **Legacy 표시를 감시 신호로 쓰던 설계는 폐기함.** 지금 Legacy 로 표시된 모델이 없음. 하한 날짜와 릴리스 노트 두 개만 남았음
- **GCP 를 쓰면 Anthropic managed 인지 Google managed 인지 먼저 확인함.** 전자는 이 페이지 날짜, 후자는 자체 일정임
- **콘솔 Usage CSV 로 어느 키가 아직 4.5 를 때리는지 확인함.** 페이지가 주는 방법은 그대로임

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude 모델 은퇴 일정](./2026-09-30-claude-model-retirement-schedule.md) | 수명주기 구조, 통지 기간 집계, 하한과 60일의 관계. 이 문서는 그 문서가 남긴 재확인 항목 셋의 답임 |
| [Claude Opus 5.5](./2026-09-22-claude-opus-5-5.md) | 대체 모델 쪽. effort 기본값 `medium`, 깨지는 요청, 진행 상황 텍스트가 thinking 으로 오는 변화가 그쪽에 정리돼 있음 |
| [API 버전 2026-10-21](../practices/2026-10-21-claude-api-version-migration.md) | 샘플링 파라미터 제거가 모델 쪽 거부에서 **API 버전 쪽 제거**로 올라간 경로 |
| [Claude Code 의 effort 해석 순서](../practices/2026-10-27-claude-code-effort-resolution.md) | 모델과 effort 가 여러 층에서 해석되는 구조. 모델 이름이 박혀 있을 자리 목록으로 쓸 수 있음 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | "나아지지 않으면 안 옮긴다"는 판단의 유효기간. 59일이 그 유효기간임 |

## 짚어야 할 것

- **하한과 은퇴일을 같은 눈으로 보면 안 됨.** Deprecated 행의 은퇴일은 날짜가 걸린 약속이고, Active 행의 "Not sooner than" 은 바닥임. 페이지는 은퇴일 열 전체를 "Tentative retirement date" 로 적어 둠
- **통지 기간 60일 계산은 이 문서가 한 것임.** 2026-11-23 과 2027-01-22 의 날짜 차이를 센 값이고 페이지에 적힌 수치가 아님. 페이지가 약속하는 문구는 "at least 60 days' notice ... for publicly released models" 하나뿐임. 과거 간격(61·61·62·60·62)도 이 저장소가 센 값임
- **"하한 하루 전에 공지"는 한 건의 관찰임.** Opus 4.5 한 사례이고 Haiku 4.5 는 하한이 40일 지났는데도 공지가 없음. 둘을 같이 놓으면 하한은 공지 시점을 예측하는 데 쓸 수 없고, **공지가 하한 직전에도 올 수 있다는 것까지만** 말함
- **Legacy 가 사라진 이유는 확인할 수 없었음.** 상태 정의에는 남아 있고 어느 모델에도 붙어 있지 않음. 표기 정책이 바뀐 것인지, 지금 해당 모델이 없을 뿐인지 페이지가 적지 않음
- **파트너 플랫폼의 실제 Opus 4.5 날짜를 확인하지 않았음.** Bedrock 과 Google managed 가 자체 일정이라는 사실까지만 확인했고 각 플랫폼 모델 표는 열지 않았음
- **파라미터 폐기 서술이 바뀌었음.** 이 저장소의 2026-10-01 문서가 인용한 SDK 별 문구(대부분의 SDK 가 타입 체크를 통과시키고 Python SDK v1.0 이상만 `TypeError`)가 **지금 표에는 없음.** 지금은 `temperature`·`top_p`·`top_k` 가 "Removed in API version `2026-10-21`" 로 적혀 있고 400 을 돌려줌. 문구가 다른 곳으로 옮겨간 것인지 삭제된 것인지는 확인하지 않았음. SDK 쪽 함정 자체는 여전히 밟을 수 있는 것으로 보고 요청 바디를 직접 훑는 편이 안전함
- **토큰 1배~1.35배는 Anthropic 이 적은 범위이고 워크로드별 실측이 아님.** 한국어 입력에서 어떻게 움직이는지는 이 범위로 알 수 없음. 올리기 전에 `count_tokens` 로 자기 프롬프트를 재는 것이 유일한 방법임
- **가중치 보존 약속은 실무 판단을 바꾸지 않음.** 은퇴일 이후 요청이 실패한다는 사실은 그대로임
- **원문에 지시문이나 명령문 형태의 문장은 없었음.** 전부 정책과 절차 서술임

## 유효기간

**2026-11-24 확인 기준**임. 공지일과 은퇴일은 날짜가 박힌 기록이라 낡지 않지만 **상태 표는 모델이 나올 때마다 바뀜.** 가장 먼저 갈릴 줄 셋. **Sonnet 4.5 가 2026-11-30 에 실제로 Retired 로 넘어갔는지**(6일 뒤), **Haiku 4.5 가 결국 폐기 공지를 받았는지**, 그리고 **Opus 4.6 이 2027-02-05 하한 앞에서 같은 패턴으로 공지를 받는지.** 셋째가 "하한 직전 공지"가 패턴인지 한 번의 일치인지를 가름. 다시 볼 때는 [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations) 의 deprecation history 첫 항목과 상태 표만 대조하면 되고, 사이의 변경은 [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api) 에서 deprecation 항목만 훑어도 잡힘. **Legacy 표기가 되살아나는지도 같이 볼 것.**
