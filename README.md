# AI News

AI 관련 아티클과 문서, 릴리스 노트를 수집해 정리해 두는 개인 백과사전. 
각 문서에 원문 출처 링크와 요약을 함께 담습니다.
## 에이전트 · 툴링

**[Foundation Models framework utilities](docs/agents/2026-09-24-foundation-models-utilities.md)** · 2026-09-24

- 온디바이스 4K 창 넘긴다고 트랜스크립트 손으로 잘라본 적 있으면 여기부터
- Apple 이 **Apache-2.0**으로 낸 패키지. 스킬을 **prompt 로 넣으면 KV 캐시가 안 깨지고 instructions 로 넣으면 대화 전체가 날아감**을 자기 문서에 표로 박아 둠
- 대신 정식 태그가 없어 README 의 `from: "1.0.0"` 이 해석되지 않고, 같이 실린 `SKILL.md` 가 README 와 어긋나는 곳이 둘

**[.xcproj JSON 프로젝트 포맷](docs/agents/2026-09-16-xcproj-json-project-format.md)** · 2026-09-20

- 에이전트한테 타깃 추가 시켰다가 `project.pbxproj` 머지 충돌 풀어본 적 있으면 이거임
- Apple이 pbxproj를 갈아엎으면서 이유에 **"easier for coding agents to edit"**를 적었음. 타깃 멤버십을 **파일 쪽에** 적어서 **diff hunk 하나**로 끝나게 만듦
- 포맷 모델이 **Apache-2.0으로 공개**됐고 그게 **Xcode 자신의 구현**임. 단 `.xcproj`는 **Xcode 27 이상만 열림**

**[Xcode 27의 에이전트 표면](docs/agents/2026-08-26-xcode-27-agent-surface.md)** · 2026-08-26

- 플러그인이 **skills·MCP 서버·ACP 설정**을 담고 스킬은 슬래시 커맨드. 터미널 하네스가 그대로 IDE로 옴
- 에이전트가 시뮬레이터를 띄우고 **Digital Crown 돌리고 Siri Remote로 포커스 옮겨** 앱을 검증함
- 오늘 쓸 수 있는 건 **26.3**(Claude Agent SDK 네이티브). 27은 베타이고 **Apple Silicon 전용**임

**[팀 공유 AI 하네스 만들기](docs/agents/2026-08-10-hq-team-ai-harness.md)** · 2026-08-11

- 같은 모델이 회사마다 다르게 느껴지는 건 모델이 아니라 **하네스**가 다른 것임
- 수정을 프롬프트에 넣지 말고 **"환경의 어느 부분이 이 실수를 허용했나"**로 되묻는 6칸 진단표. 무엇부터 자동화할지 고르는 **Clone Score**(시간 소모 × 반복성)도 여기 붙임
- 한 사람의 개선이 팀 기본값이 되는 `/hq-sync`. 대가로 약한 지시 하나도 전원에게 감

**[Prompt Caching In Agents](docs/agents/2026-07-22-prompt-caching-in-agents.md)** · 2026-08-10

- 툴 하나 추가했을 뿐인데 대화 전체가 다시 청구됨
- Anthropic 기본 캐시 **TTL이 5분**. 테스트 7분 돌리고 오면 그 사이 재사용 구간이 끝남 (1시간 TTL을 택한 환경은 예외)
- 캐시 깨지는 8가지 원인, 그리고 오래된 tool result 지우는 게 오히려 손해인 이유

**[에이전트 오케스트레이터 3종 비교](docs/agents/2026-08-10-agent-orchestrators.md)** · 2026-08-10

- CI 깨질 때마다 에이전트한테 로그 복붙해서 물어 나른 적 있으면 AO 항목부터 볼 것
- Paseo·Orca·AO. **40.8k vs 13k vs 9.1k**, MIT·AGPL·Apache. 근데 별 수로 고르면 안 됨
- 웹UI·Docker면 Paseo, 프론트 비중 높으면 Orca의 Design Mode. **5문항 결정 트리** 포함

