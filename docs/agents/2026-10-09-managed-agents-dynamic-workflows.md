---
title: Managed Agents 다이내믹 워크플로, 오케스트레이션 코드를 모델이 직접 쓴다
source: https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration
author: Anthropic
published: 2026-10-09
collected: 2026-10-10
tags: [orchestration, multi-agent, subagent, parallel, permissions, llm-cost, claude]
---

출처: [Multiagent orchestration, Dynamic workflows](https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration#dynamic-workflows) · [Workflow runs](https://platform.claude.com/docs/en/managed-agents/workflow-runs) · [API release notes 2026-10-09 항목](https://platform.claude.com/docs/en/release-notes/api) (문서 확인은 2026-10-10)

## 요약

**2026-10-09**에 Claude Managed Agents 에 다이내믹 워크플로가 베타로 붙음(베타 헤더 `managed-agents-2026-04-01`). 지금까지 "여러 에이전트를 어떤 순서로 돌릴지"는 사람이 코드로 적는 것이었는데, 이제 **모델이 그 프로그램을 직접 쓰고 서버가 백그라운드에서 실행**함. 그 실행 단위가 **workflow run** 임.

숫자가 이 기능의 성격을 말해줌. 한 run 안에서 **동시에 64 스레드**, run 수명 전체로 **에이전트 1,000개**, 기본 수명 **24시간**, 한 세션에 열어둘 수 있는 run **10개**. 서브에이전트 위임이 "1단 깊이, 세션당 자식 스레드 25개"였던 것과 자릿수가 다름.

운영 쪽에서 실제로 걸리는 건 기능이 아니라 **기본값**임. `multiagent.type` 을 `multiagent_20261001` 로 올리면 `workflows` 와 `subagents` 가 **둘 다 기본 enabled** 이고, 비용 상한인 **세션 예산은 세션을 만들 때만 걸 수 있음**. 기존 `coordinator` 타입에서 이전할 때 블록에 `workflows` 를 안 적으면 조용히 켜진 상태로 넘어감.

## 세 가지 위임 방식

같은 `multiagent` 블록 안에서 셋이 각각 켜지고 꺼짐.

| | 서브에이전트 | 다이내믹 워크플로 | 어드바이저 |
|---|---|---|---|
| 누가 다음 단계를 정하나 | 세션 에이전트가 직접 | 모델이 쓴 프로그램 | 세션 에이전트가 직접 |
| 결과를 받는 쪽 | 세션 에이전트 | 프로그램 | 세션 에이전트 |
| 후속 메시지 | 보낼 수 있음 | 못 보냄. run 끝까지 서버가 스레드를 아카이브 | 해당 없음 |
| 깊이 | 1단 | run 안에서 중첩 불가 | 해당 없음 |
| 동시 한도 | 세션당 자식 스레드 25개(유휴 포함) | run 하나에 동시 64 스레드 | 프라이머리 스레드만 |
| 목록 한도 | `subagents.predefined_agents` 20개 | `workflows.predefined_agents` 20개 | 없음(`model` 지정) |
| 기본값 | enabled | enabled | disabled |

run 의 스레드는 세션의 자식 스레드 25개 한도에서 **면제**됨. 어드바이저 스레드도 안 셈. 즉 25개로 비용이 막힐 거라고 생각하면 안 됨.

## run 의 구조

run 안에 **phase** 가 있고 phase 안에서 **에이전트 스레드**가 돎. 프로그램이 할 수 있는 것은 다섯 가지로 정리돼 있음.

- 에이전트를 동시에 여러 개 띄우기(fan out)
- 한 에이전트의 결과를 다음 에이전트에게 넘기기
- 다음에 어느 에이전트를 돌릴지와 그 프롬프트를 스스로 결정하기
- phase 안에서 반복하기(리뷰를 통과할 때까지, 또는 정해진 라운드까지)
- 에이전트 하나가 실패했을 때 처리하거나, 그 실패로 run 을 끝내기

[Graph Engineering](../practices/2026-07-20-graph-engineering.md)이 `parallel()` 과 `pipeline()` 으로 다룬 그 구분이 그대로 있는데, **그래프를 짜는 주체가 사람에서 모델로 옮겨간 형태**임. 그 문서가 "그래프를 쓰지 말아야 할 5가지 경우"로 정리한 판단을 이제 프롬프트로 전달해야 함. 공식 문서가 권하는 방법도 그것임. 시스템 프롬프트에 **언제 run 을 쓰고 언제 직접 하라**를 적는 것.

> "You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run."

run 을 시작하는 별도 API 호출은 없음. 에이전트가 `user.message` 를 읽고 스스로 판단함. 그래서 **권한 정책은 run 시작에 걸리지 않고 run 안의 툴 호출에만 걸림.**

## 켜는 법

```json
{
  "multiagent": {
    "type": "multiagent_20261001",
    "workflows": { "type": "enabled" },
    "subagents": { "type": "disabled" }
  }
}
```

업데이트 규칙이 함정이 되는 자리임.

- 안 보낸 설정은 **저장된 값을 유지**함. 단 이것은 **이미 `multiagent_20261001` 타입인 에이전트를 같은 타입으로 업데이트할 때**만임
- `coordinator` 에서 옮겨오는 업데이트는 다름. **`multiagent` 블록을 통째로 보내는 것**이라서 빠뜨린 것이 유지되지 않음. `subagents.predefined_agents` 를 안 보내면 **기존 전문 에이전트 목록이 사라지고**, `advisor` 를 안 보내면 **어드바이저가 꺼짐.** 마이그레이션 업데이트에는 지금 쓰는 에이전트 목록과 어드바이저 설정을 **다시 적어 보낼 것**
- `null` 로 보낸 설정은 **기본값**을 먹음. `workflows` 의 기본값이 enabled 라서 `null` 은 켜는 것이 됨
- `predefined_agents` 를 보내면 저장된 목록을 **통째로 교체**함
- 이미 만들어진 세션은 생성 시점의 설정을 복사해 뒀으므로 에이전트를 고쳐도 안 바뀜
- 커스텀 툴 이름에 `ant__` 접두사가 있으면 업데이트가 전부 400 으로 실패함. 예약 접두사임

## 한도와 비용

| 한도 | 값 | 넘기면 |
|---|---|---|
| run 하나에서 동시에 일하는 스레드 | **64** | 하나 끝날 때까지 더 안 만듦. 문서가 "보장하지 않는 숫자"로 적어둠 |
| 한 run 이 수명 전체에 띄우는 에이전트 | **1,000** | run 이 `thread_limit_error` 로 끝남. 실패한 에이전트를 새 스레드로 재실행하므로 스레드 수는 1,000 을 넘을 수 있음 |
| run 수명 | **24시간** 기본, 에이전트가 더 짧게 설정 가능 | `timeout_error`. **에이전트가 설정한 수명을 알려주는 이벤트가 없음** |
| 한 세션에 동시에 열린 run | **10** 기본 | `max_workflow_runs_error`. 유휴 run 도 셈 |

비용 쪽은 run 자체 가격이 없고 각 모델 요율로 토큰이 청구됨. 문제는 계측임.

- **run 하나의 사용량을 바로 주는 필드가 없음.** 세션의 스레드를 전부 나열해서 해당 `workflow_run_id` 를 가진 것들의 `usage` 를 직접 더해야 함. 아카이브된(`terminated`) 스레드도 목록에 남으니 끝난 run 도 집계는 됨
- **run 을 나열하는 엔드포인트가 아예 없음.** 재접속하면 세션 이벤트를 `types` 필터로 다시 받아 상태를 복원해야 함
- 예산에 닿으면 열린 run 이 전부 멈추고 세션이 `budget_reached` 로 유휴가 됨. 단 **각 스레드는 이미 시작한 모델 요청을 끝내므로, 일하고 있던 스레드 수만큼 예산을 넘길 수 있음.** 동시 64 스레드가 상한이라는 걸 같이 보면 초과분의 크기가 나옴
- 예산을 올리거나 없애면 멈춘 run 이 재개됨. 인터럽트도 같이 멈춘 run 은 예외
- 모델 요청은 조직의 **기존 Messages API 레이트리밋을 그대로 먹음.** run 분량을 미리 비워 두라는 것이 공식 안내임

## 운영에서 먼저 걸릴 것

**승인이 64갈래로 갈림.** run 스레드의 툴 호출은 프라이머리 스트림으로 올라오고, 세션이 `running` 인 동안에도 도착함. 그래서 `session.status_idle` 을 기다리지 말고 즉시 답해야 함. 공식 샘플 코드가 확인 분기를 **일부러 안 넣었고** 그 이유를 이렇게 적어둠. 모든 호출을 허용하는 분기를 만들면 `always_ask` 가 사실상 always allow 가 된다고. [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md)가 내린 결론, 즉 승인 정확도를 올리는 것보다 위험한 명령을 승인 대상에서 빼는 편이 싸다는 판단이 여기서는 선택이 아님. 64갈래로 들어오는 승인 요청을 사람이 읽을 수 없음.

**run 의 스레드가 샌드박스를 공유함.** 공식 서술로 "A run's threads work in the session's sandbox, so every thread works with the same files" 임. 세션이 마운트한 메모리 스토어의 파일까지 포함됨. 대화 히스토리만 각자 가짐. [OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](../security/2026-08-07-openai-hugging-face-agent-incident.md)에서 끌어낸 교훈이 **"여러 에이전트에게 공유 쓰기 가능 경로를 주지 말 것"** 이었는데, 이 기능은 그걸 설계로 깔고 있음. 그 사고에서 에이전트들이 게시판을 만든 매개가 공유 쓰기 경로 하나였음. 중첩이 안 되고 서버가 스레드를 끝까지 아카이브한다는 점에서 조건이 같지는 않지만, **파일로 서로에게 신호를 남기는 경로는 열려 있음.** 격리가 필요한 작업을 run 으로 넘기기 전에 이 한 줄을 먼저 봐야 함.

**끝났다는 신호를 세 가지로 쪼개 읽어야 함.** `result` 가 `completed` 라는 것은 **프로그램이 끝까지 돌았다는 뜻이고 일이 됐다는 뜻이 아님.** 스레드의 작업이 실패했거나 스레드를 못 만들었어도 run 은 `completed` 로 끝날 수 있다고 문서가 직접 적어둠. 실패를 찾으려면 각 스레드의 이벤트를 읽어야 함. 세션이 유휴가 된 것도 일이 끝난 신호가 아님. 멈춘 run 은 세션을 `running` 으로 붙들지 않기 때문임.

**phase 이벤트를 진행률로 쓸 때 보장이 없음.** `workflow_run.created` 가 선언한 `phases` 를 두고 문서가 요구하는 처리가 이것임. phase 가 둘 이상 열려 있는 경우, `phases` 에 없는 phase 가 시작하는 경우, **완료된 run 인데도 선언된 phase 가 한 번도 시작하지 않는 경우**를 전부 다루라는 것. phase 이벤트에는 이름도 없어서 `workflow_run_phase_id` 로 되찾아야 함.

**워크플로 코드를 볼 수 없음.** run 을 시작·관리하는 툴 호출도, 각 스레드가 프로그램에 돌려준 결과도 안 보임. 공식 문서가 제시하는 유일한 확인 경로는 run 이 끝난 뒤 **에이전트에게 워크플로를 그대로 출력해 달라고 부탁하는 것**임.

> "Print the workflow that you started the run with, word for word, in one code block. Do not start another run."

디버깅 수단이 모델에게 묻는 것뿐이라는 뜻임. [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md)이 세운 기준, 즉 되돌리기 어려운 레인은 점수와 무관하게 열지 않는다는 원칙을 적용하면, 이 기능을 쓰기 좋은 자리는 **읽기 위주의 대량 작업**임. 공식 문서가 드는 예도 그쪽임. 감사, 마이그레이션, 딥리서치, 교차 검증.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Graph Engineering](../practices/2026-07-20-graph-engineering.md) | fan out 과 phase 전달이 `parallel()`·`pipeline()` 과 같은 구조. 그래프를 짜는 주체가 모델로 옮겨간 것이 차이 |
| [에이전트 오케스트레이터 3종 비교](./2026-08-10-agent-orchestrators.md) | 같은 문제를 로컬 컨트롤 플레인으로 푸는 쪽. 워크트리 격리 대 샌드박스 공유로 갈림 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](../security/2026-08-05-agent-approval-miss-rates.md) | 동시 64 스레드의 승인 요청을 사람이 읽을 수 없다는 쪽으로 결론이 강해짐 |
| [OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](../security/2026-08-07-openai-hugging-face-agent-incident.md) | 공유 쓰기 경로. 그 문서의 교훈과 이 기능의 샌드박스 공유가 정면으로 만남 |
| [Harness · Loop · Graph](../practices/2026-07-28-loop-graph-harness.md) | loop 없이 graph 를 만들지 말라는 순서. 여기서는 graph 가 먼저 주어지므로 loop 를 뒤에서 세워야 함 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 레버 3(가시성·트립와이어·예산). 세션 예산이 세션 생성 때만 걸린다는 제약이 그 레버의 전제를 깎음 |

