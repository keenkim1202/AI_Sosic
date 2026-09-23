---
title: 툴 정의를 메시지 안에 넣기, 툴을 바꾸면서 캐시를 지키는 베타
source: https://platform.claude.com/docs/en/build-with-claude/mid-conversation-system-messages
author: Anthropic
published: 2026-09-22
collected: 2026-09-23
tags: [claude, prompt-caching, tooling, mcp, llm-cost, coding-agent]
---

출처: [Mid-conversation system messages, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/mid-conversation-system-messages) · [Tool use with prompt caching](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-use-with-prompt-caching) · [Tool search tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool) · [Claude API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api) (2026-09-22 항목, 2026-09-23 확인)

## 요약

이 저장소가 여러 문서에서 반복한 결론이 하나 있음. **툴을 추가하든 제거하든 스키마를 바꾸든 프롬프트 캐시가 통째로 날아간다**는 것. Anthropic 문서도 같은 표를 갖고 있음. "Modifying tool definitions → Entire cache (tools, system, messages)". 2026-09-22에 붙은 `inline-tools-2026-09-15` 베타가 그 결론의 **절반을 갈라냄.** 대화 중간의 `role: "system"` 메시지 안에 `tool_addition` 블록을 넣고 **툴 정의 전체를 값으로 실어 보내면**, 최상위 `tools` 배열이 손대지지 않으므로 캐시된 prefix가 그대로 남음. 스키마 교체와 제거도 같은 방식으로 **append 연산이 됨.**

기존의 deferred tool loading과 갈리는 지점이 핵심임. `defer_loading`은 **첫 요청에 정의를 이미 다 보내 놓은 툴**만 나중에 켤 수 있었음. 값으로 정의하는 경로는 **첫 요청 시점에 존재조차 몰랐던 툴**을 붙일 수 있음. MCP 서버를 세션 중간에 붙이는 것까지 됨.

대신 조건이 붙음. **`tools`에 non-deferred 툴이 하나도 없으면 첫 인라인 정의가 렌더링 시작점을 바꿔 캐시 미스가 한 번 남.** 그리고 Sonnet 5는 이 경로 자체가 없음.

## 무엇이 어떻게 갈렸나

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)가 정리한 문장은 이랬음. additive tool loading은 "단어 그대로 **추가**일 때만 통하고, 제거·교체·재정렬은 그대로 캐시를 깬다."

| 하려는 것 | 이전 | `inline-tools-2026-09-15` |
|---|---|---|
| 첫 요청에 정의를 보낸 툴을 나중에 켬 | `defer_loading` + 툴 검색으로 가능 | 그대로 가능 |
| 첫 요청에 **몰랐던** 툴을 붙임 | `tools` 편집. 전체 캐시 무효 | `tool_addition` + `tool_definition`. prefix 유지 |
| 툴 스키마를 교체함 | `tools` 편집. 전체 캐시 무효 | **같은 이름으로 새 정의**를 다시 보냄 |
| 툴을 회수함 | `tools` 편집. 전체 캐시 무효 | `tool_removal` 블록 |
| 툴 순서를 바꿈 | 전체 캐시 무효 | **문서가 다루지 않음.** `tools`가 고정이라 순서를 바꿀 자리가 없음 |

즉 "추가일 때만"이 "추가·교체·제거일 때"로 넓어진 것이고, 재정렬은 여전히 답이 없음. 애초에 재정렬을 원할 이유가 캐시 때문이라면 이제 그 동기 자체가 사라짐.

## 블록 세 가지

`tool_addition`은 참조와 값 둘 다 받음.

```json
{
  "role": "system",
  "content": [
    {
      "type": "tool_addition",
      "tool": {
        "type": "tool_definition",
        "definition": {
          "name": "db_query",
          "description": "Run a read-only SQL query against the analytics database.",
          "input_schema": {
            "type": "object",
            "properties": { "sql": { "type": "string" } },
            "required": ["sql"]
          }
        }
      }
    }
  ]
}
```

`definition`은 **표준 `tools` 엔트리 그대로**임. 커스텀 툴과 Anthropic 정의 툴을 다 받고 `cache_control`과 `defer_loading`도 실을 수 있음.

`tool_removal`은 값이 아니라 참조만 받음. 이름으로 지목하거나(`tool_reference`), MCP 툴 하나(`mcp_tool_reference`, `server_name`과 `name`), MCP 툴셋 전체(`mcp_toolset_reference`, `server_name`)를 지목함. 회수한 툴은 나중에 `tool_addition`으로 다시 꺼낼 수 있음.

스키마를 갈 때는 **같은 이름으로 다른 정의**를 보내면 새 정의가 앞의 것을 덮음. 다만 **다른 타입의 툴에 같은 이름을 재사용하면 400**이고 `tool_name_conflict`가 옴. 서버 툴을 새 버전으로 올리는 것은 허용됨.

