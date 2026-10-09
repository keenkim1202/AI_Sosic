---
title: 에이전트 웹 egress 통제 3층, allowed_hosts 가 서버측 툴까지 덮게 된 변경
source: https://platform.claude.com/docs/en/managed-agents/environments
author: Anthropic
published: 2026-10-07
collected: 2026-10-09
tags: [agent-security, sandboxing, permissions, claude, guardrail, mcp]
---

출처: [Cloud environment setup](https://platform.claude.com/docs/en/managed-agents/environments) · [Restrict web search and web fetch domains](https://platform.claude.com/docs/en/managed-agents/tools-web-restrictions) · [Web fetch tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool) · [Claude Platform 릴리스 노트](https://platform.claude.com/docs/en/release-notes/overview) (2026-10-07 항목, 2026-10-09 확인)

## 요약

Claude Managed Agents 에 **2026-10-07** 로 egress 통제 두 건이 들어갔음. 첫째, `limited` 네트워킹 환경의 **`allowed_hosts` 가 샌드박스뿐 아니라 `web_search`·`web_fetch` 에도 적용**됨. 이 두 툴은 샌드박스가 아니라 Anthropic 서버에서 도는데, 그동안 환경의 호스트 목록이 닿지 않던 경로였음. 둘째, `web_fetch` 가 **세션에 이미 등장한 URL 만** 가져옴. 모델 자기 출력, 에이전트 시스템 프롬프트, 첨부 문서, `bash`·`read`·MCP 툴의 출력에 있는 URL 은 출처로 안 쳐주고 `url_not_in_prior_context` 를 돌려줌. 릴리스 노트가 이유를 **"This reduces the risk of data exfiltration"** 으로 적었음.

실무에서 걸리는 건 셋임. **환경을 만들 때 `networking` 을 빼면 `unrestricted` 가 됨**(콘솔 폼은 반대로 Limited 로 시작함). **두 목록의 매칭 규칙이 달라서 세션 생성이 400 으로 떨어지는 조합**이 생김. 그리고 **허용은 호스트 단위지 동작 단위가 아니라서**, 한 번 열어준 호스트로는 `git push` 든 패키지 publish 든 나감.

## 세 층이 각각 어디서 걸리는가

이 제품의 웹 접근 통제는 층이 셋이고, 막히는 지점과 에러가 다 다름.

| 층 | 설정 위치 | 막히면 |
|---|---|---|
| 샌드박스 네트워크 | 환경의 `networking.allowed_hosts` | 샌드박스 요청은 **403**, 호스트 이름이 찍힘. 서버측 웹 툴은 툴 결과 에러(`url_not_allowed`) |
| 툴별 도메인 목록 | 에이전트 툴셋의 `web_search`·`web_fetch` 항목에 `allowed_domains` 또는 `blocked_domains` | `web_fetch` 는 `url_not_allowed`, `web_search` 는 결과에서 조용히 빠짐 |
| URL 출처 검사 | 설정 없음, 항상 켜짐 | `web_fetch` 가 `url_not_in_prior_context` |

세 번째 층에 스위치가 없다는 게 포인트임. 끌 수 있는 통제가 아니라 `web_fetch` 의 동작 자체임.

## 1층, allowed_hosts 가 서버측 툴까지 덮음

이번 변경의 본체임. 문서가 이렇게 적음.

> "With `limited` networking, `allowed_hosts` also applies to the `web_search` and `web_fetch` tools, which run on Anthropic's servers."

따라오는 규칙 넷이 중요함.

- **`allowed_hosts` 가 비어 있으면 두 툴은 아무것도 못 돌려줌.** `limited` 로 두고 다른 필드를 안 쓰면 허용 호스트가 0 개인 상태임. 세션에 붙인 파일·메모리 스토어·GitHub 레포는 그래도 살아 있음
- **`allow_package_managers` 와 `allow_mcp_servers` 는 이 두 툴에 호스트를 하나도 더해주지 않음.** 패키지 레지스트리를 열어둔 환경이라도 `web_fetch` 로 그 호스트를 가져오려면 `allowed_hosts` 에 따로 적어야 함
- **반대 방향으로는 샌다.** 웹 툴 쓰라고 `allowed_hosts` 에 넣은 호스트는 **샌드박스에도 열림**. 읽기 전용으로 열어주는 수단이 아님
- **`unrestricted` 와 self-hosted 샌드박스는 두 툴을 제한하지 않음.** 즉 이 통제는 `limited` 를 고른 환경에만 생김

여기서 나오는 실무 결론이 하나 있음. 웹 조사만 하는 에이전트라면 `unrestricted` 가 필요 없음. 문서도 "An agent that only uses the `web_search` and `web_fetch` tools does not need `unrestricted` networking if you can list the sites it needs" 로 같은 말을 함. 다만 그 목록이 샌드박스 허용 목록과 **같은 목록**이라는 대가가 붙음.

## 2층, 두 목록의 매칭 규칙이 달라서 400 이 남

이게 이 문서에서 제일 밟기 쉬운 함정임. 툴의 `allowed_domains` 와 환경의 `allowed_hosts` 는 **서브도메인을 다루는 방식이 반대**임.

| 목록 | 서브도메인 | 와일드카드 |
|---|---|---|
| 툴의 `allowed_domains`·`blocked_domains` | 적은 호스트와 **그 모든 서브도메인**을 덮음. `example.com` 이 `docs.example.com` 을 덮음 | `*.example.com` 은 **거부됨**. 형식 오류로 400 |
| 환경의 `allowed_hosts` | **정확히 그 호스트 하나**만. `example.com` 은 `www.example.com` 을 안 덮음 | `*.example.com` 을 써야 서브도메인이 덮임. 단 `example.com` 자신은 안 덮음 |

그래서 이런 검사가 걸림.

> "Creating a session fails with a 400 error when an enabled web tool's `allowed_domains` has an entry that is not within `allowed_hosts`. So does a session update that adds such an entry."

문서가 든 예가 그대로 함정의 모양임. 툴 항목 `docs.example.com` 은 `allowed_hosts` 가 `["example.com"]` 이면 **안 들어감**. `["docs.example.com"]` 이나 `["*.example.com"]` 이어야 들어감. 넓은 쪽을 환경에, 좁은 쪽을 툴에 적는 평소 감각으로 쓰면 바로 걸림.

고치는 방법은 둘뿐임. 호스트를 `allowed_hosts` 에 추가하거나, 그 항목을 `allowed_domains` 에서 빼는 것.

**검사 시점도 한 번이 아님.** 에이전트 생성·수정, `tools` 를 넘기는 세션 생성·수정에서 검증하고, 세션이 **툴을 처음 초기화할 때 다시** 봄. 그때 유효하지 않으면 `session.error` 를 내고 재시도 없이 `idle` 로 돌아감. 즉 배포 시점에 통과한 설정이 세션 시작 시점에 깨질 수 있음.

목록 자체의 규칙도 좁음. 한 항목에 `allowed_domains` 와 `blocked_domains` 를 같이 쓸 수 없고, 목록은 **1~64 개**이며 각 도메인은 1~255 자임. 빈 목록은 모호하다고 거부되니 제한을 없애려면 필드를 빼거나 `null` 을 보냄. `web_fetch` 쪽 도메인에는 **경로를 붙일 수 없음**(`web_search` 쪽은 `example.com/blog` 처럼 붙을 수 있음). IP 주소, 단일 라벨 이름(`intranet`), 베어 TLD(`com`, `co.uk`), 비ASCII 도메인은 전부 거부되고 국제화 도메인은 Punycode(`xn--`) 로 적어야 함.

멀티에이전트 세션이면 한 겹 더 쌓임. 스레드에 걸리는 모든 목록이 동시에 적용돼서 `allowed_domains` 는 **교집합**, `blocked_domains` 는 **합집합**으로 동작함. roster 에이전트는 좁힐 수만 있고 넓힐 수 없음. 그리고 교집합이 비면 **툴은 그대로 붙어 있는데 모든 호출이 실패**함. 조용히 사라지지 않고 매번 `url_not_allowed` 가 나는 쪽이라 디버깅은 그나마 나음.

## 3층, web_fetch 는 대화에 나온 URL 만 가져옴

Managed Agents 쪽 릴리스 노트가 적은 허용 출처와 비허용 출처임.

| 허용되는 출처 | 허용되지 않는 출처 |
|---|---|
| `user.message` 이벤트의 **텍스트** | 모델 자기 출력 |
| `web_search` 결과 | 에이전트 시스템 프롬프트 |
| 앞서 `web_fetch` 가 돌려준 페이지 | 첨부 문서 |
| | `bash`·`read`·MCP 툴의 출력 |

막힌 URL 은 `url_not_in_prior_context` 로 돌아오고, 열어주는 방법은 **`user.message` 의 텍스트로 보내는 것**임.

Messages API 쪽 `web_fetch` 문서가 같은 규칙을 더 길게 적어뒀고, 두 군데가 갈리는 지점이 하나 있음. Messages API 에서는 **클라이언트측 툴 결과가 허용 출처**이고, 그것이 모델이 만든 텍스트를 그대로 되비춘 경우에도 허용됨. 반면 code execution·MCP connector·tool search 같은 **서버측 툴 결과는 허용 출처가 아님**. Managed Agents 의 `bash`·`read` 는 샌드박스에서 도는 쪽이라 이 구분과 방향이 맞음. 다만 Managed Agents 의 **custom tool**(애플리케이션이 직접 실행해 결과를 돌려주는 것) 결과가 허용 출처인지는 두 문서 어디에도 안 적혀 있어 **확인할 수 없었음.**

자격증명 검사도 같이 붙어 있음. API 키나 비밀번호처럼 보이는 것이 들어간 URL 은 그 자격증명이 **시스템 프롬프트나 사용자 메시지 텍스트에 등장한 경우가 아니면** 거부되고 `url_not_allowed` 가 남. 툴 결과에만 나온 자격증명은 안 쳐줌.

**허용 출처 목록을 거꾸로 읽어야 하는 항목이 하나 있음.** "앞서 `web_fetch` 가 돌려준 페이지"가 허용 출처라는 것은, 한 번 가져온 페이지에 적혀 있던 URL 은 그다음부터 가져올 수 있다는 뜻임. 프롬프트 인젝션이 들어 있는 페이지가 자기 콜백 URL 을 본문에 같이 적어두는 경우가 이 규칙 안쪽에 들어옴. 원문이 이 조합을 직접 적지는 않았고, Messages API 문서가 **"there is still residual risk that you should carefully consider"** 로 잔여 위험을 인정하는 선에서 멈춤. 그쪽이 권하는 추가 수단은 툴을 아예 끄는 것, `max_uses` 로 횟수를 묶는 것, `allowed_domains` 로 좁히는 것 셋임.

## 기본값이 unrestricted 라는 것

문서가 노트로 따로 박아둔 문장임.

> "Set `networking` explicitly in API requests; a create request that omits it gets `unrestricted`. The Claude Console's form for creating an environment starts with **Limited** selected and nothing else allowed."

**API 와 콘솔의 기본값이 반대임.** 콘솔에서 눌러보고 감을 잡은 뒤 같은 설정을 코드로 옮기는 흔한 순서가 여기서 깨짐. `networking` 을 빼먹은 환경 생성 요청은 에러가 아니라 가장 넓은 모드로 통과함.

`unrestricted` 에서 무엇이 가능한지는 문서가 7개 항목으로 적어뒀는데, 이 저장소 관점에서 세 줄이 중요함. **"Anything in the sandbox can leave it"**(파일, 툴 출력, 샌드박스에 넣은 자격증명 전부), **"Nothing pauses these requests by default"**(툴셋 기본 권한 정책이 `always_allow`), **"Model behavior is not a security control"**(사이트가 막으면 다른 방법으로 재시도하는 것까지 포함). 셋을 합치면 `unrestricted` + 기본 권한 정책은 **승인 지점이 없는 자율 egress** 임.

## 호스트 단위 허용이 남기는 구멍

`limited` 로 좁혀도 끝이 아님. 문서가 경고 박스로 적어둔 것임.

> "Network access is granted per host, not per operation. The sandbox can send any request to an allowed host, including uploads such as `git push` and package publishing, with any credential the command supplies."

여기서 `allow_package_managers` 가 무거워짐. 이걸 켜면 `allowed_hosts` 와 별개로 Anthropic 이 관리하는 호스트 묶음이 열리는데, 그 안에 `github.com`, `raw.githubusercontent.com`, `gitlab.com`, `bitbucket.org` 같은 **코드 호스팅**과 npm·PyPI·crates·Maven·RubyGems·Packagist·apt·Docker 레지스트리가 들어 있음. 패키지를 받으려고 켠 플래그가 **푸시 가능한 호스트**를 같이 여는 셈임. 목록은 Anthropic 이 바꿀 수 있다고 문서가 밝혀뒀음.

문서가 주는 완화책은 둘임. `bash` 툴의 권한 정책을 `always_ask` 나 `auto` 로 두는 것, 그리고 환경이 `packages` 를 안 쓴다면 `allow_package_managers` 를 `false` 로 두고 필요한 호스트만 `allowed_hosts` 에 적는 것. 후자가 더 확실함. 참고로 환경이 `packages` 를 지정했는데 이 플래그가 `false` 면 **요청이 400 으로 거부됨**. 레지스트리 호스트를 `allowed_hosts` 에 전부 적어뒀더라도 그럼.

**3층과 이 구멍은 서로를 못 메움.** URL 출처 검사는 모델이 egress 용 URL 을 즉석에서 만들어 `web_fetch` 로 부르는 경로를 막는 것이고, 호스트 단위 허용의 문제는 **샌드박스 안에서 `bash` 가 보내는 요청**임. 후자는 URL 출처 검사를 거치지 않음. 두 통제가 다른 채널을 보고 있어서, 한쪽을 조였다고 다른 쪽이 조여지지 않음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](./2026-08-07-openai-hugging-face-agent-incident.md) | 채널 하나를 막으면 다른 채널로 옮겨간 사례. 호스트 단위 허용이 남기는 구멍이 정확히 그 표면임 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 승인 정확도를 올리는 것보다 위험한 경로를 승인 대상에서 빼는 게 싸다는 결론. `limited` + 좁은 `allowed_hosts` 가 그 구현 |
| [Anthropic 자사 사고 보고서](./2026-08-31-anthropic-alignment-security.md) | 같은 랩이 자기 평가 환경에서 인가 없는 외부 접근을 겪은 기록. 이번 통제가 어느 쪽 경험에서 나왔는지 읽는 데 쓸 만함 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | `allow_mcp_servers` 가 다루는 대상. **`limited` 네트워킹에서 이 값이 `false` 일 때만** MCP 서버 호스트가 `allowed_hosts` 에 없으면 세션 생성이 400 으로 떨어짐. `true` 로 두면 목록에 없어도 됨. `allowed_hosts` 에 넣어 푸는 쪽을 택하면 그 호스트가 `web_search`·`web_fetch` 에도 열린다 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 세션 중간에 툴의 도메인 목록을 갈아끼울 수 있다는 것. 사용 가능한 것의 목록이 바뀌는 자리라 캐시 prefix 와 같이 봐야 함 |

