---
title: 에이전트 메모리의 구성 요소, 쓰는 순간이 아니라 끝난 뒤에 고친다
source: https://www.anthropic.com/engineering/components-of-agent-memory
author: Lance Martin (Anthropic Engineering)
published: 2026-11-06
collected: 2026-11-16
tags: [agent-memory, context-engineering, claude, agent-eval, llm-cost, anthropic]
---

출처: [The components of agent memory, Anthropic Engineering](https://www.anthropic.com/engineering/components-of-agent-memory) (2026-11-06) · 같은 기법의 제품판인 [Dreams](https://platform.claude.com/docs/en/managed-agents/dreams) 문서와 [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)를 2026-11-16 에 같이 확인함

## 요약

에이전트 메모리를 **세션 중에 쓰는 노트**와 **세션이 끝난 뒤 돌리는 정리 패스(dreaming)** 둘로 갈라 놓고, 각각이 무엇을 할 수 있는지 측정한 글임. 이 글이 쓰는 메모리의 정의는 좁음. **세션 사이에 남는, 경험으로 얻은 지식**만 메모리이고 가중치 갱신·인컨텍스트 러닝·compaction 은 전부 제외함. 건질 결론은 셋임. 첫째, **메모리를 붙이면 좋아지는 게 아니라 메모리를 쓸 줄 아는 모델에서만 좋아짐.** CL-Bench 데이터베이스 과제에서 Opus 5 는 **0.564 에서 0.828** 로 올랐는데 Sonnet 4.6 은 **0.330 에서 0.364** 로 거의 안 움직임. 둘째, **메모리를 무엇에 쓰는지는 알려주고 어떻게 쓰는지는 지시하지 말 것.** "실패만 기록해라" 같은 고정 규칙을 걸자 Opus 5 가 **-0.234**, Opus 4.7 이 **-0.150** 으로 이득을 반납했음. 셋째, 노트는 틀린 채로 굳고 그걸 고치는 자리는 **쓰는 순간이 아니라 트랜스크립트와 대조하는 사후 패스**임. 그리고 이 사후 패스는 글에만 있는 게 아니라 **Dreams 라는 이름으로 Claude API 에 research preview 로 올라와 있음**(베타 헤더 `dreaming-2026-04-21`).

## 메모리의 정의를 좁게 잡은 것이 이 글의 출발점

원문의 정의는 "knowledge that Claude gains from experience" 이고 세션 사이에 **나중에 읽을 컨텍스트로 저장**되는 것만 포함함. 명시적으로 빼는 것이 셋임.

| 빼는 것 | 이유 |
|---|---|
| 가중치 갱신 | 경험이 파라미터로 가는 경로는 이 글의 범위 밖 |
| 인컨텍스트 러닝 | 한 세션 안에서 끝나므로 세션 사이에 남지 않음 |
| compaction | 컨텍스트를 줄이는 것이지 지식을 남기는 것이 아님 |

이 저장소에 compaction 문서가 둘 있는데([SelfCompact](./2026-06-22-self-compacting-agents.md), [서버측 compaction](./2026-09-20-claude-api-compaction.md)) **그 둘과 이 글은 같은 줄에 놓이는 문서가 아님.** 저쪽은 "지금 이 대화를 어떻게 줄일까"이고 이쪽은 "이 대화가 끝난 뒤 무엇을 남길까"임. 섞어 읽으면 둘 다 흐려짐.

핵심 비유는 **메모리 쓰기가 예측이라는 것**임. 원문 표현으로 "By writing down a rule, an agent predicts that the pattern it's describing will be relevant again." 규칙을 적는다는 건 그 패턴이 또 온다고 베팅하는 것이고, 그래서 메모리 품질 문제는 저장 용량 문제가 아니라 **예측 정확도 문제**가 됨.

## CL-Bench, 이득이 모델 세대마다 완전히 다름

CL-Bench 는 원문 설명으로 "a benchmark that measures an agent's ability to use memory in order to tackle long-horizon tasks" 임. 데이터베이스 과제의 mean reward 를 stateless 와 메모리 있는 경우로 나눠 잰 값이 이거임.

| 모델 | stateless | 메모리 | 차이 |
|---|---|---|---|
| Opus 5.5 | 0.655 | **0.838** | +0.183 |
| Sonnet 5.5 | 0.644 | **0.833** | +0.189 |
| Opus 5 | 0.564 | **0.828** | +0.264 |
| Opus 4.7 | 0.383 | **0.700** | +0.317 |
| Sonnet 5 | 0.306 | **0.610** | +0.304 |
| Sonnet 4.6 | 0.330 | **0.364** | +0.034 |

읽는 법 둘.

- **Sonnet 4.6 줄이 제일 중요함.** 메모리를 붙였는데 거의 안 올랐음. 즉 메모리는 모델에 붙이는 부품이 아니라 **모델이 쓸 줄 알아야 작동하는 기능**임. 작은 모델이나 구세대를 쓰는 설계에서 "메모리를 붙이면 장기 과제가 풀린다"를 가정으로 깔면 안 됨
- **원문이 세대별로 무엇이 달라졌는지도 적어 둠.** Opus 4.7 은 노트를 합치면서 사실에 확인·미확인 표시를 달았고, Sonnet 5 는 답을 확인됨으로 표시하고 정정을 적었고, **Opus 5 가 처음으로 루프를 닫았다**("Claude Opus 5 was the first of these models to close the loop"). 규칙과 근거와 경고를 같이 기록하는 수준에 도달한 것이 그 지점임

이 저장소 독자에게 직접 걸리는 지점은 온디바이스 쪽임. [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md)이 다루는 급의 모델은 이 표의 아래쪽보다 더 아래에 있음. **기기 안의 작은 모델에 세션 간 메모리를 붙이는 설계는 이 표가 말리는 쪽**이고, 메모리를 쓸지 말지보다 **누가 메모리를 쓰는지**를 먼저 정해야 함.

## 살리언스, 많이 쓴 쪽이 짐

원문에서 가장 그림이 선명한 사례가 포켓몬 레드임.

| 모델 | 메모리 | 배지 |
|---|---|---|
| Sonnet 3.7 | **413 파일 / 225 폴더** | 1 |
| Claude Fable 5 | **2 폴더에 9 파일** + 최상위 `global.md` | 4 |
| Opus 5.5 | 목표를 추적하는 작은 노트 묶음 | 게임 클리어 |

"Sonnet 3.7 assumed everything might matter and wrote to memory indiscriminately." 반대로 Fable 5 쪽은 "memory files about strategy, plans, and tested rules exist only if Claude works them out" 임. 즉 **적은 쪽 파일들은 복사해 올 수 없는 것들**임. 좌표나 아이템 목록은 게임에서 다시 읽으면 되지만 전략과 검증된 규칙은 모델이 직접 만들어야만 생김. 그림 설명 기준으로 Fable 5 쪽은 **10 파일 43,406자**이고, 중간에 **17 스텝 동안 6 파일을 만들고 9 파일을 지운** 구간이 있음. 지우는 동작이 같이 돌고 있다는 게 요점임.

여기서 바로 나오는 운영 규칙 하나. **메모리 품질을 "쓴 양"으로 재면 안 됨.** 파일 수, 바이트 수, 쓰기 호출 수는 전부 Sonnet 3.7 쪽이 압도적으로 높고 결과는 반대임. [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md)이 경고한 얕은 지표의 교과서적 사례고, 에이전트에게 "메모리를 잘 남겨라"를 지표로 주면 정확히 413 파일이 나옴.

## 조종, 무엇에 쓰는지만 말하라

원문 문장이 그대로 결론임. "We've found that it helps to tell Claude what memory is for, but not how to use it."

**고정 규칙을 걸면 강한 모델일수록 손해임.** "record only failures" 규칙을 걸었을 때 메모리 이득이 줄어든 폭.

| 모델 | 이득 변화 |
|---|---|
| Opus 5 | **-0.234** |
| Opus 4.7 | **-0.150** |
| Sonnet 4.6 | -0.020 |

**그런데 약한 모델에서는 방향이 뒤집힘.** Sonnet 4.6 에 "메모리를 읽고 네 가설을 검증해라"를 넣자 **0.364 에서 0.641** 로 거의 두 배가 됐음. 원문이 그 차이의 원인으로 드는 것이 자발성임. Sonnet 5 는 답하기 전에 메모리를 **10번 중 9번** 스스로 읽었고 Sonnet 4.6 은 절반에 못 미쳤음. **안 하던 동작을 지시로 채워 넣는 것은 이득이고, 이미 하던 동작을 규칙으로 좁히는 것은 손해**라는 구도임.

프롬프트 설계로 옮기면 이렇게 됨.

- 메모리 지침에 **목적**을 적음: 이 저장소는 무엇에 쓰이고 다음 세션의 누가 읽는지
- **금지·할당 규칙은 최소로**: "실패만", "매 턴 하나씩", "500자 이내" 같은 형식 규칙이 강한 모델의 이득을 깎음
- 모델이 약할 때만 **절차를 지시**: 읽어라, 검증해라, 모순되면 고쳐라

## Dreaming, 틀린 노트를 고치는 자리는 사후임

원문이 지적하는 메모리의 구조적 약점은 하나임. "a note that's wrong from the start or that drifts as the environment changes can be read, and trusted, by Claude in a later session." **쓰는 순간의 에이전트는 자기 노트가 틀렸는지 알 방법이 없음.** 그 세션의 컨텍스트가 전부니까.

Dreaming 은 **로그를 다시 읽어서 실수를 고치거나 새 노트와 표현을 만드는 오프라인 과정**임. 원문 사례가 깔끔함. 포켓몬 센터 위치를 틀리게 적어 둔 노트 때문에 Claude 가 Mt. Moon 에서 일방통행 절벽으로 떨어지는 루프에 갇혔는데, dreaming 이 그 노트를 **원본 트랜스크립트와 대조해서** 고쳤음. 고친 쪽은 Cerulean City 까지 갔고 안 고친 쪽은 산을 못 넘었음.

이 구조가 중요한 이유는 **검증의 근거가 모델의 기억이 아니라 트랜스크립트라는 점**임. "Dreaming gives Claude a way to test its notes against the original transcripts." 노트가 요약이고 트랜스크립트가 원본이므로, 사후 패스는 **요약을 원본에 다시 맞추는 작업**이 됨. 이 저장소가 [SelfCompact](./2026-06-22-self-compacting-agents.md)에서 본 문제(요약 시점이 잘못되면 계산 중간 결과가 날아감)와 방향이 반대임. 저쪽은 **버리기 전에 판단**하는 것이고 이쪽은 **버린 뒤에 원본으로 복구**하는 것임.

## 위키, dreaming 이 표현 자체를 만드는 경우

가장 큰 실험이 이거임. Stainless 의 코퍼스 **약 350,000 파일, 약 420M 토큰**을 대상으로 Opus 5 에이전트들을 **128 세션** 돌려 위키를 만들게 함. 위키의 정의는 "a collection of pages written and maintained by Claude that gathers what Claude knows about one subject" 임. 25개 질문으로 채점한 결과.

| 조건 | graded criteria 충족률 |
|---|---|
| 위키만 (2 세션 분량) | 13% |
| 위키만 (128 세션 분량) | **48%** |
| 검색만 | 53% |
| 검색 + 위키 | **61%** (위키가 커지면서 54% 에서 올라감) |

덧붙는 수치 둘. 검색과 위키를 같이 쓰면 **문항당 약 20,000 토큰**을 아꼈고, Harvey LAB 합병 문항(여러 문서를 가로지르고 판단이 필요한 항목)에서는 **위키 87% 대 검색 단독 73%** 로 벌어짐.

여기서 판단을 둘 붙여야 함.

- **위키는 검색을 대체하지 못함.** 위키만 쓴 최고가 48% 이고 검색 단독이 53% 임. 원문도 둘을 같이 쓰는 쪽을 결론으로 둠. "문서를 다 읽혀서 지식 베이스를 만들면 검색이 필요 없어진다"는 쪽으로 읽으면 안 됨
- **이득 8점의 가격이 적혀 있음.** 그림 설명이 위키 생성을 **2M 에서 128M dreaming 토큰**까지 키운 축으로 그림. 즉 **한 자릿수 점수를 사는 데 1억 토큰대**가 들어감. 문항당 20,000 토큰을 아낀다는 수치와 같이 보면 **질문을 몇 천 번은 던져야 본전**이라는 계산이 나옴. 이 계산은 원문에 없고 위 두 수치로 이 문서가 붙인 것임

## 목표를 좁히면 손실이 사라짐

계약 조항 과제가 이 글에서 제일 실무적인 사례임. 여러 에이전트가 **공유 파일에 같이 쓰게** 했더니 브리프에 **57개 사실 중 41개(72%)** 만 들어갔음. 원인이 모델 능력이 아니었음. 원문 표현으로 "The contract terms were the most important part of the wiki, but the instructions didn't say that."

구조를 바꾼 뒤 결과가 **100%** 가 됐음. 바꾼 것 둘.

- **계약 하나당 페이지 하나**로 쪼개서 동시 쓰기 경합을 없앰
- **조항을 그대로 인용**하게 해서 요약 과정에서 날아가는 것을 막음

앱에 옮기면 그대로 쓸 수 있는 규칙임. 여러 에이전트가 한 메모리에 붙는 구조라면 **쓰기 단위를 엔터티 단위로 쪼개는 것**이 모델을 올리는 것보다 먼저임.

## 주입 메모리의 상한, Claude Code 가 숫자를 박아 둠

노트를 매 세션 컨텍스트로 승격시키는 경우의 비용을 원문이 짚음. "Injected memories use up valuable tokens, and an incorrect note can be carried into every session." 길수록 지시 준수율도 떨어짐.

그래서 실제 제품에 상한이 있음. **Claude Code 는 `MEMORY.md` 의 앞 200줄 또는 25KB 까지만 읽음**("It loads only the first 200 lines or 25 KB of `MEMORY.md`"). 그 아래는 안 읽힘. 이 저장소의 [CLAUDE.md 프롬프트 감사](../prompting/2026-09-25-claude-md-prompt-audit.md)와 바로 붙는 수치고, **메모리 파일을 계속 늘리는 운영은 조용히 잘리는 쪽으로 굴러감.**

## 글이 아니라 API 로 올라와 있는 부분

dreaming 은 개념 설명으로 끝나지 않고 **Dreams** 라는 이름으로 Managed Agents 에 붙어 있음. 2026-11-16 확인 기준 **research preview** 이고 Claude API 에서만 됨.

| 항목 | 값 |
|---|---|
| 베타 헤더 | `managed-agents-2026-04-01` + **`dreaming-2026-04-21`** (후자 없으면 접근 안 됨) |
| 입력 | 기존 memory store **1개** + 세션 **1~100개** |
| 출력 | **새 memory store.** 입력 store 는 수정되지 않아서 결과가 나쁘면 버리면 됨 |
| 소요 | 비동기이고 "minutes to a few hours". 입력 트랜스크립트 수에 좌우됨 |
| 과금 | 고른 모델의 **표준 API 토큰 요금**. 비용이 입력 세션의 수와 길이에 거의 선형으로 비례 |
| 조종 | `instructions` 로 무엇을 모을지 지시. **줄 단위 명령형 지시는 대체로 효과 없음** |
| 데이터 | **ZDR 비적격**(self-hosted 제외), HIPAA 비적격. 세션 트랜스크립트가 지울 때까지 남음 |

API 릴리스 노트 기준으로 모델 지원이 **2026-10-01 에 Opus 5.5 · Fable 5.1 · Sonnet 5.5 로 시작해 2026-11-09 에 Fable 5.2** 가 붙었고, 문서의 가용성 표에는 Fable 5 · Opus 5 · Sonnet 5 · Opus 4.8 · Opus 4.7 · Sonnet 4.6 까지 **10종**이 올라와 있음.

문서에서 글과 정확히 겹치는 문장이 하나 있어서 적어 둘 값어치가 있음. **`instructions` 는 "focus on coding-style preferences" 같은 상위 지침에 쓰는 것이고, "X 문장을 Y 로 바꿔라" 같은 줄 단위 명령은 파이프라인이 합성 패스라서 대체로 아무 변화를 못 만든다**는 것. 위에서 본 "무엇에 쓰는지는 말하고 어떻게 하는지는 말하지 말라"가 제품 문서에도 같은 모양으로 적혀 있음.

실무에서 먼저 걸릴 자리 둘.

- **ZDR 비적격이 설계 제약임.** 사용자 데이터가 트랜스크립트로 남아야 dreaming 이 돌아감. 앱에서 민감한 대화를 다룬다면 이 기능은 self-hosted 쪽으로 가거나 포기하는 선택이 됨
- **`input_memory_store_too_large` 에러가 목록에 있는데 상한 수치가 문서에 없음.** store 를 키우다 보면 만나는 벽인데 숫자가 안 적혀 있어서 실측해야 함

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Memory Engineer](./2026-08-02-memory-engineer.md) | 같은 주제의 반대편. 저쪽이 "무엇을 버릴까"를 네 연구소 관점으로 정리했다면 이쪽은 "무엇을 남길지를 모델이 스스로 고르게 두면 어떻게 되는가"를 측정함 |
| [SelfCompact](./2026-06-22-self-compacting-agents.md) | 이 글이 메모리에서 명시적으로 제외한 compaction. 요약 시점을 사전에 판단하는 쪽과 사후에 원본으로 복구하는 쪽의 대비 |
| [서버측 compaction](./2026-09-20-claude-api-compaction.md) | 같은 Claude API 안에서 컨텍스트를 줄이는 기능과 지식을 남기는 기능이 따로 있음. 둘을 같은 것으로 보면 안 되는 이유 |
| [자기개선 에이전트 루프의 7가지 규칙](./2026-08-10-self-improving-agent-loops.md) | 메모리 품질을 파일 수로 재면 413 파일이 나옴. 얕은 지표 함정의 메모리판 |
| [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md) | CL-Bench 류의 메모리 평가를 머지 게이트로 쓸 수 있는가. 모델별 이득 폭이 이렇게 갈리면 단일 임계값이 안 세워짐 |
| [CLAUDE.md 프롬프트 감사](../prompting/2026-09-25-claude-md-prompt-audit.md) | 주입 메모리의 200줄 / 25KB 상한. 길어질수록 준수율이 떨어진다는 쪽과 직접 붙음 |
| [Foundation Models 실전](./2026-08-21-foundation-models-in-practice.md) | 작은 모델에 메모리를 붙이는 설계. Sonnet 4.6 줄이 그 위쪽 한계를 보여줌 |
| [Managed Agents 동적 워크플로](../agents/2026-10-09-managed-agents-dynamic-workflows.md) | Dreams 가 올라가 있는 같은 Managed Agents 표면. 베타 헤더와 비동기 작업 모양이 같은 계열 |

