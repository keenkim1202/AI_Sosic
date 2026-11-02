---
title: Usage and Cost Analytics API, 비용 계기판이 5분에서 4시간으로 느려지는 교환
source: https://platform.claude.com/docs/en/manage-claude/usage-cost-analytics-api
author: Anthropic
published: 2026-10-29
collected: 2026-10-31
tags: [claude, anthropic, llm-cost, finops, observability, migration]
---

출처: [Usage and Cost Analytics API](https://platform.claude.com/docs/en/manage-claude/usage-cost-analytics-api) · [Usage and Cost API (legacy)](https://platform.claude.com/docs/en/manage-claude/usage-cost-api) · [API 릴리스 노트 2026-10-29](https://platform.claude.com/docs/en/release-notes/api) (전부 **2026-10-31 확인**)

## 요약

2026-10-29에 **Usage and Cost Analytics API**가 베타로 나오고, 같은 날 기존 Usage and Cost API(`/v1/organizations/usage_report/messages`, `/v1/organizations/cost_report`)가 **legacy**로 내려감. 레거시 페이지는 "is a legacy API and remains supported"까지만 적고 **제거 날짜를 적지 않았음.** 새 쪽은 엔드포인트 둘(`/v1/organizations/analytics/usage`, `/v1/organizations/analytics/cost`)이고 `anthropic-beta: analytics-usage-cost-2026-08-24` 헤더가 매 요청 필요함.

이 교체에서 실제로 중요한 것은 차원이 늘었다는 홍보 문구가 아니라 **신선도를 내주고 차원을 받았다는 것**임. 레거시는 데이터가 **보통 5분 안에** 올라왔음. 새 API는 **보통 4시간, 최대 24시간**이고, 값이 **월말 뒤 약 7일까지 수정됨.** 즉 레거시 위에 분 단위 예산 가드레일이나 알림을 올려 둔 코드는 그대로 옮기면 조용히 늦게 도는 계기판이 됨.

읽을 때 틀리기 쉬운 자리도 둘 바뀜. `amount`는 **달러가 아니라 센트의 소수 6자리 문자열**이고(`"41280.000000"`이 **$412.80**), **기본 워크스페이스가 `workspace_id: null`에서 실제 `wrkspc_` ID로** 바뀜. 그리고 `limit`의 의미가 **시간 버킷 수에서 행 수로** 옮겨감.

## 두 엔드포인트와 붙이는 모양

| 엔드포인트 | 돌려주는 것 | `order_by` 허용값 |
|---|---|---|
| `GET /v1/organizations/analytics/usage` | 토큰 종류별 수와 `requests` | `total_tokens`(기본), `output_tokens`, `uncached_input_tokens`, `requests`, `time` |
| `GET /v1/organizations/analytics/cost` | `amount`, `list_amount`, `currency`(항상 `"USD"`), `requests` | `amount`(기본), `list_amount`, `time` |

인증은 조직 종류로 갈림. **Claude Console 조직은 Admin API 키**(또는 `org:admin` 스코프 OAuth 토큰), **Claude Enterprise 조직은 `read:analytics` 스코프 키**임. 문서가 Admin API 키를 쓸 거면 **수집 서비스에 전용 키를 주고 시크릿 매니저에 넣고 주기적으로 교체하라**고 적고, 가능하면 Workload Identity Federation 규칙으로 단기 토큰을 받으라고 권함. 장기 키를 아예 두지 않는 쪽이 낫다는 뜻임.

```bash
curl --globoff "https://api.anthropic.com/v1/organizations/analytics/usage?\
starting_at=2026-09-01T00:00:00Z&\
ending_at=2026-09-08T00:00:00Z&\
time_bucket_width=1d&\
group_by[]=model&\
limit=10" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
  -H "x-api-key: $ANTHROPIC_API_KEY"
```

SDK와 `ant` CLI는 두 헤더를 대신 보내고 남은 페이지를 읽는 동안 알아서 가져옴. 그래서 **SDK 경로에서 `limit`은 페이지 크기일 뿐**임.

행의 모양은 셋으로 갈림. `starting_at`·`ending_at`이 구간, `group` 이 모든 차원, `usage` 또는 `cost` 가 지표임. 규칙 넷이 여기 붙음.

- **`group_by[]`에 넣지 않은 차원은 `group`에서 전부 `null`.** 넣었는데도 값이 없는 행(코드 실행 사용량의 `model` 같은 것)도 `null`임
- **`group_by[]`는 최대 5개.** 비용 쪽의 `cost_type`과 `token_type`은 이 5개 제한에 들어가지 않고 `limit`도 안 걸림. 그래서 **페이지가 `limit`보다 많은 행을 담을 수 있음**
- **`time_bucket_width`에 기본값이 없음.** 안 주면 전 구간 합계 한 행이 나옴. `1m`·`1h`·`1d` 중 하나를 주면 UTC 버킷으로 쪼개짐. 사용량이 없는 버킷은 **행이 아예 안 나오므로 0을 직접 채워야 함**
- **기본 정렬이 시계열이 아니라 랭킹.** `order_by` 지표 기준 큰 것부터임. 시계열이 필요하면 `time_bucket_width`와 `order_by=time`을 같이 줘야 함

필터는 대괄호 표기이고(`models[]=claude-opus-5-5`) 한 필터에 **최대 100개 값**임. 같은 필터의 여러 값은 OR, 다른 필터끼리는 AND. 그룹화와 필터는 독립이라 그룹화하지 않은 차원으로도 필터할 수 있음.

날짜 범위 제약이 레거시보다 빡빡함. `starting_at`은 **2026-01-01 이후이고 최근 365일 안**, 한 요청은 **최대 31일**, `time_bucket_width`를 주면 `ending_at`이 필수이고 **`1m`은 최대 24시간**임. 양 끝은 버킷 경계로 내림되고, `time_bucket_width`가 없으면 시 단위로 내림됨.

## 레거시에서 실제로 갈리는 것

| 항목 | Usage and Cost API (legacy) | Usage and Cost Analytics API |
|---|---|---|
| 베타 헤더 | 없음 | `analytics-usage-cost-2026-08-24` 매 요청 |
| 신선도 | 보통 **5분** | 보통 **4시간**, 최대 **24시간**, 월말 뒤 **약 7일**까지 값이 수정됨 |
| 응답 모양 | 시간 버킷 목록(오래된 것부터, 빈 버킷 포함) 안에 `results` | 랭킹된 행 목록. 사용량 없는 조합은 행이 없음 |
| 버킷 파라미터 | `bucket_width`, 기본 `1d` | `time_bucket_width`, 기본값 없음 |
| `limit` | 페이지당 **시간 버킷 수** | 페이지당 **행 수**(1~1,000, 기본 20) |
| 날짜 범위 | 제한 없이 앞으로 페이징 | 2026-01-01 이후, 최근 365일, 요청당 31일 |
| 사용자 차원 | `account_ids[]`, `service_account_ids[]` | `actor_ids[]` 하나로 합치고 `group_by[]=actor` |
| 비용 분해 | `group_by[]=description` 의 문자열 | `description` 없음. `cost_type`·`token_type` 등 구조화 차원 |
| 오타난 ID | 에러 | **조용히 아무것도 안 맞음** |
| 기본 워크스페이스 | `workspace_id: null` | 실제 `wrkspc_` ID |
| 금액 | `"123.45"` | `"123.450000"` 과 `list_amount` |
| 코드 실행 | 사용량 리포트에 없음 | `model: null` 인 사용량 행으로 나옴 |

레거시 일간 리포트를 그대로 재현하려면 `time_bucket_width=1d`에 같은 `group_by[]`를 주고 `limit=1000`으로 페이징한 뒤 `starting_at`으로 묶고 빈 날을 0으로 채우면 됨. 날짜 순서가 필요하면 `order_by=time&order=asc`.

**"오타난 ID가 조용히 아무것도 안 맞는다"가 이 표에서 제일 위험한 줄임.** 레거시는 모르는 API 키나 워크스페이스 ID에 에러를 줬는데, 새 API는 틀린 `models[]` 이름까지 포함해 그냥 빈 결과를 줌. 대시보드 쿼리에 모델 ID를 하드코딩해 두고 모델이 바뀌면 **0원이 정상처럼 보임.** 필터를 쓰는 쿼리는 필터 없는 합계와 대조하는 검증을 같이 두는 편이 안전함.

## 신선도가 이 API 의 성격을 정함

응답마다 `data_refreshed_at`이 실리고, **그 시점 이후 사용량은 결손이거나 부분적**임. 아직 아무 데이터도 없으면 `data_refreshed_at`이 `null`이고 `data`가 빔. 한 쿼리의 모든 페이지는 같은 스냅샷을 읽으므로 페이지 사이에 값이 흔들리지는 않음.

문서가 권하는 사용법이 둘로 갈림. **반복 호출에서 안정된 값이 필요하면 `ending_at`을 이미 본 `data_refreshed_at` 이하로 두고**, **최종 총액이 필요하면 끝난 지 7일 넘은 달만 조회**하라는 것임. 문서의 예시가 "10월 1일 값은 11월 7일경까지 바뀔 수 있다"임.

여기서 나오는 판단이 하나임. **이 API는 사후 귀속과 정산용이고, 실시간 지출 차단기를 올릴 자리가 아님.** 레거시의 5분을 전제로 "분당 폴링해서 임계값 넘으면 기능을 끈다"는 구조를 만들어 뒀다면, 옮기는 것이 아니라 **다른 신호를 찾아야 함.** 요청 응답의 `usage` 블록을 직접 집계하는 쪽이 그 목적에는 맞음. [OpenTelemetry 내보내기](2026-10-14-claude-api-opentelemetry.md)가 "과금 숫자는 Usage API 로 보라"고 문서가 지시한다고 적어뒀는데, 그 지시의 대상인 Usage API가 이번에 **느린 쪽으로 갈린 것**임. 즉 지금은 **빠르지만 불완전한 경로(스팬, 응답 `usage`)와 느리지만 권위 있는 경로(이 API) 둘을 따로 두는 모양**이 맞음.

## actor 와 api_key_id 는 다른 질문에 답함

`actor`는 요청이 귀속된 주체임. 사용자, 서비스 계정, 연합된 클라우드 신원, 그리고 그 어느 것도 아닐 때는 **API 키 자신**임. 귀속이 안 되는 사용량은 `actor: null`.

| 필터 | 맞추는 것 |
|---|---|
| `api_key_ids[]=apikey_...` | 그 키가 인증한 **모든 요청**. 누구에게 귀속됐는지와 무관 |
| `actor_ids[]=apikey_...` | **키 자신에게 귀속된** 요청만 |

개인 키와 서비스 계정 키는 연결된 신원처럼 동작해서 `actor`에 그 신원이, `api_key_id`에 키가 들어감. 반면 **레거시 워크스페이스 키는 신원에 연결되지 않아 `actor`와 `api_key_id`가 둘 다 그 키**가 됨. 예외가 하나 있음. **Claude Code 가 로그인 시 만들어 주는 키**는 Claude Code 워크스페이스에 살고 사용량이 **그 멤버에게** 귀속됨.

API 키 없이 들어온 요청(Console, API 키 없는 Claude Code 사인인, WIF 토큰)은 `api_key_id`가 `null`임. 그 몫을 `api_key_ids[]` 필터에 포함하려면 **빈 항목 `api_key_ids[]=`를 넣어야 함.** 모르고 지나치면 필터 쓴 합계가 전체보다 작게 나옴.

Claude Code 의 사용자별 비용은 `products[]=claude_code&group_by[]=actor`로 뽑힘. 문서가 이 FAQ를 따로 둠.

## 비용 숫자를 읽을 때

`cost` 객체가 넷을 가짐. `amount`(할인 적용 후, 크레딧 적용 **전**), `list_amount`(정가), `currency`, `requests`. **`group_by[]`에 `cost_type`이나 `token_type`을 주면 `requests`가 모든 행에서 `null`**이 되므로, 같은 쿼리로 단가를 역산하려 들면 분모가 사라짐.

Priority Tier 가 함정임. 약정으로 처리된 요청은 사용량 쪽에 `service_tier: "priority"`로 나오지만 **`amount`와 `list_amount`에 토큰 비용을 0으로 기여함.** 약정 수수료는 따로 청구되고 **이 데이터에 없음.** 그 요청의 웹 검색은 표준 단가로 과금되고, 약정 용량을 넘긴 요청은 표준 티어로 처리되고 그렇게 가격이 매겨짐. 레거시는 Priority Tier 비용을 **비용 엔드포인트에서 아예 빼 버렸으므로**, 이쪽이 더 정확해진 것이 아니라 **0 이 섞인 행이 보이게 된 것**임. 약정을 쓰는 쪽은 이 API 합계를 그대로 "우리가 쓴 돈"으로 적으면 안 됨.

토큰 타입은 비용 쪽에서 모델당 **다섯 행**(`uncached_input_tokens`, `cache_creation` 두 종, `cache_read_input_tokens`, `output_tokens`)으로 갈리고, 거기에 `web_search`·`code_execution`·`session_usage` 가 각각 한 행 붙음. [캐시 진단](2026-09-23-prompt-cache-diagnostics.md)으로 미스 원인을 보고, 절감이 실제로 청구서에 나타났는지는 여기서 `token_type=cache_read_input_tokens` 행으로 대조하는 짝이 됨.

## 앱을 운영하는 쪽에 걸리는 자리

차원 목록에 **`user_profile_id`**가 있음. [User Profiles](2026-10-20-user-profiles-end-customer-attribution.md)로 최종 사용자별 귀속을 붙여 둔 앱이라면, 그 ID가 그대로 비용 분해 축이 된다는 뜻임. **사용자당 원가를 뽑는 경로가 이것으로 처음 생김.** 단 신선도 때문에 사용자별 과금을 실시간으로 깎는 데는 못 쓰고, 월 단위 정산 쪽임.

플랫폼별로 비는 칸이 있음. **Claude Platform on AWS**는 두 엔드포인트가 다 있지만 **호출한 엔드포인트의 AWS 리전에 있는 워크스페이스만** 덮으므로 리전마다 따로 불러야 하고, `user_profile_id`·`claude_tag_category`·`claude_tag_user_id`와 Managed Agents 차원 넷(`agent_id`, `environment_id`, `deployment_id`, `session_id`)이 **모든 행에서 `null`**임. **Claude Models on Google Cloud (Anthropic managed)**는 조직 레벨 base URL과 Google Cloud 자격으로 부르고 `usageAnalytics.list`·`costAnalytics.list` 권한이 필요함.

내 조직에 해당하지 않는 차원은 모든 행에서 `null`이고 그 필터는 아무것도 안 맞음. **예외가 `rbac_group_id` 하나**인데, teams 를 못 쓰는 조직에서는 `null`이 아니라 **에러**를 돌려줌. 차원을 루프로 돌려 보는 탐색 스크립트가 여기서 걸림.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude API OpenTelemetry 내보내기](2026-10-14-claude-api-opentelemetry.md) | 그 문서가 "과금 숫자는 Usage API 로 보라"고 적은 **그 Usage API 가 이번에 갈렸음.** 스팬은 빠르고 불완전, 이쪽은 느리고 권위 있음 |
| [User Profiles](2026-10-20-user-profiles-end-customer-attribution.md) | `user_profile_id` 가 그룹화 차원으로 들어옴. 최종 사용자별 원가를 뽑는 경로가 여기서 열림 |
| [대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md) | 그쪽의 라우팅 30%·캐싱 50% 절감을 **사후에 검증할 자리.** 다만 월말 뒤 7일까지 값이 흔들림 |
| [Cache diagnostics](2026-09-23-prompt-cache-diagnostics.md) | 미스 원인은 그쪽, 절감의 금액 확인은 `token_type=cache_read_input_tokens` 행 |
| [API 버전 2026-10-21](2026-10-21-claude-api-version-migration.md) | 이 API를 부르는 SDK 가 그 버전에서 메이저로 뛴 것들임. `anthropic-version` 헤더가 매 요청 필요하다는 서술이 같은 자리 |
| [Fast mode 정식 공개](2026-10-15-fast-mode-opus-5-5-ga.md) | `speed` 가 그룹화 차원에 있어 fast 와 standard 의 실제 지출 비중을 가를 수 있음 |

