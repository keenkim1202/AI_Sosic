---
title: MCP 2026-07-28 스펙, 세션을 버리고 stateless로
source:
  - https://blog.modelcontextprotocol.io/posts/2026-07-28/
  - https://blog.modelcontextprotocol.io/posts/mcp-roadmap/
author: Model Context Protocol
published: 2026-07-28
collected: 2026-09-03
tags: [mcp, protocol, agent-tooling, stateless, authorization, prompt-caching, migration, roadmap]
---

출처: [The 2026-07-28 Specification](https://blog.modelcontextprotocol.io/posts/2026-07-28/) · [Key Changes 전문](https://modelcontextprotocol.io/specification/2026-07-28/changelog) · [스펙 본문](https://modelcontextprotocol.io/specification/2026-07-28) (스펙은 2026-08-21 확인) · [The New MCP Roadmap](https://blog.modelcontextprotocol.io/posts/mcp-roadmap/) (2026-08-22, David Soria Parra · Den Delimarsky, 2026-09-03 확인)

## 요약

MCP가 **양방향 상태 유지 프로토콜에서 요청·응답 stateless로** 바뀜. `initialize`/`initialized` 핸드셰이크와 `Mcp-Session-Id` 헤더가 없어지고, 요청마다 프로토콜 버전과 클라이언트 정보를 `_meta`에 실어 보냄. 로드밸런서 뒤의 서버 인스턴스가 **상태를 공유할 필요가 없어짐**. 서버를 직접 운영하지 않더라도 알아둘 게 둘 있음. **Roots·Sampling·Logging이 폐기 예정**으로 들어갔고, 툴 목록에 **캐시 힌트(`ttlMs`, `cacheScope`)와 결정적 정렬 권고**가 생겨서 프롬프트 캐시 적중률에 직접 영향을 줌. 폐기 창은 **최소 12개월**. 한 달 뒤 나온 로드맵(2026-08-22)까지 같이 봄. 다음 개정은 **Tasks의 코어 승격**과 **점진적 툴 발견** 쪽으로 감.

## 왜 지금 보는가

Xcode 26.3이 Claude Agent SDK를 네이티브로 넣고 Xcode 27이 3사 에이전트를 받으면서, iOS 개발자도 **에이전트에 무엇을 어떻게 붙이는가**가 실무로 내려옴. 그 연결 규격이 MCP고, 이번 개정은 **기존 서버가 그대로 안 도는 종류의 변경**임. Tier 1 SDK가 월 **5억 회에 가까운 다운로드**를 찍고 TypeScript·Python SDK가 각각 누적 **10억 다운로드**를 넘긴 시점의 변경이라 파급이 큼.

## stateless로 간 것

**세션 제거.** `Mcp-Session-Id` 헤더와 프로토콜 수준 세션이 Streamable HTTP 전송에서 빠짐. `tools/list`, `resources/list`, `prompts/list`가 더 이상 커넥션마다 다른 결과를 내지 않음. 호출 간 상태가 필요하면 **서버가 발급한 핸들을 평범한 툴 인자로 넘기는 방식**으로 바꿔야 함.

**핸드셰이크 제거.** `initialize` / `notifications/initialized`가 사라지고 모든 요청이 `_meta`에 프로토콜 버전과 클라이언트 능력을 실음 (`io.modelcontextprotocol/protocolVersion`, `io.modelcontextprotocol/clientCapabilities`). 버전이 안 맞으면 `UnsupportedProtocolVersionError`가 돌아옴.

**`server/discover` 신설.** 서버는 이걸 **반드시 구현**해서 지원 프로토콜 버전·능력·신원을 알려야 함. 클라이언트는 사전 버전 선택용으로 먼저 호출하거나 STDIO에서 하위 호환 탐침으로 씀.

**알림 경로 재편.** HTTP GET 엔드포인트와 `resources/subscribe`/`unsubscribe`가 **`subscriptions/listen` 하나로** 대체됨. 클라이언트가 원하는 유형(`toolsListChanged`, `promptsListChanged`, `resourcesListChanged`, `resourceSubscriptions`)만 옵트인함. 진행률·로그 같은 요청 범위 알림은 여전히 해당 요청의 응답 스트림으로 흐름.

**제거된 것들.** `ping`, `logging/setLevel`, `notifications/roots/list_changed`. 로그 레벨은 요청마다 `_meta`의 `io.modelcontextprotocol/logLevel`로 지정하고, 이 필드가 없는 요청에 대해 서버는 `notifications/message`를 **보내면 안 됨**.

**SSE 재개 기능 제거.** `Last-Event-ID` 헤더와 SSE 이벤트 ID가 빠짐. 응답 스트림이 끊기면 진행 중이던 요청이 날아가고, 클라이언트가 **새 요청 ID로 다시 보내야 함.** 장시간 작업을 스트림 재개에 기대고 있었으면 여기서 설계가 바뀜.

## MRTR, 서버가 되묻는 방식

서버가 클라이언트에게 요청을 거는 패턴(`roots/list`, `sampling/createMessage`, `elicitation/create`)이 **Multi Round-Trip Requests**로 대체됨.

흐름은 이럼. 서버가 `resultType: "input_required"`인 결과를 돌려주고 `inputRequests`에 필요한 것을 담음. 클라이언트는 **원래 요청을 재시도**하면서 `inputResponses`에 답을 실음. 스트림을 열어둘 필요가 사라짐.

부수 효과로 **모든 결과에 `resultType` 필드가 필수**가 됐음 (`"complete"` 또는 `"input_required"`). 이전 프로토콜 서버가 이 필드를 생략하면 클라이언트는 `"complete"`로 취급해야 함.

## 캐시와 라우팅, 게이트웨이를 위한 변경

**표준 헤더 필수.** Streamable HTTP POST에 `Mcp-Method`, `Mcp-Name` 헤더가 필요함. 게이트웨이가 **JSON 본문을 파싱하지 않고 라우팅·계측**할 수 있게 하는 목적임.

**목록 결과에 캐시 필드 필수.** `tools/list`, `prompts/list`, `resources/list`, `resources/read`, `resources/templates/list` 결과에 `ttlMs`(신선도 힌트)와 `cacheScope`(`"public"` / `"private"`)가 붙음. 폴링을 줄이려는 것이고 기존 `listChanged` 알림을 대체하는 게 아니라 보완함.

그리고 이 항목이 이 저장소 관점에서 제일 값어치 함.

> 서버는 `tools/list`가 반환하는 툴을 **결정적 순서로** 내보내야 함(SHOULD). 클라이언트 캐싱을 가능하게 하고 **LLM 프롬프트 캐시 적중률을 높이기** 위함임.

툴 정의가 프롬프트 prefix에 들어가므로 **순서가 흔들리면 그 뒤가 전부 캐시 미스**가 됨. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)와 [에이전트 생태계 레포 지형도](../agents/2026-08-10-agent-ecosystem-repos.md)에서 정리한 "사용 가능한 것의 목록을 바꾸면 캐시가 깨진다"가 이제 **프로토콜 권고로 올라온 것**임.