## 짚어야 할 것

- **수치 전부가 Anthropic 자체 측정이고 비교 대상이 Anthropic 모델뿐임.** 메모리 이득이 모델 세대를 따라 커지다가 최신에서 다시 줄어드는(Opus 4.7 +0.317 → Opus 5.5 +0.183) 모양은 자사 모델 라인업 안에서만 그린 그림임
- **CL-Bench 1.0 의 공개 여부와 과제 구성을 확인하지 못했음.** 위 표는 데이터베이스 과제 하나의 mean reward 이고 다른 과제의 수치는 이 글에서 확인할 수 없었음
- **위키 실험의 채점자가 누구인지 확인할 수 없었음.** "graded criteria" 비율이라고만 적혀 있고 사람 채점인지 모델 채점인지가 본문에서 확인되지 않음. 모델 채점이면 [Eval Engineering](./2026-08-01-eval-engineering-merge-gate.md)이 경고한 자기 채점 문제가 걸림
- **dreaming 의 비용이 달러로 적히지 않았음.** 그림 설명의 2M~128M dreaming 토큰이 유일한 단서고, 위에서 본전 계산을 붙였지만 그 계산은 원문에 없는 이 문서의 추정임
- **포켓몬 사례는 수정 전후 한 쌍의 비교임.** 반복 횟수나 분산이 적히지 않아서 일화 이상으로 읽으면 안 됨
- 다만 **원문이 부정 사례를 숨기지 않음.** 법률 브리프 과제 중 하나에서는 위키가 뚜렷한 이득을 못 냈다고 적어 뒀고, 위키만 쓸 때(48%)가 검색만 쓸 때(53%)보다 낮다는 것도 그대로 적음
- 원문 본문에 읽는 쪽을 조종하려는 지시문이나 명령문은 없었음. Dreams 문서 쪽도 같음
- 위 CL-Bench 표의 수치는 원문 본문과 그림 설명에서 읽은 것이고, 그중 **Opus 5.5 와 Opus 5 두 줄만 원문 문장으로 다시 대조했음**

## 유효기간

**2026-11-16 확인 기준이고 글은 2026-11-06, Dreams 는 research preview 임.** 글의 측정 결과는 시점이 박힌 기록이라 낡지 않지만 제품 쪽은 움직임. 다시 볼 때 확인할 것 셋. **Dreams 가 research preview 를 벗어나 `dreaming-2026-04-21` 헤더가 필요 없게 됐는지**, **`input_memory_store_too_large` 의 상한이 숫자로 문서에 적혔는지**, 그리고 **`MEMORY.md` 의 200줄 / 25KB 상한이 바뀌었는지**. 앞의 둘은 [Dreams 문서](https://platform.claude.com/docs/en/managed-agents/dreams)와 [API 릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서, 셋째는 Claude Code 문서에서 확인하면 됨. CL-Bench 가 공개되면 위 표를 갱신하지 말고 공개판 기준으로 새로 재는 편이 나음.