## 짚어야 할 것

- **개인 계정은 이 API를 못 씀.** 문서 상단이 "The Admin API is unavailable for individual accounts"로 못 박고 Console 에서 조직을 만들라고 넘김. 1인 개발자도 조직 설정이 전제임
- **베타 헤더 날짜가 `2026-08-24`인데 공개는 2026-10-29임.** 두 달 차이의 이유는 어느 페이지에도 없고 **확인할 수 없었음.** 헤더 이름을 날짜로 읽어 공개 시점을 추정하면 안 되는 사례임
- **레거시의 제거 일정이 없음.** "remains supported" 까지만 적혀 있고 폐기 날짜도 공지 경로도 없음. 언제까지 쓸 수 있는지는 **확인할 수 없었음**
- **레이트 리밋을 확인할 수 없었음.** 레거시 문서는 "분당 1회 폴링"을 권고로 적어뒀는데 새 문서에는 해당 FAQ가 없음. 폴링 주기를 레거시 감각으로 잡으면 안 됨
- **실제 호출로 대조하지 못했음.** Admin API 키가 없어 응답을 직접 받지 못했고, 위 필드 이름과 값은 전부 문서 본문과 문서의 예시 JSON에서 읽은 것임. 응답 예시의 토큰 수와 금액은 **문서가 만든 예시 값이고 실측이 아님**
- 문서가 통합 구축 시 **User-Agent 헤더를 설정하라**고 권하는데, 그것이 레이트 리밋이나 지원에 영향을 주는지는 적혀 있지 않음

## 유효기간

**2026-10-31 확인 기준이고 베타임.** 신선도 수치(4시간 / 24시간 / 7일)와 `limit` 범위(1~1,000, 기본 20)는 베타 동안 움직일 수 있는 종류임. 다시 볼 때 확인할 것 셋. **베타를 벗어나 헤더가 필요 없게 됐는지**, **레거시 Usage and Cost API 에 폐기 일정이 붙었는지**, 그리고 **레이트 리밋이 문서에 숫자로 적혔는지.** 앞의 둘은 [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서 analytics 항목이 더 붙었는지로 먼저 잡힘. 차원 목록은 제품이 늘 때마다 늘어나므로 위 표를 고치지 말고 [usage](https://platform.claude.com/docs/en/api/beta/organization/analytics/usage/list)·[cost](https://platform.claude.com/docs/en/api/beta/organization/analytics/cost/list) 레퍼런스에서 새로 받을 것.