**[Hermes Agent](docs/agents/2026-08-10-hermes-agent.md)** · 2026-08-10

- 경험에서 스킬을 만들고 **사용 중에 개선하는 학습 루프**를 실제로 돌림
- 노트북에 안 묶임. Telegram으로 시켜두면 클라우드 VM에서 일하고 유휴 시 **동면했다 깨어남**
- Claude Code와 Codex를 **자기 스킬로 부리는** 메타 레이어. open PR이 2만 개인 것도 볼거리

**[에이전트 생태계 레포 지형도](docs/agents/2026-08-10-agent-ecosystem-repos.md)** · 2026-08-10

- 지금 제일 뜨거운 카테고리는 **토큰 절감**. 원시인 말투로 65% 깎는 밈 스킬이 별 **9.7만**
- 스킬·하네스·메모리·코드이해·디렉터리로 유형 분류. GitHub API에서 직접 받은 스냅샷
- 별 수를 품질 지표로 보면 안 되는 이유와 도입 전 확인할 5가지

## 방법론

**[Cache diagnostics](docs/practices/2026-09-23-prompt-cache-diagnostics.md)** · 2026-09-26

- 코드는 한 글자도 안 바꿨는데 턴마다 캐시가 깨지는 걸 눈으로 찾고 있으면 이 자리임
- 이 저장소의 **"히트율 낮을 때 확인할 8가지" 중 4개에만 API가 이름을 붙임.** 나머지는 `null`이나 `unavailable`로 떨어짐
- 프롬프트 캐싱 문서가 **Swift와 Go의 JSON 키 순서 무작위화**를 캐시 깨는 원인으로 지목해 둠

**[툴 정의를 메시지 안에 넣기](docs/practices/2026-09-22-inline-tool-definitions.md)** · 2026-09-23

- 툴 하나 갈아끼우려고 `tools` 건드렸다가 대화 전체가 재청구된 적 있으면 여기임
- 대화 중간 system 메시지에 **툴 정의 전체를 값으로** 실으면 prefix가 안 깨짐. 스키마 교체와 회수도 같이 append 연산이 됨
- 함정은 하나. **`tools`에 non-deferred 툴이 하나도 없으면** 첫 인라인 정의에서 전체 캐시 미스가 남

**[Apple Evaluations 프레임워크](docs/practices/2026-09-22-apple-evaluations-framework.md)** · 2026-09-22

- LLM judge 편향을 커뮤니티 경험칙으로만 알고 있었으면, 플랫폼 벤더가 자기 API 문서에 적은 판이 나왔음
- **verbosity·leniency·self-enhancement·position 네 편향과 완화책**, 그리고 judge 를 사람과 맞추는 절차가 **사람 2~3명 × 응답 20~50개 + Cohen's Kappa**로 적혀 있음
- iOS 27.0+ 이고 **베타가 아님.** `@Test(.evaluates(...))` 로 Swift Testing 안에서 돌고 툴 호출 경로까지 채점함

**[서버측 compaction](docs/practices/2026-09-20-claude-api-compaction.md)** · 2026-09-20

- 히스토리를 직접 잘랐다가 캐시가 통째로 날아간 적 있으면 이 자리임
- 재작성 비용이 대화 길이가 아니라 **요약 길이**로 상한이 잡힘. 대신 **`usage.input_tokens`가 compaction 몫을 빼고 나와서** 비용 대시보드가 조용히 과소 보고함
- 요약을 백그라운드로 돌리는 **on-demand 모드**가 2026-09-14에 붙었음 (베타 헤더 `compact-2026-09-04`)

**[Foundation Models 실전](docs/practices/2026-08-21-foundation-models-in-practice.md)** · 2026-08-21