## 짚어야 할 것

- **베타임.** 베타 헤더 `managed-agents-2026-04-01` 로 들어가고, 타입 식별자가 `multiagent_20261001` 이라 다음 개정이 예고된 형태임
- 동시 64 스레드는 문서 스스로 **"the API doesn't guarantee this number"** 로 적어둔 값임. 이 숫자를 전제로 설계하면 안 됨. phase 순서도 같은 식으로 보장이 빠져 있음
- run 과 phase 의 `name`·`description` 은 **모델이 쓴 텍스트**이고, 문서가 "요청의 단어를 되풀이할 수 있다"고 명시함. 서버가 각각 64자·256자로 자름. 사용자에게 그대로 렌더하는 UI 라면 신뢰 경계가 여기 있음
- 세션 런타임 요금이 토큰과 별도로 붙는지는 pricing 페이지 쪽 서술이고 이번에 **확인하지 않았음.** `list_cost` 합계가 세션 런타임을 빼고 나온다는 서술만 확인함
- 원문의 다이어그램 둘(run 의 계층, run 의 상태 전이)은 **SVG 이미지라 내용을 읽지 못했고** 대체 텍스트만 확인함. 위 구조 서술은 전부 본문 산문에서 가져온 것임
- iOS 앱이 직접 쓰는 기능이 아님. 서버측 에이전트 인프라임. 앱 개발 쪽에서 값어치가 나오는 자리는 레포 감사, 대량 마이그레이션, 리뷰 팬아웃 같은 **빌드 바깥의 작업**임

## 유효기간

**2026-10-10 확인 기준이고 베타임.** 다시 볼 때 확인할 것 셋. **`workflows` 의 기본값이 enabled 로 남아 있는지**, **한도 수치(64 / 1,000 / 10 / 24시간)가 바뀌었는지**, 그리고 **run 을 나열하는 엔드포인트가 생겼는지**. 셋째는 지금 없어서 재접속 때 이벤트를 다시 긁어야 하는 부분임. [릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서 `workflow` 항목이 더 붙었는지부터 보는 편이 빠름.
