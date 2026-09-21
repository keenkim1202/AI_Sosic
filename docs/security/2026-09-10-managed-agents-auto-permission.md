---
title: Managed Agents 의 auto 권한 정책, 승인 판단을 사람에서 서버로 옮김
source: https://platform.claude.com/docs/en/managed-agents/permission-policies
author: Anthropic
published: 2026-09-10
collected: 2026-09-21
tags: [agent-security, permissions, human-in-the-loop, claude, mcp, agent-tooling]
---

출처: [Permission policies, Claude Docs](https://platform.claude.com/docs/en/managed-agents/permission-policies) · [API 릴리스 노트 2026-09-10](https://platform.claude.com/docs/en/release-notes/api)

## 요약

Claude Managed Agents 의 권한 정책에 세 번째 값 **`auto`** 가 붙었음(2026-09-10, 베타 헤더 `managed-agents-2026-04-01`). 툴 호출마다 **서버가 직접 판정**해서 **실행 / 거부 / 승인 대기** 셋 중 하나로 보냄. 판정 근거에 툴 이름뿐 아니라 **그 호출의 입력과 그 시점까지의 세션 내용**이 들어가므로, 같은 툴의 두 호출이 다르게 취급될 수 있음. 이 저장소가 [승인 miss rate 문서](./2026-08-05-agent-approval-miss-rates.md)에서 적어둔 결론("승인 정확도를 올리려 하지 말고 위험한 명령을 승인 대상에서 빼라")과 방향이 같지만, 공식 문서가 **"`auto` 는 사람 검문소가 아니다"** 를 경고 박스로 따로 박아둔 것이 핵심임. 그리고 **`user.message` 에 넣는 내용이 그대로 "내 의도"로 읽히기 때문에**, 여기에 신뢰할 수 없는 사용자 입력을 흘려보내면 거부됐을 호출이 통과할 수 있음.

## 세 가지 정책과 기본값

| 정책 | 동작 | 어느 툴셋의 기본값인가 |
|---|---|---|
| `always_allow` | 확인 없이 실행 | **agent 툴셋**(`agent_toolset_20260401`) |
| `always_ask` | 세션을 멈추고 내 승인을 기다림 | **MCP 툴셋**(`mcp_toolset`) |
| `auto` | 호출마다 서버가 판정 | **없음. 명시적으로 켜야 함** |

MCP 툴셋이 `always_ask` 기본인 이유를 문서가 밝혀둠. MCP 서버에 **새 툴이 추가돼도 내 승인 없이는 안 돌게** 하려는 것임. 서버를 신뢰한다면 `default_config.permission_policy` 로 풀어주는 구조.

정책은 두 자리에 둘 수 있음. 툴셋 전체에 거는 `default_config`, 그리고 툴 하나만 덮어쓰는 `configs` 배열. 기본을 `auto` 로 두고 `bash` 만 `always_ask` 로 올려두는 조합이 문서의 예제임.

```json
{
  "type": "agent_toolset_20260401",
  "default_config": { "permission_policy": {"type": "auto"} },
  "configs": [
    {"name": "bash", "permission_policy": {"type": "always_ask"}}
  ]
}
```

**이미 돌고 있는 세션은 만들어질 때의 툴셋 설정을 그대로 유지함.** 에이전트를 업데이트해도 그 뒤에 생성된 세션부터 적용됨. 사고 대응으로 정책을 조이는 상황이라면 이 점이 그대로 지연이 됨.

**커스텀 툴은 권한 정책의 적용 대상이 아님.** 실행 주체가 내 애플리케이션이라 `agent.custom_tool_use` 를 받고 내가 직접 판단함. 즉 `auto` 를 켜도 커스텀 툴 쪽 판단 코드는 그대로 내 몫임.

## auto 가 내리는 세 가지 결론

| 결론 | 일어나는 일 |
|---|---|
| 실행 | 안전하다고 판정되면 `always_allow` 와 동일하게 그냥 돎 |
| 거부 | 고위험으로 판정되면 실행 안 됨. 에이전트는 `is_error: true` 와 `Permission to use {tool_name} has been denied.` 를 툴 결과로 받고, **세션은 계속 진행됨** |
| 승인 대기 | 판정이 서지 않으면(`indeterminate`) `always_ask` 처럼 멈춤 |

거부 쪽에 걸린 문장 하나가 설계에 영향을 줌. **클라이언트가 이 거부를 뒤집을 수 없음.** 서버가 `auto` 로 거부한 호출에 `user.tool_confirmation` 을 보내면 **400 에러**임. 사람이 "괜찮으니 진행해" 라고 개입할 경로가 없다는 뜻이고, 되돌리려면 그 툴의 정책 자체를 바꿔 새 세션을 만드는 수밖에 없음.

반대로 거부돼도 세션이 죽지 않고 계속 간다는 점도 같이 봐야 함. 에이전트는 툴 하나가 막혔다는 것만 알고 다음 수를 찾음. [Hugging Face 공격 사고](./2026-08-07-openai-hugging-face-agent-incident.md)가 보여준 "채널을 막으면 다른 채널로 옮겨간다"는 패턴이 여기서도 그대로 성립할 수 있는 구조임. 원문은 이 지점을 다루지 않음.

## 무엇이 판정을 움직이나

문서가 의도(intent)의 출처를 명시적으로 갈라둔 부분이 이 글에서 가장 값어치 있음.

- **`user.message` 이벤트에 내가 올린 내용은 내 의도로 계산됨.** 그래서 같은 호출이 맥락에 따라 통과할 수 있음
- 반면 **툴 결과, 가져온 웹페이지, MCP 서버의 응답, 세션 스레드 사이의 메시지에서는 의도를 읽지 않음.** 내용을 평가는 하되 **지시로 받지는 않음**
- 그래도 **누가 요청하든 고위험으로 판정되는 호출**이 따로 있음

여기서 바로 따라 나오는 경고도 문서에 있음. **신뢰할 수 없는 최종 사용자 입력을 `user.message` 로 중계하면 그 입력도 내 의도로 읽힘.** 앱이 사용자 채팅을 그대로 에이전트에 넘기는 구조라면 `auto` 의 판정면이 사용자에게 열려 있는 셈임. 문서의 처방은 단순함. **그 사용자가 검토 없이 돌리게 두지 않을 툴에는 `always_ask` 를 걸어라.**

이게 프롬프트 인젝션 방어선을 어디에 긋는지에 대한 답이기도 함. 툴 결과와 웹 콘텐츠는 지시로 안 받는다고 선을 그었고, 대신 `user.message` 가 신뢰 경계가 됨. **그 경계를 앱이 지켜야 함.**

## 이벤트에서 판정 결과 읽기

`auto` 를 안 쓰더라도 관측 표면이 넓어졌음. 모든 정책에서 `agent.tool_use` 와 `agent.mcp_tool_use` 이벤트가 **`evaluated_permission`** 을 실어 보냄(`"allow"` / `"ask"` / `"deny"`). 대부분의 이벤트는 **`evaluation`** 객체도 같이 오고, 그 `type` 이 어떤 정책이 그 결과를 냈는지 말해줌. `auto` 일 때는 서버의 판정과 `reason_code` 까지 들어감.

```json
{
  "type": "agent.tool_use",
  "name": "bash",
  "input": { "command": "rm -rf /workspace/reports" },
  "evaluated_permission": "deny",
  "evaluation": {
    "type": "auto",
    "evaluated_permission": { "type": "deny", "reason_code": "high_risk" }
  }
}
```

문서가 이름을 밝힌 `reason_code` 는 둘임. 거부 쪽의 **`high_risk`**, 승인 대기 쪽의 **`indeterminate`**. 그리고 이 값은 **클라이언트가 분기하고 감사 기록에 남기라고 있는 것이지 최종 사용자에게 보여줄 문구가 아니라고** 못 박아둠.

`evaluation` 이 아예 안 오는 경우가 둘 있음. **세션에서 활성화되지 않은 툴을 에이전트가 부른 경우**(정책 평가 없이 바로 `deny`), 그리고 **`evaluation` 도입 이전에 기록된 이벤트**임. 후자는 `allow` 면 `always_allow`, `ask` 면 `always_ask` 로 읽으라는 안내가 붙어 있음. 로그를 소급 집계하는 코드를 쓸 거면 이 두 갈래를 먼저 넣어야 함. 문서도 **모르는 `evaluation.type` 과 `reason_code` 를 견디게 작성하라**고 적어둠.

## 승인 대기가 걸렸을 때의 흐름

1. `agent.tool_use` 또는 `agent.mcp_tool_use` 이벤트가 나감
2. 세션이 `session.status_idle` 로 멈춤. `stop_reason.type` 이 `requires_action` 이고 막고 있는 이벤트 ID 들이 `stop_reason.event_ids` 에 들어 있음. **세션은 무한정 기다림**
3. 각 ID 에 대해 `user.tool_confirmation` 을 보냄. `result` 는 `"allow"` 또는 `"deny"` 이고, 거부 이유를 `deny_message` 로 붙일 수 있음. 여러 건을 한 요청에 묶어 보낼 수 있음
4. 다 풀리면 세션이 `running` 으로 돌아감. 거부된 툴은 안 돌고, 에이전트는 내 `deny_message` 가 담긴 거부 결과를 받음

**무한정 기다린다**는 것이 [Claude Cowork 의 Dispatch](../practices/2026-08-10-claude-cowork.md)와 정반대임. 그쪽은 **10분 안에 응답 없으면 자동 거부하고 그 행동 없이 계속 진행**함. 자리를 비웠을 때 한쪽은 반쪽짜리 결과가 나오고, 다른 쪽은 아무것도 안 나온 채 세션이 살아 있음. 어느 쪽이 나은지는 워크로드에 달렸지만, **같은 회사의 두 제품이 같은 상황에서 반대로 동작한다**는 것은 알고 써야 함.

같은 날 `ant` CLI **1.32.0** 에 `ant beta:sessions connect` 가 붙어서, 터미널을 세션에 붙여 대기 중인 호출을 대화형으로 허용·거부할 수 있음. `--web` 을 주면 Console 의 세션 뷰어를 로컬에서 띄움.

## auto 는 사람 검문소가 아님

문서가 경고 박스로 따로 박아둔 문장임.

> `auto` is not a human checkpoint. If the server determines that a call is safe, the call runs before anyone sees it, and its effects might not be reversible. If a person must review a tool's calls before they run, configure `always_ask` on that tool.

이 한 줄이 `auto` 의 위치를 정함. `auto` 는 **`always_allow` 를 조이는 장치이지 `always_ask` 를 대체하는 장치가 아님.** 지금 `always_ask` 로 두고 사람이 보고 있는 툴을 `auto` 로 내리는 것은 검토 단계를 없애는 변경이고, 지금 `always_allow` 인 툴을 `auto` 로 올리는 것은 순수한 개선임. 도입 방향을 한쪽으로만 잡는 게 맞음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 같은 문제의 반대편 답. 그쪽은 사람 승인이 보안 경계로 결함이 있다는 데이터고, 이쪽은 그 판정을 서버로 옮긴 구현. 단 그쪽이 권한 것은 **승인 대상에서 빼기**였지 **다른 모델에게 승인시키기**가 아니었음 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 제품 쪽의 `Auto` 모드가 같은 발상임. 다만 거기서는 불안전한 행동을 차단하고 **명시적 허가를 요청**한다고 적혀 있고, 여기 `auto` 의 거부는 클라이언트가 뒤집을 수 없음. 승인 타임아웃도 10분 대 무한으로 갈림 |
| [OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](./2026-08-07-openai-hugging-face-agent-incident.md) | 거부돼도 세션이 계속 간다는 설계가 "막힌 채널을 우회한다"는 관찰과 만나는 지점 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | MCP 툴셋이 `always_ask` 기본인 이유(새 툴 자동 추가)가 스펙 쪽 툴 목록·발견 논의와 같은 문제를 봄 |

## 짚어야 할 것

- **판정 주체가 무엇인지 문서가 밝히지 않음.** "서버가 평가한다"고만 적혀 있고 분류기인지 모델인지, 어떤 기준표를 쓰는지, 오탐·미탐률이 얼마인지 아무 수치가 없음. [승인 miss rate 문서](./2026-08-05-agent-approval-miss-rates.md)에는 사람 쪽 수치라도 있었는데 여기는 비교할 숫자 자체가 없음
- **`reason_code` 의 전체 목록을 확인할 수 없었음.** 문서에 나온 것은 `high_risk` 와 `indeterminate` 둘뿐이고, "모르는 값을 견디게 작성하라"는 안내로 보아 더 있거나 늘어날 것으로 보임
- 베타임(`managed-agents-2026-04-01`). 필드 이름과 기본값이 바뀔 수 있음
- **`auto` 를 켠 뒤 승인 프롬프트가 줄어든다는 보장이 없음.** `indeterminate` 가 얼마나 자주 나오는지에 달렸는데 그 비율에 대한 언급이 없음. 승인 피로를 줄일 목적이라면 실제로 재보기 전에는 효과를 가정하면 안 됨
- 이 문서는 공식 문서와 릴리스 노트만 읽고 정리한 것임. **실제로 `auto` 를 켜서 어떤 호출이 어느 쪽으로 가는지는 확인하지 않았음**

## 유효기간

**2026-09-21 확인 기준**이고 `auto` 는 2026-09-10 릴리스 노트 항목임. 베타 단계라 정책 이름, `evaluation` 객체 모양, `reason_code` 값이 움직일 수 있음. 다시 볼 때 확인할 것 셋. **`auto` 가 베타를 벗어났는지**, **어느 툴셋의 기본값이 `auto` 로 바뀌었는지**(지금은 어느 쪽도 아님), 그리고 **`reason_code` 목록이 문서에 정리됐는지**. 두 번째는 기존 에이전트의 동작을 조용히 바꾸는 변경이라 릴리스 노트에서 먼저 봐야 함.