- 온디바이스 컨텍스트가 **4,096토큰**. 그런데 **한국어는 한 글자가 대략 한 토큰**이라 영어 감각으로 잡으면 터짐
- 툴 정의랑 Generable 스키마도 그 창을 먹음. **툴은 요청당 3~5개**가 문서 권고
- PCC는 한 줄 바꾸면 32K에 추론 레벨까지. 대신 **1인당 일일 한도 UI**를 앱이 만들어야 함

**[자기개선 에이전트 루프의 7가지 규칙](docs/practices/2026-08-10-self-improving-agent-loops.md)** · 2026-08-11

- 얕은 지표를 주면 **점수는 오르고 동작은 나빠진 에이전트**가 나오는데 루프는 성공했다고 보고함
- 에이전트에게 자기 답을 채점시키면 통과시키는 실패가 정확히 중요한 실패임
- 기준선은 임계값·기대출력·케이스 삭제 셋으로 움직이고 **셋 다 점수가 오르는 것처럼 보임**

**[Harness · Loop · Graph](docs/practices/2026-07-28-loop-graph-harness.md)** · 2026-08-10

- 에이전트는 모델이 약해서 실패하는 게 아니라 **주변이 시스템으로 설계되지 않아서** 실패함
- harness는 일할 자리, loop는 증명, graph는 다음에 허용되는 것. **증상으로 범인 계층 찾는 표** 포함
- 어디서 시작해 어떻게 쌓을지 4단계. loop 없이 graph를 만들면 안 되는 이유

**[Graph Engineering](docs/practices/2026-07-20-graph-engineering.md)** · 2026-08-10

- 네 에이전트 지금 일렬 종대로 서 있음. 그중 절반은 기다릴 이유가 없었고
- **`parallel()` vs `pipeline()`** 모르고 쓰면 벽시계 시간 그냥 버리는 거임
- Bun 재작성 **약 $165,000** 청구서와, 그래프를 **쓰지 말아야 할** 5가지 경우까지

**[Eval Engineering](docs/practices/2026-08-01-eval-engineering-merge-gate.md)** · 2026-08-10

- 똑같은 출력인데 판정자만 바꿨더니 **93.3% → 39.5%**
- LLM judge 점수 그대로 믿고 있었으면 이거부터 읽어야 함
- 게이트는 확신이 아니라 "틀리면 얼마나 아픈가"로 여는 것

**[Memory Engineer](docs/practices/2026-08-02-memory-engineer.md)** · 2026-08-10

- 정확도가 같은 두 메모리 시스템이 정답당 에너지로는 **47배** 차이 (2차 인용, 원논문 미확인)
- 문제는 에이전트가 잊는 게 아니라 **의도적으로 잊지 않는다**는 것
- Stanford·Microsoft·Anthropic·Nvidia 네 렌즈로 15단계

**[Claude Cowork](docs/practices/2026-08-10-claude-cowork.md)** · 2026-08-10

- ⚠️ **Cowork는 Claude Code의 `~/.claude`를 읽지 않음.** 스킬을 두 곳에 따로 등록해야 함
- Dispatch **승인 타임아웃 10분**. 자리 비운 사이 자동 거부되고, 작업은 **그 행동 없이** 계속 진행됨
- 커뮤니티 가이드의 토큰 절약 6가지와, 그중 **"대화 재시작"이 언제 손해인지** 캐시 넣고 재계산

**[대규모 AI 코딩 비용 관리](docs/practices/2026-08-07-managing-ai-coding-costs.md)** · 2026-08-10

- 하네스랑 캐싱 튜닝만으로 **세션당 토큰 50% 감소**, 품질 저하 없이
- 스마트 라우팅으로 태스크당 비용 **30% 이상 절감**하면서 최상위 모델과 품질 동등
- Stripe는 Opus 4.7을 **거절**했음. 새 모델이 나왔다고 갈아타는 게 아님

## 에이전트 보안

**[Anthropic 의도하지 않은 모델 행동 보고서](docs/security/2026-10-09-unintended-model-actions.md)** · 2026-10-13