## 캐시가 유지되는 조건

문서가 규칙으로 못 박아 둔 것들임. 여기가 실무에서 틀리기 쉬운 자리.

| 규칙 | 왜 |
|---|---|
| `tools`에 **non-deferred 툴을 최소 하나** 남길 것 | non-deferred가 0인 대화에서 첫 인라인 정의가 렌더링된 프롬프트의 **시작점을 바꿈.** 전체 캐시 미스가 한 번 남 |
| 아는 툴은 처음부터 `tools`에 선언할 것 | 모델에게 아직 안 보이고 싶으면 `defer_loading: true`를 붙임. 값으로 정의하는 것은 **첫 요청에 모르거나 나중에 바뀌는 것만** |
| `cache_control`은 블록이나 정의 중 **한쪽에만** | 양쪽에 두면 안 됨. 그리고 요청당 중단점 한도(4개)를 같이 먹음 |
| deferred 정의에는 `cache_control` 금지 | 인라인이든 `tools`든 같음. 400이 남 |
| 앞 메시지는 **바이트 단위로 동일**해야 함 | 새로 처리되는 것은 덧붙인 system 메시지 하나뿐임 |

앞의 두 줄이 서로 맞물림. "필요할 때만 로드"를 극단으로 밀어서 `tools`를 전부 deferred로 채워 두면, 인라인 정의를 처음 쓰는 순간 아끼려던 것보다 큰 미스를 한 번 맞음. 툴 검색을 쓸 때 검색 툴 자신을 non-deferred로 남기라는 기존 규칙(전부 deferred면 400)이 여기서 같은 역할을 함.

## MCP 툴셋을 대화 중간에 붙이기

`mcp-client-2026-09-15`를 같이 보내면 `definition`에 `mcp_toolset`을 넣을 수 있음.

```json
{
  "type": "tool_addition",
  "tool": {
    "type": "tool_definition",
    "definition": { "type": "mcp_toolset", "mcp_server_name": "calendar" }
  }
}
```

**서버 URL과 토큰은 `mcp_servers`에 그대로 두고 이 블록에는 절대 넣지 않음.** 그리고 응답의 맨 앞에 `mcp_tool_listing` 블록이 붙어 서버가 내려준 툴 목록을 기록함. 이 블록을 다음 요청에 **그대로 되돌려 보내면 목록이 고정**되고 서버를 다시 조회하지 않음.

이게 [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md)에 적어둔 것과 정확히 만남. 그쪽 스펙이 "툴 목록을 결정적 순서로 내보내라"고 권고한 이유가 프롬프트 캐시 적중률이었음. 지금은 **클라이언트 쪽에서 받은 목록을 핀으로 박는 수단**이 생긴 것이라, 서버가 권고를 안 지켜도 대화 하나 안에서는 목록이 흔들리지 않음. 서버를 믿는 대신 응답을 보관하는 방향임.

응답 형태가 바뀌는 것도 같이 봐야 함. 문서가 **`content[0]`을 읽지 말고 블록 타입으로 고르라**고 명시함. 첫 블록이 텍스트라고 가정한 파서는 여기서 깨짐.

## 한계와 에러

400과 `error.details.error_code`가 `available_tools_limit_exceeded`로 오는 조건 넷임.

| 조건 | 한도 |
|---|---|
| 어느 메시지 이후든 사용 가능한 deferred 툴 수 | **10,000개** 초과 |
| 첫 user 메시지 이후 정의돼 사용 가능해진 툴 수 | **10,000개** 초과 |
| 첫 user 메시지 이후 보낸 툴 정의의 총 바이트 | **4MB**(4,194,304바이트) 초과 |
| 렌더링된 툴 텍스트 크기 | **4MB** 초과 |

배치 위치 제약도 그대로 걸림. 대화 중간 system 메시지는 배열 첫 자리에 올 수 없고, user 턴(또는 서버 툴 결과로 끝나는 assistant 턴) 바로 뒤여야 하며, `tool_use`와 그 `tool_result` 사이에 끼일 수 없음. 그리고 **일시 정지된 assistant 턴 직후에는 `tool_addition`·`tool_removal`이 거부됨.** 턴을 먼저 재개하고 다음 system 메시지에서 보내야 함. `text` 블록은 그 자리에서도 받음.

## 왜 이게 비용 얘기로만 안 끝나는가

[Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)에서 앞 턴을 편집하면 400이 남. Opus 5.5도 같은 검사를 갖고 있고, 두 모델 모두 **2026-08-31 00:00 UTC 이후 만든 계정에는 기본 강제**임. 그리고 그 검사 대상에 **`tools`가 들어감.** thinking 블록이 만들어진 뒤 `system`·`tools`·앞 메시지 중 하나라도 바뀌면 걸림.

