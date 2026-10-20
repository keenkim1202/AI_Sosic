---
title: Scion, 하루 머지 3만 건에서 git 을 버린 기록
source: https://www.anthropic.com/engineering/redesigning-source-control
author: Anthropic
published: 2026-10-13
collected: 2026-10-20
tags: [anthropic, source-control, monorepo, coding-agent, ci, code-review, architecture]
---

출처: [Redesigning source control at Anthropic](https://www.anthropic.com/engineering/redesigning-source-control) (2026-10-13 발행, **2026-10-15 갱신**, 2026-10-20 확인)

## 요약

Anthropic 이 모노레포를 git 에서 떼내고 자체 버전 관리 시스템 **Scion** 으로 옮긴 기록임. 이유가 한 줄로 요약돼 있음. **하루 머지가 2026-05 약 2,500건에서 09 3만 건 이상으로 열 배 넘게 늘면서** 코딩이 아니라 사람 규모로 만들어진 개발 인프라가 병목이 됐다는 것. 설계 전제가 바뀐 지점이 핵심임. **클라이언트가 거의 다 에이전트와 잡이라 항상 온라인**이고, 그러면 git 의 분산·오프라인 우선 설계가 비용만 남김. 결과는 이전 git 미러링 구성 대비 **인프라 풋프린트 약 200배 축소**, 체크아웃 **약 100배 가속**(two orders of magnitude, 분 단위에서 초 단위), **CI CPU 여유 25%**. 다만 본문이 **"Scion is by no means a suitable replacement for git for general use."** 라고 못박고 공개 배포 계획도 없다고 적음. 그래서 읽을 값어치는 시스템이 아니라 **머지 큐와 리뷰를 어떻게 바꿨는가** 쪽에 있음.

## 왜 지금 보는가

iOS 앱 하나를 혼자 또는 소규모로 만드는 입장에서 수만 명 규모 모노레포 얘기는 그대로 쓸 데가 없음. 그런데 이 저장소에 이미 같은 압력을 다른 각도로 적어둔 문서가 둘 있음. [InfoQ Culture & Methods Trends 2026](../industry/2026-08-07-infoq-culture-trends-2026.md)의 **GitHub PR 10억 → 140억 전망**이 예측 쪽이고, [.xcproj JSON 프로젝트 포맷](../agents/2026-09-16-xcproj-json-project-format.md)이 포맷 층에서 충돌을 줄이는 쪽임. 이 글은 그 압력을 실제로 받은 조직이 **무엇을 먼저 버렸는지**를 보여줌. 버린 것 목록에 전역 잠금 기반 머지 검증과 줄 단위 리뷰가 들어 있다는 게 규모와 무관하게 읽을 부분임.

## 숫자

| 항목 | 값 |
|---|---|
| 하루 머지 수 | 2026-05 약 **2,500건** → 09 **3만 건 이상** |
| main 브랜치 히스토리 | **100GB에 근접** |
| 압축 워킹 카피 | **25GB 이상** |
| CI 체크아웃 | 하루 약 **2,500만 회** |
| 그걸 순진하게 fetch 하면 | 하루 **600PB 이상**의 git 트래픽 |
| 대응으로 운영했던 내부 git 미러 | **수백 개** |
| 마이그레이션 | **5개월, 7명** |
| 인프라 풋프린트 (CPU·메모리) | 이전 git 미러링 대비 **약 200배 작음** |
| 체크아웃 속도 | **약 100배**(two orders of magnitude) 빠름. 분 단위 → 초 단위 |
| CI 가 얻은 여유 | CPU 용량 **25%** |
| 캐시 계층별 히트율 | 각 계층 **99% 초과** |
| 인제스션 코드를 Claude 가 재작성한 결과 | 메모리 **20배 적게**, 실행 **4배 빠르게** |

수치는 전부 Anthropic 자체 보고임. 비교 대상인 이전 미러링 구성의 세부는 공개되지 않았음.

## git 이 안 맞았던 지점

문제는 저장소 크기만이 아니었음. **클라이언트의 성격**이 바뀐 것임. 원문은 거의 모든 클라이언트가 에이전트와 잡이고 "online by definition" 이라고 적음. 에이전트 샌드박스도 거기 포함됨. 로컬에 전체 히스토리를 들고 오프라인에서도 돌게 만든 설계가, 서버가 늘 닿는 환경에서는 복제 비용으로만 남음.

중간 대응은 **내부 git 미러를 수백 개 띄우는 것**이었고, 머지가 늘면서 그 운영이 더 어려워졌음. 하루 2,500만 회 체크아웃을 순진하게 처리하면 **600PB/일** 이라는 숫자가 나오는 자리임.

## 에이전트 규모 소스 컨트롤의 원칙 6가지

Scion 이 따른 원칙으로 본문이 제시한 것임. 각 항목이 git 의 어떤 전제를 뒤집는지 같이 봐야 함.

- **클라이언트는 항상 온라인**: 서버가 닿는다고 가정하고, 필요할 때 데이터를 받아옴
- **클라이언트·서버 비대칭**: 히스토리·blame·커밋·머지·리베이스를 **서버가** 처리하고 클라이언트는 상한 있는 로컬 캐시와 워킹 카피만 가짐
- **API 접근 우선**: 소스 컨트롤 질의를 프로그램으로 노출해서, 많은 잡이 로컬 저장소 없이 돌게 함
- **수평 확장**: 독립적으로 확장되는 컴포넌트에 데이터를 분산. 전체를 구현한 복사본도, packfile 유지보수도 없음
- **싸고 락 없는 커밋**: 커밋은 서버에서 일어나고 비용이 **저장소 크기가 아니라 변경 크기**에 비례함. ref 는 main 과 릴리스 브랜치 외에는 권장하지 않음
- **워킹 카피는 선택이고 sparse**: CI 와 쓰기 작업 상당수가 체크아웃 없이 API 로 돌고, 체크아웃할 때는 읽는 것만 가져옴

여기서 ref 를 권장하지 않는다는 항목이 눈에 걸림. 브랜치를 많이 파는 워크플로가 전제에서 빠져 있다는 뜻이고, trunk 기반 개발을 깔고 있음. 각주도 "people only care about history or blame for commits on the main line" 이라고 그 전제를 밝힘.

## Scion 이 어떻게 생겼는가

**changeset.** 작업 단위가 커밋이 아니라 changeset 이고, 원문은 이걸 "an entry in a write-ahead log" 로 설명함. **첫 부모 기준으로 변경된 파일 목록**을 저장하고, 파일 내용은 blob 스토리지의 압축본을 content-addressed 포인터로 가리킴. **"Changesets are created server-side and are available everywhere, instantly."** 즉 생성과 가시성이 서버 쪽에 있음.

**ref 이동은 따로, 그리고 싸게.** changeset 생성과 ref 갱신이 분리돼 있고, ref 를 옮기는 것은 **"a compare-and-swap on a single Spanner row"** 임. 머지가 하루 3만 건이어도 여기가 안 막히는 이유임.

**derived data 는 재귀 정의.** blame 같은 파생 데이터를 타입마다 부모 값에서 계산되게 정의해서, 타입별로 즉시 계산하거나 미뤄 계산하는 선택이 생김.

**3층이 같은 RPC 를 말함.** `scion-daemon`(클라이언트 쪽), `scion-cache`(리전 캐시), `scion-server`(권위 서버)가 **동일한 RPC 에 응답함.** 클라이언트는 캐시 토폴로지를 몰라도 됨. 데몬은 주기적으로 갱신되는 스냅샷에서 시작해 자기 리전의 `scion-cache` 만 호출하고, 미스는 상위로 넘어감. fetch 는 **루트 트리에서 시작해 없는 트리와 blob 을 재귀로 요청**하는 방식임. 각 계층이 **히트율 99% 초과**이고 오래 남는 상태를 들지 않음.

CI 소비자가 **자기 로컬 저장소를 유지할 필요가 없어진 것**이 200배·약 100배 수치의 출처임. 체크아웃이 분 단위에서 초 단위로 내려갔다고 적음.

## 사람의 일이 바뀐 자리

이 절이 규모와 무관하게 읽을 부분임.

- **엔지니어의 역할은 거부권이었음.** 원문 표현이 **"The role of the engineers was to tell Claude “no.”"** 임. Claude 가 시스템 제약을 위반하는 설계를 내거나 가상의 우려에 대해 과설계하는 경우를 사람이 잘라냈다고 적음
- **리뷰가 줄 읽기에서 의도 확인으로 옮겨감.** 사람 리뷰어는 **"they now examine the intent of the change as opposed to reading every line of code."** 상태임
- **코드 리뷰 도구 Antique 를 같이 만듦.** 의미 기반 승인 규칙을 표현할 수 있어서, 주석만 바뀐 변경과 민감한 로직 변경을 분리함
- **머지 큐의 전역 잠금 방식을 버림.** 원문은 **"We haven't been able to effectively replicate the industry standard of a global repository lock on the merge queue"** 이고, 그 대신 **머지 후 테스트와 배포 전 테스트**에 투자했다고 적음. 큐 자체가 사라진 것은 아님: 같은 글이 운영 감시 대상으로 **"Antique merge queue superfan"** 을 든다
- **컴포넌트마다 "superfan" Claude 에이전트**를 붙여 감시시킴
- 테스트와 최적화에도 Claude 를 썼고, 인덱스를 놓치고 있던 ancestry 질의의 알고리즘 문제를 Claude 가 찾았다고 적음. 저장소를 **"among the most well-tested and well-defended codebases we have ever worked on"** 으로 평가함
- 7명이 Scion 과 Antique 와 주변 시스템을 만들고 **5개월에 마이그레이션까지** 끝냈으며, 그 사이 하루 머지가 열 배로 늘었음. 과거라면 수년과 더 많은 인원이 들었을 일이라고 적음

## 안 하겠다고 적은 것

- **일반 대체재가 아님.** **"Scion is by no means a suitable replacement for git for general use."** 수만 명 규모이거나 에이전트 산출이 많은 아주 큰 모노레포에만 맞는다고 함
- **공개판을 낼 계획 없음.** 다른 조직이 쓰려면 코드 리뷰·코드 검색·CI 를 포함한 개발 생애주기 전체를 같이 옮겨야 하고, 내부 컴포넌트가 **"highly bespoke"** 라는 이유임. 모두에게 맞추려 만든 버전은 쓸모가 없을 것을 우려한다고 적음
- 대신 설계 아이디어를 공유해서 git 플랫폼과 OSS 가 일부를 가져가길 바란다고 함. 예로 **서버사이드 RPC**, 더 나은 sparse·shallow 기능을 들고, **"A git server doesn’t have to work the same way as a git client does."** 로 요약함

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [InfoQ Culture & Methods Trends 2026](../industry/2026-08-07-infoq-culture-trends-2026.md) | PR 10억 → 140억 전망의 실측 대응판. 사람 규모 리뷰가 무의미해진다는 예측에 대해 이쪽의 답은 리뷰를 없애는 게 아니라 **줄 읽기에서 의도 확인으로 옮기는 것**이었음 |
| [Eval Engineering](../practices/2026-08-01-eval-engineering-merge-gate.md) | 머지 게이트를 어떻게 여는가. 그쪽은 게이트 설계가 주제인데 이쪽은 **전역 잠금으로 머지를 직렬화하는 방식을 버리고** 머지 후·배포 전으로 검증을 옮겼음. 결론이 갈리는 자리라 둘을 같이 봐야 함 |
| [.xcproj JSON 프로젝트 포맷](../agents/2026-09-16-xcproj-json-project-format.md) | 에이전트 머지 압력에 대한 다른 층의 답. 포맷을 고쳐 충돌 자체를 줄이는 쪽과 버전 관리 시스템을 갈아치우는 쪽 |
| [대규모 AI 코딩 비용 관리](../practices/2026-08-07-managing-ai-coding-costs.md) | 비용이 토큰에서 **주변 인프라**로 옮겨간 사례. CI CPU 25% 와 풋프린트 200배가 그 항목 |
| [MCP 2026-07-28 스펙](./2026-07-28-mcp-stateless-spec.md) | "클라이언트가 상시 온라인이면 상태를 서버에 두는 게 맞다" 는 같은 방향의 설계 전환. 한쪽은 프로토콜, 한쪽은 저장소 |

## 짚어야 할 것

- **자체 보고임.** 머지 건수, 200배, 약 100배, 99% 히트율 모두 Anthropic 이 밝힌 값이고 외부 검증은 없음
- **비교 기준이 공개되지 않음.** "이전 git 미러링 인프라" 의 구성과 측정 방법이 없어서 200배와 25% 가 무엇 대 무엇인지 재현할 수 없음
- **새 아이디어 주장이 아님.** 2026-10-15 갱신으로 붙은 각주가 Sapling 의 서버 컴포넌트 **Mononoke** 가 "pioneered many of the techniques we use in Scion" 이라고 밝히고, 특히 Bonsai 데이터 모델을 지목함. 갱신 사유 자체가 선행 프로젝트 명시였음
- **"Claude 가 구현 대부분을 했다" 는 서술은 검증 수단이 없음.** 메모리 20배·4배 같은 수치도 사내 프로토타입과의 비교이고 코드가 공개되지 않음
- Antique, superfan 에이전트, 머지 후 테스트 구성은 **이름과 역할만** 공개됨. 승인 규칙을 어떻게 쓰는지, 머지 후 테스트가 깨진 main 을 어떻게 되돌리는지는 **확인할 수 없었음**
- 개인·소규모 개발에 옮겨 쓸 수 있는 것은 시스템이 아니라 판단임. **머지 큐를 유지할지, 리뷰를 의도 단위로 올릴지**가 그것이고, 둘 다 되돌리기 어려운 결정임
- 원문 본문에 읽는 쪽을 조종하려는 지시문은 없었음

## 유효기간

**2026-10-20 확인 기준**이고 수치는 **2026-09 스냅샷**임. 발행 이틀 뒤에 각주 갱신이 붙은 글이라 선행 프로젝트 귀속이 더 늘 수 있음. 다시 볼 때 확인할 것 둘. **git 플랫폼(GitHub, GitLab)이 서버사이드 RPC 나 sparse·shallow 개선을 실제로 가져갔는지**, 그리고 **Anthropic 이 입장을 바꿔 공개판이나 더 자세한 설계 문서를 냈는지**. 후자는 본문이 현재로서는 안 한다고 적은 부분임.