## 인증 강화

- 인가 서버는 응답에 [RFC 9207](https://datatracker.ietf.org/doc/html/rfc9207)의 `iss` 파라미터를 넣어야 하고(SHOULD), 클라이언트는 인가 코드를 교환하기 전에 **기록된 발급자와 대조해 검증해야 함**(MUST)
- 클라이언트 자격증명은 **발급한 인가 서버에 묶임.** 발급자 식별자로 키를 잡아 저장해야 하고, 다른 인가 서버에 재사용하면 안 되며, 인가 서버가 바뀌면 재등록해야 함
- **Dynamic Client Registration(RFC 7591)이 폐기 예정.** 대체는 **Client ID Metadata Documents(CIMD)**. DCR은 CIMD를 지원하지 않는 인가 서버와의 하위 호환용으로만 남음
- DCR을 계속 쓴다면 OpenID Connect 리디렉트 URI 충돌을 피하려고 `application_type`을 명시해야 함

## 확장과 폐기 정책

**Tasks가 코어에서 확장으로 나감.** 실험적 tasks가 공식 확장 `io.modelcontextprotocol/tasks`가 됐고, 블로킹 방식 `tasks/result`가 **폴링 방식 `tasks/get`**으로 바뀜. 클라이언트에서 서버로 입력을 주는 `tasks/update`가 생기고 `tasks/list`는 제거됨. 서버가 요청별 옵트인 없이 태스크 핸들을 먼저 돌려줄 수 있게 됨. `ClientCapabilities`/`ServerCapabilities`에 `extensions` 필드가 생겨 코어 밖 기능을 선언함.

**정식 폐기 정책이 생김.** Active / Deprecated / Removed 세 상태와 **최소 12개월 폐기 창**, 그리고 폐기 기능 레지스트리를 둠. 프로토콜이 기존 구현을 깨지 않고 진화하기 위한 장치임.

폐기 목록과 권장 대체 경로.

| 폐기 | 대체 |
|---|---|
| **Roots** | 디렉터리·파일을 툴 파라미터, 리소스 URI, 서버 설정으로 넘기기 |
| **Sampling** | LLM 제공자 API에 직접 통합 |
| **Logging** | stdio에서는 `stderr`, 아니면 OpenTelemetry |
| **HTTP+SSE 전송** | Streamable HTTP |
| `includeContext`의 `"thisServer"`, `"allServers"` | 생략하거나 `"none"` |
| **OAuth DCR** | Client ID Metadata Documents |

## 그 밖의 변경

- 리소스 없음 에러 코드가 `-32002`에서 **`-32602`(Invalid Params)**로 바뀜. JSON-RPC 정렬 목적
- 에러 코드 할당 정책 신설. `-32000`~`-32019`는 구현 정의(기존 SDK 사용분은 그대로 인정), `-32020`~`-32099`는 스펙 예약. 이번 초안 코드가 재번호됨(`HeaderMismatch` `-32001` → `-32020` 등)
- `inputSchema`/`outputSchema`가 JSON Schema 2020-12 키워드 전체를 허용하고 `structuredContent`가 임의 JSON 값을 허용. `$ref` 해석 요건과 합성 키워드 리소스 상한이 추가됨
- `_meta`에 OpenTelemetry 트레이스 컨텍스트(`traceparent`, `tracestate`, `baggage`) 전파 규약이 문서화됨
- SDK는 TypeScript·Python·Go·C#이 갱신되고 **Rust는 베타**

## 마이그레이션할 때 볼 순서

1. **세션 ID에 의존하는 코드부터.** 스펙 변경 중 실제로 깨지는 게 여기임. 서버 발급 핸들을 툴 인자로 넘기는 구조로 바꿈
2. `initialize` 핸드셰이크 제거에 맞춰 `_meta` 필드 채우기, `server/discover` 구현
3. 스트림 재개에 기대던 장시간 작업을 **Tasks 확장 폴링**으로 옮기기
4. `Mcp-Method`·`Mcp-Name` 헤더와 목록 결과의 `ttlMs`·`cacheScope` 추가
5. 툴 목록 **정렬을 결정적으로** 고정. 캐시 적중률에 직접 영향
6. Roots·Sampling·Logging 사용처를 대체 경로로 옮기기. 12개월 창이 있지만 신규 구현에는 쓰지 말 것
7. DCR을 쓰고 있으면 CIMD 지원 여부 확인

## 다음 개정판이 어디로 가는가

스펙 발표 한 달 뒤인 **2026-08-22에 새 로드맵**이 나왔음. 리드 메인테이너 둘(David Soria Parra, Den Delimarsky)이 썼고 **목표 날짜나 버전은 명시하지 않음.** "앞으로 몇 달"과 "다음 스펙 릴리스와 그 이후"라고만 함. 우선순위 다섯 개인데, 이번 개정에서 stateless로 밀어낸 것들의 **뒷정리와 연장**에 해당함.

| # | 영역 | 구체적으로 무엇 |
|---|---|---|
| 1 | **에이전틱 메시징 프리미티브** | "현대 에이전틱 워크로드는 더 이상 표준 요청·응답 패턴에 맞지 않음". 서버가 먼저 거는 이벤트(웹훅·채널)로 클라이언트 폴링을 없애고, **Tasks 확장(SEP-2663)을 코어로 승격**시키는 것이 목표. `subscriptions/listen`·진행률 알림과 어떻게 합성되는지를 워킹그룹들이 같이 검토 |
| 2 | **HTTP 네이티브 전송 통일** | "원격 MCP 서버는 이제 다른 HTTP 워크로드와 다르지 않음". **로컬 서버의 stdio까지 Streamable HTTP로 덮어서** 배포 형태에 상관없이 전송 하나로 통일 |
| 3 | **에이전트 신원과 엔터프라이즈 보안** | "사용자가 자리에 없는 상태에서 자기 신원을 갖고 도는 클라우드 워크로드"를 전제로 함. **DPoP([RFC 9449](https://datatracker.ietf.org/doc/html/rfc9449)) 확정과 채택**, **Workload Identity Federation**을 통한 에이전트 신원·위임 경로 정의, Enterprise-Managed Authorization 뒤의 ID-JAG 그랜트 활용, 표준 토큰 익스체인지. IETF OAuth·WIMSE 워킹그룹과 같이 감 |
| 4 | **프리미티브 개선** | 둘임. 하나는 **툴 결과 처리 표준화**. "`tools/call` 응답이 같은 출력을 두 가지 이상의 형태로 실을 수 있는데" 클라이언트가 어느 쪽을 써야 하는지 정해진 규약이 없음. 다른 하나는 **점진적 툴 발견**. 툴 표면 전체를 미리 드러내는 대신 "서버가 작은 진입점만 제공하고 대화가 좁혀질수록 카탈로그를 더 드러내는" 방식 |
| 5 | **SDK 개발자 경험** | 에르고노믹스, 스펙 적합성 테스트, 문서 정확도 |

### 여기서 미리 대비할 것

- **4번의 점진적 툴 발견이 이 저장소 관점에서 제일 중요함.** 이번 개정이 "툴 목록을 결정적 순서로 내보내라"로 캐시를 지켰다면, 점진적 발견은 **대화 중간에 툴 목록이 늘어나는 것을 프로토콜이 공식화**하는 방향임. [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 additive tool loading이 정확히 그 문제인데, 거기 결론은 **"추가"일 때만 캐시가 살고 제거·교체·재정렬은 그대로 깬다**였음. 규격이 어느 쪽으로 정해지는지가 캐시 비용을 좌우함
- **1번이 오면 Tasks가 확장이 아니라 코어가 됨.** 이번에 장시간 작업을 Tasks 확장 폴링으로 옮기라고 적어둔 마이그레이션 3번 항목이, 다음 개정에서는 선택이 아니게 될 가능성이 큼
- **2번은 로컬 stdio 서버를 쓰는 쪽에 영향이 큼.** Xcode의 `xcrun mcp-server`나 Claude Code 로컬 플러그인처럼 stdio로 붙는 구성이 전송 계층에서 바뀜

### 프로세스 쪽

우선순위 영역마다 담당 코어 메인테이너와 워킹그룹이 배정돼 있고, **그 영역 안의 SEP는 우선 리뷰를 받고 채택 확률이 높다**고 명시함. 스펙에 넣고 싶은 게 있으면 이 다섯 개 중 하나에 붙이는 게 유리하다는 뜻임. Contributor Ladder도 정식 도입됨.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 툴 정의가 prefix에 들어가는 문제. 결정적 정렬 권고가 그 대응임 |
| [에이전트 생태계 레포 지형도](../agents/2026-08-10-agent-ecosystem-repos.md) | "사용 가능한 것의 목록을 바꾸면 캐시가 깨진다"가 프로토콜 권고로 올라옴 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | IDE 안의 에이전트에 도구를 붙이는 규격이 이것임. Xcode 자신이 MCP 서버가 되는 쪽까지 감 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md) | 인가 강화(발급자 검증, 자격증명 바인딩)가 노리는 위협면 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 로컬 MCP 플러그인 지원과 한도. 클라이언트 쪽 제약 |

## 짚어야 할 것

- **이 문서는 스펙 변경 목록을 정리한 것이고 실제 마이그레이션을 해본 기록이 아님.** 각 SDK가 언제 어떤 형태로 따라오는지는 SDK 릴리스 노트를 봐야 함
- 이전 개정은 **2025-11-25**임. 그 사이 버전을 건너뛰고 올라오면 확인할 변경이 더 많음
- 검색 결과 중 "MCP Apps"를 이번 릴리스 항목 또는 공식 확장으로 소개하는 글이 있는데, **공식 Key Changes 문서에도 2026-08-22 로드맵에도 나오지 않음.** 두 번 확인했고 이 문서에는 넣지 않았음
- **로드맵은 계획이지 확정이 아님.** 목표 날짜도 버전도 명시되지 않았고, 다섯 영역 중 무엇이 다음 개정에 실제로 들어가는지는 SEP 진행 상황을 봐야 함
- Rust SDK는 베타임. 프로덕션 전제로 잡으면 안 됨

## 유효기간

스펙 부분은 **2026-08-21**, 로드맵 부분은 **2026-09-03 확인 기준**임. 스펙 자체는 `2026-07-28`로 고정된 개정판이라 문언은 안 바뀌지만, SDK 지원 범위와 클라이언트(Claude Code, Xcode 등) 적용 시점은 계속 움직임. 다시 볼 때는 [폐기 기능 레지스트리](https://modelcontextprotocol.io/specification/2026-07-28/deprecated)로 12개월 창이 어디까지 왔는지 확인하고, 로드맵 절은 다섯 영역의 SEP가 실제로 스펙에 들어갔는지로 대조하는 편이 나음. **로드맵 절은 다음 개정판이 나오면 통째로 낡음.**
