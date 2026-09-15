# 점검 결정 기록

`Daily audit` 워크플로가 매 실행마다 같은 문서를 다시 저울질하지 않도록, **사람이 내린
판단과 그 근거**를 남기는 파일임. 점검은 이 파일을 읽고, 여기 적힌 결정을 뒤집는 제안을
하지 않는다.

## 쓰는 법

- 사람이 제거·병합 여부를 결정할 때마다 아래 표에 한 줄 추가한다
- **근거를 적는다.** 결론만 있으면 다음에 같은 논쟁이 반복됨
- 결정이 바뀌면 줄을 지우지 말고 새 줄을 추가하고 날짜로 구분한다. 왜 뒤집혔는지가 남아야 함
- **뒤집을 때는 이전 줄의 근거 칸 맨 앞에 `(대체됨 → YYYY-MM-DD)` 를 붙인다.** 표시를 안 하면
  같은 문서에 상반된 줄이 둘 남고, 점검은 어느 쪽을 따라야 할지 알 수 없게 됨
- **한 문서에 줄이 여럿이면 날짜가 가장 최근인 줄이 유효한 결정이다.** 이전 줄은 기록용이고
  판단 근거로 쓰지 않는다
- 문서가 실제로 제거되면 그 줄은 남겨둔다. 되살리자는 제안을 막는 역할도 함

## 제거하지 않기로 한 문서

| 문서 | 결정일 | 근거 |
|---|---|---|
| `practices/2026-07-28-loop-graph-harness.md` | 2026-09-07 | "본문 스스로 근거를 무효화" 기준에 걸리지만, 무효화된 것은 30초 요약표와 증상별 진단표 **둘의 출처 귀속**이지 내용 전체가 아님. 계층 정의·loop 7요소·값비싼 실수 5가지·프로덕션 체크리스트는 원문 산문에서 확보한 것임. 무엇이 원문이고 무엇이 정리자 것인지 갈라 적은 것은 이 저장소가 요구하는 태도에 부합함. 6개 문서가 링크하는 허브이기도 함 |
| `agents/2026-08-10-agent-ecosystem-repos.md` | 2026-09-07 | 별 수 스냅샷이 낡았으나 수치 노후는 제거 사유가 아님. 문서 자신이 스냅샷 시점과 "표를 갱신하지 말고 새로 조사하라"는 재조사 방법을 적어 둠 |
| `agents/2026-08-10-hermes-agent.md` | 2026-09-07 | 위와 같음. 릴리스가 주 단위로 나온다는 것과 릴리스 노트를 직접 보라는 안내가 본문에 있음 |
| `agents/2026-08-10-agent-orchestrators.md` | 2026-09-07 | 위와 같음. 3종 비교 수치가 2026-08-10 기준이라고 명시돼 있음 |

## 병합하지 않기로 한 쌍

| 쌍 | 결정일 | 근거 |
|---|---|---|
| `industry/2026-06-08-apple-ai-platform-for-ios.md` + `practices/2026-08-21-foundation-models-in-practice.md` | 2026-09-07 | 겹치는 것은 PCC 무료 구간 한 항목뿐이고 그것도 경고와 상대경로 링크로 이미 해소돼 있음. 발표 쪽에만 있는 것(Core AI의 Core ML 후계 선언, MLX 수치)과 실전 쪽에만 있는 것(4K 컨텍스트와 한국어 토크나이저, 툴 3~5개 권고)이 서로 안 겹침 |
| `practices/2026-07-20-graph-engineering.md` + `practices/2026-07-28-loop-graph-harness.md` | 2026-09-07 | 원리(계층 지도) 대 운영(구현과 비용). 층위가 다른 조합임 |
| `practices/2026-08-01-eval-engineering-merge-gate.md` + `practices/2026-08-10-self-improving-agent-loops.md` | 2026-09-07 | 두 문서가 "경로를 채점하라" 대 "경로 말고 동작을 채점하라"로 정면 충돌하고, 서로를 링크하며 역할을 갈라 둠. **그 긴장이 값어치임** |
| `security/2026-08-07-openai-hugging-face-agent-incident.md` + `security/2026-08-31-anthropic-alignment-security.md` | 2026-09-07 | 같은 계열 사건이지만 한쪽은 외부 재구성, 다른 쪽은 랩의 자체 보고임. 원인 서술의 깊이 차이가 그 자체로 읽을거리 |
| `agents/2026-08-10-agent-ecosystem-repos.md` + `agents/2026-08-10-agent-orchestrators.md` | 2026-09-07 | 지형도 대 심층 비교. 지형도가 해당 항목에서 비교 문서로 넘기는 구조가 이미 서 있음 |
| `practices/2026-08-10-claude-cowork.md` + `agents/2026-08-10-hq-team-ai-harness.md` | 2026-09-07 | 후자가 전자를 "폴더를 버리라는 주장과의 정면 대비"로 인용함. 충돌 자체가 값어치라 병합 대상이 아님 |

