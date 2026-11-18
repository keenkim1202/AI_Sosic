---
title: Linked billing, 과금만 고객 조직으로 넘어가고 레이트 리밋은 내 것으로 남음
source: https://platform.claude.com/docs/en/manage-claude/oauth-apps/organization-billing-connections
author: Anthropic
published: 2026-11-17
collected: 2026-11-18
tags: [claude, anthropic, billing, oauth, gateway, user-profiles, rate-limits]
---

출처: [Linked billing](https://platform.claude.com/docs/en/manage-claude/oauth-apps/organization-billing-connections) · [API 릴리스 노트 2026-11-17](https://platform.claude.com/docs/en/release-notes/api) · [User profiles for end-customer attribution](https://platform.claude.com/docs/en/manage-claude/user-profiles) (전부 **2026-11-18 확인**)

## 요약

**2026-11-17에 linked billing이 베타로 열렸음.** 고객이 자기 Claude 조직을 내 앱에 **OAuth로 한 번 연결**해 두면, 그다음부터 요청마다 `billing_policy.bill_to`로 **누가 낼지를 고를 수 있음.** 베타 헤더는 `user-profiles-billing-2026-10-01`이고, 연결 단위는 [user profiles](./2026-10-20-user-profiles-end-customer-attribution.md)의 `uprof_` 프로필 그대로임. 설계할 때 제일 먼저 박아야 할 문장은 과금이 아니라 그 옆 문장임. **"Requests run in your organization, so your rate limits, model access, service tier, data retention, and encryption keys apply."** 즉 **돈만 넘어가고 레이트 리밋·모델 접근·서비스 티어·데이터 보관은 전부 내 조직 것**임. 고객의 ZDR·BAA·CMEK는 이 트래픽에 **적용되지 않음**. 그리고 이 기능은 문서가 대상을 못 박아 둠. **"developers whose app users are primarily Claude Platform customers."** 고객이 Claude 조직을 안 가진 일반 소비자 앱이면 해당 사항이 없음.

## 적용 대상이 좁음, 먼저 여기서 갈림

| 고객 유형 | 결과 |
|---|---|
| Claude 조직이 **Anthropic에 직접** 결제 (선불 크레딧 또는 인보이스) | 됨 |
| AWS·Google Cloud·Azure **마켓플레이스**를 통해 결제 | `403 user_profile_not_billable` |
| **Claude 구독 플랜만** 있음 (API 조직 없음) | `403 user_profile_not_billable` |
| Claude 조직 자체가 없음 (일반 소비자) | 애초에 연결 플로를 못 탐 |

iOS 앱 쪽으로 번역하면 이렇게 됨. B2C 앱은 **이 기능의 대상이 아님.** 소비자가 자기 Claude 조직을 만들고 Admin 또는 Billing 역할로 승인하는 플로를 통과할 리가 없음. 쓸 자리는 **고객사가 Claude를 이미 쓰고 있는 B2B 제품**, 즉 사내 배포 앱이나 기업 고객용 도구임. 그 바깥이면 비용 구조는 그대로 내 부담이고, iOS 앱에서 키 없이 과금을 워크스페이스로 보내는 경로는 여전히 [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md)의 App Attest 쪽임. 둘은 **푸는 문제가 다름.** App Attest는 "앱에 키를 안 넣는 법"이고, linked billing은 "내가 안 내는 법"임.

그리고 **고객이 자기 데이터 계약을 들고 오지 못함.** 요청이 내 조직에서 돌기 때문임. HIPAA·FERPA·CMEK가 필요한 고객은 연결하지 말라고 문서가 직접 적음.

## `billing_policy` 와 요청 모양

```json
{
  "model": "claude-sonnet-5-5",
  "max_tokens": 1024,
  "messages": [{"role": "user", "content": "Hello, Claude"}],
  "billing_policy": {"type": "billing_policy", "bill_to": "user_profile"}
}
```

헤더는 둘임. `anthropic-user-profile-id: uprof_...` 와 `anthropic-beta: user-profiles-billing-2026-10-01`. 베타 헤더는 `billing_policy`를 **설정할 때마다** 필요하고, `bill_to: "self"`이거나 필드를 아예 빼면 선택 사항임.

지원 범위가 좁음.

| 엔드포인트 | `bill_to: "user_profile"` |
|---|---|
| `/v1/messages` | 됨. 단 **과금되는 code execution 기능을 쓰면 400** |
| `/v1/chat/completions` | 됨 |
| `/v1/messages/count_tokens` | 필드를 **받고 무시함**. 보낼 본문 그대로 토큰을 셀 수 있게 |
| `/v1/messages/batches` | **안 됨.** 배치 안의 해당 요청만 `errored`로 떨어지고 아무도 과금되지 않음. 나머지 요청은 정상 실행 |

`"user_profile"`을 붙이는 기준은 문서가 책임을 내게 넘김.

> "Set `bill_to: "user_profile"` only on requests the customer directs you to make: the Claude usage that the customer and their users cause in your product. Don't bill a customer for your own usage, such as testing, evaluations, or work for other customers. The Claude API can't check this for you, so it's your responsibility."

내 테스트와 평가를 고객에게 떠넘겨도 API는 못 잡는다는 뜻임. [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md)처럼 CI에서 평가를 돌리는 하네스가 프로덕션과 같은 코드 경로를 쓰고 있으면, 그 경로에 `bill_to`가 어떻게 들어가는지를 배포 전에 확인해야 하는 자리임.

## 설계에서 제일 아픈 곳, 레이트 리밋은 안 넘어감

문서 한 문장이 전부를 정함.

> "Requests run in your organization, so your rate limits, model access, service tier, data retention, and encryption keys apply."

고객 수가 늘면 **비용 곡선은 평평해지는데 레이트 리밋 곡선은 그대로 가파름.** linked billing을 "고객이 내니까 우리는 무제한"으로 읽으면 정확히 반대로 터짐. 돈이 안 드니까 호출을 아낄 이유가 사라지고, 그 늘어난 호출이 전부 내 조직의 분당 한도를 먹음. 용량 계획은 과금 모델과 **분리해서** 세워야 함.

[대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md)의 레버들 중 캐시와 모델 선택은 여기서 **값어치가 비대칭**이 됨. 프롬프트 캐시는 내 돈을 아끼지 않지만(고객이 내므로) 내 레이트 리밋은 아껴 줌. 반대로 싼 모델로 내리는 것은 고객 청구서를 줄이는 쪽이라 **내게 직접 돌아오는 이득이 없음.** 이 비대칭을 모르면 최적화 우선순위를 거꾸로 잡게 됨.

## 거절이 설계의 일부임

`bill_to: "user_profile"`은 **조용히 내게 청구되지 않음.** 과금이 불가능하면 요청이 거절됨. 그래서 거절 처리가 선택이 아님.

| 코드 | 의미 | 할 일 |
|---|---|---|
| `403 user_profile_not_billable` | 과금 불가. **이유는 안 알려줌** | 재시도 안 됨(새 연결 직후 10초만 예외). `bill_to: "self"`로 재전송하면 내 비용으로 서빙 가능 |
| `402 linked_organization_spend_limit_reached` | 고객이 **스스로 건 한도**(내 앱용 spend cap 또는 조직 월 한도)에 닿았거나 크레딧이 없음. 둘 중 뭔지는 안 알려줌 | 재시도 안 됨. Claude가 답을 시작하기 **전에** 거절됨 |
| `402 linked_organization_billing_unavailable` | 조직이 정지됐거나, **답변이 시작된 뒤** 크레딧이 떨어짐 | 재시도 안 됨. 고객에게 계정 확인을 안내 |
| `503 overloaded_error` | 과금 가능 여부를 지금 확인 못 함 | `retry-after` 지켜 백오프. 서빙도 과금도 안 됨 |
| `400 user_profile_required` | `bill_to: "user_profile"`인데 프로필 헤더가 없음 | 내 버그 |

**402 둘이 갈리는 지점이 운영에서 중요함.** `spend_limit_reached`는 생성 전 거절이라 토큰이 안 나가고, `billing_unavailable`은 "Claude가 답을 시작한 뒤 크레딧이 떨어진" 경우를 포함함. 즉 후자는 **스트리밍 도중에 끊기는 모양**이 될 수 있는 쪽임. 둘을 한 핸들러로 뭉개면 UI에서 구분이 사라짐.

그리고 응답은 **누가 냈는지 말해 주지 않음.** 문서가 "A successful response doesn't say who was billed, so keep your own record"라고 적음. 요청별로 어느 `bill_to`를 보냈는지는 내 로그에만 남음.

## 고객이 건 한도는 내 쪽에서 안 보임

고객은 **내 앱 전용 월 spend cap**을 USD 정수로 걸 수 있음. 연결 승인 화면에서 처음 정하고, 이후에는 Console의 Linked platforms 페이지에서 바꿈. 여기서 설계에 걸리는 비대칭이 둘임.

- **cap 값도, 소진율도 내가 읽을 수 없음.** 경고 없이 402가 시작됨. 남은 예산을 보여 주는 UI를 만들 수 없다는 뜻이고, 접근 가능한 신호는 402가 오기 시작했다는 사실 하나뿐임
- **거절은 UTC 기준 다음 달 1일 00:00까지, 또는 고객이 한도를 올릴 때까지 계속됨.** 한도 변경이 반영되는 데 약 1분 걸림

cap은 **협상 할인 전 정가(list price)로 토큰 비용과 웹 검색 요금을 셈.** 그래서 고객이 실제로 받는 청구서와 cap이 센 금액이 어긋남. 고객 지원에서 이 질문이 오면 답할 쪽은 Anthropic이 아니라 나임. 문서가 지원 책임을 내게 둠.

앱이 심사 중이거나 미승인 상태면 조건이 더 셈.

| | 테스트·심사·미승인 | 정식 공개 후 |
|---|---|---|
| 승인 화면의 월 한도 | **필수, $50 USD 이하** | 선택 |
| 금액 입력칸 초깃값 | **$50 USD** | 비어 있음 |
| "한도 없이 계속" 버튼 | 없음 | 있음 |
| Console에서 cap 제거 | 불가 | 가능 |
| 고객이 걸 수 있는 최대 cap | **$50 USD** | 실질 상한 없음 |

$50 상한은 **파일럿 설계를 직접 제약함.** 정식 공개 전에는 고객 한 곳당 월 $50 이상을 태울 수 없으므로, 실사용 부하 검증을 이 경로로 하려는 계획은 성립하지 않음.

## 연결이 끊긴 걸 알려주는 경로가 없음

고객이 Console에서 내 앱을 끊을 수 있고, **그때 webhook도 이메일도 오지 않음.** 문서 문장이 "Anthropic doesn't notify you when they do: you get no webhook event and no email"임. 상태를 알려면 직접 읽어야 함.

```bash
curl "https://api.anthropic.com/v1/user_profiles/uprof_01HV9Z?oauth_app_id=YOUR_CLIENT_ID" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "anthropic-version: 2026-10-21" \
  -H "anthropic-beta: user-profiles-search-2026-08-30"
```

`connection_status.type`이 셋임. `connected`(연결 하나 이상 살아 있음), `disconnected`(연결됐었고 전부 끝남, `last_revoked_at`과 `last_revoked_reason.type`이 붙음), `not_connected`(한 번도 연결된 적 없음). 모르는 값과 `null`은 **unknown으로 취급**하라고 문서가 적음.

그런데 `connected`가 성공을 보장하지 않음.

> "`connected` doesn't promise that a request will succeed. It says only that a connection is active."

즉 **상태를 폴링해도 거절 처리를 생략할 수 없음.** 폴링은 "끊긴 고객에게 계속 보내는 것"을 줄이는 용도지 게이트가 아님. 이 모양은 [user profiles](./2026-10-20-user-profiles-end-customer-attribution.md)가 집행 상태 webhook이 아직 안 온다고 적어 둔 것과 같은 자리임. 그 문서가 "webhook 수신기보다 폴링을 먼저 만들라"고 결론 낸 것이 여기서도 그대로 유효함.

## 사용량 집계에서 0 이 두 종류가 됨

Console과 [Usage and Cost Analytics API](./2026-10-29-usage-cost-analytics-api.md)는 **고객이 낸 요청도 센다.** 다만 금액 쪽 처리가 이렇게 됨.

> "On the cost endpoint, a request that your customer pays for adds 0 to `amount` and its list price to `list_amount`."

그 문서에 이미 적어 둔 Priority Tier 함정과 **모양이 다름.** 약정 요청은 `amount`와 `list_amount` **둘 다** 0을 기여하고, 고객 결제 요청은 `amount`만 0이고 `list_amount`는 정가가 들어감. 그래서 cost 엔드포인트의 행을 읽을 때 갈래가 셋이 됨.

| `amount` | `list_amount` | 읽는 법 |
|---|---|---|
| 0 | 0 | Priority Tier 약정으로 처리됨. 약정 수수료는 이 데이터에 없음 |
| 0 | 0 초과 | **대개 그 행의 요청을 고객이 전부 결제함** |
| 0 초과 | 0 초과 | 내가 낸 보통 요청 |

API에 **누가 냈는지를 가리키는 필드도 필터도 `group_by[]` 값도 없음.** 위 조합이 유일한 단서임. 문서도 "usually means"라고 적지 단정하지 않음.

그리고 이 수치로 고객을 끊으려 들면 안 됨. Console은 약 1시간마다 갱신되고 요청이 보이기까지 몇 시간 걸림. 실시간 집행용 데이터가 아니라는 경고가 두 문서에 다 있음.

## 인증 구조, 오퍼레이터 서비스 계정 하나가 전부를 들고 있음

연결을 거쳐 과금하려면 요청이 **앱의 operator service account**로 인증돼야 함. Workload Identity Federation으로 만들고, 한 서비스 계정은 **앱 하나만** 운영할 수 있음. 클라이언트 시크릿은 Anthropic이 발급하지 않고, 백엔드가 RFC 7523 방식 `private_key_jwt` client assertion을 보냄.

여기 경고가 하나 박혀 있음. **그 자격증명은 이 앱으로 연결된 아무 고객에게나 과금할 수 있음.** 방어는 `anthropic-user-profile-id` 헤더가 의도한 고객과 맞는지를 내가 확인하는 것뿐임. 멀티테넌트 서버에서 헤더를 요청 컨텍스트가 아니라 캐시나 전역에서 끌어오는 코드면 **남의 고객에게 청구되는 사고**가 됨. 그리고 성공 응답이 누가 냈는지 말해 주지 않으므로, 그 사고는 응답으로는 안 보임.

연결별 access token을 쓰면 폭발 반경이 줄어듦. 토큰 하나는 **그 고객만** 과금할 수 있고, 수명은 최대 1시간임. `offline_access`를 같이 받으면 refresh token이 오고 최대 1년, 갱신해도 그 1년은 안 늘어남. 다만 연결 토큰으로는 `bill_to: "self"`를 보낼 수 없음(400). 즉 **402를 받고 내 비용으로 재전송하려면 백엔드가 오퍼레이터 자격증명을 따로 들고 있어야 함.** 두 경로를 다 구현해야 한다는 뜻이고, 이것이 이 설계에서 제일 쉽게 빠뜨리는 조각임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [User Profiles](./2026-10-20-user-profiles-end-customer-attribution.md) | 같은 `uprof_` 프로필을 쓰는 확장임. 그쪽은 **집행 폭발 반경**을 쪼개는 용도였고, 여기서 같은 ID가 **과금 주체**가 됨. 프로필을 안 붙이고 있었다면 선행 작업이 그 문서임 |
| [Usage and Cost Analytics API](./2026-10-29-usage-cost-analytics-api.md) | `amount` 0 행의 의미가 하나 더 늘어남. 그쪽이 적어 둔 Priority Tier 0 과 모양이 달라서 `list_amount`로 갈라야 함 |
| [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md) | iOS 앱에서 과금 주체를 옮기는 **다른 경로**. App Attest는 앱에 키를 안 두는 것이고 과금은 내 워크스페이스로 옴. linked billing은 과금을 고객에게 넘기는 것이라 푸는 문제가 반대쪽임 |
| [거절 응답과 과금](./2026-09-24-refusal-billing-and-fallback.md) | "요청이 실패해도 돈이 나가는가"를 다루는 같은 계열. 여기서는 402·403이 **생성 전 거절이라 과금이 없다**는 쪽이 명시돼 있고, `billing_unavailable`만 생성 도중 사례를 포함함 |
| [대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md) | 비용 레버의 값어치가 뒤집히는 자리. 캐시는 레이트 리밋을 아껴 내게 이득이고, 모델 다운그레이드는 고객 청구서만 줄임 |

## 짚어야 할 것

- **Claude Platform OAuth 전체가 Beta Service임.** 문서 맨 위 경고가 "Endpoints, parameters, and behavior can change during the beta. It might not be suitable for production use"임. 프로덕션 과금 경로로 올리기 전에 읽어야 하는 문장이고, 이 저장소 판단으로는 **지금 단계에서 매출 경로를 여기 걸면 안 됨**
- **이 저장소 주인에게 당장 쓸 자리는 없을 가능성이 큼.** 문서가 대상을 "app users are primarily Claude Platform customers"로 못 박았음. 소비자 iOS 앱이면 해당 없음. 적어 두는 이유는 B2B 제품으로 갈 때 **과금 구조를 처음부터 다르게 잡을 수 있다**는 것과, 레이트 리밋이 안 넘어간다는 함정이 뒤늦게 알면 비싸기 때문임
- **고객 조직의 적격 여부를 내가 미리 확인할 길이 없음.** 문서가 "You can't see that from your side, so ask your Anthropic account team"이라고 적음. 즉 마켓플레이스 결제 고객인지 아닌지는 **403을 받고 나서야** 알게 됨. 온보딩 UX에서 이 실패를 설계에 넣어야 함
- **403 `user_profile_not_billable`이 이유를 말해 주지 않음.** 내 인증 문제, 연결 종료, 고객 조직 부적격이 전부 같은 코드로 옴. 문서가 확인 순서를 적어 두지만 응답만으로는 갈라지지 않음
- **spend cap을 읽는 API가 없음.** 값도 소진율도 못 봄. 확인한 범위에서 조회 경로는 존재하지 않고, 이게 없으면 "예산 80% 경고" 같은 기능을 만들 수 없음
- **레이트 리밋이 프로필 단위로 나뉘는지는 확인할 수 없었음.** 문서는 "your rate limits ... apply"까지만 적고, 한 고객이 내 조직 한도를 독식하는 것을 막는 장치가 있는지는 어느 절에도 없음. 고객별 쿼터는 내 게이트웨이에서 만들어야 하는 것으로 읽힘
- **고객이 낸 금액을 내가 볼 수 없음.** "Neither shows what the customer paid or the customer's rates." 고객에게 사용량 리포트를 내보내는 제품이면 금액은 내 쪽 추정값이 됨
- **연결 토큰 경로와 오퍼레이터 경로를 둘 다 구현해야 하는 것이 문서에 한 줄로만 있음.** 연결 토큰은 `bill_to: "self"` 재전송을 못 하므로, 402 폴백을 쓰려면 오퍼레이터 자격증명 경로가 따로 살아 있어야 함
- **마이그레이션 함정 하나.** `org:billing_attribution` 스코프가 deprecated 이고, Permissions 탭에서 한 번 제거하면 **다시 추가할 수 없음**. 되돌릴 수 없는 설정이라 순서를 틀리면 복구가 안 됨

## 유효기간

**2026-11-18 확인 기준이고 베타 헤더가 `user-profiles-billing-2026-10-01`, 링크 상태 조회 쪽이 `user-profiles-search-2026-08-30` 임.** 베타 서비스 고지가 붙어 있어 이 문서에서 가장 빨리 낡는 것은 **엔드포인트와 에러 코드의 모양**임. 다시 볼 때 확인할 것 넷. **베타를 벗어났는지**(벗어나면 프로덕션 판단이 달라짐), **Message Batches 가 지원 목록에 들어왔는지**, **spend cap 을 읽는 경로가 생겼는지**(생기면 예산 UI 를 만들 수 있음), 그리고 **고객이 끊었을 때의 webhook 이벤트가 생겼는지**. 마지막 둘은 지금 폴링으로 메우고 있는 구멍이라 생기면 설계가 바뀜. 위 표의 $50 테스트 상한은 앱 심사 정책 쪽이라 별도로 움직일 수 있음.
