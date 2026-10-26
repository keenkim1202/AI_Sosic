---
title: User Profiles, 한 사용자의 위반이 앱 전체 차단으로 번지는 것을 막는 헤더 하나
source: https://platform.claude.com/docs/en/manage-claude/user-profiles
author: Anthropic
published: 2026-10-20
collected: 2026-10-21
tags: [claude, anthropic, policy, gateway, privacy, compliance]
---

출처: [User profiles for end-customer attribution](https://platform.claude.com/docs/en/manage-claude/user-profiles) · [API 릴리스 노트 2026-10-20](https://platform.claude.com/docs/en/release-notes/api) · [User Profiles API 레퍼런스](https://platform.claude.com/docs/en/api/user_profiles) · [Organization webhooks](https://platform.claude.com/docs/en/manage-claude/organization-webhooks) (전부 **2026-10-21 확인**)

## 요약

앱이 자기 API 키로 사용자 요청을 Claude에 중계하면, 그 요청은 전부 **내 조직 이름으로** 도착함. Anthropic은 API 트래픽에 자동 안전 분류기를 돌리는데, 요청에 최종 사용자 식별자가 없으면 **사용자 한 명이 유발한 집행 조치가 조직 전체에 걸림.** 2026-10-20에 나온 user profiles가 그 폭발 반경을 한 명으로 줄이는 장치임. 사용자를 `POST /v1/user_profiles`로 한 번 등록해 `uprof_` ID를 받아 두고, 그 사용자 몫 요청마다 **`anthropic-user-profile-id` 헤더**에 실어 보내면 됨. 공식 문서 문장이 그대로 이것임. "A safety enforcement action, such as a restriction or a ban, can then apply to one end customer instead of your whole organization."

**프로필은 과금 대상이 아니고 가격에 영향이 없음.** 즉 안 붙일 이유가 비용 쪽에는 없고, 붙이지 않은 요청은 문서가 "내 조직 자신의 사용량"으로 간주함. 함정은 셋임. **Bedrock과 Google Cloud는 ID 형식만 보고 프로필을 조회하지 않아서 200이 와도 귀속이 됐다는 증거가 아니고**, 집행 상태를 알려주는 webhook은 **등록은 되지만 Anthropic이 아직 이벤트를 쏘지 않으며**, 옛 `user-profiles-*` 베타 헤더를 아직 보내는 코드는 응답 모양이 조용히 옛 형태로 내려옴.

## 폭발 반경이 왜 조직 단위인가

구조가 단순해서 놓치기 쉬움. 앱이 온디바이스 모델 대신 서버 경유로 Claude를 부르는 순간, Anthropic 쪽에서 보이는 주체는 사용자가 아니라 **API 키를 가진 조직 하나**임. 그래서 위반 트래픽을 만든 주체를 Anthropic이 알 방법이 없고, 조치는 알 수 있는 단위, 즉 조직에 걸림.

프로필은 그 사이에 식별자 한 겹을 넣는 것임. 대신 **Anthropic은 내가 보낸 내용을 검증하지 않음.** 문서가 "Anthropic doesn't verify the details"라고 적어 둠. 즉 이것은 신원 증명 장치가 아니라 **귀속 단위 선언**임. 프로필에 대화 내용은 들어가지 않고, 내가 보낸 필드와 Anthropic이 붙인 프로그램 상태만 들어감.

문서가 먼저 요구하는 전제가 하나 있음. **최종 사용자 트래픽을 보내는 조직과 내 내부 사용량을 쓰는 조직을 분리할 것.** 프로필은 워크스페이스가 아니라 조직에 속하므로 한 조직 안에서 워크스페이스를 갈라도 분리가 안 됨.

## access_type 둘

| 트래픽의 주체 | 프로필 | `access_type` |
|---|---|---|
| 내 앱에 가입한 사용자 | 가입 시점에 사용자당 하나 | `application` (기본값, 생략하면 이것) |
| API 접근을 재판매하는 상대 회사 | 회사당 하나. 그 회사 안의 개인별로 만들지 않음 | `passthrough` (기본값이 아니라 명시해야 함) |
| 내 조직 자신 | 만들지 않음. ID 없는 요청이 내 사용량으로 집계됨 | 해당 없음 |

일반 iOS 앱이면 전부 `application`임. `passthrough`는 게이트웨이나 리셀러 쪽 모양이고, 이 경우 **회사는 식별되지만 요청을 실제로 한 사람은 식별되지 않음.** 문서는 그때 Messages 요청의 `metadata.user_id`를 따로 보내라고 함. 프로필의 `metadata`가 아니라 **요청 자신의 `metadata`**라고 명시해 둔 것이 헷갈리기 쉬운 지점임.

## 붙이는 쪽 실제 모양

등록은 한 번, 귀속은 매 요청임.

```bash
# 1. 가입 플로에서 한 번
curl https://api.anthropic.com/v1/user_profiles \
  -H 'Content-Type: application/json' \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{"external_user_details": {"reference_id": "user-8f2a1c", "country": "KR"}}'
# -> {"id": "uprof_011CZ...", ...}

# 2. 그 사용자 몫 요청마다
curl https://api.anthropic.com/v1/messages \
  -H "anthropic-user-profile-id: $PROFILE_ID" ...
```

`POST /v1/user_profiles`는 **멱등하지 않음.** `reference_id`가 기존 프로필과 같아도 호출마다 새 프로필이 생김. 그래서 받은 `id`를 사용자 레코드에 저장해 두고 각 서버 인스턴스가 거기서 읽는 구조여야 함. 인스턴스마다 알아서 만들게 두면 사용자 하나에 프로필이 여러 개 생기고 귀속이 흩어짐. 기존 사용자 기반을 한꺼번에 등록하려면 생성에 레이트 리밋이 있으니 먼저 Anthropic 계정 팀에 문의하라고 적혀 있음.

SDK와 CLI는 헤더를 대신 보내는 인자를 가짐. `user_profile_id` (java `userProfileId`, php `userProfileID`, csharp·go `UserProfileID`, cli `--user-profile-id`). 헤더는 **token counting, Message Batches, OpenAI SDK 호환 경로**에서도 같게 동작하고, 배치에 붙이면 배치 안 모든 요청에 적용됨.

## 검증 강도가 플랫폼마다 다름

이 표가 이 문서에서 제일 실무적인 부분임.

| 플랫폼 | 프로필 생성 | ID 전달 위치 | 매 요청 검증 |
|---|---|---|---|
| Claude API | 가능 | `anthropic-user-profile-id` 헤더 | 조회함. 형식이 틀리거나 내 조직 것이 아니면 **400** |
| Claude Platform on AWS | 가능 | `anthropic-user-profile-id` 헤더 | 조회함 |
| Microsoft Foundry (Anthropic 호스팅) | 가능 | `anthropic-user-profile-id` 헤더 | 조회함. Azure 호스팅 배포는 생성 불가 |
| Google Cloud | 불가. Claude API나 AWS 쪽에서 만듦 | `anthropic-user-profile-id` 헤더 | **형식만 봄** |
| Claude in Amazon Bedrock | 불가. 위와 같음 | 요청 본문 최상위 `user_profile_id` 필드 | **형식만 봄** |

아래 두 줄이 조용한 사고의 자리임. Bedrock과 Google Cloud는 프로필을 조회하지 않으므로 **형식만 맞으면 아무 ID나 통과함.** 문서 문장이 "a successful response doesn't confirm that the ID matches a profile"임. 오타가 난 ID, 삭제된 프로필의 ID, 다른 조직에서 받은 ID를 몇 달간 보내면서 "귀속되고 있다"고 믿을 수 있음. 반대로 Claude API 쪽은 틀리면 바로 400이라 배포 즉시 드러남.

Message Batches에도 비슷한 틈이 있음. 배치를 **만들 때는 형식만** 보고, 조직이 안 맞는 ID면 배치는 생성된 뒤 **안의 요청들이 각각 `errored` 결과로** 떨어짐. 배치 생성 응답만 보고 성공으로 처리하는 코드면 못 잡음.

삭제도 즉시가 아님. 프로필을 지우면 프로필 엔드포인트는 바로 404를 주지만, **약 10분간은 지워진 ID를 실은 추론 요청이 그대로 성공할 수 있음.** 그 뒤로는 검증하는 플랫폼이 400을 주고, Bedrock과 Google Cloud는 조회를 안 하므로 계속 받음.

## 무엇을 보내고 무엇을 안 보낼 수 있나

모든 필드가 선택임. `name`, `metadata`, 그리고 `external_user_details` 안의 `entity_type`, `country`, `reference_id`, `onboarded_at`, `account_status`, `name_hash`, `email_hash`.

개인정보 쪽 설계가 들어가 있어서 이 레포에 걸림. **이름과 이메일을 넘길 수 없으면 `name`을 비우고 `name_hash`, `email_hash`를 보내면 됨.** 프로필에 이메일 주소 자체를 담는 필드는 **없음.** 해시 함수는 Anthropic이 정하지 않고 불투명 문자열로만 취급함(각 1~255자). 대신 조건이 붙음. **모든 프로필에 같은 함수와 같은 대소문자·공백 정규화를 쓸 것.** 같은 방식으로 계산한 값만 서로 매칭되므로, 중간에 함수를 바꾸면 그 이후 프로필이 이전 것들과 연결되지 않음.

규칙 몇 개는 한 번 틀리면 되돌리기 어려움.

- **모르는 값은 플레이스홀더를 넣지 말고 빼라.** `external_user_details`의 값은 나중에 교체할 수 있지만 **지울 수 없음.** 업데이트에 `null`을 보내면 400임
- `country`는 내 나라가 아니라 **최종 사용자의 나라**(ISO 3166-1 alpha-2 대문자)
- `reference_id`는 내 쪽 안정적 식별자. **유일성을 강제하지 않음**
- `metadata`는 키 최대 **16개**, 키 64자, 값 512자. 업데이트에서 `metadata`는 키 단위로 병합되고 **빈 문자열을 보내면 그 키가 삭제됨.** 반면 `name`과 `access_type`은 통째로 교체됨
- 사용자가 계정을 닫거나 데이터 삭제를 요구하면 프로필을 삭제함. 단 **Anthropic이 그 프로필 ID에 대해 보관하는 신뢰·안전 기록은 삭제되지 않음.** 사용자가 삭제 요구 후에도 앱에 남는다면 새 프로필을 만들어 ID를 갈고 옛 것을 지우라고 적혀 있음

## 레거시 베타 헤더가 응답 모양을 바꿈

이 API는 지금 베타 헤더 없이 쓰는 형태가 기준임. 공식 가이드의 예제가 `anthropic-beta`를 보내지 않음. 그런데 이전 베타 헤더들이 **아직 동작하고, 보내면 응답 모양이 옛 형태로 내려옴.**

| 보내는 헤더 | 달라지는 것 |
|---|---|
| `user-profiles-2026-03-24` | `external_user_details`와 `access_type`이 없음. `reference_id`가 최상위 `external_id`로 옴 |
| `user-profiles-2026-08-18` | `external_user_details`가 없음. `external_id`와 `external_user_onboarded_at`을 씀 |
| `user-profiles-2026-09-04` 및 그 이전 | `programs` 대신 `trust_grants`가 옴. `enforcement_status`·`enforcement_actions` 필드가 빠지고 `enforcement_status[]` 목록 필터가 **400**을 냄 |

이것이 실무에서 걸리는 모양은 이렇다. 예전에 붙여둔 베타 헤더를 상수로 박아 두고 지우지 않으면, 새로 생긴 집행 상태 필드가 **응답에 아예 안 들어오고** 코드는 "집행 조치가 없다"로 읽음. 에러가 아니라 필드 부재라서 조용함. 헤더를 지우는 것이 그대로 수정임.

## 집행 상태를 받는 길은 아직 반만 열림

조직 webhook이 같은 날 베타로 붙었음(`webhooks-2026-10-07`). 이벤트 둘, `user_profile.enforcement_action_opened`와 `user_profile.enforcement_action_resolved`. 이벤트는 내용이 아니라 ID만 실어 오고(`data.id`가 프로필 ID, `data.enforcement_action_id`가 `enac_` 접두사), 무엇이 바뀌었는지는 프로필을 다시 조회해서 `enforcement_actions`와 `enforcement_status`(`banned` / `restricted` / `unrestricted`)로 봐야 함. 배달은 Standard Webhooks 규격이고 **5초 안에 2xx**를 줘야 하며, 중복이 올 수 있어 이벤트 `id`로 중복 제거해야 함.

⚠️ 다만 문서가 스스로 이렇게 적어 둠. **"Event delivery isn't live: Anthropic doesn't send the two events, so an endpoint you register receives nothing."** 즉 지금은 핸들러를 만들어 둘 수만 있음. 접근 자체도 계정 팀 문의가 필요하고, HIPAA readiness가 켜진 조직과 기본 워크스페이스가 CMEK를 쓰는 조직은 쓸 수 없음.

그래서 **오늘 돌아가는 유일한 감지 경로는 폴링임.** 프로필 목록을 `enforcement_status[]=restricted`, `enforcement_status[]=banned`로 조회하는 것. 문서가 여기에 함정을 하나 더 적어 뒀는데, 응답이 `limit`보다 적게 오거나 비어 있어도 끝이 아닐 수 있으므로 **`next_page`가 `null`이 될 때까지 계속 요청해야 함.** 첫 페이지가 비었다고 멈추면 제재당한 사용자를 놓침. 그리고 집행 필드는 **내 조직에 집행 조치 기능이 켜진 뒤에야** 응답에 나타남.

## iOS 앱이면 지금 할 것

1. 서버 경유로 Claude를 부르는 앱이면 **가입 플로에 프로필 생성을 넣고 `id`를 사용자 레코드에 저장.** 과금이 없으니 미루는 이유가 비용 쪽에는 없음
2. 추론이 Bedrock이나 Google Cloud에서 돈다면 **200을 귀속 증거로 쓰지 말 것.** 생성은 Claude API나 Claude Platform on AWS 조직에서 하고, 그 ID가 실제로 유효한지는 생성 쪽 조직에서 따로 확인해야 함
3. 내부 개발용 호출과 사용자 트래픽을 **다른 조직으로** 분리. 워크스페이스 분리로는 안 됨
4. 이름·이메일을 Anthropic에 넘기지 않기로 했다면 **해시 함수와 정규화 규칙을 먼저 고정**하고 문서에 적어 둘 것. 나중에 바꾸면 그 전후 프로필이 서로 연결되지 않음
5. 기존 코드에 `user-profiles-*` 베타 헤더가 박혀 있으면 지움

반대로 **지금 안 해도 되는 것**은 webhook 수신기임. 이벤트가 안 오므로 폴링을 먼저 만드는 편이 맞음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md) | 그 문서의 레버 2가 말하는 게이트웨이·라우터가 여기서 `passthrough` 프로필의 대상임. 중앙 게이트웨이를 두면 귀속 책임도 거기로 모임 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 검사가 **계정·조직 단위로 켜졌는지에 따라 갈리는** 같은 패턴. 그쪽은 히스토리 편집 검사가 2026-08-31 이후 계정에만 강제되고, 여기는 집행 필드가 조직에 기능이 켜진 뒤에만 응답에 나타남 |
| [서버측 compaction](./2026-09-20-claude-api-compaction.md) | 베타 헤더를 상수로 박아 두면 응답 해석이 조용히 어긋나는 문제. 그쪽은 `usage.input_tokens`가 과소 보고되고, 여기는 집행 필드가 아예 빠짐 |