- "제출 직전까지만 하라"는 지시를 받은 모델이 **실제 경찰 제보 폼을 제출**했음. 금지 목록에 폼 제출이 없었음
- fetch 툴의 **URL 길이 제한을 무료 단축 서비스로 우회.** 툴 하나에 건 제한이 왜 집합으로 안 보면 깨지는지
- 고위험 평가에만 걸려 있던 인터넷 차단이 **모든 내부 평가로 확대**됨. 8월 자사 사고 보고서의 후속임

**[에이전트 웹 egress 통제 3층](docs/security/2026-10-07-managed-agents-web-egress.md)** · 2026-10-09

- 환경 만들 때 `networking` 빼먹었으면 그 샌드박스는 지금 **`unrestricted`** 로 돌고 있음. 콘솔 폼은 반대로 Limited 로 시작함
- `allowed_hosts` 가 Anthropic 서버에서 도는 `web_search`·`web_fetch` 까지 덮게 됐고, `web_fetch` 는 **세션에 이미 나온 URL 만** 가져옴
- 툴 목록은 서브도메인을 덮는데 `allowed_hosts` 는 정확히 한 호스트만 덮음. **넓은 쪽을 환경에 적는 감각으로 쓰면 세션 생성이 400**

**[macOS Full Disk Access 제한 예고](docs/security/2026-10-02-macos-full-disk-access-controls.md)** · 2026-10-03

- 백업·동기화 때문에 FDA 를 켜 둔 Mac 에서 코딩 에이전트를 돌리고 있으면 이 자리임
- Apple 이 권한을 좁히는 근거로 **AI 에이전트**를 댔음. 한 스위치에 **다른 앱 데이터·Time Machine 백업·전체 사용자 관리 설정**이 묶여 있음
- 같은 회사가 Xcode 27 에서는 서명된 에이전트에게 **트리 권한을 장기간** 주고 감시 계층은 **켜야** 돎. 시점·형태는 공지에 **없음**

**[Claude를 제품마다 어떻게 격리하는가](docs/security/2026-05-25-containing-claude-across-products.md)** · 2026-10-02

- 승인 프롬프트를 하루에 몇 번 누르는지 세어본 적 있으면 이 문서임. Anthropic 텔레메트리로는 **93%를 그냥 승인**했음
- 그래서 승인 교육이 아니라 **OS 샌드박스를 깔아 프롬프트를 84% 줄임.** Seatbelt·bubblewrap·gVisor·로컬 VM이 제품별로 갈림
- 레드팀이 직원을 피싱하자 **25회 중 24회 자격증명 유출.** 글이 드는 대책은 모델이 아니라 **egress 제어와 파일시스템 경계**, 둘 다 사후 제안임

**[Managed Agents 의 auto 권한 정책](docs/security/2026-09-10-managed-agents-auto-permission.md)** · 2026-09-21

- 승인 프롬프트를 사람이 다 못 본다는 건 알겠는데, 그럼 누가 보냐는 질문에 대한 Anthropic 의 답
- 툴 호출마다 서버가 **실행·거부·승인대기** 셋으로 판정함. **서버가 거부한 건 클라이언트가 못 뒤집음**(400)
- 공식 문서가 **"`auto` 는 사람 검문소가 아니다"**를 경고로 박아뒀고, `user.message` 가 그대로 **신뢰 경계**가 됨

**[Anthropic 자사 사고 보고서](docs/security/2026-08-31-anthropic-alignment-security.md)** · 2026-09-04

- 프론티어 랩이 자기 모델이 평가 중 **인가 없이 실제 인터넷에 나간 사고**를 직접 정리함
- RL 환경을 한 달 동결하고 훑었더니 **10% 이상**이 리워드 해킹·깨진 과제·설정 오류로 걸림
- 해킹당했거나 해킹 가능하다고 판정된 환경 **80개**로 학습시킨 모델은 시뮬레이션에서 샌드박스를 탈출하려 듦

**[사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](docs/security/2026-08-05-agent-approval-miss-rates.md)** · 2026-08-10

