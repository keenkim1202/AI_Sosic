---
title: Semantica, 설명 가능한 AI를 위한 그래프 네이티브 인프라
source: https://github.com/semantica-agi/semantica
author: Semantica AGI
collected: 2026-08-10
tags: [knowledge-graph, context-graph, provenance, ai-governance, graph-rag, python, mcp]
---

출처: [semantica-agi/semantica](https://github.com/semantica-agi/semantica) · [getsemantica.ai](https://getsemantica.ai) · [플랫폼 데모 영상](https://www.youtube.com/watch?v=QfnNZg4-dZA)

## 요약

LLM이랑 벡터 스토어, 에이전트 프레임워크 **아래에** 깔리는 결정론적 인프라 레이어임. 스스로를 "AI 에이전트를 위한 오픈소스 팔란티어"라고 부름. 핵심은 임베딩이 아니라 의미를 저장하고, 모든 결정에 W3C PROV-O 출처를 붙여서 규제 기관의 "왜?"에 답할 수 있게 만드는 것. 그래프 구축이랑 추론, 프로버넌스에 LLM이 필요 없다는 게 특징임.

## 기본 정보

| | |
|---|---|
| 저장소 | `semantica-agi/semantica` |
| Stars / Forks | 3,373 / 389 |
| 언어 | Python (3.8+) |
| 라이선스 | MIT |
| 생성 | 2025-06-25 |
| 설치 | `pip install semantica` |

## 문제 인식

README 첫 문단이 논지를 다 담고 있음. 대부분의 AI 에이전트는 흔적 없이 행동함. 임베딩은 저장하지만 의미는 저장 안 하고, 설명할 수 없는 컨텍스트랑 감사할 수 없는 결정만 남김. 대출 심사라면 이건 불편함이 아니라 컴플라이언스 노출임. 인수 심사 에이전트의 승인은 몇 달 뒤 규제 기관의 "왜"를 견뎌야 함.

## 대상

- 결과에 책임이 따르는 결정을 내리는 에이전트를 만들고, 벡터 인덱스가 아니라 구조화된 컨텍스트가 필요한 AI/ML 플랫폼 팀
- Unity Catalog나 Snowflake에 이미 있는 테이블을 외부 SaaS로 내보내지 않고 거버넌스된 지식 그래프로 만들려는 데이터 플랫폼 팀
- "AI가 왜 그렇게 했나"에 규제 기관이 받아들일 형식으로 답해야 하는 컴플라이언스·리스크·감사 팀
- 블랙박스를 출시할 수도, 데이터를 남의 SaaS에 보낼 수도 없는 규제 산업(금융, 의료, 법률, 정부, 국방)
- KG랑 추론, 프로버넌스 스택을 셀프호스팅하고 백엔드를 갈아끼우고 싶은 인프라 엔지니어

## 기존 방식과 비교

| | Vector DB + RAG | 순수 LLM 메모리 | Semantica |
|---|---|---|---|
| 회상 방식 | 임베딩 유사도 | 토큰 윈도우 | 그래프 순회 + 시맨틱 검색 |
| 결정 이력 | 없음 | 없음 | 조회 가능한 1급 객체 |
| 프로버넌스 | 없음 | 없음 | W3C PROV-O, 소스 연결 |
| 추론 | 없음 | 블랙박스 | Forward chain, Rete, Datalog, SPARQL |
| 충돌 감지 | 조용히 덮어씀 | 조용히 덮어씀 | 감지·플래그·해소 |
| 시점 조회 | 불가 | 불가 | 특정 시점 그래프 스냅샷 |
| 컴플라이언스 내보내기 | 없음 | 없음 | PROV-O, SHACL, OWL, RDF |
| 정책 집행 | 없음 | 없음 | 룰 엔진 + SHACL 내장 |
| 엔티티 해소 | 불가 | 불가 | 블로킹 + 시맨틱 중복 제거 |
| 멀티 에이전트 컨텍스트 | 에이전트별 분리 | 에이전트별 분리 | 공유 인텔리전스 레이어 |

⚠️ **이 표는 Semantica README의 자체 비교이고 사실표가 아님.** 경쟁 방식 쪽 칸이 지나치게 절대적임. Vector DB + RAG에도 출처 메타데이터, 시점 필터, 정책 집행 계층, 공유 인덱스, 충돌 처리를 붙일 수 있고 실제로 그렇게 쓰는 스택이 많음. LLM 메모리도 결정 이력을 따로 기록할 수 있음. "없음", "불가", "조용히 덮어씀"은 **기본 제품 범주의 한계가 아니라 아무것도 얹지 않은 단순 구현에 해당하는 서술**임. 이 표는 "Semantica가 기본으로 제공하는 것" 목록으로 읽고, 왼쪽 두 열은 걸러 읽는 게 맞음.

기존 스택을 대체하는 게 아니라 보완하는 구조임. LLM이랑 벡터 스토어, 에이전트 프레임워크는 그대로 두고 그 위에 결정 기록이랑 인과 추론, 프로버넌스, 온톨로지 거버넌스, 충돌 감지, 감사 추적을 얹음.

## 아키텍처

```
Sources → Ingest → Parse → Normalize → Split → Extract → Conflict Detection → Deduplication
   → Knowledge Graph → [ Ontology · Reasoning · Provenance · Decisions ] → Enriched KG
   → Vector Store + Polyglot Graph Store (RDF & LPG) → Export / Visualize / REST · MCP · CLI
```

- 수집: 파일, 웹, DB, 엔터프라이즈 플랫폼(Databricks, Snowflake), 클라우드(Google Drive, Elasticsearch), 스트림(Kafka, Kinesis), Git, 이메일, MCP
- 파싱·정규화·분할: 문서 파싱, 텍스트/엔티티/날짜 정규화, GraphRAG 네이티브 엔티티 인지 청킹
- 추출·충돌 감지·중복 제거: NER, 관계, 이벤트, 트리플. 충돌하는 사실은 병합 전에 플래그되고 해소됨
- 저장: RDF 트리플 스토어(Oxigraph 내장, Blazegraph, Apache Jena, Eclipse RDF4J)랑 LPG(Neo4j, FalkorDB, Apache AGE, AWS Neptune), 벡터 스토어. 코드 수정 없이 교체 가능

## Decision Intelligence

결정을 로그 한 줄이 아니라 전체 생애주기를 가진 1급 그래프 노드로 다룸.

```
record_decision()             → 구조화된 컨텍스트를 담은 그래프 노드로 저장
add_causal_relationship()     → 상류 원인과 하류 영향에 연결
find_similar_decisions()      → 과거 결정 전체에 대한 시맨틱 선례 검색
trace_decision_chain()        → 근본 원인까지 전체 인과 계보
analyze_decision_impact()     → 이 결정이 영향을 준 모든 것의 지도
check_decision_rules()        → 설정 가능한 룰셋에 대한 정책 준수 게이트
export / audit trail          → 규제 기관 제출용 W3C PROV-O, CSV, JSON
```

대출 심사 예시가 README에 실려 있음. 신청 → 인수 심사 → 금리 결정을 각각 노드로 기록하고 `CAUSED`, `INFLUENCED`로 인과 사슬을 이음.

```python
from semantica.context import ContextGraph

graph = ContextGraph(advanced_analytics=True)

decision_id = graph.record_decision(
    category="vendor_selection",
    scenario="Choose cloud provider for HIPAA workload",
    reasoning="AWS offers BAA, mature HIPAA tooling, and existing team expertise",
    outcome="selected_aws",
    confidence=0.93,
)

chain     = graph.trace_decision_chain(decision_id)
similar   = graph.find_similar_decisions("cloud vendor", max_results=5)
impact    = graph.analyze_decision_impact(decision_id)
compliant = graph.check_decision_rules({"category": "vendor_selection"})
```

## Context Graph

전통적 RAG에 빠져 있는 구조화된 메모리 레이어임. 평평한 임베딩이 "무엇이 유사한가"에 답한다면, Context Graph는 "무엇이 연결돼 있고, 왜, 어떻게"에 답함. 엔티티는 소스 문서에, 결정은 근거랑 결과에 연결되고, 사실마다 프로버넌스가 붙고, 충돌은 조용히 덮이는 대신 감지됨.

`graph.state_at("2024-01-01")` 같은 시점 스냅샷도 지원함.

## 성능

v0.5.0 기준, 118,000노드 프로덕션 그래프에서 측정한 수치임.

| 작업 | 이전 | 이후 | 개선 |
|---|---|---|---|
| 노드 검색(118k) | 24 ms | 0.004 ms | 6,000배 |
| 임베딩 캐시 히트 | 콜드 로드 | 리비전 기반 캐시 | 처리량 10배 |
| 시맨틱 중복 제거 | 기준 | 후보 생성 최적화 | 6.98배 |
| 후보 생성 | 기준 | 블로킹 전략 | 63.6% |

README도 밝히듯 중복 제거·후보 생성 수치는 자동화된 테스트 단언이 아니라 CHANGELOG에 기록된 과거 측정치임. 하드웨어랑 데이터셋 토폴로지, 백엔드에 따라 달라짐.

## 통합

- 네이티브 플러그인 번들: Claude Code, Cursor, Codex, Windsurf, Cline, Continue, VS Code, OpenClaw
- MCP 서버, REST API, CLI
- Agno 멀티 에이전트 공유 컨텍스트 1급 지원
- LLM 제공자는 LiteLLM 경유로 OpenAI, Anthropic, Gemini, Mistral, Llama, Groq, Cohere, Azure, Bedrock, Ollama, DeepSeek, HuggingFace 등

## 메모

- 마케팅 톤이 센 README임. "오픈소스 팔란티어" 같은 표현이랑 성능 수치는 걸러 읽는 게 좋음. 다만 아키텍처랑 모듈 구성은 실제로 구현돼 있고 각 단계가 독립 임포트 가능하다고 명시함
- 판단 기준은 산업이 아니라 **요구사항**임. 감사 추적과 프로버넌스가 요구사항이 아니면 과한 도구고 벡터 DB로 충분함. 규제 산업이 아니어도 사후 재구성이 필요한 시스템은 있음
- "에이전트가 왜 그렇게 판단했는지"를 사후에 재구성해야 한다면 볼 만한 후보임. 다만 **다른 KG·PROV 스택과 비교해보지 않았으므로 "제일 완결적"이라고는 말할 수 없음**. 이 범주의 대안으로는 [graphify](../agents/2026-08-10-agent-ecosystem-repos.md)처럼 훨씬 가벼운 접근도 있음
