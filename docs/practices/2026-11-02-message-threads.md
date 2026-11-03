---
title: Message threads, 서버가 대화를 5분 들고 있는 대신 내놓는 것들
source: https://platform.claude.com/docs/en/build-with-claude/message-threads
author: Anthropic
published: 2026-11-02
collected: 2026-11-03
tags: [claude, prompt-caching, llm-cost, context-engineering, compaction, thinking, effort]
---

출처: [Message threads, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/message-threads) · [API 릴리스 노트의 2026-11-02 항목](https://platform.claude.com/docs/en/release-notes/api) · [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) (문서 확인 **2026-11-03**)

## 요약

**2026-11-02** 릴리스. Messages API 에 `thread` 필드가 붙어서, 서버가 대화를 짧게 들고 있고 그다음 턴부터는 **새 메시지만** 보냄. 베타 헤더는 `message-threads-2026-08-12`. 얻는 것은 문서가 처음부터 좁혀 적어둠. **요청 크기와 TTFT** 이고, **비용이 아님.** 가격과 레이트 리밋은 그대로이고 **"A thread does not cache your prompt for you"** 라고 못 박음. 대가가 셋임. 첫째, 스레드는 **마지막 요청에서 5분**이면 만료되고 그때 continue 는 **404**, 복구는 전체 대화 재전송임. 이 저장소가 [프롬프트 캐시 문서](../agents/2026-07-22-prompt-caching-in-agents.md)에서 함정으로 적어둔 그 5분이 **두 번째 층에 또 생긴 것**임. 둘째, **저장됐는지 알려주는 응답 필드가 없음.** 다음 continue 가 404 로 떨어질 때 알게 됨. 셋째, **서버측 compaction 과 토큰 카운팅이 같이 막힘.** 그리고 `output_config` 가 고정 필드라서, [Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)이 풀어준 "대화 중간에 effort 바꾸기"가 스레드 안에서는 **400** 임.

## create · continue · replay

동작이 셋이고, 셋째를 처음부터 구현해야 하는 것이 이 기능의 성격임.

| 동작 | 보내는 것 | 돌아오는 것 |
|---|---|---|
| **create** | 전체 대화 + `system` + `tools` + `"thread": {"type": "create"}` | 평범한 message. 그 응답 `id` 가 스레드의 손잡이가 됨 |
| **continue** | 직전 응답 이후의 **새 메시지만** + `{"type": "continue", "previous_message_id": "<id>"}`. `system`·`tools` 는 빼야 함 | 평범한 message. 그 `id` 로 다음 턴을 이음 |
| **replay** | 전체 대화를 **새 create 요청으로** 다시 | continue 가 404 일 때의 유일한 복구 경로 |

문서가 앱의 역할을 한 문장으로 정함.

> "Your application stays the source of truth for the conversation. Keep your own copy of every message, and build the replay path from the start."

즉 **대화를 서버에 맡기는 기능이 아님.** 내 쪽 저장은 그대로 두고, 업로드만 줄이는 것임. SDK 헬퍼(`client.beta.messages.thread(...)`)가 헬퍼 객체 수명 동안 메모리에 사본을 들지만 그 밖에는 아무 데도 안 쓴다고 같이 적어둠. 프로세스보다 오래 살아야 하는 대화면 직접 들고 있어야 함.

## 매 요청에 무엇을 보내는가

이 표를 틀리면 400 임. 특히 가운데 칸이 설계를 묶음.

| 묶음 | 필드 | 규칙 |
|---|---|---|
| **create 전용** | `system`, `tools` | create 에만. continue 에 보내면 **`null` 이어도 400** |
| **고정** | `model`, `tool_choice`, `thinking`, `mcp_servers`(서버별 인증 토큰은 예외), `context_management`, `output_config`, `container` 의 `skills`, 최상위 `cache_control`, 그리고 **`anthropic-beta` 값의 집합** | 매 요청에 **같은 값**으로. 다르면 `thread_fingerprint_mismatch` 400 이고 에러 메시지가 어긋난 필드 이름을 알려줌 |
| **자유** | `max_tokens`, `stop_sequences`, `stream`, `metadata`, `service_tier`, 그리고 이 페이지가 이름을 안 댄 나머지 | 턴마다 바꿔도 됨 |

고정 칸에서 실무에 바로 걸리는 것 둘.

**`output_config` 가 고정임.** 이 저장소가 [Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) 문서에서 "대화 중간에 effort 를 바꿔도 프롬프트 캐시가 깨지지 않음"을 그 모델의 값어치로 적었고, 어려운 단계만 올리고 반복 단계는 내리는 패턴을 거기서 권했음. **스레드를 쓰면 그 패턴이 막힘.** 캐시가 깨지는 문제가 아니라 요청이 **400** 으로 거절되는 문제로 바뀜. effort 를 턴마다 움직이는 하네스라면 스레드와 Fable 의 중간 effort 변경 중 하나를 골라야 하고, 둘을 섞은 설계는 성립하지 않음.

**`anthropic-beta` 값의 집합이 고정임.** 베타 헤더를 상황에 따라 붙이고 떼는 코드가 흔한데, 스레드 안에서는 그것 자체가 fingerprint 불일치임. 이 저장소가 다룬 베타가 이미 여럿이라([compaction](./2026-09-20-claude-api-compaction.md), [인라인 툴 정의](./2026-09-22-inline-tool-definitions.md), [preserved thinking](./2026-10-12-preserved-thinking-bindings.md)) 헤더를 조립하는 자리를 한 곳으로 모아두지 않으면 밟음.

저장 안 되는 것도 명시돼 있음. **MCP 서버 URL, MCP 인증 토큰, `container` 의 `id`.** 그래서 `mcp_servers` 와 `container` 는 쓰는 요청마다 다시 보내야 하고, 인증 토큰만은 요청 사이에 바뀌어도 됨.

## 비용 기능이 아님

제일 오해하기 쉬운 자리임. 서버가 대화를 들고 있으니 재청구가 줄 것 같지만 아님.

> "A thread does not cache your prompt for you. Prompt caching works as it does without a thread: you choose where the cache breakpoints go."

그리고 **"Threads do not change pricing or rate limits."** 줄어드는 것은 업로드하는 바이트, 서버가 매 턴 다시 검증하고 토크나이즈하는 양, 그리고 그에 따른 TTFT 임. 토큰 과금은 캐시를 어디에 걸었느냐로 결정되고 그것은 스레드 없이 할 때와 같음.

캐시를 붙일 때 스레드가 새로 만드는 제약이 둘.

- **`cache_control` breakpoint 가 요청당 최대 4개.** 최상위 `cache_control` 이 **1개를 먹고**, create 요청의 `system`·`tools` 에 건 breakpoint 는 **continue 요청마다 다시 셈.** 저장된 옛 메시지에 걸린 것은 안 셈. 하나 넘치면 `thread_unsupported_request` 400
- **한 번 보낸 breakpoint 는 옮기거나 지울 수 없음.** continue 요청이 옛 메시지를 다시 보낼 수 없기 때문임

최상위 `cache_control` 을 매 요청에 보내는 것이 문서가 권하는 기본형이고, API 가 그것을 그 요청 `messages` 의 **마지막 캐시 가능 블록**에 붙임. 그러면 턴마다 앞 대화를 읽고 새 턴을 씀. 이것도 고정 필드라 값이 매 요청 같아야 함.

## 5분이 두 번째 층에 생김

이 저장소에서 가장 많이 인용된 수치가 캐시 TTL **5분**임. 스레드의 수명도 **마지막 요청에서 5분**이고, 저장된 응답마다 5분이 다시 시작함. 삭제는 마지막 사용에서 **1시간 이내**.

두 5분은 별개 장치인데 **깨지는 상황이 같음.** [프롬프트 캐시 문서](../agents/2026-07-22-prompt-caching-in-agents.md)가 든 예가 그대로 적용됨. 모델에 물어보고, 테스트를 7분 돌리고, 돌아와서 다음 요청을 보내는 흐름. 전에는 결과가 조용한 비용 증가였는데 이제는 **404 + 전체 재전송**이 하나 더 붙음. 그리고 재전송은 create 요청이라 **캐시 쓰기 비용까지 같이** 발생함. 두 손실이 겹치는 자리임.

앱 쪽에서 이게 더 아픈 이유가 있음. 서버 에이전트 루프는 분 단위로 쉬지 않고 돌지만, **사용자가 타이핑하는 채팅 화면은 5분을 아주 쉽게 넘김.** 사람이 화면을 보다가 다른 앱으로 갔다 오는 시간이 그 길이임. 그래서 사용자 대면 대화에 스레드를 붙이면 replay 경로가 예외가 아니라 **평상 경로**가 됨. 그 경우 얻는 것은 사실상 없고 코드만 늘어남. 스레드가 값을 하는 쪽은 문서가 적은 대로 **사람을 기다리지 않는 긴 에이전트 루프**임.

## 저장됐다고 알려주는 응답이 없음

이 저장소가 반복해서 적어온 유형, 즉 에러 없이 조용히 갈리는 자리임.

> "No response, to a create or a continue request, says whether the API stored that turn. You find out on the next continue request."

404 `thread_not_found` 가 나는 이유를 문서가 나열하는데, **응답은 어느 이유인지 말하지 않음.**

- 만료됨
- API 가 응답은 줬지만 저장하지 않음. 예로 **`max_tokens` 에 잘린 툴 호출로 끝난 응답**은 저장되지 않음
- 직전 응답을 **fallback 모델**이 씀
- 직전 응답이 아직 생성 중임
- 다른 조직·워크스페이스에서 온 요청
- **스레드를 만든 것과 다른 사용자·서비스 계정**에서 온 요청. 레거시 워크스페이스 키로 만든 스레드는 그 키에 묶임
- 새 메시지가 저장된 대화가 받을 수 없는 것을 더함. 예로 **이미지가 없던 대화의 첫 이미지**

없는 모델을 부른 요청도 404 라서, 문서가 **replay 전에 `error.details.error_code` 를 먼저 보라**고 적음. 그리고 replay 직후의 continue 가 또 404 면 그 대화는 서버가 저장할 수 없는 것(예로 아주 큰 대화)일 수 있으니 `thread` 없이 보내라고 함. SDK 헬퍼는 `unsupported_request_behavior` 를 `continue_unthreaded` 로 만들어야 그렇게 동작하고, 안 그러면 **매 턴 replay** 를 함. 즉 설정 하나를 안 켜면 "스레드를 쓰는데 매번 전체를 보내는" 상태로 조용히 들어감.

## 못 쓰게 되는 기능들

여기가 도입 판단의 핵심임. 이 저장소가 문서를 하나씩 써 둔 기능들이 목록에 있음.

| 막히는 것 | 어떻게 막히는가 |
|---|---|
| **Token counting** | `thread` 가 들어간 토큰 카운팅 요청은 **400**. 전체 대화를 `system`·`tools` 와 함께 따로 세라고 함 |
| **Compaction** | `compaction` 파라미터나 `compact-2026-09-04` 블록이 있으면 400. **compaction on demand 는 스레드에서 아예 없음** |
| **Context editing** | `clear_thinking_20251015` 의 `keep: "all"` 하나만 예외고 나머지 전부 막힘 |
| **Message Batches** | `thread` 가 들어간 요청 항목은 배치 결과에서 `invalid_request_error` 로 끝남 |
| 조직 설정 | **HIPAA readiness** 를 켠 조직, **Inference hooks** 를 켠 조직은 못 씀 |

대부분 `thread_unsupported_request` 400 이고, `error.details.unsupported` 에 `context_management`·`cache_control`·`fallbacks`·`hipaa`·`inference_hooks` 중 하나가 들어옴. **여러 조건에 걸려도 에러는 하나만 이름을 댐**이라고 적혀 있어서, 하나 고치고 다시 받는 반복이 생길 수 있음. 목록에 값이 더 붙을 수 있다고도 적음.

둘이 특히 아픔.

**토큰 카운팅이 막히는 것.** [요청 크기 상한 문서](./2026-10-28-request-size-and-image-limits.md)에 적어둔 대로 히스토리의 이미지까지 예산에 들어가는데, 그 예산을 재는 공식 수단이 스레드와 같이 안 돎. 전체 대화를 따로 세라는 처방은 곧 **앱이 전체 대화 사본을 들고 있어야 하는 이유가 하나 더 생기는 것**임. 업로드를 줄이려고 붙인 기능이 계측을 위해 전체를 다시 조립하게 만듦.

**compaction 과 배타적인 것.** [서버측 compaction](./2026-09-20-claude-api-compaction.md) 문서의 값어치가 "히스토리를 직접 자르지 않고 재작성 비용을 요약 길이로 묶는 것"이었음. 스레드는 append-only 라 그 자리를 못 줌. 문서가 주는 대안은 **요약을 내 코드로 써서 새 create 요청의 평범한 메시지로 보내는 것**, 즉 compaction 을 다시 손으로 하라는 것임. [SelfCompact 논문 문서](./2026-06-22-self-compacting-agents.md)가 측정한 대로 이기는 쪽은 요약 품질이 아니라 **안 할 자리에서 안 하는 판정**이라, 그 판정을 직접 들고 있어야 하는 구성으로 되돌아감. 긴 대화를 오래 끌고 가는 것이 목적이면 스레드와 compaction 중 **compaction 이 먼저**임.

## thinking 블록이 또 걸림

고정 필드를 바꾸려면 새 스레드를 만들어야 하는데, 그 create 요청에서 한 번 더 걸리는 자리가 있음. **Fable 5.1 · Mythos 5.1 · Opus 5.5 · Sonnet 5.5 · Haiku 5.5** 에서는 `system` 이나 `tools` 를 바꾸면 대화에 이미 들어 있는 `thinking`·`redacted_thinking` 블록이 무효가 되고 create 요청이 400 날 수 있음. 문서의 회피법은 **보내는 대화에서 thinking 블록을 전부 빼는 것**이고, 그러면 모델은 그 추론 없이 답함.

[thinking 블록의 세 가지 바인딩](./2026-10-12-preserved-thinking-bindings.md)이 정리한 구조가 그대로 반복됨. 그 문서가 적은 처방, 즉 세션 시작에 `system` 을 고정하고 바뀌는 지시는 `messages` 에 덧붙이라는 쪽이 스레드에서는 더 강한 요구가 됨. `system` 을 움직이는 설계면 스레드를 새로 만들 때마다 추론을 버리게 됨.

## 거절과 fallback 에서 갈리는 지점

[거절 청구와 fallback](./2026-09-24-refusal-billing-and-fallback.md) 문서와 바로 맞물림.

**거절(`stop_reason: "refusal"`)이면 continue 하지 말아야 함.** API 가 그 거절 턴을 이미 들고 있을 수 있고 continue 로는 못 지움. 거절을 부른 턴을 빼거나 바꿔서 **새 스레드**로 시작해야 함. SDK 헬퍼는 거절 턴을 보관하지 않고 다음 턴을 새 스레드로 시작함.

**fallback 은 스레드와 사이가 나쁨.** 스레드는 모델 하나를 들고 있으므로 **fallback 모델이 쓴 턴은 저장되지 않음.** 응답은 정상으로 오지만 그 `id` 로는 이을 수 없고, 판별은 `usage.iterations` 의 `fallback_message` 항목(스트림이면 마지막 `message_delta`)으로 함. 여기에 sticky routing 이 겹침. 한 번 fallback 이 일어나면 **약 1시간** 동안 `fallbacks` 를 포함한 create 요청이 요청한 모델을 건너뛰고 바로 fallback 으로 갈 수 있고, **그 턴들은 전부 저장되지 않으며** 응답할 때마다 1시간이 다시 시작함. 즉 **매 턴 전체 대화를 보내는 상태가 1시간 단위로 이어질 수 있음.** continue 요청은 이 라우팅을 시작하지도 쓰지도 않음. 그리고 `fallbacks` 는 스레드의 모든 요청에 넣거나 하나도 안 넣는 둘 중 하나여야 하고, 중간에 붙이거나 떼면 fingerprint 400 임.

정리하면 **fallback 을 쓰는 구성에서 스레드로 얻는 이득은 fallback 이 걸리는 동안 사라짐.** 거절·fallback 이 잦은 워크로드면 스레드는 코드만 늘림.

## 스트리밍과 중단

응답 `id` 는 `message_start` 에 오지만 **`message_stop` 을 받은 뒤에만** 이어야 함. 먼저 보낸 continue 는 404 가 될 수 있음. 스트림이 `message_stop` 전에 끊기면 같은 continue 를 다시 보내고, `thread_already_continued` 나 `thread_continue_in_flight` 가 오면 replay 함. 문서가 이유를 적어둠. 연결이 끊긴 시점이 서버가 생성 중이었을 때일 수도 있고 저장을 마친 뒤였을 수도 있어서임. 그래서 **"Replaying is always safe"** 가 기준이 됨. SDK 헬퍼는 **끊긴 스트림의 continue 를 스스로 재시도하지 않음.**

이어서 알아둘 것 둘. 스레드는 **append-only** 이고 **응답 `id` 하나는 한 번만 이을 수 있음.** 그래서 앞 메시지를 고치거나 분기하는 UI("마지막 메시지 수정", 대화 분기)는 전부 새 스레드, 즉 전체 재전송임. `stop_reason: "pause_turn"` 은 예외적으로 그 `id` 에서 **빈 `messages: []`** 로 이어 재개하고, 그동안 새 `user` 메시지를 넣으면 거절됨.

## Swift 가 또 없음

SDK 가 create·continue·replay·거절·fallback 처리를 대신해 주는 thread 헬퍼를 제공하는데, 이 페이지의 예제 언어는 **Python · TypeScript · C# · Go · Java · PHP · Ruby 일곱 개이고 Swift 는 없음.** [Claude API OpenTelemetry](./2026-10-14-claude-api-opentelemetry.md) 문서에서 같은 모양을 이미 적어뒀음. 앱에서 API 를 직접 부르는 구성이면 위에 나온 404 분류, replay, breakpoint 4개 세기, fallback 판별을 **전부 직접 만들어야 함.** 헬퍼가 알아서 하는 것과 사용자가 처리해야 하는 것을 문서가 두 목록으로 갈라 놨는데, Swift 쪽에서는 그 두 목록이 하나로 합쳐짐.

이 기능의 이득이 TTFT 와 요청 크기뿐이라는 것과 같이 놓으면 판단이 쉬워짐. **앱에서 직접 부르는 경로라면 지금 붙일 이유가 약함.** 중간에 서버를 두는 구성이면 그 서버의 언어에 헬퍼가 있을 가능성이 높고, 거기서 쓰는 것이 맞음.

## 이름이 겹치는 것 주의

문서가 노트로 따로 적어둠. **Managed Agents 의 session threads 와 다른 것임.** 이 저장소는 [Managed Agents 다이내믹 워크플로](../agents/2026-10-09-managed-agents-dynamic-workflows.md)에서 "동시 64 스레드", "자식 스레드 25개" 같은 수치를 다뤘는데 그쪽은 **에이전트 실행 단위**이고, 여기의 message thread 는 **Messages API 요청의 대화 상태**임. 한도도 수명도 엔드포인트도 공유하지 않음.

## 데이터 보관

- **ZDR 대상이 아님.** 다만 이 기능을 쓰는 것이 다른 ZDR 적격 기능의 사용에 영향을 주지는 않는다고 적음
- 스레드 내용(대화, 시스템 프롬프트, 툴)은 **메모리에만** 두고 서버 관리 키로 암호화하며 **영속 저장소에 쓰지 않음.** 마지막 사용에서 **1시간 이내** 삭제
- **스레드를 나열·조회·삭제하는 API 가 없음.** 사용자가 대화 삭제를 요청해도 서버측 스레드를 즉시 지우는 수단이 없고 위 1시간에 의존함. 개인정보 삭제 흐름을 약속한 앱이면 이 한 줄을 먼저 봐야 함
- 스레드는 **파일을 참조로** 들고 있음. 스레드를 쓰는 동안 대화가 쓰는 파일을 계속 살려둬야 함

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | 5분 TTL 함정이 스레드 수명으로 한 층 더 생김. 404 뒤 replay 는 캐시 쓰기까지 같이 치름 |
| [서버측 compaction](./2026-09-20-claude-api-compaction.md) | 스레드와 **배타적임.** compaction on demand 가 아예 없고 요약을 직접 만들어 새 스레드로 가야 함 |
| [SelfCompact](./2026-06-22-self-compacting-agents.md) | 요약 판정을 다시 내 코드로 들고 와야 하는 구성이 됨 |
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 대화 중 effort 변경이 그 모델의 값어치였는데 `output_config` 가 고정 필드라 스레드에서는 400 |
| [thinking 블록의 세 가지 바인딩](./2026-10-12-preserved-thinking-bindings.md) | `system`·`tools` 교체가 thinking 블록을 무효로 만드는 같은 구조. 새 스레드를 만들 때마다 걸림 |
| [거절 청구와 fallback](./2026-09-24-refusal-billing-and-fallback.md) | 거절 턴은 continue 금지, fallback 턴은 저장 안 됨. sticky routing 1시간이 이득을 지움 |
| [요청 크기 상한 128MB](./2026-10-28-request-size-and-image-limits.md) | 업로드를 줄이는 기능인데 그 예산을 재는 토큰 카운팅이 같이 막힘 |
| [Claude API OpenTelemetry 내보내기](./2026-10-14-claude-api-opentelemetry.md) | SDK 지원 언어에 Swift 가 없는 같은 모양 |
| [Managed Agents 다이내믹 워크플로](../agents/2026-10-09-managed-agents-dynamic-workflows.md) | **이름만 같고 다른 것.** 그쪽 스레드는 에이전트 실행 단위임 |

## 짚어야 할 것

- **공식 문서와 릴리스 노트만 읽었고 직접 호출해 보지 않았음.** 위의 에러 코드, 5분·1시간, breakpoint 4개 상한은 전부 문서에 적힌 값이고 실측이 아님
- **속도 이득의 크기가 수치로 제시되지 않음.** 문서의 서술은 "requests are smaller, and time to first token stays lower and steadier as the conversation grows" 까지임. 대화 길이별로 TTFT 가 얼마나 줄어드는지는 **확인할 수 없었음.** 도입 판단을 하려면 자기 워크로드에서 재야 함
- **베타 헤더 문자열이 `message-threads-2026-08-12` 인데 릴리스 노트 항목은 2026-11-02 임.** 헤더 날짜가 공개 날짜보다 약 3개월 앞서 있음. 이것이 내부 도입 시점인지 다른 이유인지는 문서에 설명이 없고 **확인할 수 없었음.** 헤더 문자열을 날짜로 읽어 기능 출시 시점을 추정하면 안 된다는 사례로만 둠
- **"스레드를 쓰면 싸진다"는 서술은 문서 어디에도 없음.** 가격·레이트 리밋 불변이 명시돼 있으므로, 비용 절감을 기대해 붙이면 아무 효과가 없음
- 지원 모델 목록에 `claude-opus-4-5-20251101`·`claude-sonnet-4-5-20250929`·`claude-haiku-4-5-20251001` 같은 구형 스냅샷도 들어 있고 페이지가 **"Message threads are in beta on every model"** 로 적음. 다만 Haiku 5.5 처럼 **파라미터 거절 규칙이 다른 모델**이 있으므로 모델 교체는 여전히 따로 확인해야 함
- SDK 의 베타 tool runner 가 **기본으로 스레드를 씀**(끌 수 있음)이라고 적혀 있음. 툴 러너를 쓰고 있었다면 위 제약이 **모르는 사이에 이미 걸려 있을 수 있음.** 특히 compaction·토큰 카운팅을 같이 쓰던 구성이면 확인할 자리임
- 플랫폼은 Claude API · Claude Platform on AWS · Claude Models on Google Cloud(Anthropic managed) · Microsoft Foundry 넷에서 베타임. **게이트웨이를 끼운 경로에서 `thread` 가 어떻게 취급되는지는 이 문서가 다루지 않음**

## 유효기간

**2026-11-03 확인 기준이고 베타임.** 다시 볼 때 확인할 것 넷. **베타를 벗어났는지**, **5분 수명이 늘었는지**, **스레드를 나열·삭제하는 API 가 생겼는지**, 그리고 **compaction 과 토큰 카운팅 금지가 풀렸는지.** 넷째가 풀리면 위의 "compaction 이 먼저" 판단이 뒤집힘. [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서 `thread` 항목이 더 붙었는지부터 보는 편이 빠르고, 문서 페이지의 `featureMetadata.status` 와 `Features that threads don't support` 절을 같이 대조하면 됨.