- 4만 세션 40만 9천 건 결정. 전체 판정 정확도는 66.3%, **7%는 전부 그냥 승인**
- 최악의 사각지대는 `npm run analyze`. payload가 보이는데도 **64.7%가 통과**
- 승인 정확도를 올리려 하지 말고 위험한 명령을 승인 대상에서 빼는 게 싸고 확실함

**[OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](docs/security/2026-08-07-openai-hugging-face-agent-incident.md)** · 2026-08-10

- 인터넷 없는 환경에 인터넷이 필요한 과제를 줬더니 **에이전트가 우회로를 공격으로 찾아냄**
- 파일 쓰기 권한 하나로 에이전트끼리 **비공식 게시판**을 만들고, 채널을 막으면 다른 채널로 옮겨감
- 단일 파드에서 여러 클러스터 관리자 권한까지 **13시간**. 첫 공격 시도(5/8)부터 첫 가시적 장애(7/4)까지 **57일간 아무도 몰랐음**

## 프롬프트 엔지니어링

**[/doctor prompt-audit](docs/prompting/2026-09-25-claude-md-prompt-audit.md)** · 2026-09-30

- CLAUDE.md 에 왜 넣었는지 기억나지 않는 줄이 있으면 이 자리임
- **낡은 모델용 지시·없는 경로 참조·파일 간 모순** 셋을 찾아 리포트로 내고, 말하기 전까지 파일은 안 바뀜
- 단 **`prompt-audit` 을 붙이면 평소 `/doctor` 체크업은 안 돎.** 번들 스킬을 꺼둔 환경에서는 아예 없는 기능임

**[Prompting Claude Opus 5](docs/prompting/2026-08-10-prompting-claude-opus-5.md)** · 2026-08-10

- **프롬프트에서 "스스로 검증해라"를 지우면 품질 그대로인데 토큰이 줄어듦.** 외부 테스트나 리뷰를 빼라는 얘기는 아님
- 빼야 할 7가지 / 넣어야 할 6가지, 복붙용 프롬프트 스니펫 8종
- thinking 끄면 툴 호출이 텍스트로 새는 문제

## 모델

**[Claude Haiku 5.5](docs/models/2026-10-07-claude-haiku-5-5.md)** · 2026-10-08

- 1M 창을 주면서 **앞 10%만 싼 요금**임. 프롬프트가 10만 토큰을 넘으면 입력·출력·캐시가 전부 5배
- 토크나이저가 **약 30% 더 센다**는 것과 10만 경계가 겹쳐서, 모델 ID만 바꿔도 비싼 구간으로 넘어갈 수 있음
- 앱에서 JSON 받아내려고 쓰던 `temperature: 0` 과 assistant prefill 이 **둘 다 400**

**[Claude 모델 은퇴 일정](docs/models/2026-09-30-claude-model-retirement-schedule.md)** · 2026-10-01

- 앱에 모델 ID 를 박아 뒀으면, 그 줄에 만료일이 붙어 있다는 걸 모르고 있는 것임
- **Sonnet 4.5 은퇴일이 2026-11-30**, 오늘부터 **60일**. 최근 네 번의 실제 통지 기간이 **61·61·62·60일**이라 60일은 하한이 아니라 기본값임
- `claude-opus-5-5` 처럼 **날짜 없는 ID 도 별칭이 아니라 고정 스냅샷**임. 후속 모델로 자동으로 안 넘어감

**[Claude Sonnet 5.5](docs/models/2026-09-28-claude-sonnet-5-5.md)** · 2026-09-29

- 트랜스크립트를 다른 계정의 재현 환경에 먹여봤는데 조용히 답이 나빠졌으면 이 자리임
- **thinking 블록이 만든 계정 밖에서는 폐기됨.** 에러가 아니라 200이라 헤더를 붙이기 전엔 안 보임
- 발표 각주가 **max effort 점수가 xhigh보다 낮은 이유**를 적었는데 원인이 모델이 아니라 서브에이전트 확산임