## 짚어야 할 것

- **이것은 신원 증명이 아님.** Anthropic이 내가 보낸 값을 검증하지 않으므로, 프로필은 "내가 이 트래픽의 주체를 이렇게 구분한다"는 선언까지임. 사용자 인증과 남용 차단은 여전히 내 앱 몫임
- **집행이 실제로 어떤 기준에서 걸리는지는 이 문서들에 없음.** 자동 안전 분류기를 돌린다는 사실과, 조치가 restriction 또는 ban이라는 것까지만 확인했음. 어떤 트래픽이 어느 조치로 이어지는지는 [Usage Policy](https://www.anthropic.com/legal/aup) 쪽 문제이고 이 문서는 다루지 않음
- **프로필을 붙이면 조직 전체 차단을 면한다는 보장은 문서에 없음.** 문서 문언은 조치가 한 최종 사용자에게 적용될 수 **있다**(can apply)는 것임. 조직 단위 조치가 사라진다는 서술은 아님
- **webhook 이벤트는 배달되지 않는 상태이고, 그 사실을 문서가 직접 적음.** 설계에 넣되 유일한 경로로 삼으면 안 됨
- **생성 레이트 리밋의 구체적 수치는 확인할 수 없었음.** "profile creation is rate limited"와 대량 등록 전 계정 팀 문의 권고까지만 적혀 있음
- `enac_` 접두사가 붙는 집행 조치 객체를 **직접 조회하는 엔드포인트가 있는지 확인할 수 없었음.** 확인한 경로는 프로필을 다시 읽어 `enforcement_actions`를 보는 것뿐임
- 프로필 ID를 **클라이언트(앱)에 내려보내도 되는지에 대한 지침이 문서에 없음.** 헤더는 서버가 붙이는 전제로 쓰여 있고, 이 레포 판단으로는 앱 바이너리에 넣을 값이 아님

## 유효기간

**2026-10-21 확인 기준.** user profiles 본체는 베타 헤더 없이 쓰는 형태이고, 조직 webhook은 `webhooks-2026-10-07` 베타임. 다시 볼 때는 [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서 user profiles와 webhooks 항목이 더 붙었는지부터 보고, 확인할 것 셋. **webhook 이벤트 배달이 실제로 시작됐는지**(지금은 문서가 안 쏜다고 명시), **집행 조치 기능이 계정 팀 문의 없이 켜지는 기본값이 됐는지**, 그리고 **Bedrock·Google Cloud가 ID를 조회하게 바뀌었는지.** 마지막 것은 바뀌면 조용히 틀린 ID를 보내던 코드가 400으로 드러나므로 배포 전에 알아야 하는 변경임.
