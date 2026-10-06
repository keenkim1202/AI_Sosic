---
title: Models API의 line 과 capabilities, 모델 목록을 코드에 박지 않는 방법
source: https://platform.claude.com/docs/en/api/models/list
author: Anthropic
published: 2026-10-01
collected: 2026-10-06
tags: [claude, routing, effort, thinking, migration, tooling]
---

출처: [List Models (API 레퍼런스)](https://platform.claude.com/docs/en/api/models/list) · [릴리스 노트 2026-10-01](https://platform.claude.com/docs/en/release-notes/api) · [Models overview, Using the Models API](https://platform.claude.com/docs/en/models/overview#using-the-models-api) (2026-10-06 확인)

## 요약

**2026-10-01 에 Models API 응답에 `line` 필드가 붙었음.** 모델이 속한 계열 이름을 그대로 돌려주고, 값은 `haiku`·`sonnet`·`opus`·`fable`·`mythos` 다섯임. 릴리스 노트가 드는 예가 **"Claude Opus 4.5 와 Claude Opus 4.6 이 둘 다 `opus` 를 보고한다"**이고, 문서는 **"`line` 을 모델의 `id` 에서 추론하지 말고 읽어라"**라고 못 박음. 계열이 없는 모델에서는 `null` 임.

왜 이게 필요해졌는지가 더 중요함. **4.6 세대부터 모델 ID 에 날짜가 안 붙음.** `claude-opus-5-5`, `claude-sonnet-5-5` 처럼 날짜 없는 ID 자체가 고정 스냅샷이 됐고, 거기에 `claude-fable-5-1`·`claude-mythos-5-1` 같은 이름까지 들어오면서 **문자열 패턴으로 계열과 세대를 추측하는 코드가 전부 깨짐.** `line` 은 그 추측을 API 쪽으로 옮기는 장치임.

같은 응답의 `capabilities` 객체는 effort 레벨·thinking 타입·context management 전략·배치·PDF 입력 같은 것을 모델별로 켜짐/꺼짐으로 알려줌. 다만 **이 저장소가 모델 문서마다 적어 둔 "지뢰"는 대부분 `capabilities` 에 없음.** `tool_choice` 강제 호출이 400 을 내는지, thinking 을 `between_tools` 로 꺼야 하는지는 여기서 알 수 없음. 모델 선택 UI 를 앱에 넣는다면 **목록과 계열은 API 에서 받고, 모델별 제약은 손으로 들고 있어야 한다**는 결론이 됨.

## `GET /v1/models` 가 돌려주는 것

목록은 **최근 출시 모델이 먼저** 나오고, `after_id`·`before_id`·`limit` 으로 페이지를 넘김. `limit` 기본값 **20**, 범위 **1~1000**. 응답에 `first_id`·`last_id`·`has_more` 가 같이 옴.

항목 하나의 필드는 이것뿐임.

| 필드 | 내용 |
|---|---|
| `id` | 모델 식별자. **날짜 없는 ID 도 고정 스냅샷임** |
| `display_name` | 사람이 읽는 이름 |
| `line` | 계열. `haiku`·`sonnet`·`opus`·`fable`·`mythos`, 또는 `null` |
| `created_at` | RFC 3339 출시 시각. **출시일을 모르면 epoch 값이 들어갈 수 있음** |
| `max_input_tokens` | 입력 컨텍스트 창 상한. `null` 가능 |
| `max_tokens` | `max_tokens` 파라미터에 넣을 수 있는 상한. `null` 가능 |
| `capabilities` | 기능별 지원 여부 객체. **객체 자체가 `null` 일 수 있음** |
| `type` | 항상 `"model"` |

`null` 이 되는 자리가 넷(`line`, `max_input_tokens`, `max_tokens`, `capabilities`)이라는 게 구현에서 제일 먼저 걸림. 모델 피커를 만들면 **"계열 없음" 그룹과 "한도 미표기" 상태를 UI 가 가질 수 있어야 함.**

## `capabilities` 로 분기할 수 있는 것

문서가 **"알려진 모든 capability 의 키는 항상 존재한다"**고 적음. 즉 키 유무로 판단하지 말고 `supported` 값을 보는 구조임.

| 키 | 하위 구조 |
|---|---|
| `effort` | `supported`, 그리고 `low`·`medium`·`high`·`max` 각각의 지원 여부. `xhigh` 는 **`null` 가능** |
| `thinking` | `supported`, `types.adaptive`, `types.enabled` |
| `context_management` | `supported`, `clear_thinking_20251015`, `clear_tool_uses_20250919`, `compact_20260112` (각각 `null` 가능) |
| `batch` | Batch API 지원 여부 |
| `structured_outputs` | structured output / JSON 모드 / strict tool schema 지원 여부 |
| `image_input` | 이미지 콘텐츠 블록 수용 여부 |
| `pdf_input` | PDF 콘텐츠 블록 수용 여부 |
| `citations` | 인용 생성 지원 여부 |
| `code_execution` | 코드 실행 툴 지원 여부 |

여기서 실제로 값어치 있는 분기는 `effort` 와 `thinking` 쪽임. 모델 비교표가 **Claude Haiku 4.5 는 effort 미지원**이라고 적고 있으니, effort 를 쓰는 코드 경로를 `capabilities.effort.supported` 로 가리면 모델 이름 하드코딩이 하나 줄어듦. 다만 **실제 응답으로 대조하지는 못했음**(아래 참조).

## 이 API 가 알려주지 않는 것

이 절이 이 문서의 요지임. 스키마에 **없는** 것들이고, 전부 이 저장소의 다른 문서가 "모델 바꿀 때 터지는 지점"으로 적어 둔 것들임.

- **`tool_choice` 제약.** `any`·`tool` 강제 호출이 400 을 내는 모델이 늘고 있는데(Fable 5.1, Mythos 5.1, Opus 5.5, Sonnet 5.5) `capabilities` 에 해당 키가 없음
- **thinking 을 끄는 방법.** `types` 에 있는 값은 `adaptive` 와 `enabled` 둘뿐임. Sonnet 5.5 가 선행 thinking 을 끄는 데 쓰는 `between_tools` 는 여기 없고, Opus 5.5 는 `disabled` 와 `enabled` 둘 다 400 임
- **라이프사이클.** deprecated / retired 상태도 은퇴 예정일도 필드로 없음. 문서 쪽 정의는 **retired 면 요청이 실패**하는 것이고, 은퇴 **최소 60일 전**에 통보한다고 적혀 있음. 즉 이 API 만으로는 "이 모델 곧 죽는다"는 경고를 피커에 띄울 수 없음
- **가격, 기본 effort, 지연시간.** 비교표에만 있음. 비용 계산을 런타임에 하려면 값을 따로 들고 있어야 함
- **compaction 의 on-demand 모드.** `context_management` 에 나열된 전략은 셋인데 `compact_20260112` 까지임. 2026-09-14 에 붙은 최상위 `compaction` 파라미터 쪽은 전략 목록에 안 나타남
- **베타 기능 전반.** 이 메서드의 `anthropic-beta` 헤더는 **deprecated 로 표시되고 "향후 릴리스에서 이 메서드에서 제거된다"**고 적혀 있음. 베타를 보려면 `client.beta.models` 쪽 메서드를 쓰라는 안내임

정리하면 **"어떤 모델이 있고 어느 계열인가"는 API 로 넘겼지만 "그 모델에 무엇을 보내면 안 되는가"는 아직 사람 몫**임. 모델 피커를 API 로 자동 채우면 목록은 최신으로 유지되는데, 새로 등장한 모델이 기존 요청 형태를 거부해서 런타임에 400 이 나는 조합이 생김. 자동 채우기를 쓸 거면 **지원 모델 허용 목록을 한 겹 더 두는 쪽**이 안전함.

## Apple 쪽과 같은 모양

이 패턴은 iOS 앱 개발에서 이미 본 것임. Foundation Models 프레임워크도 `model.capabilities.contains(.reasoning)` 처럼 **런타임에 능력을 물어보게** 돼 있고, 백엔드를 교체 가능하게 만든 뒤로는 그게 필수가 됨. 양쪽 다 **"모델 이름으로 분기하지 말고 능력으로 분기하라"**는 같은 지시를 하고 있음.

차이는 커버리지임. Apple 쪽은 컨텍스트 창 같은 핵심 제약이 프레임워크 문서에 고정값으로 박혀 있고, Claude 쪽은 그걸 API 로 내주면서 **대신 모델별 금지 조합을 안 내줌.** 어느 쪽이든 하드코딩이 완전히 사라지지는 않음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | `tool_choice` 400, 히스토리 편집 400 같은 모델별 제약. `capabilities` 가 안 알려주는 항목의 실제 사례 |
| [대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md) | 스마트 라우팅. 라우팅 대상 목록을 어디서 받을지의 문제 |
| [서버측 compaction](./2026-09-20-claude-api-compaction.md) | `context_management` 전략 목록과 on-demand 모드의 간극 |
| [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md) | `model.capabilities` 로 런타임 능력 확인. 같은 패턴의 Apple 쪽 |
| [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md) | effort 레벨 선택. `capabilities.effort` 로 가릴 수 있는 범위 |

## 짚어야 할 것

- **실제 API 응답으로 대조하지 못했음.** 이 문서는 레퍼런스 스키마와 릴리스 노트, 모델 비교표만 읽은 것임. API 키가 없어 `GET /v1/models` 를 직접 호출해 `line` 과 `capabilities` 의 실제 값을 확인할 수 없었음
- **레퍼런스 페이지의 예시 응답은 플레이스홀더임.** `claude-opus-5` 항목에 `"line": "haiku"` 가 들어 있고 `max_input_tokens` 와 `max_tokens` 가 둘 다 `0` 임. 값 자체를 참고하면 안 됨. 필드 이름과 구조만 쓸 것
- **`line` 값 집합은 고정이 아님.** 문서가 "더 많은 계열이 추가될 수 있으니 값 집합을 고정으로 취급하지 말라"고 명시함. `switch` 문을 전수로 쓰면 다음 릴리스에 깨짐
- **Claude Haiku 4.5 의 effort 미지원은 비교표에서 읽은 것**이고 `capabilities.effort.supported` 가 실제로 `false` 로 오는지는 확인하지 못했음
- 파트너 플랫폼(Amazon Bedrock, Google Cloud)이 자체 모델 ID 체계와 자체 라이프사이클을 쓴다는 것은 문서가 밝혀 둠. **게이트웨이를 거치면 이 API 의 목록이 실제 호출 가능한 목록과 다를 수 있음.** 그 경우의 동작은 확인하지 못했음

## 유효기간

**2026-10-06 확인 기준**임. `line` 이 붙은 지 며칠이라 **아직 움직일 자리**임. 다시 볼 때 확인할 것 셋. **`line` 값 집합에 새 이름이 추가됐는지**, **라이프사이클 상태나 은퇴 예정일이 필드로 들어왔는지**(들어오면 이 문서의 결론 한 줄이 바뀜), 그리고 **`anthropic-beta` 헤더가 이 메서드에서 실제로 제거됐는지.** 세 번째는 문서가 이미 예고해 둠. `capabilities` 하위 키는 기능이 추가될 때마다 늘어나므로 표를 갱신하지 말고 레퍼런스에서 새로 받는 편이 나음.