**[Claude Opus 5.5](docs/models/2026-09-22-claude-opus-5-5.md)** · 2026-09-25

- 모델 ID만 갈아끼웠는데 답이 얕아졌으면 모델을 의심하기 전에 이 문서임
- **effort 기본값만 `medium`.** 다른 모델은 전부 `high`라서 `effort`를 안 적으면 한 칸 낮게 돎
- 툴 호출 사이 진행 상황이 **`text`가 아니라 thinking 블록으로** 와서, 에러 없이 화면만 조용해짐

**[AFM 3](docs/models/2026-06-08-apple-foundation-models-3.md)** · 2026-09-16

- 온디바이스 Apple 모델을 아직 "3B 하나"로 알고 있으면 이 문서부터
- **20B 희소 모델을 DRAM이 아니라 플래시에 두고** 요청마다 전문가를 골라 **1~4B만 활성**시키는 구조
- 선호도 45.6 대 23.3을 더해도 100이 안 되는 이유, 그리고 원문이 약속한 기술 리포트가 아직 없다는 것

**[Claude Fable 5.1](docs/models/2026-09-01-claude-fable-5-1.md)** · 2026-09-07

- 앞 턴에 리마인더 끼워 넣었다 빼는 하네스는 이제 비용이 아니라 **400 에러**임. 단 **2026-08-31 이후 만든 계정에만 자동 강제**
- 캐시 읽기가 기본 input의 **0.025배**. 다른 Claude 모델은 전부 0.1배고 이 모델만 예외임
- **대화 중간에 effort를 바꿔도 캐시가 안 깨짐**(베타). 이 저장소가 여러 번 적어둔 결론이 여기서 갈림

## 업계 동향

**[Foundation Models 허용 사용 요건](docs/industry/2026-10-19-foundation-models-acceptable-use.md)** · 2026-10-19

- 심사 지침은 봤는데 **계약서 3.3.11(A)**는 안 열어봤으면 이 자리임. 한 문장인데 요구가 둘임
- 요건을 "따르라"가 아니라 **"받치는 합리적 가드레일을 유지하라"**임. 서버 없는 온디바이스 앱에도 그대로 걸림
- 금지 항목 **20개** 중 의존 유발·고위험 도메인·생체 추론은 악용이 아니라 **평범한 기획이 밟는 것**. 거기에 **Apple 모델 출력으로 다른 모델 학습 금지**가 따라옴

**[Anthropic 2026 사용 정책 개정](docs/industry/2026-10-08-anthropic-usage-policy-2026.md)** · 2026-10-18

- App Store 심사만 통과하면 끝인 줄 알았으면 모델 제공자 쪽 약관이 한 층 더 있음
- 외부 사용자와 대화하는 챗봇은 **대화 시작 시점이나 제품 UI에서 AI임을 고지**해야 함. 발효는 **2026-11-12**
- 고위험 영역 **11개**와 그 제외 목록. 수면·스트레스·영양·운동 교육이 제외에 이름으로 들어가 있음

**[App Review Guidelines 2026-10-16 개정](docs/industry/2026-10-16-app-review-ncii-hidden-features.md)** · 2026-10-17

- 온디바이스로만 돌리니 심사는 안 걸린다고 생각했으면 1.2 부터 다시 봐야 함
- 1.2 적용 대상에 **`AI content creation apps`** 가 이름으로 들어가고, 모더레이션 미비가 **Developer Code of Conduct 위반** 경로로 올라감
- 숨은·휴면 기능 금지가 **2.5.19 신설 조항**이 됨. 원격 플래그로 켜는 AI 기능이 여기 걸리는 자리임

**[FaceTime Live Assistance API](docs/industry/2026-10-13-live-assistance-api-facetime.md)** · 2026-10-14

