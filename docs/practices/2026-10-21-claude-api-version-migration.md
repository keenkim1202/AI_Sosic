---
title: API 버전 2026-10-21, 2023-06-01 이후 첫 버전 올림과 SDK 메이저 동반 승격
source: https://platform.claude.com/docs/en/api/migration-2026-10-21
author: Anthropic
published: 2026-10-21
collected: 2026-10-22
tags: [claude, migration, authorization, gateway, compaction, opensource]
---

출처: [Migrating to API version 2026-10-21](https://platform.claude.com/docs/en/api/migration-2026-10-21) · [Versions](https://platform.claude.com/docs/en/api/versioning) · [API 릴리스 노트 2026-10-21](https://platform.claude.com/docs/en/release-notes/api) (전부 2026-10-22 확인)

## 요약

Claude API 에 **`2026-10-21`** 버전이 생겼음. 버전 히스토리상 `2023-01-01`, `2023-06-01` 다음 세 번째이고 **`2023-06-01` 이후 첫 새 버전**임. 와이어 레벨 파괴 변경은 셋뿐인데 그중 하나가 **`temperature`·`top_p`·`top_k` 제거**이고, 기본값을 보내도 모든 모델에서 400 임. 더 넓게 번지는 쪽은 SDK 임. 같은 날 **Python 2.0.0, TypeScript 1.0.0, C# 13.0.0, Go 2.0.0, Java 3.0.0, PHP 1.0.0, Ruby 2.0.0, `ant` CLI 2.0.0** 이 나왔고 이 버전들이 기본으로 `2026-10-21` 을 보냄. SDK 쪽에는 API 버전과 무관한 파괴 변경이 따로 얹혀 있음. **API 키가 `X-Api-Key` 가 아니라 `Authorization: Bearer` 로 나가고**, 한 클라이언트가 자격증명을 **하나만** 들 수 있게 됐고, 라이선스가 **MIT 에서 Apache 2.0** 으로 바뀜.

## 버전을 올리는 주체가 둘임

이 변경의 성격을 가르는 지점이라 먼저 정리함.

| 호출 방식 | `anthropic-version` 을 정하는 쪽 | 올리지 않으면 |
|---|---|---|
| 직접 HTTP | 요청 헤더에 직접 적음 | **아무 일도 안 일어남.** 헤더를 바꾸는 날이 마이그레이션 날임 |
| SDK · `ant` CLI | 설치된 메이저 버전이 정함 | 패키지를 올리는 순간 같이 올라감 |

문서 표현은 "your requests use the API version that was current when your SDK's major version was released" 임. 즉 SDK 사용자는 **버전 올림과 SDK 파괴 변경을 한 번에** 받고, 직접 호출하는 쪽은 둘을 분리해 집행할 수 있음. 공식 SDK 목록에 **Swift 가 없으므로** iOS 앱이나 Swift 서버에서 Claude API 를 직접 때리고 있으면 후자에 해당함. 당장 깨질 것은 없고, 올리는 시점을 스스로 고르면 됨.

새 버전이 도는 곳은 Claude API, Claude Platform on AWS, Microsoft Foundry, Google Cloud 임. **Amazon Bedrock 은 Claude 4.7 이상 모델에만** 제공되고, 그보다 오래된 모델을 Bedrock 에서 부르고 있으면 지금 쓰는 SDK 버전에 머물러야 함. 올리려면 모델을 먼저 4.7 이상으로 옮기는 순서임.

## 와이어 레벨 파괴 변경은 셋뿐

Messages API 와 토큰 카운팅, 그리고 `2026-10-21` 로 만든 Message Batch 의 각 요청에 적용됨.

1. **`assistant` 메시지에 `document` 블록을 직접 넣으면 400.** `user` 메시지로 보내라는 것이고, 다른 블록 안에 중첩된 document(예: Claude 가 돌려준 web fetch 툴 결과)는 그대로 받음
2. **`user` 메시지의 `text` 블록에 비어 있지 않은 `citations` 배열이 있으면 400.** `search_result` 블록 안쪽도 포함됨. `assistant` 메시지의 citations, 그리고 document·search result 에 붙이는 `"citations": {"enabled": true}` 설정은 그대로 유효함
3. **`temperature`·`top_p`·`top_k` 제거.** 세 개 중 하나라도 보내면 **기본값이어도** 400 이고 **모든 모델에서** 그럼

Admin API 와 Claude Enterprise Analytics API 쪽은 이름 바뀐 필드의 구명이 사라지는 변경이고(`roles` → `role_ids`, `actor.email` → `actor.email_address` 등) 이 저장소 독자층과는 거리가 있어 옮기지 않음. 하나만 적어두면 API 키의 `workspace_id` 가 `scope` 로 바뀌면서 **의미도 달라짐**. `2023-06-01` 에서는 Default Workspace 키와 전체 워크스페이스 키가 **둘 다 `null`** 이었는데, 이제 전자는 `scope.workspace_id` 에 실제 ID 가 들어가고 후자는 `scope` 가 `{"type": "organization"}` 이 됨. `null` 을 Default Workspace 로 읽던 코드는 조용히 틀림.

### 샘플링 파라미터 제거는 "이미 그랬다"가 아님

이 저장소는 [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) 에서 `temperature`·`top_p`·`top_k` **비기본값 금지**를 모델별 제약으로 적어뒀음. 이번 변경은 그 제약을 **버전 레벨로 끌어올린 것**이라 성격이 다름. 문서가 직접 구분해 적음.

- Claude 4.7 이상처럼 **이미 기본값만 받던 모델**은 샘플링 동작이 그대로임. 파라미터만 지우면 끝
- **다른 값을 받던 모델**(문서가 든 예는 Claude Sonnet 4.6)에서 일관된 출력을 위해 `temperature` 를 낮춰 쓰고 있었으면 **그 효과를 잃음.** 대체 설정은 없다고 명시함
- 비기본값에 의존하는 요청이 있으면 선택지는 둘. 파라미터를 버리거나, **그 요청만 `2023-06-01` 에 남겨두는 것**

마지막 항목이 실질적인 탈출구임. 버전은 요청 단위 헤더이므로 전부 올리지 않아도 됨. 다만 SDK 를 쓰면 클라이언트 전체가 한 버전이라 이 분리가 쉽지 않음.

## 자격증명이 헤더를 옮겼음

SDK 와 CLI 가 API 키를 `X-Api-Key` 가 아니라 **`Authorization: Bearer <key>`** 로 보냄. **Claude API 자체는 두 헤더를 다 받으므로** 직접 호출하는 프로그램은 바꿀 게 없음. 깨지는 자리는 중간에 뭔가 끼어 있을 때임.

- **`X-Api-Key` 를 읽는 프록시·게이트웨이**: `Authorization` 을 읽도록 고치거나, 클라이언트에서 헤더를 되돌려야 함. 언어마다 처방이 다름. TypeScript 는 `defaultHeaders: { "X-Api-Key": apiKey, Authorization: null }`, Python 은 httpx2 의 `auth` 함수로 요청마다 바꿔치기, C# 은 `ExtraHeaders` 임
- **`ant` CLI 에는 헤더를 추가하는 플래그가 없음.** 프록시를 `X-Api-Key` 로 읽게 두면 CLI 는 맞출 방법이 없고 프록시를 고쳐야 함
- **커스텀 `fetch` 나 인터셉터로 키를 들여다보던 코드**도 같이 걸림. `authorization` 에서 `Bearer ` 뒤를 읽어야 함
- Foundry 클라이언트는 **여전히 `x-api-key`** 로 보내고, Bedrock Mantle 류는 원래 베어러였음. 플랫폼 클라이언트마다 다르니 쓰는 것만 확인하면 됨

여기서 파생된 변경이 **"클라이언트는 자격증명을 하나만 든다"** 임. 키와 토큰이 같은 헤더를 쓰게 되면서 둘을 동시에 들 수 없음. 전에는 키와 토큰을 함께 주면 각자 헤더로 둘 다 나갔고, 이제는 생성 시점에 `ValueError`·`TypeError`·`ArgumentException` 류로 막힘(언어별로 예외 타입이 다름). 아무것도 넘기지 않았을 때 찾는 순서도 정해졌음. **`ANTHROPIC_API_KEY` → `ANTHROPIC_AUTH_TOKEN` → `ANTHROPIC_PROFILE` → workload identity federation 변수 → 디스크의 활성 프로필** 순이고, 앞의 둘이 모두 설정돼 있으면 경고를 한 번 찍음.

⚠️ **게이트웨이 앞에 토큰, API 에 키를 동시에 보내던 구성은 설계를 바꿔야 함.** 문서의 처방은 토큰을 `auth_token` 으로 주고 `X-Api-Key` 를 기본 헤더로 직접 박은 뒤, **게이트웨이가 전달 전에 자기 `Authorization` 헤더를 제거**하는 것임. 이유도 한 줄로 적어둠. "The Claude API expects one credential for each request."

## SDK 에서 사라진 것

API 버전과 무관하게 메이저 버전이 집행하는 변경임. 전수는 원문에 있고, 이 저장소에 영향이 있는 것만 추림.

| 변경 | 범위 | 하게 되는 일 |
|---|---|---|
| **`MessageParam` 이 역할별 유니온으로 쪼개짐** | 전 언어 | 역할을 리터럴로 적는 코드는 대개 무사함. `role` 이 변수인 코드, `MessageParam` 을 상속·구현하던 코드, 역할 안 가리고 `content` 를 갈아끼우던 코드가 깨짐 |
| **툴 러너의 클라이언트측 compaction 제거** | TypeScript, Ruby | `compactionControl` / `compaction_control:` 이 없어짐. 서버측 compaction(`betas: ["compact-2026-01-12"]` + `context_management.edits`)으로 옮기거나 `compactBeforeNextTurn()` 을 직접 부름 |
| **Text Completions 제거** | TypeScript, Go, Java, Ruby, CLI | `client.completions` 와 `HUMAN_PROMPT`·`AI_PROMPT`, `ant completions create` 가 사라짐. 응답 텍스트 경로가 `completion` 에서 `content.0.text` 로 옮겨감 |
| **단일값 enum 이 상수로** | C#, Go, Java, PHP, Ruby, CLI 플래그 하나 | 값이 하나뿐인 필수 필드의 enum 타입이 없어짐. 보내는 JSON 은 같음 |
| **키를 커맨드라인으로 넘기는 플래그 제거** | CLI | `--api-key`·`--auth-token` 이 `flag provided but not defined` 로 멈춤. 이유를 "셸 히스토리와 프로세스 목록에 보인다"로 적음. `--api-key-stdin` 이나 `ant auth login` 으로 감 |
| **Go 1.25 이상 요구** | Go | 런타임 요건 상향 |

클라이언트측 compaction 제거는 이 저장소 문서 하나의 결론을 직접 건드림. [서버측 compaction](./2026-09-20-claude-api-compaction.md) 은 서버측을 1차 전략으로 권하면서도 **"SDK 의 클라이언트측 compaction(`tool_runner`)도 남아 있음"** 을 대안으로 적어뒀음. TypeScript·Ruby 에서는 그 대안이 메이저 버전과 함께 없어졌음. 문서가 제시한 대체 경로가 바로 서버측 compaction 설정이라 방향 자체는 그 문서와 같음. **임계값 최소 50,000 토큰**도 그대로임.

## 라이선스가 Apache 2.0 으로

새 메이저 버전부터 SDK 와 CLI 가 **Apache License 2.0** 이고, 이전 버전은 MIT 로 남음. 코드 변경은 필요 없고 의존성 라이선스를 추적하면 항목을 고치라고만 적음. TypeScript 플랫폼 패키지(`@anthropic-ai/bedrock-sdk` 등)와 C# 보조 패키지(`Anthropic.Bedrock` 등)는 **각자의 1.0.0 릴리스에서** 바뀌고, Java 는 `com.anthropic` 전 아티팩트가 해당됨.

앱 번들에 라이선스 고지를 넣는 쪽이면 이게 실제 작업임. Apache 2.0 은 MIT 와 달리 **NOTICE 파일 취급과 특허 조항**이 있어서 고지 화면 생성 도구 설정을 한 번 확인해야 함. 이 저장소 기준으로는 **올해 Apple 이 `.xcproj` 포맷 모델을 Apache-2.0 으로 낸 것**([.xcproj JSON 프로젝트 포맷](../agents/2026-09-16-xcproj-json-project-format.md))과 같은 방향의 선택이고, 둘 다 MIT 보다 특허 쪽을 명시하는 쪽으로 움직였음.

## 올릴 때 조용히 틀리는 자리

문서가 "에러가 안 나고 틀린다"고 명시한 것들만 모음. 자동화에 맡기기 어려운 쪽임.

- **Go CLI 의 옛 설치 경로.** `go install github.com/anthropics/anthropic-cli/cmd/ant@latest` 는 계속 **최신 v1 을 설치**함. 에러도 경고도 없음. `/v2` 를 붙여야 함
- **TypeScript 플랫폼 패키지를 같이 안 올리면** 패키지 매니저가 `@anthropic-ai/sdk` **옛 복사본을 하나 더** 설치함
- **PHP 의 위치 인자 순서.** `role` 이 첫 파라미터가 되면서 옛 순서는 `UnhandledMatchError` 를 던지는데 **PHPStan 이 경고하지 않음**
- **Go 의 메시지 복사.** `MessageParam` 을 복사하면 같은 메시지를 가리킴. v1 에서 `duplicate.Content` 를 바꾸면 복사본만 바뀌었는데 이제 `duplicate.OfUser.Content` 를 바꾸면 **원본도 바뀜**
- **Ruby 의 `to_h`.** `.new` 로 만든 인스턴스는 `to_h` 에 `:role` 이 없고 `message[:role]` 이 `nil` 임. `message.role` 은 그대로 동작함
- **PHP 의 enum 되읽기.** 넘긴 enum case 가 value 로 보관돼서 case 와 비교하면 **false 가 나오고 아무것도 던지지 않음**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | `temperature`·`top_p`·`top_k` 비기본값 금지가 모델 제약에서 **버전 제약**으로 올라감. 그 문서의 "모델에 따라 갈림" 서술이 `2026-10-21` 에서는 성립하지 않음 |
| [서버측 compaction](./2026-09-20-claude-api-compaction.md) | 그 문서가 대안으로 남겨둔 클라이언트측 compaction(`tool_runner`)이 TypeScript·Ruby 새 메이저에서 제거됨. 대체 경로가 그 문서의 서버측 설정 그대로임 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 캐시 키에 영향을 주던 요청 필드 셋(`temperature`·`top_p`·`top_k`)이 아예 사라짐. 요청 모양이 단순해지는 방향 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 같은 "한 번에 깨고 버전으로 가른다" 패턴. 저쪽은 폐기 레지스트리와 12개월 창, 이쪽은 요청 헤더로 두 버전을 동시에 돌리는 방식임 |
| [.xcproj JSON 프로젝트 포맷](../agents/2026-09-16-xcproj-json-project-format.md) | 올해 두 벤더가 공개 구현체를 Apache-2.0 으로 낸 사례가 둘 생김 |
| [대규모 AI 코딩 비용 관리](./2026-08-07-managing-ai-coding-costs.md) | 게이트웨이를 끼워 라우팅하는 구성이 `Authorization` 전환의 1차 피해 지점임 |

## 짚어야 할 것

- **원문은 마이그레이션 가이드이고 설계 설명이 아님.** `temperature` 를 왜 지웠는지, 왜 베어러로 옮겼는지는 적혀 있지 않음. 키 플래그 제거 이유("셸 히스토리와 프로세스 목록")만 예외적으로 적혀 있음
- **`2023-06-01` 의 지원 종료 시점은 확인할 수 없었음.** 버전 페이지는 "최신 버전을 쓰는 것을 권한다"까지만 적고 폐기 일정이 없음. MCP 쪽처럼 명시된 창이 있는 것과 다름
- **SDK 변경 전수를 옮기지 않았음.** `betas` 가 비베타 Models 메서드에서 제거된 것, 언어별 Optional 반환 전환 같은 항목은 생략했으니 쓰는 언어 탭은 원문에서 직접 읽을 것
- **직접 돌려본 것이 아니라 문서만 읽었음.** 버전별 패키지 번호와 예외 타입은 원문 표기를 옮긴 것이고 설치해 확인하지 않았음
- **Swift 가 공식 SDK 목록에 없다는 것은 릴리스 노트의 목록에서 읽은 것**이고, 별도 공지를 찾아본 결과가 아님
- 같은 날 릴리스 노트에 함께 올라온 user profiles 쪽 변경(`enforcement_status`, `enforcement_actions`)은 이 문서 범위 밖임

## 유효기간

**2026-10-22 확인 기준**임. 버전 자체와 파괴 변경 목록은 `2026-10-21` 에 고정된 기록이라 낡지 않음. 반면 움직이는 것이 둘 있음. **`2023-06-01` 의 폐기 일정이 공지되는지**, 그리고 **Amazon Bedrock 의 구형 모델 제약이 풀리는지**. 다시 볼 때는 [Versions](https://platform.claude.com/docs/en/api/versioning) 의 버전 히스토리에 항목이 더 붙었는지부터 보고, SDK 쪽은 각 저장소의 CHANGELOG 로 대조하는 편이 나음.