그래서 툴을 바꾸는 행위의 대가가 모델에 따라 둘로 갈림.

| | 구형 계정·구형 모델 | Fable 5.1 / Opus 5.5 + 신규 계정 |
|---|---|---|
| `tools`를 편집함 | 캐시 전체 무효. **비용 문제** | 재생한 thinking 블록이 **400**. 요청 자체가 실패 |

Opus 5.5 문서가 이 대목에 직접 처방을 적어 둠.

> "Keep the conversation append-only so the question never arises: change instructions or tools with mid-conversation system messages rather than edits."

즉 이 베타는 최적화 옵션이 아니라 **그 모델들에서 툴을 바꾸는 유일하게 문서화된 방법**에 가까움. [서버측 compaction](./2026-09-20-claude-api-compaction.md)이 히스토리 축소 쪽에서 같은 위치를 차지한 것과 짝이 맞음. 한쪽은 줄이는 일, 다른 한쪽은 툴을 바꾸는 일을 append 연산으로 옮겨 놓은 것임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | "툴 로드아웃이 캐시를 깨는 이유"와 "히트율 낮을 때 확인할 8가지" 5번. 추가만 안전하다는 서술이 여기서 넓어짐 |
| [서버측 compaction](./2026-09-20-claude-api-compaction.md) | 같은 append-only 요구에서 나온 짝. 히스토리 대 툴 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 히스토리 편집 400. 그 검사 대상에 `tools`가 포함됨 |
| [에이전트 생태계 레포 지형도](../agents/2026-08-10-agent-ecosystem-repos.md) | 스킬 설치·제거와 MCP 툴 정의 변경이 무효화로 적힌 표. 그 표의 전제가 베타에서 갈림 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 툴 목록 결정적 정렬 권고. `mcp_tool_listing`이 클라이언트 쪽 대응 수단 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 플러그인이 MCP 서버를 담는 구조. IDE가 세션 중간에 서버를 붙일 때 걸리는 자리 |

## 짚어야 할 것

- **베타 헤더 조합을 확정하지 못했음.** 문서의 Model Support 절은 대화 중간 툴 변경에 `mid-conversation-tool-changes-2026-07-01`이 필요하다고 적고, 값으로 정의하는 경로에는 `inline-tools-2026-09-15`가 필요하다고 적음. 릴리스 노트는 후자가 "참조로 추가·제거하는 것까지 같은 헤더가 덮는다"고 함. **둘을 함께 보내야 하는지, 후자만으로 되는지는 두 출처로 갈리지 않게 좁힐 수 없었음.** 붙이기 전에 작은 요청으로 직접 확인할 것
- **플랫폼 범위도 갈림.** 대화 중간 툴 변경은 Claude API·Amazon Bedrock·Google Cloud에서 된다고 적혀 있는데, 릴리스 노트는 인라인 정의를 **"in beta on the Claude API"**로만 적음. Bedrock·Google Cloud에서 값 정의가 되는지는 **확인하지 못했음**
- **Sonnet 5에는 대화 중간 system 메시지 자체가 없음.** 최상위 `system` 필드를 쓰라고 문서가 적음. Sonnet 5로 도는 하네스에서는 이 절 전체가 적용되지 않음
- **캐시가 유지된다는 것은 문서의 서술이고, 이 문서는 직접 측정하지 않았음.** `usage`의 `cache_read_input_tokens`로 확인해야 함. 특히 non-deferred 툴이 하나도 없는 경로에서 미스가 한 번만 나는지는 재보지 않았음
- 토큰 절감 수치는 이 페이지들에 없음. 툴 검색 문서에 있는 "다중 MCP 서버 구성에서 정의만 약 55k 토큰, 툴 검색이 85% 이상 줄임"은 **툴 검색 쪽 수치이고 인라인 정의의 수치가 아님.** 섞어 인용하면 안 됨
- 회수한 툴을 모델이 이미 부른 뒤에 회수했을 때 진행 중인 `tool_use`가 어떻게 처리되는지는 **문서에서 찾지 못했음**

## 유효기간

**2026-09-23 확인 기준**이고 베타 헤더가 `inline-tools-2026-09-15`, MCP 툴셋 쪽이 `mcp-client-2026-09-15`임. 릴리스 노트 항목이 2026-09-22이라 아직 움직이는 중임. 다시 볼 때 확인할 것 둘. **베타를 벗어났는지**, 그리고 **위 "짚어야 할 것"의 첫 두 항목(헤더 조합과 플랫폼 범위)이 문서에서 정리됐는지.** [Tool use with prompt caching](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-use-with-prompt-caching)의 무효화 표에 인라인 정의 줄이 추가되면 그때가 정리된 시점으로 봐도 됨.