## 짚어야 할 것

- **문서만 읽었고 직접 돌려보지 않았음.** 400 이 나는 조합과 `url_not_in_prior_context` 의 실제 에러 본문은 재현해 확인하지 않았음
- **Managed Agents 는 `managed-agents-2026-04-01` 베타 헤더가 붙은 베타임.** 필드 이름과 기본값이 바뀔 수 있음
- **릴리스 노트에 적힌 두 변경의 적용 범위를 더 좁히지 못했음.** 기존 환경과 에이전트에 소급 적용되는지, 즉 변경 전에 만든 환경이 다음 세션부터 새 규칙으로 도는지는 어느 페이지에도 안 적혀 있어 **확인할 수 없었음**
- **custom tool 결과가 `web_fetch` 의 허용 출처인지 확인할 수 없었음.** 위에 적은 대로임
- `allow_package_managers` 로 열리는 호스트 목록은 **Anthropic 이 관리하며 바뀔 수 있다고 문서가 밝혀둠.** 이 문서에 옮긴 목록을 고정된 사실로 쓰면 안 됨
- **조직 단위 설정은 항목마다 적용 시점이 다름.** 콘솔 Capabilities 의 **Allow web search 는 2026-10-06 부터** Managed Agents 세션에도 적용됨. 즉 이 문서의 확인 시점(2026-10-09)에 이미 적용돼 있었음. **웹 검색 도메인 제한은 2026-10-14**, **웹 가져오기 끄기와 도메인 제한은 2026-10-20** 부터이고 둘 다 이 문서 이후임. 검색 결과는 조직 제한과 에이전트 `web_search` 의 `allowed_domains`·`blocked_domains` 를 **둘 다 통과한 것만** 옴
- outcome-driven 세션의 grader 는 이 설정과 무관하게 `web_search`·`web_fetch` **없이** 돎. 채점자에게 웹을 주는 설계를 세웠다면 여기서 안 됨

## 유효기간

**2026-10-09 확인 기준이고, 변경 자체는 2026-10-07 항목임.** 베타 제품의 설정 표면이라 필드 이름과 검증 규칙이 움직임. 다시 볼 때는 [릴리스 노트](https://platform.claude.com/docs/en/release-notes/overview)에서 Managed Agents 항목이 더 붙었는지부터 보고, 확인할 것 둘. **환경 생성 시 `networking` 의 기본값이 `limited` 로 바뀌었는지**, 그리고 **`allow_package_managers` 로 열리는 호스트 목록이 바뀌었는지**. 전자는 문서가 노트로 경고할 만큼 밟기 쉬운 자리라 바뀔 가능성이 있고, 후자는 문서 자신이 변경 가능하다고 적어둔 부분임.