## 제거한 문서

병합으로 사라진 문서도 여기 적는다. 원문 출처를 다시 만났을 때 어느 문서에 들어갔는지 알아야
같은 내용을 새로 쓰지 않음.

| 문서 | 제거일 | 근거 |
|---|---|---|
| `industry/2026-02-22-will-ai-replace-programmers.md` | 2026-08-21 | PR #2 |
| `industry/2026-04-01-ai-trends-2026.md` | 2026-08-21 | PR #2 |
| `industry/2026-08-01-hn-ai-digest.md` | 2026-08-21 | PR #2 |
| `agents/2026-08-10-agent-orchestrator-ao.md` | 2026-08-21 | PR #2. `agents/2026-08-10-agent-orchestrators.md` 로 병합. 원문 출처를 다시 만나도 그쪽 문서에 이미 들어가 있음 |
| `agents/2026-08-10-paseo-vs-orca-agent-orchestrators.md` | 2026-08-21 | PR #2. 위와 같은 곳으로 병합. 두 문서가 같은 3종 비교를 따로 하고 있었음 |
| `practices/2026-07-02-claude-cowork-setup.md` | 2026-08-21 | PR #2. `practices/2026-08-10-claude-cowork.md` 로 병합 |
| `practices/2026-07-24-graph-engineering-explained.md` | 2026-08-21 | PR #2. `practices/2026-07-20-graph-engineering.md` 로 병합 |
| `practices/2026-08-07-automate-your-life-with-claude.md` | 2026-09-04 | PR #6 |
| `models/2026-07-27-kimi-k3.md` | 2026-09-04 | PR #9. 본문이 "초록과 메타데이터만 정리했음"이라고 밝힌 채 멈춰 있었고 벤치마크·라이선스·서빙 요구사항이 전부 미확인 |
| `infra/2026-08-10-semantica-context-graph.md` | 2026-09-04 | PR #9. 제품 README 요약이고 핵심 비교표를 본문에서 스스로 "사실표가 아님"으로 무효화해 둠 |
| `agents/2026-02-03-xcode-agentic-coding.md` | 2026-09-04 | PR #9. Xcode 27 서술이 6월 뉴스룸 기반 예고였고 베타 6으로 확인한 문서가 이미 있어 병합함 |
| `agents/2026-08-10-human-review-skill.md` | 2026-09-16 | **이 파일을 만들게 한 문서임.** 점검 실행마다 유지 / 제거로 판단이 갈려서(2026-09-06 유지, 09-07 제거, 09-07 유지) 사람이 직접 읽고 결론을 냄. `설치`·`사용`·`할 수 있는 것`·`구성` 네 절이 원 저장소 README를 순서대로 옮긴 것이고, `## 짚어야 할 것`도 `## 유효기간`도 겹치는 지점 표도 없음. 판단이 담긴 곳은 `## 메모`의 "코드 diff 리뷰에는 안 맞음" 한 줄뿐임. 별 수 654도 수집 시점이 없어 DRAFTING의 "stars는 받은 시점을 명시한다"에 어긋남. 인바운드 4곳(README, `xcode-27-agent-surface`, `hq-team-ai-harness`, `self-improving-agent-loops`)을 정리함 |