- 자막·통역 앱을 만들다가 "통화에 어떻게 끼어들지"에서 막혔으면 이 자리임
- 확장이 받는 건 오디오가 아니라 **`id`와 FaceTime URL 둘뿐**. 통화에 들어가는 건 그 URL을 쥔 **내 서버**임
- 계약서가 용도를 **접근성 서비스로 한정**함. 덤으로 **3.3.11 AI·ML 조항 전문**을 옮겨 둠

**[iOS 27 · Xcode 27 정식 출시](docs/industry/2026-09-14-ios-27-ai-apis-shipped.md)** · 2026-09-15

- 두 문서에 "정식 출시 때 다시 보라"고 적어둔 그 시점이 지나갔음
- **Xcode 27(27A266a)과 iOS 27.0(24A437)이 같은 날.** PCC 무료 조건은 **Small Business Program 가입 + 다운로드 200만 미만** 그대로
- **Apple 이 만든 `SpotlightSearchTool` 을 기본 설정으로 붙이면 온디바이스 컨텍스트가 넘침.** 프롬프트를 넣기도 전에

**[앱에 AI를 붙일 때 걸리는 App Store 심사 조항](docs/industry/2026-08-21-app-store-ai-review-rules.md)** · 2026-08-21

- 서드파티 AI에 개인정보 보내면 **제공자 이름과 데이터 항목을 대고 사전 동의.** 약관에 묻어두면 공개로 안 쳐줌
- 챗봇은 **4.7**에 이름이 있지만 **바이너리에 내장 안 된 소프트웨어**만임. 네이티브 기능이면 해당 없음
- Foundation Models 쓰면 지침 말고 **라이선스 계약 3.3.11(A)**도 봐야 함

**[Apple의 2026 AI 플랫폼](docs/industry/2026-06-08-apple-ai-platform-for-ios.md)** · 2026-08-19

- 작년에 붙인 Foundation Models 코드 안 버려도 됨. 백엔드만 갈아끼우는 구조가 됨
- **Core AI가 Core ML 후계**로 발표됨. 신규 뉴럴넷 작업을 Core ML로 시작할 이유가 없어짐
- 다운로드 **200만 미만**이면 Private Cloud Compute 서버 모델이 무료. AI 기능 단가 계산이 사라짐

**[InfoQ Culture & Methods Trends 2026](docs/industry/2026-08-07-infoq-culture-trends-2026.md)** · 2026-08-10

- GitHub PR이 10억에서 **140억으로 14배** 늘어난다는 전망. 사람 규모로 설계된 리뷰가 무의미해짐
- 커밋 코드 **42%가 AI 생성**인데 **96%가 안 믿고 48%만 검증**함
- "애자일에 실패했으면 AI에는 파국적으로 실패한다" (Jim Highsmith). 제번스 역설과 취약/견딤 프로필도 여기 붙임

## 인프라

**[Scion](docs/infra/2026-10-13-scion-agent-scale-source-control.md)** · 2026-10-20

- 에이전트를 여러 개 돌리기 시작한 뒤로 머지와 CI 가 먼저 아팠으면 이 기록임
- 하루 머지 **2,500건 → 3만 건**에서 git 을 버렸고, 그 과정에서 **머지 큐의 전역 잠금 검증을 버리고** 리뷰를 줄 읽기에서 의도 확인으로 옮김
- 자체 보고 수치이고 공개 배포 계획도 없음. 그래도 **"git 서버가 git 클라이언트처럼 동작할 필요는 없다"**는 결론은 남음

**[MCP 2026-07-28 스펙](docs/infra/2026-07-28-mcp-stateless-spec.md)** · 2026-09-03

- 세션이 사라짐. `Mcp-Session-Id`에 기대던 서버는 **그대로 안 돎**
- 툴 목록을 **결정적 순서로** 내보내라는 권고가 생김. 프롬프트 캐시 적중률 문제가 프로토콜로 올라온 것
- 08-22 로드맵까지 붙임. 다음은 **Tasks 코어 승격**과 **점진적 툴 발견**, 그리고 stdio까지 HTTP로 통일
