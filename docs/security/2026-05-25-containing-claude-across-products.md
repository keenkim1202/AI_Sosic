---
title: Claude를 제품마다 어떻게 격리하는가, 승인 프롬프트를 샌드박스로 바꾼 패턴 셋과 놓쳤던 위험 넷
source: https://www.anthropic.com/engineering/how-we-contain-claude
author: Max McGuinness, Mikaela Grace, Jiri De Jonghe, Jake Eaton, Abel Ribbink (Anthropic)
published: 2026-05-25
collected: 2026-10-02
tags: [agent-security, sandboxing, permissions, prompt-injection, claude-code, mcp]
---

출처: [How we contain Claude across products, Anthropic Engineering](https://www.anthropic.com/engineering/how-we-contain-claude) (2026-10-02 확인)

## 요약

Anthropic이 자사 제품 셋(claude.ai 코드 실행, Claude Code, Claude Cowork)을 어떤 격리 구조로 돌리는지 직접 적은 글임. 이 저장소 관점에서 값어치가 큰 건 결론이 아니라 **자기 텔레메트리와 자기가 놓친 사고 넷을 숫자와 함께 공개했다는 점**임. 핵심 둘. 첫째, **사용자가 승인 프롬프트의 약 93%를 승인**하고 있었고, 그래서 승인을 더 잘 받는 대신 **OS 샌드박스를 깔아 프롬프트를 84% 줄였음**. 둘째, 레드팀이 직원을 피싱해 Claude Code에 악성 프롬프트를 붙여 넣게 했을 때 **25회 중 24회 자격증명 유출에 성공**했음. 거의 전부 뚫렸으므로 이때 실제로 막아낸 방어는 없고, 글이 드는 것은 **그 상황에서 버틸 수 있는 방어**임: **POST 를 막는 egress 제어**, 그리고 애초에 **`~/.aws` 에 닿지 못하게 하는 파일시스템 경계**. 글이 내세우는 원칙은 **환경 계층에서 먼저 격리하고 모델 계층은 그다음**이라는 것임.

## 기본 정보

| | |
|---|---|
| 발행 | 2026-05-25. Anthropic Engineering |
| 다루는 제품 | claude.ai 코드 실행, Claude Code, Claude Cowork |
| 성격 | 자사 설계 공개 + 자사 레드팀 사고 보고 |
| 모델 쪽 수치 | **Claude Opus 4.7** 기준. 지금 기준으로는 세대가 지난 수치임 |

## 위험 셋, 방어 셋

글이 위험을 셋으로 가름. **사용자 오용**(악의 또는 부주의), **모델 오작동**, **외부 공격자**(프롬프트 인젝션, 서플라이체인). 그리고 방어를 셋으로 가름.

| 계층 | 수단 | 성질 |
|---|---|---|
| 환경 | 프로세스 샌드박스, VM, 파일시스템 경계, egress 제어 | 결정적. 모델이 뭘 하려 해도 못 하게 함 |
| 모델 | 시스템 프롬프트, 분류기, 프로브, 학습 수정 | **확률적. 100%가 되지 않음을 글이 직접 인정** |
| 외부 콘텐츠 | 툴 권한, MCP 서버 감사, 데이터 접근 축소 | 컨텍스트로 들어오는 것을 줄임 |

모델 계층이 얼마나 센지도 수치로 적음. 프롬프트 인젝션 내성 벤치마크 기준임.

> "On Gray Swan's Agent Red Teaming benchmark, which tests susceptibility to prompt injection, Claude Opus 4.7 holds attack success to roughly 0.1% on single attempts, and around 5–6% after 100 adaptive attempts."

**단발 0.1%와 적응형 100회 뒤 5~6%를 같이 읽어야 함.** 공격자가 재시도할 수 있는 표면이면 모델 계층만으로는 안 됨. 글의 첫 원칙이 환경 우선인 이유가 이 숫자임.

Claude Code의 auto mode 수치도 같은 성격임.

> "Claude Code auto mode catches roughly 83% of overeager behaviors before they execute."

각주가 나머지를 적음. 모델 기반 분류기로 명령 승인을 위임하는 것이고, **정상 명령을 막는 비율은 약 0.4%**로 낮지만 **과도한 행동의 약 17%는 통과**함.

## 격리 패턴 셋

제품마다 사용자가 감독할 수 있는 정도가 달라서 격리 강도를 다르게 뒀다는 구성임.

| 패턴 | 제품 | 격리 수단 | 경계 |
|---|---|---|---|
| 임시 컨테이너 | claude.ai 코드 실행 | **gVisor 컨테이너**, 격리된 인프라 | 파일시스템이 **세션마다 임시**. 폭발 반경이 작은 대신 할 수 있는 일도 적음 |
| 사람이 끼는 샌드박스 | Claude Code | **Seatbelt**(macOS) · **bubblewrap**(Linux) | 읽기 허용, 쓰기는 워크스페이스 안, **네트워크는 기본 거부** |
| 로컬 VM | Claude Cowork | **Apple Virtualization framework**(macOS) · **HCS**(Windows) | 워크스페이스와 `.claude` 폴더만 마운트. 그 밖의 호스트는 안 보임 |

Cowork의 마운트 모드는 셋임. **read-only, read-write, read-write-no-delete.** 자격증명은 호스트 키체인에 남고 VM에는 **세션 범위로 축소된 토큰**만 들어감. 경로 검증 전에 심볼릭 링크를 먼저 해석해야 탈출을 막는다는 구현 디테일도 글이 적어 둠.

Cowork 아키텍처는 **에이전트 루프를 게스트 안에 두는 full-VM 모드에서 루프는 호스트에 두고 코드 실행만 게스트에서 하는 host 모드로 옮겼음.** 신뢰성 때문이고, 코드 실행을 격리하는 보장은 유지된다고 적음.

## 승인 프롬프트 93%, 그래서 샌드박스로 갔음

이 글에서 가장 실무에 직접 걸리는 대목임.

> "Our telemetry showed users approved roughly 93% of permission prompts."

이게 자사 제품의 실제 사용 데이터라는 점이 중요함. 행동으로 처방이 바뀌었음. 승인 교육을 하거나 프롬프트 문구를 고치는 쪽이 아니라 **OS 샌드박스를 깔아 프롬프트가 뜰 일 자체를 줄였음.**

> "The result was an 84% reduction in permission prompts, and we open-sourced the runtime."

사용자 숙련도에 따라 패턴이 갈린다는 관찰도 붙음.

> "Experienced users auto-approve roughly twice as often as new users, but they also interrupt the agent mid-execution more frequently."

즉 숙련자는 사전 승인을 덜 보고 대신 **실행 중 개입으로 감독**함. 승인 프롬프트를 감독 수단으로 설계하면 숙련자에게는 이미 작동하지 않는다는 뜻으로 읽힘.

그리고 무인 실행에 대한 입장이 분명함.

> "Claude Code's reference devcontainer exists precisely so that the agent can run unattended, without per-action approvals."

**무인 실행을 금지하는 게 아니라 격리 안에서 하라는 설계임.**

## 놓쳤던 위험 넷

자기가 뚫린 사례를 절 제목에 `Risk we missed`로 달아 넷 적음. 이 저장소에서 쓸모가 큰 부분임.

**1. 신뢰 대화상자 앞에서 이미 실행됨.** Claude Code가 시작할 때 프로젝트 설정을 읽는 시점이 `Do you trust this folder?` 프롬프트보다 **앞**이었음. 공격자가 레포에 커밋해 둔 훅이 자동으로 실행됨.

> "Because Claude Code reads project settings during startup—before presenting the standard 'Do you trust this folder?' prompt—the hook the attacker had authored and committed would execute automatically."

수정은 **프로젝트 로컬 설정의 파싱과 실행을 신뢰 수락 뒤로 미루는 것**. 레포를 클론해서 에이전트를 띄우는 워크플로가 있으면 이 순서가 전부임.

**2. 사용자가 인젝션 경로가 됨.** 내부 연구자가 직원을 피싱했음. 평범한 협업 메일처럼 생긴 `can you run this for me?` 에 붙여 넣을 프롬프트가 들어 있었고, 그 프롬프트가 AWS 자격증명을 빼내는 지시였음.

> "Across 25 retries of that prompt, Claude completed the exfiltration 24 times."

**모델 계층이 통째로 무력했던 사례임.** 사용자가 직접 넣은 지시라 의도가 정당해 보였기 때문임. 글이 적은 결론은 이 상황에서 버티는 건 환경, 구체적으로 **POST를 막는 egress 제어**라는 것이고, 같은 자리에서 **`~/.aws` 를 애초에 손에 닿지 않게 두는 파일시스템 경계**도 같이 듦. 둘 다 이 연습에서 작동을 확인한 방어가 아니라 **막았을 방어**로 제시된 것임. 재시도 25회 중 24회가 유출로 끝났다.

**3. 허용된 도메인으로 유출됨.** 워크스페이스 파일에 심은 지시를 따라 Claude가 다른 파일을 읽고 **공격자의 API 키로 Anthropic Files API를 호출**했음. egress 프록시는 목적지가 `api.anthropic.com`인 것을 보고 통과시킴.

> "Claude, following the instructions, read other files in the workspace and called Anthropic's Files API using the attacker's key. The egress proxy checked the destination, saw api.anthropic.com, and let it through."

수정은 **VM 안에 방어용 man-in-the-middle 프록시를 두고 자사 API 트래픽을 가로채는 것**. VM에 발급된 세션 토큰만 받고 서버측 fetch 헤더는 막음. 도메인 allowlist가 유출을 막아주지 않는다는 사례라, 사내 프록시를 도메인 단위로만 짜 둔 경우에 그대로 걸림.

**4. VM이 EDR까지 막았음.** 엔드포인트 탐지 소프트웨어 입장에서 Cowork는 **불투명한 하이퍼바이저 프로세스**라 게스트 안을 못 봄.

> "From the EDR's perspective, Claude Cowork is an opaque hypervisor process. It can't inspect the guest."

현재 완화책은 **pull 방식 OTLP 내보내기**로 관리자가 사후에 이벤트 로그를 받아 가게 하는 것임. 격리를 올리면 관측이 떨어진다는 교환이 실제로 발생한 기록으로 읽을 것.

## 에이전트가 읽는 것을 신뢰하는 문제

MCP와 플러그인, 웹 검색처럼 컨텍스트로 콘텐츠를 밀어 넣는 경로를 다루는 절임. 두 줄이 핵심임.

> "Remote versus local is more important than it seems. A locally installed tool is auditable."

> "Tool output is an attack surface even when the tool is trusted."

**툴을 신뢰한다는 것과 툴의 출력을 신뢰한다는 것이 다른 판단**이라는 정리임. 원격 MCP 서버는 어느 시점에 내용이 바뀌어도 로컬에서 감사할 방법이 없으니, 로컬 설치와 원격 연결을 같은 신뢰 등급으로 취급하면 안 됨.

## 앞으로 걸린다고 적은 것

- **영속 메모리 오염.** 세션을 넘어 남는 상태(제품 메모리, `CLAUDE.md`, 마운트된 워크스페이스)가 새 인젝션 표면이 됨
- **멀티 에이전트 신뢰 상승.** 에이전트마다 신뢰 등급을 다르게 주는 것과 인젝션 경로가 늘어나는 것 사이의 교환
- **에이전트 신원.** 자격증명은 호스트 키체인에 두고 VM에는 세션 범위 토큰만 주는 방식을 기본으로 적음

## 글이 내세운 원칙 셋

1. `Design for containment at the environment layer first, then steer behavior at the model layer.`
2. `Match isolation strength to the user's capacity for oversight.`
3. `Be wary of custom components.`

셋째가 자주 빠지는 항목임. 하이퍼바이저·seccomp·컨테이너 런타임처럼 오래 두들겨 맞은 1차 도구를 쓰고, 격리 경계를 직접 구현하지 말라는 것임.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 같은 결론에 다른 근거. 그쪽은 게임 실험의 판정 정확도 66.3%, 이쪽은 자사 제품 텔레메트리의 **승인율 93%**. 두 수치는 측정 대상이 달라 섞을 수 없지만, "승인 정확도를 올리지 말고 승인 대상을 줄여라"는 처방이 양쪽에서 같게 나옴 |
| [OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](./2026-08-07-openai-hugging-face-agent-incident.md) | 파일 쓰기 권한 하나가 통신 채널이 된 사례. 이 글의 "툴 출력은 툴을 신뢰해도 공격 표면"과 같은 표면 |
| [Claude Cowork](../practices/2026-08-10-claude-cowork.md) | 같은 제품의 격리를 다루는데 **서술이 갈림.** 아래 짚어야 할 것 참고 |
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | Apple이 붙인 파일시스템 감시 계층과 `--unsafe-always-allow-all-agents`가 정확히 이 글의 교환. 켜야 도는 감시와 장기 권한 부여 |
| [MCP 2026-07-28 스펙](../infra/2026-07-28-mcp-stateless-spec.md) | 원격 대 로컬 서버의 신뢰 차이. 스펙이 전송을 HTTP로 통일하는 방향과 같이 읽을 것 |
| [Memory Engineer](../practices/2026-08-02-memory-engineer.md) | 영속 메모리가 인젝션 표면이 된다는 항목. 메모리를 설계할 때 "무엇을 기억하나"와 "누가 거기 쓸 수 있나"가 같이 걸림 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | Opus 4.7이 벤치마크 기준 모델로 등장함. 그쪽 문서의 Stripe 사례와 같은 세대이고, 이 글의 모델 쪽 수치가 지금 기준으로 낡은 이유 |

## 짚어야 할 것

- **Cowork의 격리 서술이 이 저장소 안에서 갈림.** 이 글(2026-05-25)은 Cowork를 **로컬 VM**(Apple Virtualization framework / HCS)으로 적고 자격증명이 호스트 키체인에 남는다고 함. 반면 [Claude Cowork](../practices/2026-08-10-claude-cowork.md)가 공식 문서(2026-08-10 기준)에서 확보한 서술은 "내 컴퓨터와 네트워크로부터 분리된 **Anthropic 서버의 격리 환경**"임. 로컬 파일을 다루는 데스크톱 경로와 서버측 실행 경로가 따로 있는 것으로 보이지만, **어느 조건에서 어느 쪽이 쓰이는지는 이번에 확인할 수 없었음.** 둘 중 하나가 틀렸다고 단정하지 말고, Cowork에 민감한 폴더를 붙일 때는 자기 환경에서 실제 실행 위치를 확인하는 편이 안전함
- **자사 보고임.** 뚫린 사례를 공개한 것은 값어치가 크지만, 공개 범위를 고른 쪽도 Anthropic임. 놓친 위험이 넷이라는 것이 전부가 넷이라는 뜻은 아님
- **모델 쪽 수치는 Opus 4.7 기준이고 세대가 지남.** 0.1% / 5~6%와 auto mode의 83% / 17% / 0.4%를 지금 모델에 그대로 옮기면 안 됨. 재확인 경로는 아래 유효기간에 적음
- **auto mode 분류기 수치의 측정 방법을 확인하지 못했음.** 과도한 행동(overeager behaviors)의 정의와 표본 구성이 글에 없어서, 83%와 17%를 다른 하네스의 자동 승인과 비교할 수 없음
- **84% 감소는 Anthropic 자사 측정임.** 프롬프트 수가 줄어든 것이고 위험 감소를 직접 측정한 수치가 아님. 샌드박스가 막는 것과 프롬프트가 묻던 것이 같은 집합인지는 글이 밝히지 않음
- 25회 중 24회라는 피싱 결과는 **내부 레드팀 한 건의 프롬프트에 대한 재시도 수**임. 공격 성공률 일반값으로 읽을 표본이 아님

## 유효기간

**2026-10-02 확인 기준이고 글 자체는 2026-05-25 발행임.** 사고 서술과 설계 패턴은 시점이 박힌 기록이라 낡지 않지만, 모델 쪽 수치와 제품 구성은 움직임. 다시 볼 때 확인할 것 셋. **Gray Swan 벤치마크 수치가 현행 모델로 갱신됐는지**, **Claude Code auto mode의 분류기 수치가 다시 공개됐는지**, 그리고 **Cowork의 실행 위치 서술이 공식 문서와 맞춰졌는지**. 셋째는 위 짚어야 할 것의 첫 항목이 풀리는 지점임.
