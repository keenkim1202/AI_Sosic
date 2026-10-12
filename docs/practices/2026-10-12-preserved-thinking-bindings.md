---
title: thinking 블록의 세 가지 바인딩, 모델·대화·계정
source: https://platform.claude.com/docs/en/build-with-claude/preserved-thinking
author: Anthropic
collected: 2026-10-12
tags: [claude, thinking, prompt-caching, compaction, coding-agent, llm-cost]
---

출처: [Preserved thinking, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking) · [API release notes](https://platform.claude.com/docs/en/release-notes/api) (2026-09-14 · 2026-09-28 · 2026-10-07 항목, 2026-10-12 확인)

## 요약

[Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)에서 "앞 턴을 편집하면 400"으로 처음 걸린 그 검사가, 지금은 **Fable 5.1·Opus 5.5·Sonnet 5.5·Haiku 5.5 넷에 모두 적용**됨. 그 문서가 재확인 과제로 남긴 "전체 계정으로 확대됐는지"는 **아직 아님**. 계정 생성일 기준(2026-08-31 00:00 UTC)이 그대로임. 대신 이번에 새로 생긴 것이 둘이고 둘 다 운영에 직접 걸림.

첫째, 바인딩이 하나가 아니라 **셋**임. 모델 바인딩(이 모델이 읽을 수 있는 블록인가), prefix 바인딩(앞선 `system`·`tools`·`messages`가 그대로인가), 그리고 **계정 바인딩**임. 셋의 실패 처리가 서로 달라서, 하나로 묶어 생각하면 틀림.

둘째, 계정 바인딩은 **에러를 내지 않음.** Sonnet 5.5와 Haiku 5.5가 만든 thinking 블록을 다른 계정이 보내면 API가 그 블록을 **버리고 요청은 성공**함. 모델은 그 추론 없이 답함. 베타 헤더를 안 붙이면 **버렸다는 사실조차 응답에 안 나옴.** 이 저장소가 [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md)에서 정리한 유형, 즉 지표는 통과하는데 품질만 조용히 내려가는 실패가 정확히 이 모양임.

그리고 옛 계정에서 히스토리 편집을 **강제 없이 탐지하는 경로**가 2026-09-14에 생김. 헤더만 붙이고 `prefix_mismatch_behavior`를 비워 두면, 검사에 걸린 블록이 모델에 그대로 가면서 `input_transformations`에 `thinking_mismatch_allowed`로 보고됨.

## 바인딩 셋을 갈라 보기

| 바인딩 | 무엇을 검사 | 실패하면 | 과금 | 보고 |
|---|---|---|---|---|
| 모델 | 현재 모델이 그 블록을 읽을 수 있는가 | **항상 버림.** `prefix_mismatch_behavior`와 무관 | 안 됨 | `thinking_dropped` / `model_binding_mismatch` |
| prefix (대화) | 블록보다 앞선 `system`·`tools`·`messages`가 그대로인가 | 설정에 따라 **400** 또는 **그 블록부터 뒤 전부 버림** | 버린 건 안 됨 | `thinking_dropped` / `prefix_binding_mismatch`, 또는 강제 off면 `thinking_mismatch_allowed` |
| 계정 | 블록을 만든 계정(또는 연결된 계정)인가 | **버리고 요청은 성공** | 문서에 명시 없음 | `thinking_dropped` / `organization_binding_mismatch` |

세 보고가 전부 `input_transformations` 배열로 오고, **이 배열은 `thinking-binding-controls-2026-08-01` 헤더를 붙여야 생김.** 헤더가 없으면 모델 바인딩과 계정 바인딩은 무음으로 처리됨. 스트리밍에서는 `message_start`의 `message` 객체에 실려 옴.

## 계정 바인딩, 이번에 새로 걸린 자리

적용 범위가 좁음. **Sonnet 5.5와 Haiku 5.5가 만든 블록만** 해당하고, 그 이전 모델 블록은 영향이 없음.

문제는 범위가 아니라 실패 방식임. 400이면 코드가 멈추니까 알게 되는데, 이건 **요청이 성공함.** 릴리스 노트 문언이 이럼.

> "When another account sends one of these blocks, the API drops the block before the model sees it, and the request succeeds."

실제로 이걸 밟게 되는 구성이 셋 떠오름.

- 개발 계정에서 캡처한 트랜스크립트를 **스테이징이나 CI의 다른 계정 키로 재생**하는 회귀 테스트
- 조직이 둘로 갈린 상태에서 **세션 저장소를 공유**하는 경우
- 서로 다른 계정 키로 요청을 흘리는 **게이트웨이·프록시**를 중간에 둔 경우

세 번째는 문서가 따로 한 절을 둬서 경고함. 라이브러리나 프록시가 중간에 있으면 그쪽이 한 재작성도 편집으로 세고, 그 사용자는 그걸 볼 수도 고칠 수도 없음. 그래서 **caller의 `anthropic-beta` 값과 `thinking.block_binding`을 손대지 말고 통과시키고, `input_transformations`를 그대로 돌려주라**는 것이 문서의 권고임. 모르는 키를 거부하는 옵션 스키마가 사용자에게서 `"drop_block"` 선택권을 빼앗는다는 지적도 같이 있음.

## 어느 모델이 어느 블록을 읽는가

2026-10-12 기준 문서 서술임. `+` 로 적은 것은 플랫폼에 따라 갈리는 항목임.

| 모델 | 읽는 블록 | 못 읽는 블록 |
|---|---|---|
| Fable 5.1 / Mythos 5.1 | 서로의 블록, 이전 Claude 모델, Opus 5 · **(Claude API에서) Opus 5.5** | 문서가 따로 적지 않음 |
| Opus 5.5 | Opus 5, 이전 Opus·Sonnet·Haiku, **(Claude API·Google Cloud에서) Sonnet 5.5 · Haiku 5.5** | Fable · Mythos |
| Sonnet 5.5 | Sonnet 5, Opus 4.8, Haiku 4.5와 그 이전, **(Claude API·Google Cloud에서) Haiku 5.5** | Opus 5, Opus 5.5, Fable · Mythos |
| Haiku 5.5 | Sonnet 5, Opus 4.8, Haiku 4.5와 그 이전 | Opus 5, Opus 5.5, Sonnet 5.5, Fable · Mythos |

여기서 나오는 실무 결론은 [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)에 적은 것과 같되 방향이 하나 더 붙음. **라우터나 폴백이 모델을 바꾸면 블록이 버려지지만, API가 내 `messages` 배열을 고치지는 않음.** 그래서 히스토리를 읽을 수 있는 모델로 되돌리면 블록도 같이 돌아옴. 문서의 처방은 "전체 히스토리를 thinking 블록까지 그대로 계속 보내고, 현재 모델이 못 읽는 것은 API가 버리게 두라"임. 클라이언트가 모델별로 블록을 걸러내는 코드를 짜면 그게 곧 prefix 편집이 돼서 더 비싼 실패를 만듦.

## drop_block 은 공짜가 아님

prefix 검사에 걸렸을 때의 선택지가 둘임.

| 값 | 동작 |
|---|---|
| `"error"` (강제되는 곳의 기본값) | 400 `invalid_request_error`. 첫 실패 블록을 지목함 |
| `"drop_block"` | 실패 블록과 **그 뒤 모든 thinking 블록**을 버림. 요청은 성공하고, 버린 블록은 과금 안 됨. 단 **프롬프트 캐시는 편집 지점에서 다시 시작**함 |

`"drop_block"`이 응급 처치로 읽히기 쉬운데, 문서가 **자체 측정치**를 붙여 그게 아니라고 적음. 멀티턴 코딩 벤치마크에서 출력 토큰이 이렇게 늘었음.

| 드롭 빈도 | 출력 토큰 증가 |
|---|---|
| 세션당 한 번 | **2.5%**(low effort) ~ **20.1%**(max) |
| 첫 턴 이후 매 턴 | **4.6%**(low) ~ **67.1%**(max) |

버린 추론을 모델이 다시 만들기 때문임. effort가 높을수록 손해가 커지는 구조라, [Prompting Claude Opus 5](../prompting/2026-08-10-prompting-claude-opus-5.md)의 결론(필요 없는 자기검증 지시를 빼서 토큰을 줄인다)과 같은 자리에서 반대 방향으로 비용이 샘. 그리고 **캐시가 편집 지점에서 끊기므로** [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md)의 계산이 한 번 더 걸림. 즉 `"drop_block"`은 켜 두고 잊는 플래그가 아니라 **편집 지점을 찾아 없앨 때까지의 임시 경로**임.

한 가지 단서. Sonnet 5.5와 Haiku 5.5에서는 `block_binding`이 **`thinking: {"type": "adaptive"}`에서만 동작**함. Sonnet 5.5의 `between_tools`나 Haiku 5.5의 `disabled`를 쓰고 있으면 이 선택지 자체가 없고, 히스토리를 append-only로 유지하거나 편집 지점 이후의 thinking 블록을 직접 걷어내는 것만 남음.

## 내가 쓴 요약으로 compaction 하면 깨지는 자리

[서버측 compaction](./2026-09-20-claude-api-compaction.md)에 "서버측 compaction은 이 검사를 통과하는 경로"라고 적어뒀음. 그 반대쪽이 이번에 명확해짐. **요약을 클라이언트가 직접 쓰면서 최근 턴을 원문으로 남기는 방식(keep-tail)은 검사에 걸림.** 남겨둔 assistant 턴의 thinking 블록은 "원래 앞 턴들"을 전제로 만들어진 것인데 그 앞이 요약으로 바뀌었기 때문임.

- 요약을 **API가 쓰게 하면**(on-demand compaction) 남긴 턴의 thinking이 유효하게 남을 수 있음. 조건은 compaction 쪽 문서에 따로 있음
- 요약을 **내가 쓰면** 남긴 턴에서 `thinking`·`redacted_thinking`을 직접 떼거나 `"drop_block"`을 켜야 함
- **백그라운드 compaction을 직접 구현한 경우**가 제일 고약함. 요약을 만드는 동안 쌓인 thinking이 **요약을 끼워 넣는 순간 한꺼번에 무효**가 됨
- 요약 하나만 남기고 이전 턴을 전부 지우는 단순 compaction은 애초에 실패할 thinking이 안 남아서 문서가 권하는 쪽임

중간 턴만 골라 빼는 것은 어떤 compaction 방식으로도 못 피함. 지시를 바꾸려고 턴을 뺐던 거라면 대화 중간 시스템 메시지를 덧붙이는 쪽으로, 오래된 tool result나 thinking만 걷어내려는 거라면 서버측 context editing으로 가라는 것이 문서의 대안임.

## 세션을 저장해 두고 다음날 재개하기

FAQ에 있는 항목인데 앱에서 바로 걸리는 자리라 옮겨둠. 재개는 평범한 후속 요청이고, `system`·`tools`·앞선 `messages`의 **값**이 마지막으로 보낸 것과 같아야 함. JSON 포맷이나 키 순서는 무관함.

그래서 저장해야 하는 것은 입력이 아니라 **보낸 것과 받은 것 그대로**임. 렌더링된 시스템 프롬프트, 툴 정의, 반환된 그대로의 assistant 턴. 날짜나 갱신된 지시 파일, 버전이 올라간 툴 정의처럼 **그사이 바뀔 수 있는 입력으로 다시 렌더링하면 안 됨.** 새로 생긴 것은 전부 뒤에 붙이는 메시지로 감.

`AGENTS.md`나 `CLAUDE.md`처럼 세션 중에 바뀌는 지시 파일도 같은 처방임. 세션 시작에 한 번 읽고 상단 `system`은 고정하고, 파일이 바뀌면 그 시점의 `messages`에 새 버전을 **덧붙임**. 운영자가 주는 지시면 대화 중간 시스템 메시지로, 신뢰할 수 없는 파일 텍스트면 시스템 권한을 주지 말고 다음 `user` 턴에 넣으라고 갈라 적음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | prefix 검사와 400의 최초 기록. 그 문서가 남긴 재확인 과제 중 "전체 계정 확대"는 아직 아니고, 적용 **모델**이 넷으로 늘어난 것이 이번 차이 |
| [서버측 compaction](./2026-09-20-claude-api-compaction.md) | 통과하는 경로(서버측)와 걸리는 경로(클라이언트 keep-tail)의 경계 |
| [Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) | `"drop_block"`이 편집 지점에서 캐시를 다시 시작시킴. 가지치기의 역설이 여기서 한 번 더 걸림 |
| [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md) | 계정 바인딩은 요청이 성공하면서 품질만 내려가는 실패임. 루프가 성공으로 보고하는 실패 유형 |
| [Claude Cowork](./2026-08-10-claude-cowork.md) | "이어붙일까 truncate할까"를 캐시 배수로 따진 표. truncate 쪽에 400 또는 추론 재생성 비용이 추가로 붙음 |

## 짚어야 할 것

- **문서 페이지만 읽었고 직접 호출해 확인하지 않았음.** 에러 문언과 `input_transformations` 모양은 문서에 실린 예시를 옮긴 것임
- **계정 바인딩에서 버려진 블록의 과금 여부를 확인할 수 없었음.** prefix·모델 바인딩으로 버려진 블록은 과금되지 않는다고 명시돼 있는데, 계정 쪽은 같은 문장이 없음
- **"연결된 계정(linked account)"의 정의를 확인할 수 없었음.** 조직 간 어떤 관계가 연결로 인정되는지는 이 페이지에 없음
- 출력 토큰 증가 수치(2.5~67.1%)는 **Anthropic 자체 측정이고 멀티턴 코딩 벤치마크 기준**임. 문서도 자기 워크로드에서 재보라고 적어둠. 이 저장소의 iOS 작업 패턴에 그대로 옮기면 안 됨
- 모델별 읽기 가능 조합은 **플랫폼(Claude API, Google Cloud, Bedrock 등)에 따라 갈림.** 위 표의 `+` 항목이 그 부분이고, 게이트웨이를 끼우면 다시 확인해야 함
- 적용 범위가 모델마다 다른 상태라 **"Claude는 이렇다"로 일반화하면 틀림.** Mythos 5.1과 그 이전 모델은 prefix 검사를 아예 돌리지 않음

## 유효기간

**2026-10-12 확인 기준**임. 이 페이지는 모델이 추가될 때마다 표가 늘어나는 종류라 모델 릴리스마다 낡음. 다시 볼 때 확인할 것 셋. **계정 생성일 기준이 전체 계정으로 확대됐는지**(문서가 예고해 둔 변경임), **`thinking-binding-controls-2026-08-01`이 베타를 벗어나 `input_transformations`가 기본으로 오는지**, 그리고 **계정 바인딩이 Sonnet 5.5·Haiku 5.5 밖으로 넓어졌는지**. 첫째는 [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md)이 이미 같은 항목을 재확인 과제로 적어둔 것이라, 둘을 같이 갱신하는 편이 나음.
