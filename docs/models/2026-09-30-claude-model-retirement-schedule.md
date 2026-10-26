---
title: Claude 모델 은퇴 일정, 앱에 박아 둔 모델 ID 에 만료일이 붙어 있음
source: https://platform.claude.com/docs/en/about-claude/model-deprecations
author: Anthropic
published: 2026-09-30
collected: 2026-10-01
tags: [claude, model-deprecation, model-lifecycle, api, ios, llm-cost]
---

출처: [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations) · [Models overview](https://platform.claude.com/docs/en/models/overview) · [Claude Platform release notes](https://platform.claude.com/docs/en/release-notes/api) · [Commitments on Model Deprecation and Preservation](https://www.anthropic.com/research/deprecation-commitments) (2025-11-04) (2026-10-01 확인)

## 요약

2026-09-30 에 **Claude Sonnet 4.5(`claude-sonnet-4-5-20250929`)가 폐기 공지**됐고 은퇴일이 **2026-11-30** 임. 이 문서를 쓰는 2026-10-01 기준으로 **정확히 60일** 남았음. 은퇴한 모델에 요청하면 실패함. 여기까지는 릴리스 노트 한 줄인데, 그 뒤의 구조가 앱 개발에 더 크게 걸림. Anthropic 이 약속한 통지 기간은 **"공개 릴리스 모델에 대해 최소 60일"** 이고, 최근 네 번의 실제 통지 기간은 **61·61·62·60일**이었음. 즉 **60일은 하한이 아니라 사실상의 기본값**으로 굴러가고 있음.

iOS 앱에 걸리는 지점은 이것임. 앱은 심사와 사용자 업데이트를 거치므로 서버처럼 당일 롤아웃이 안 됨. **모델 ID 를 바이너리에 박아 두면 60일 안에 심사까지 통과시켜야 하는 작업이 예고 없이 생김.** 그리고 현재 라인업에 있는 **Claude Haiku 4.5 의 은퇴 하한이 2026-10-15**, 즉 2주 뒤임. 하한이 지난다고 은퇴하는 것은 아니지만, 그날 이후로는 폐기 공지가 언제 와도 일정상 이상하지 않다는 뜻임.

## 수명주기 네 단계

| 상태 | 뜻 |
|---|---|
| **Active** | 완전 지원, 사용 권장 |
| **Legacy** | 더 이상 갱신 없음. 앞으로 폐기될 수 있음 |
| **Deprecated** | 아직 동작하지만 권장하지 않음. 대체 모델과 **은퇴일이 지정됨** |
| **Retired** | 사용 불가. **요청이 실패함** |

중간 단계가 둘인 게 요점임. Legacy 는 날짜가 없고 Deprecated 부터 날짜가 붙음. 그러니 운영에서 감시할 신호는 "Deprecated 로 바뀌었나"가 아니라 **"Legacy 로 내려갔나"** 임. Legacy 로 내려간 시점이 60일 시계가 아직 시작되지 않은 유일한 구간임.

2026-10-01 기준 Legacy 로 표시된 모델은 Claude Fable 5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, Opus 4.5, Claude Sonnet 5, Sonnet 4.6 임. 다만 Models overview 의 Legacy 목록과 deprecations 페이지의 상태 표가 같은 모델을 각각 "Legacy"와 "Active"로 적고 있음. 두 페이지의 어휘가 어긋나는 것인지 둘 중 하나가 낡은 것인지는 **확인할 수 없었음.** 날짜가 걸린 쪽은 deprecations 페이지이므로 운영 판단은 그쪽 표로 하는 편이 안전함.

## 지금 날짜가 걸려 있는 것

deprecations 페이지의 상태 표에서 가까운 순으로 뽑은 것임. "Not sooner than" 은 **그날 은퇴한다는 뜻이 아니라 그 전에는 은퇴하지 않는다는 하한**임.

| 모델 ID | 상태 | 날짜 |
|---|---|---|
| `claude-sonnet-4-5-20250929` | Deprecated | **은퇴 2026-11-30** (공지 2026-09-30) |
| `claude-haiku-4-5-20251001` | Active | 하한 2026-10-15 |
| `claude-opus-4-5-20251101` | Active | 하한 2026-11-24 |
| `claude-opus-4-6` | Active | 하한 2027-02-05 |
| `claude-sonnet-4-6` | Active | 하한 2027-02-17 |
| `claude-mythos-preview` | Deprecated | 은퇴일 **To be announced** (공지 2026-06-09) |

마지막 줄이 예외임. 폐기 공지는 2026-06-09 에 나갔는데 은퇴일이 넉 달 가까이 미정으로 남아 있음. Glasswing 참가자용 프리뷰 모델이라 "공개 릴리스 모델"에 걸리는 60일 약속의 바깥에 있는 것으로 읽히지만, 페이지가 그 이유를 적지는 않았음.

신규 모델의 하한은 출시일에서 **정확히 1년** 뒤로 찍힘. Opus 5.5 는 2026-09-22 출시에 하한 2027-09-22, Sonnet 5.5 는 2026-09-28 출시에 하한 2027-09-28, Fable 5.1 은 2026-09-01 출시에 하한 2027-09-01 임. 표에 적힌 날짜들을 맞춰 본 것이고 Anthropic 이 "1년"이라는 규칙을 문서로 약속한 것은 아님. **다만 계획을 "출시일 + 1년 + 60일"로 잡으면 60일을 더 버는 것으로 착각함.** 표의 날짜는 "Not sooner than" 하한이고 60일 통지는 그 하한과 무관한 별개 약속이라, 공지가 1주년 전에 와서 은퇴가 1주년 당일에 걸릴 수 있음. **계획의 바닥은 "출시일 + 1년"으로 잡고, 60일은 공지가 뜬 뒤에 쓸 수 있는 시간으로 따로 셈.**

## 통지 기간이 실제로 얼마였나

페이지의 deprecation history 절에 있는 공지일과 은퇴일 쌍을 날짜 차이로 계산한 것임. 계산은 이 문서가 했고 페이지에 적힌 수치가 아님.

| 공지 | 대상 | 은퇴 | 간격 |
|---|---|---|---|
| 2026-09-30 | Sonnet 4.5 | 2026-11-30 | 61일 |
| 2026-06-05 | Opus 4.1 | 2026-08-05 | 61일 |
| 2026-04-14 | Sonnet 4 · Opus 4 | 2026-06-15 | 62일 |
| 2026-02-19 | Haiku 3 | 2026-04-20 | 60일 |
| 2025-12-19 | Haiku 3.5 | 2026-02-19 | 62일 |
| 2025-10-28 | Sonnet 3.7 | 2026-02-19 | 114일 |
| 2025-06-30 | Opus 3 | 2026-01-05 | 189일 |

**2026 년에 들어오면서 간격이 60일대로 수렴함.** 2025 년의 114일·189일을 기준으로 일정을 잡으면 안 됨. 공지가 뜬 날부터 심사 통과까지 두 달이 전부라고 보는 편이 맞고, 앱스토어 심사 반려 한 번을 감안하면 실제 여유는 그보다 짧음.

## 모델 ID 에 날짜가 없어도 핀 고정임

Models overview 각주가 이렇게 적음.

> "Every Claude model ID is a pinned snapshot, including the dateless IDs used from the 4.6 generation on."

> "For models before the 4.6 generation, the alias is a convenience pointer that resolves to the dated ID. Dateless IDs are their own pinned snapshot; the alias row repeats them."

여기서 갈림이 생김. `claude-haiku-4-5-20251001` 같은 **4.6 이전 세대**는 날짜 없는 `claude-haiku-4-5` 가 별칭이고 날짜 박힌 쪽을 가리킴. 반면 `claude-opus-5-5` 처럼 **4.6 세대 이후의 날짜 없는 ID 는 별칭이 아니라 그 자체가 고정 스냅샷**임. 겉보기에 둘 다 "날짜 없는 짧은 ID"지만 성격이 다름.

실무에 걸리는 결론은 하나임. **`claude-opus-5-5` 가 날짜가 없다고 해서 자동으로 후속 모델로 넘어가지 않음.** 5.6 이 나와도 이 ID 는 5.5 를 계속 가리키고, 은퇴일이 오면 그대로 죽음. 모델 ID 를 하드코딩하는 대신 원격 설정으로 빼 두는 이유가 여기 있음.

## 플랫폼마다 날짜가 다름

deprecations 페이지의 날짜가 모든 곳에 적용되지 않음.

| 플랫폼 | 은퇴 일정 |
|---|---|
| Claude API · Claude Platform on AWS · Microsoft Foundry | 이 페이지의 날짜를 따름 |
| Amazon Bedrock | **자체 일정** |
| Google Cloud (Vertex AI) | **자체 일정** |

> "Partner-operated platforms (Amazon Bedrock and Google Cloud) set their own retirement schedules, so a model's lifecycle status and dates can differ."

백엔드를 Bedrock 이나 Vertex 로 두고 있으면 **이 문서의 날짜를 그대로 캘린더에 옮기면 안 됨.** 각 플랫폼의 모델 표를 따로 봐야 함. 반대로 Foundry 는 Claude API 일정을 따른다고 명시돼 있음.

## 파라미터도 폐기됨

모델만 죽는 게 아님. `temperature`, `top_p`, `top_k` 가 **Claude Opus 4.7 이후 모델에서 기본값이 아닌 값으로 설정되면 400 에러**임.

여기에 SDK 별 함정이 하나 붙음.

> "Most SDKs keep deprecated parameters in their request types so existing code continues to type-check. The Python SDK (v1.0 and later) removes `temperature`, `top_p`, and `top_k`, so passing them raises a `TypeError`."

**대부분의 SDK 는 타입 체크를 통과시킴.** 즉 컴파일이나 린트에서 안 걸리고 런타임 400 으로만 드러남. Python SDK v1.0 이상만 `TypeError` 로 즉시 터짐. Swift 로 직접 요청 바디를 만드는 코드면 타입 시스템이 아무것도 안 잡아 주므로, 모델을 올릴 때 **샘플링 파라미터를 보내고 있는지 요청 바디를 직접 훑는 단계**가 필요함. 대체는 파라미터가 아니라 프롬프트로 하라는 게 문서 권고임.

## 쓰고 있는 모델 찾기

페이지가 주는 방법은 콘솔 내보내기임. [Usage](https://platform.claude.com/usage) 페이지에서 **Export** 를 눌러 CSV 를 받으면 **API 키별·모델별**로 사용량이 갈려 나옴. 키가 환경별로 갈려 있으면 이 CSV 하나로 "어느 환경이 아직 낡은 모델을 때리고 있나"가 나옴.

프로그래밍으로는 [Models API](https://platform.claude.com/docs/en/api/models/list) 가 있고 응답에 `max_input_tokens`, `max_tokens`, `capabilities` 가 들어 있음. 다만 **이것이 은퇴일을 돌려주는지는 확인할 수 없었음.** 목록에 남아 있는지로 간접 확인하는 것과 은퇴일을 미리 받는 것은 다른 일임.

## 가중치는 보존한다고 약속돼 있음

은퇴가 삭제는 아님. 2025-11-04 의 [Commitments on Model Deprecation and Preservation](https://www.anthropic.com/research/deprecation-commitments) 이 **공개 릴리스된 모든 모델과 내부에서 유의미하게 쓰인 모든 모델의 가중치를 최소한 Anthropic 이 회사로 존속하는 기간 동안 보존**하겠다고 적음. 은퇴 시점에 모델을 대상으로 한 인터뷰를 진행해 기록을 남기고, 비용이 내려가면 일부 은퇴 모델을 다시 공개하는 것도 탐색 중이라고 함.

deprecations 페이지도 은퇴의 단점을 스스로 나열함. 특정 모델을 선호하는 사용자가 이주해야 한다는 것, 연구자가 비교 연구용 모델을 잃는다는 것, 그리고 안전성과 모델 복지 관련 위험이 생긴다는 것.

**실무 판단에는 영향이 없음.** 가중치가 보존되는 것과 API 로 호출할 수 있는 것은 다른 얘기이고, 은퇴일 이후 요청이 실패한다는 사실은 그대로임. 다만 "언젠가 다시 열릴 수 있다"는 문장이 공식 문서에 있다는 것은 기록해 둘 값어치가 있음.

## 앱 쪽에서 할 일

- **모델 ID 를 바이너리에 박지 않음.** 원격 설정이나 서버 프록시로 빼서 앱 업데이트 없이 바꿀 수 있게 함. 60일 안에 심사를 통과시켜야 하는 상황을 없애는 게 이 항목의 전부임
- **앱이 직접 Claude API 를 때리고 있으면 특히 급함.** 서버를 경유하면 모델 교체가 배포 한 번인데, 단말에서 직접 호출하면 교체가 앱 릴리스가 됨
- **Legacy 전환을 감시 신호로 씀.** Deprecated 를 기다리면 이미 60일 시계가 돌고 있음
- **콘솔 Usage CSV 를 분기마다 한 번 받음.** 테스트 코드나 오래된 배치 잡이 낡은 ID 를 계속 때리고 있는 경우가 여기서 드러남
- **Bedrock·Vertex 를 쓰면 날짜를 따로 관리함.** 1차 플랫폼 일정과 다름

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | "새 모델이 나왔다고 갈아타지 않는다"는 레버 1 과 정면으로 만남. 아래 절 참고 |
| [Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md) | 하한이 2027-09-01 로 가장 뒤에 있는 현행 모델. 모델 교체 비용을 아끼려면 수명도 변수임 |
| [서버측 compaction](../practices/2026-09-20-claude-api-compaction.md) | 베타 헤더와 모델 세대에 묶인 기능들. 모델을 올릴 때 같이 깨질 수 있는 자리 |

### 비용 문서와의 긴장

[대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md)는 Stripe 가 Opus 4.7 을 거절한 사례를 근거로 **새 모델이 나와도 자기 워크로드로 재보고 안 나아지면 안 옮긴다**고 정리했음. 그 판단은 그대로 유효하지만, 이 문서가 한 칸을 덧붙임. **안 옮기겠다는 선택은 은퇴일까지만 유효한 선택임.** 4.7 에 머무르기로 한 팀도 2027-04-16 하한 이후 언젠가는 옮겨야 하고, 그때는 "나아지는가"가 아니라 "언제까지"가 기준이 됨.

그래서 두 문서를 같이 읽으면 이렇게 됨. **평가는 갈아탈지 말지를 정하는 데 쓰는 게 아니라, 갈아탈 때 어디로 갈지를 정하는 데 쓰는 것임.** 갈아타는 것 자체는 선택지가 아님.

## 짚어야 할 것

- **은퇴일은 전부 "tentative"로 표시돼 있음.** Deprecated 상태의 확정 은퇴일(Sonnet 4.5 의 2026-11-30)과 Active 상태의 하한을 같은 줄에 두고 읽으면 안 됨. 후자는 약속이 아니라 바닥임
- **통지 기간 계산은 이 문서가 한 것임.** 61·61·62·60일은 페이지에 적힌 수치가 아니라 공지일과 은퇴일의 차이를 센 값임. 페이지가 약속하는 문구는 "at least 60 days' notice ... for publicly released models" 하나뿐임
- **"출시일 + 1년"은 관찰이지 약속이 아님.** 지금 표의 하한들이 그 패턴에 맞을 뿐임
- **Legacy 목록이 두 페이지에서 어긋남.** Models overview 가 Legacy 로 적은 모델들이 deprecations 상태 표에서는 Active 임. 어느 쪽이 낡았는지 확인하지 못했음
- **Models API 가 은퇴일을 돌려주는지 확인하지 못했음.** 응답 필드 목록에 그런 필드가 나와 있지 않았고 별도 문서를 열어 확인하지 않았음
- **파트너 플랫폼(Bedrock·Vertex)의 실제 날짜를 이 문서는 확인하지 않았음.** "자체 일정"이라는 사실까지만 확인함
- 이 페이지는 상시 갱신되는 문서라 아래 유효기간을 반드시 같이 볼 것

## 유효기간

**2026-10-01 확인 기준**임. 날짜가 박힌 사실(공지일, 은퇴일, 과거 은퇴 이력)은 낡지 않지만, **상태 표는 모델이 나올 때마다 바뀜.** 가장 먼저 갈릴 줄 둘. **Sonnet 4.5 가 2026-11-30 에 실제로 Retired 로 넘어갔는지**, 그리고 **Haiku 4.5 가 2026-10-15 하한을 지나 폐기 공지를 받았는지**. 둘 다 이 문서 발행 시점에서 두 달 안의 일임. 다시 볼 때는 [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)의 상태 표와 deprecation history 첫 항목만 대조하면 됨.
