---
title: Claude API OpenTelemetry 내보내기, 토큰 회계가 스팬 단위로 갈라짐
source: https://platform.claude.com/docs/en/manage-claude/opentelemetry
author: Anthropic
published: 2026-10-14
collected: 2026-10-15
tags: [claude, opentelemetry, observability, llm-cost, prompt-caching, compaction, eval]
---

출처: [OpenTelemetry for the Claude API](https://platform.claude.com/docs/en/manage-claude/opentelemetry) · [Use OpenTelemetry with the Claude SDK](https://platform.claude.com/docs/en/manage-claude/opentelemetry/sdk) · [Use OpenTelemetry with the Claude API](https://platform.claude.com/docs/en/manage-claude/opentelemetry/api) · [OpenTelemetry span reference](https://platform.claude.com/docs/en/api/opentelemetry/spans) · [API 릴리스 노트 2026-10-14](https://platform.claude.com/docs/en/release-notes/api) (2026-10-15 확인)

## 요약

Messages API 요청과 Managed Agents 세션을 OpenTelemetry 스팬으로 내보내는 기능이 **2026-10-14 에 베타로 붙음.** 경로가 둘이고 성질이 다름. **SDK 추적은 코드 몇 줄이고 권한이 필요 없는데**, **API 추적은 Console 에 목적지를 등록해야 하고 admin 역할이 필요함.** 이 저장소에 걸리는 것은 트레이스 그림이 아니라 **토큰 회계**임. [서버측 compaction](2026-09-20-claude-api-compaction.md)이 적어둔 과소 보고가 **같은 모양으로 한 번 더** 나타나서, `gen_ai.usage.*` 는 compaction 몫을 빼고 나오고 `anthropic.usage.compaction.*` 를 따로 더해야 총액이 됨. 반대쪽 함정도 있음. 한 요청에 SDK 스팬과 API 스팬이 **같은 이름**으로 둘 생기므로 둘 다 더하면 전부 두 배가 됨.

그리고 문서가 스스로 선을 그음. **내보낸 스팬은 API 사용량의 완전한 기록이 아님.** 전달이 best effort 라 유실될 수 있고 `traceparent` 의 sampled 플래그를 끄면 아예 안 나옴. 과금 숫자는 Usage API 로 보라고 문서가 직접 지시함. 즉 이것은 **비용 계기판을 옮길 곳이 아니라 디버깅과 eval 재료를 뽑을 곳**임.

## 두 경로를 먼저 갈라야 함

문서의 비교표에서 설계에 걸리는 행만 뽑음.

| 받는 것 | SDK 추적 | API 추적 |
|---|---|---|
| Messages API · Sessions API 요청 | 됨 | 됨 |
| 요청 안의 **모델 실행 하나하나** | 안 됨 | 됨 |
| Managed Agents **턴** | 안 됨 | 됨 |
| API 가 돌리는 **서버 툴** | 안 됨 | 됨 |
| 툴 러너가 돌리는 **클라이언트 툴** | 됨 | 안 됨 |
| 파일 업로드 같은 그 밖의 요청 | 됨 | 안 됨 |
| 설정하는 곳 | 코드 | Console |
| 필요한 권한 | 없음 | **admin** |
| 적용 범위 | SDK 를 쓰는 클라이언트 | 모든 클라이언트 |

읽는 순서는 이럼. 내 코드가 무엇을 했는지 보려면 SDK 쪽, API 안에서 무엇이 돌았는지 보려면 API 쪽임. 서버 툴 호출과 모델 재실행은 클라이언트에서 보이지 않으므로 **둘을 같이 켜야 요청 하나가 끊기지 않고 이어짐.** 단 Claude Platform on AWS 로 보낸 요청의 API 스팬은 별도 트레이스로 떨어진다고 적혀 있음.

SDK 는 **Python, TypeScript, C#, Go, Java, PHP, Ruby** 일곱 언어에서 스팬을 냄. 켜는 방법은 트레이서 프로바이더를 등록하는 것뿐이고, SDK 호출 코드는 그대로 둠.

## 토큰을 두 번 세지 않는 규칙

문서에 `Counting tokens` 절이 따로 있음. 규칙이 다섯 줄인데 하나라도 빠뜨리면 합계가 틀림.

- **한 요청에 소스가 둘이면 하나만 더함.** SDK 스팬과 API 스팬이 **둘 다 `anthropic.messages.create`** 라서 이름으로는 구분이 안 됨. 가르는 것은 span kind 임. SDK 가 `CLIENT`, 그 자식인 API 가 `SERVER`. `service.name` 으로도 갈림 (API 쪽이 `claude-platform`)
- **API 스팬은 `anthropic.messages.create` 아니면 그 자식 `anthropic.model_request` 중 하나만.** 둘 다 더하면 안 됨. 사용량 속성이 두 층에 겹쳐 있음
- **compaction 은 어느 쪽에도 안 들어 있음.** SDK 스팬이면 `anthropic.usage.compaction.input_tokens` 와 `..._output_tokens` 를 더함. API 스팬이면 `anthropic.compaction` 스팬을 더하고, **그러면서 같은 스팬의 `anthropic.usage.compaction.*` 속성은 더하지 않음.** 같은 토큰임
- **`anthropic.messages.count_tokens` 스팬의 숫자는 사용량이 아님.** 속성 설명이 "This count isn't usage. Don't include it when you sum token usage." 라고 못 박아 둠
- **캐시 속성은 더하지 않음.** `gen_ai.usage.input_tokens` 가 이미 캐시 읽기와 쓰기를 포함함

여기에 두 가지가 더 붙음. [fallback 모델](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback)이 응답을 서비스한 요청이면 그 스팬에는 **실제로 서비스한 모델의 사용량만** 들어 있고, API 는 fallback 이 끼어도 요청당 스팬을 하나만 냄. 그리고 스팬은 **같은 것이 두 번 도착할 수 있음**(at least once). 관측 플랫폼이 중복을 안 걷으면 trace ID 와 span ID 로 직접 한 번만 세야 함.

## 과소 보고가 같은 모양으로 한 번 더

[서버측 compaction](2026-09-20-claude-api-compaction.md)이 적어둔 결론이 이럼. 최상위 `usage.input_tokens` 가 compaction 몫을 빼고 나오므로 그 값을 읽던 대시보드는 compaction 을 켜는 순간 조용히 과소 보고로 바뀜. **텔레메트리 층에서 같은 일이 반복됨.** `gen_ai.usage.*` 는 `usage.input_tokens` 와 똑같이 compaction 을 뺀 값이고, 문서가 그 이유를 명시해 둠.

> "the `gen_ai.usage.*` attributes don't include compaction tokens"

읽어야 할 것은 이것임. **계기판을 OpenTelemetry 로 옮기는 것 자체가 해결이 아님.** 분리해서 내보내 주니 더할 수 있게 된 것이고, 더하는 코드를 안 쓰면 숫자는 전과 똑같이 작게 나옴. [대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md)의 레버 3(가시성·트립와이어·예산)을 세울 때 손대야 하는 자리가 `usage` 파싱에서 스팬 집계로 한 칸 옮겨간 것뿐임.

## 캐시 수치가 5분과 1시간으로 갈려 나옴

토큰 사용량 속성에 캐시 쪽이 넷 있음.

| 속성 | 내용 |
|---|---|
| `gen_ai.usage.cache_read.input_tokens` | 캐시에서 읽은 토큰 |
| `gen_ai.usage.cache_write.input_tokens` | 캐시에 쓴 토큰 |
| `anthropic.usage.cache_creation.ephemeral_5m_input_tokens` | **5분 캐시**에 쓴 토큰. Messages API 스팬에만 붙음 |
| `anthropic.usage.cache_creation.ephemeral_1h_input_tokens` | **1시간 캐시**에 쓴 토큰. Messages API 스팬에만 붙음 |

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 "히트율 낮을 때 확인할 8가지"를 세어둔 문서인데, 그 확인을 **TTL 별로 갈라서 할 계측이 처음 1차 문서에 생긴 것**임. 5분 쓰기만 늘고 1시간 쓰기는 비어 있다면 TTL 선택이 의도와 다르게 적용되고 있다는 신호로 읽힘.

주의할 데가 두 곳임. 스트리밍 응답의 **SDK 스팬**에는 이 두 값이 **첫 출력 토큰 전에 쓴 분량까지만** 들어감. 그리고 스팬에 들어 있는 것은 **토큰 수뿐이고 요율이 아님.** 비용으로 바꾸려면 모델별 배수를 직접 곱해야 하는데, 그 배수가 모델마다 같지 않음. [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)에 적어둔 캐시 읽기 예외가 그 얘기임. 스팬 집계로 비용 그래프를 그리면 **모델을 섞어 쓰는 순간 단가 테이블이 코드 안에 또 하나 생김.**

## 스팬은 과금 기록이 아님

문서가 Note 로 직접 적음.

> "Exported spans aren't guaranteed to be a complete record of your API usage."

빠지는 경로가 둘임. 전달이 **best effort** 라 유실된 스팬은 다시 보내지지 않고, 호출하는 쪽이 `traceparent` 헤더의 sampled 플래그를 끄면 **그 요청의 스팬도 그 아래 스팬도 아예 생성되지 않음.** 유출된 키를 잡는 것처럼 API 키별 사용량을 전부 봐야 하는 일에는 Usage API 를 쓰라고 같은 Note 가 지시함.

목적지 쪽 운영 조건도 숫자로 박혀 있음. 조직당 목적지는 **10개**까지, 엔드포인트는 **443 포트 HTTPS 의 OTLP/HTTP(protobuf)** 여야 하고, TLS 핸드셰이크 5초·응답 헤더 5초·전달 전체 10초의 타임아웃이 걸림. 그리고 목적지가 **자동으로 비활성화되는 조건이 셋**임. `3xx` 리다이렉트를 주거나, 호스트명이 공인 IP 가 아닌 주소로 풀리거나, **24시간 동안 전달이 전부 실패**하면 멈춤. 비활성화되면 그 사이 스팬은 안 보내지고, 사람이 Console 에서 다시 켜야 함.

여기서 나오는 실무 결론 하나. **관측 플랫폼이 죽은 구간은 트레이스에 구멍으로 남고 메워지지 않음.** 사후 분석을 이 데이터에만 의존하는 설계는 안 됨.

## 콘텐츠 모드를 올리면 데이터 경로가 하나 늘어남

기본값은 **`metadata_only`** 임. 식별자·타이밍·상태·토큰 수와 내가 붙인 이름(에이전트·툴 이름, MCP 서버 이름)까지만 나감. `content` 로 올리면 **프롬프트와 응답 본문**이 스팬에 들어감. 걸리는 조건들.

- 스팬당 콘텐츠 기본 상한이 **0.75MB**. 많은 관측 플랫폼과 프록시의 1MB 요청 제한 아래로 잡아둔 값이고, 영어 **200k 토큰** 창이 들어가는 크기라고 적혀 있음. **1M 토큰 창은 약 4MB** 가 필요함
- 넘치면 잘리고 `anthropic.content.truncated` 가 `true` 로 붙음. **요청 스팬은 시스템 프롬프트와 가장 오래된 메시지부터** 버리고, 툴 스팬은 결과의 뒤쪽 블록부터 버림. 이미지와 문서는 타입만 남음
- 조직에 **HIPAA·FERPA·K-12 데이터 취급 제약**이 걸려 있으면 목적지는 모드와 무관하게 메타데이터만 내보냄. **SDK 스팬은 그 제약을 받지 않음**
- **zero data retention 에서도 `content` 를 쓸 수 있음.** Messages API 요청은 전달 전까지 메모리에만 들고 있다고 적혀 있음. Managed Agents 는 애초에 ZDR 대상이 아님

판단은 이럼. `content` 를 켜는 것은 성능 옵션이 아니라 **사용자 입력이 흘러가는 경로를 하나 더 만드는 일**임. 전달된 뒤의 보존과 접근 통제는 Anthropic 이 아니라 그 플랫폼 것이라고 문서가 못 박아 둠. 앱에서 받은 텍스트가 그 경로를 타면 [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md)에 정리한 서드파티 전송 공개 요건과 같은 질문이 한 번 더 생김. 다만 **심사 지침이 관측 플랫폼으로의 전송을 직접 지목한다는 서술은 확인하지 못했음.** 이 문단은 지침 해석이 아니라 데이터 경로를 세는 얘기임.

## MCP 쪽으로 트레이스가 이어짐

[MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md)에 `_meta` 로 OpenTelemetry 트레이스 컨텍스트를 전파하는 규약이 문서화됐다고 적어뒀는데, 그 규약을 쓰는 1차 구현이 이것임. MCP 커넥터로 붙인 서버에는 API 가 보내는 요청과 알림마다 **`params._meta` 에 W3C `traceparent`** 가 들어감. `tools/call` 요청이면 그 툴 사용의 `anthropic.mcp_tool_use` 스팬을, 나머지는 `anthropic.messages.create` 스팬을 가리킴. 추적하지 않는 요청(예: sampled 플래그가 꺼진 요청)에는 `traceparent` 를 안 보냄.

클라이언트 쪽은 균일하지 않음. 문서 기준으로 **Python `mcp` 2.0 이상과 C# `ModelContextProtocol`** 은 트레이스 컨텍스트를 `_meta` 로 보내지만, **TypeScript·Go·Java·Ruby·PHP MCP SDK 는 스스로 하지 않음.** 내 MCP 서버의 스팬이 트레이스에 안 붙으면 먼저 볼 자리가 여기임.

## 처음 켤 때의 순서

1. **SDK 쪽만 먼저 켬.** admin 권한이 필요 없고 코드 안에서 끝남. 콘솔 스팬 익스포터로 모양을 먼저 봄
2. **`metadata_only` 를 유지함.** 프롬프트 본문은 필요가 분명해질 때 켬
3. **토큰 합산 코드를 쓸 때 소스를 하나로 고정함.** span kind 로 SDK 와 API 를 가르고, compaction 을 더하는 분기를 같이 넣음
4. **과금 숫자는 Usage API 로 맞춤.** 스팬 집계는 추이와 원인 분석용으로만 씀
5. SDK 버전을 **고정함.** 베타 동안 스팬 이름·속성·옵션·환경변수가 바뀔 수 있다고 문서가 예고함

[Eval Engineering](2026-08-01-eval-engineering-merge-gate.md)의 4단계가 "최고의 테스트는 이미 로그에 있다"였음. `content` 모드의 `gen_ai.input.messages` 와 `gen_ai.output.messages` 가 그 로그를 OpenTelemetry 규약 모양으로 내보내 주는 것이라, 실제 트래픽에서 케이스를 뽑는 경로가 짧아짐. 다만 위의 유실·샘플링 조건이 그대로 걸리므로 **여기서 뽑은 케이스 묶음은 모수가 아니라 표본임.**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [서버측 compaction](2026-09-20-claude-api-compaction.md) | `usage.input_tokens` 가 compaction 을 빼고 나오는 문제가 `gen_ai.usage.*` 에서 같은 모양으로 반복됨 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 히트율 진단에 쓸 1차 계측. 5분·1시간 쓰기가 따로 나옴 |
| [대규모 AI 코딩 비용 관리](2026-08-07-managing-ai-coding-costs.md) | 레버 3 의 가시성 대시보드를 무엇으로 세울지. 과금 기록으로는 쓰면 안 되는 이유 |
| [Eval Engineering](2026-08-01-eval-engineering-merge-gate.md) | 실제 트래픽에서 케이스를 뽑는 경로. 표본 편향이 어디서 들어오는지 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | `_meta` 트레이스 컨텍스트 전파 규약의 1차 구현 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 토큰 수를 비용으로 바꿀 때 모델별 캐시 배수가 갈리는 지점 |

## 짚어야 할 것

- **베타임.** SDK 문서가 "the SDK options and the span schema can change in ways that break your code, your queries, or your dashboards" 라고 적고 버전 고정을 권고함. 스팬 이름이나 속성으로 짠 대시보드와 쿼리가 깨질 수 있는 쪽임
- **일곱 언어 목록에 Swift 가 없음.** 확인한 것은 릴리스 노트의 그 목록까지임. iOS 앱이 Claude API 를 직접 호출하는 구조라면 SDK 추적을 그대로 쓸 수 없고, API 추적이나 자체 계측으로 가야 한다는 뜻임. Swift SDK 자체의 존재 여부는 이 문서로 확인하지 않았음
- **내보내기 자체에 요금이 붙는지는 문서에서 확인할 수 없었음.** 관측 플랫폼 쪽 수집 비용은 당연히 그쪽 과금임
- 문서가 파트너 플랫폼 **12곳**을 나열하고 각각의 연동 문서로 링크하는데, **각 플랫폼에서 실제로 어떻게 보이는지는 확인하지 않았음.** OTLP 를 받는 곳이면 어디든 된다는 서술까지가 확인한 것임
- API 추적은 **admin 역할**이 있어야 등록·변경이 됨. developer 역할은 조회만 되고, 워크스페이스 단위로 범위를 가를 수 있음. 혼자 쓰는 조직이면 걸림돌이 아니지만 **권한이 없으면 SDK 쪽만 쓸 수 있다는 것**이 설계 제약으로 들어감
- **Claude Code·Agent SDK·Cowork 는 이 문서의 범위가 아님.** 문서가 각각 별도 모니터링 문서로 넘기고 있고, 이 정리는 Messages API 와 Managed Agents 쪽임

## 유효기간

**2026-10-15 확인 기준이고 기능은 2026-10-14 베타 시작임.** 스팬 이름과 속성은 베타 동안 움직이므로 이 문서의 속성 이름을 그대로 쿼리에 박기 전에 [스팬 레퍼런스](https://platform.claude.com/docs/en/api/opentelemetry/spans)로 대조할 것. 다시 볼 때 확인할 것 둘. **베타를 벗어났는지**, 그리고 **SDK 지원 언어 목록이 늘었는지.** 후자는 이 저장소 주인에게 Swift 때문에 직접 걸리는 항목임. 콘텐츠 상한(0.75MB)과 목적지 한도(10개) 같은 숫자는 [설정 문서](https://platform.claude.com/docs/en/manage-claude/opentelemetry/api)의 Limits 표에서 다시 받는 편이 나음.
