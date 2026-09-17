---
title: AFM 3, 온디바이스 20B를 플래시에 두는 Apple 3세대 모델
source: https://machinelearning.apple.com/research/introducing-third-generation-of-apple-foundation-models
author: Apple Machine Learning Research
published: 2026-06-08
collected: 2026-09-16
tags: [apple, ios, foundation-models, on-device-llm, model-release]
---

출처: [Introducing the Third Generation of Apple's Foundation Models](https://machinelearning.apple.com/research/introducing-third-generation-of-apple-foundation-models) (2026-06-08 발행, 본문 하단에 **2026-09-09 갱신** 표시)

## 요약

이 저장소는 Apple 모델을 지금까지 "온디바이스 3B 아니면 PCC"로 다뤄 왔음. 3세대는 그 이분법이 아니라 **다섯 모델의 가족**임. 가장 크게 갈리는 게 **AFM 3 Core Advanced**로, **20B 희소 모델을 DRAM이 아니라 플래시(NAND)에 두고** 요청마다 전문가 집합을 골라 **1~4B만 활성**시킴. 서버 쪽은 주력 **AFM 3 Cloud**, 이미지 전용 **ADM 3 Cloud**, 최상위 **AFM 3 Cloud Pro** 셋이고 **Cloud Pro만 NVIDIA GPU에 최적화**됐다고 밝힘. 다만 이 글에 실린 품질 수치는 전부 **Apple 인하우스 사람 평가**이고 외부 벤치마크가 아니며, 원문이 "여름 중 공개하겠다"고 적은 **기술 리포트는 2026-09-16 현재 확인되지 않음.**

## 다섯 모델

원문 표현으로 "Google과 협업해 맞춤 제작한(custom-built in collaboration with Google) 다섯 모델의 가족"임.

| 모델 | 위치 | 원문이 밝힌 것 |
|---|---|---|
| **AFM 3 Core** | 온디바이스 | **3B 밀집(dense)** 모델의 차세대. 품질 개선 |
| **AFM 3 Core Advanced** | 온디바이스 | Apple의 가장 강력한 온디바이스 모델. **태생부터 멀티모달**이고 표현력 있는 음성과 고정확도 받아쓰기를 담당. **20B 희소, 요청에 따라 1~4B 활성** |
| **AFM 3 Cloud** | 서버 (PCC) | 속도·효율·성능에 최적화된 서버 주력 |
| **ADM 3 Cloud (Image)** | 서버 (PCC) | 이미지 생성·편집. 사진 편집 도구와 Image Playground를 구동 |
| **AFM 3 Cloud Pro** | 서버 (PCC) | 가장 강한 서버 모델. **에이전틱 툴 사용과 복잡한 추론** 같은 부하 큰 용도 |

⚠️ **"20B 희소, 1~4B 활성"은 서버 모델이 아니라 온디바이스 Core Advanced임.** 플래시 저장이라는 설명이 붙는 것도 그래서고, 이 대목은 2차 보도에서 자주 뒤바뀜.

## AFM 3 Core Advanced, 플래시에 사는 20B

이 글에서 기술적으로 제일 값어치 있는 부분임. 구조 이름은 **Instruction-Following Pruning(IFP)** 기반의 희소 활성 아키텍처.

전제부터 원문이 적어 둠. 보통의 LLM은 가중치 전체가 활성 메모리(DRAM)에 있어야 하고, 그 발자국이 소비자 하드웨어에서 규모를 막음. 그래서 **전체 모델을 플래시(NAND)에 둠.** 문제는 NAND에서 DRAM으로의 대역폭이 **토큰 단위로 갈아끼우기에는 너무 느리다**는 것이고, 해법이 라우팅 단위를 바꾸는 것임.

- 라우팅 결정을 **토큰마다가 아니라 프롬프트마다** 함
- 가벼운 밀집 블록이 초기 처리에서 **고정된 전문가 집합을 고르고**, 생성 중에 주기적으로 다시 고름
- 상시 활성인 **shared expert** 비중을 높게 두고, 입력에 의존하는 **routed expert**만 필요할 때 DRAM으로 올림

원문이 이 설계의 요점으로 드는 것은 성능이 아니라 **추론 시점의 탄력성(inference-time elasticity)** 임. 모든 작업에 단일 모델을 쓰거나 작은 모델 여럿을 앙상블로 관리하는 대신, **용도별로 활성 파라미터 수를 미리 정해** 두고 요청 난이도에 따라 가중치를 점진적으로 올림.

**이 저장소 관점에서 짚을 것.** [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)에 적어둔 설계 분기는 "온디바이스로 안 되면 PCC"였음. 활성 파라미터를 용도별로 정하는 모델이 온디바이스에 들어오면 그 분기가 **2지선다가 아니게 됨.** 다만 이 글은 모델 이야기만 하고 **프레임워크에서 Core Advanced를 앱이 직접 고를 수 있는지는 말하지 않음.** 아래 `## 짚어야 할 것` 참고.

## 서버 쪽, PT-MoE와 하드웨어가 갈리는 지점

**AFM 3 Cloud.** Private Cloud Compute 위에서 도는 멀티모달 추론 모델임. 개선 축으로 **Parallel-Track Mixture-of-Experts(PT-MoE)** 기반을 든 뒤, 그 효과를 학습 안정화와 함께 **"컨텍스트 윈도우 안의 정보를 추론하고 정확히 회상하는 능력"** 으로 적음. 컨텍스트 크기 자체는 이 글에 숫자로 나오지 않음.

**ADM 3 Cloud (Image).** 종횡비와 해상도에 걸쳐 일반화되고, 기본 모델이 생성·편집·Genmoji를 직접 처리함. 그 위에 **전용 어댑터**가 개별 편집 경험을 담당한다고 밝힘. 예로 드는 것이 사진 앱의 **Spatial Reframing**, 터치 기반 이미지 수정, Image Playground의 개인화임.

**하드웨어가 갈림.** 넷과 하나로 갈림.

> "AFM 3 Core, AFM 3 Core Advanced, and AFM 3 Cloud, along with ADM 3 Cloud, are all purpose-built for Apple silicon."

> "For AFM 3 Cloud Pro, we worked with Google and NVIDIA to extend Private Cloud Compute to NVIDIA GPUs in Google Cloud"

**Cloud Pro 쪽은 칩 벤더만 갈리는 게 아니라 인프라도 갈림.** PCC 를 Google Cloud 의 NVIDIA GPU 로 **확장했다**고 적혀 있음. 그런데도 결론부는 다섯 모델 전부에 대해 이렇게 적음.

> "To protect our users' privacy, these models run exclusively on-device and on Private Cloud Compute."

즉 Apple 의 서술에서 **PCC 는 Apple 데이터센터라는 장소가 아니라 보증의 이름**임. 그 보증이 남의 클라우드에 놓인 GPU 위에서 어떤 기술적 근거로 유지되는지는 **이 글에 없음.** PCC 의 검증 가능성을 근거로 도입을 판단하는 자리라면 여기부터 따로 확인해야 함.

## 학습

- **사전학습**을 최신 세대 클라우드 **TPU** 가속기에서 크게 확장함
- 모든 모델이 **공통의 초기 기반을 공유**한 뒤 각자의 구조와 용도로 특화됨. 이때 오디오, 이미지 이해, 롱컨텍스트 추론, 고품질 시각 생성이 붙음
- **후학습**은 지도 파인튜닝에 **다단계 강화학습**을 결합
- **Quantization Aware Training**으로 정확도를 유지한 채 크게 압축

데이터 쪽 진술 둘을 그대로 옮김. 공개 정보, 서드파티에서 라이선스하거나 구매한 데이터, 오픈소스 데이터, 전용 연구로 얻은 데이터, 합성 데이터의 혼합이라고 적고, 이어서 이렇게 씀.

> "We do not use our users' private personal data or user interactions when training our foundation models."

웹 퍼블리셔의 학습 거부(opt out) 권리를 존중한다는 문장도 붙음. 결론부 갱신 표시에는 반대 방향의 항목이 하나 있음. **2026-09-09 자로 "Siri와 Apple Intelligence 개선에 도움을 주도록 사용자가 옵트인할 수 있다"** 가 추가됨. 기본값이 아니라 옵트인이라는 것까지가 원문에서 확인되는 범위임.

## 자체 평가

전부 **인하우스 사람 채점자**가 Instruction Following, Truthfulness, Presentation 축으로 채점한 것이고, 이미지 프롬프트에는 Image Understanding이 추가됨.

| 비교 | 텍스트 선호도 | 이미지 이해 선호도 |
|---|---|---|
| AFM 3 Core 대 2025 기준선 | **45.6% 대 23.3%** | **61% 이상** |
| AFM 3 Cloud 대 2025 AFM Server | **64.7% 대 8.7%** | **37.8% 대 9.6%** |
| AFM 3 Cloud Pro 대 AFM 3 Cloud | 상대 **약 10%** 개선 | 상대 **14%** 개선 |

단측 평가로는 AFM 3 Cloud가 2025 AFM Server 대비 전체 응답 만족도 **약 36%**, 지시 따르기 **21%** 상대 개선.

**14% 가 원문에 두 번 나오는데 서로 다른 것임.** 표의 14% 는 이미지 이해 쪽이고, 같은 숫자가 수학 범주에도 따로 나옴. 우연히 같은 값이라 한쪽을 다른 쪽으로 옮겨 적기 쉬움.

> "AFM 3 Cloud Pro provides an even further improvement over our AFM 3 Cloud, achieving a relative improvement in overall response satisfaction of roughly 10 percent for text and 14 percent for image understanding overall."

> "AFM 3 Cloud Pro excels in specific task categories such as Math, showing a relative 14 percent improvement over AFM 3 Cloud."


**사이드바이사이드 수치를 읽을 때 주의.** 45.6 대 23.3을 더해도 100이 안 됨. 나머지가 무승부인지 어떻게 처리됐는지 **원문이 밝히지 않음.** 그래서 이 표는 "몇 퍼센트 좋아졌다"로 옮기면 안 되고 방향으로만 써야 함.

### 음성 쪽은 기준선이 모델이 아님

여기가 다른 표와 성격이 다름. 비교 대상이 이전 세대 모델이 아니라 **Apple의 기존 프로덕션 TTS 시스템**임. 5점 MOS.

| 항목 | 기존 TTS | AFM 3 Core Advanced |
|---|---|---|
| General Voice | 3.87 | **4.15** |
| Conversational Voice | 3.82 | **4.24** |

일반 음성에서 **+0.28**, 대화체에서 **+0.42**로 격차가 벌어짐. 그리고 이 점수가 **1B 활성 크기에서 나온 것**이라고 명시함. 위 20B 희소 구조의 요점이 숫자로 드러나는 유일한 대목임.

받아쓰기는 일곱 축(Overall Quality, Punctuation, Casing, Layout, Meaning Capture, Disfluency Handling, Style)으로 이전 받아쓰기 시스템과 비교했고, 역시 **1B 활성 크기**에서 Overall Quality **44.7% 대 17.6%** 로 선호됨. 나머지 여섯 축도 일관되게 앞섰다고 적음.

## 로케일과 안전

지원 로케일을 한 문장으로 나열하지 않고 **각주의 묶음 약어**로 표기함. 확인되는 묶음은 셋임.

| 약어 | 포함 |
|---|---|
| **PFIGSCJK** | 포르투갈어, 프랑스어, 이탈리아어, 독일어, 스페인어, 중국어, 일본어, **한국어** |
| **DDNSTV** | 덴마크어, 네덜란드어, 노르웨이어, 스웨덴어, 터키어, 베트남어 |
| **AFIHHMPRTU** | 아랍어, 핀란드어, 인도네시아어, 히브리어, 힌디어, 말레이어, 폴란드어, 러시아어, 태국어, 우크라이나어 |

영어는 미국·영국·호주·인도 방언을 아우르는 표기로 따로 씀. 안전 쪽은 민감 콘텐츠 분류 체계를 두고, **다국어 후학습 정렬**, **언어별 가드레일 모델**, 지원 로케일 전반에 걸친 **원어민 레드팀**을 한다고 적음.

**로케일별 품질 수치는 없음.** 한국어가 목록에 있다는 것과 품질이 영어와 같다는 것은 다른 얘기이고, 이 글로는 후자를 말할 수 없음. 한국어 토큰 효율 문제는 모델이 아니라 토크나이저 쪽인데 이 글에 토크나이저 서술이 아예 없음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 그쪽이 프레임워크 API와 제약(4K/32K, 쿼터, 툴 개수)을 다루고 이 문서가 그 아래 모델 계층을 다룸. 그쪽 `## 모델 라인업` 표가 이 글의 다섯 모델을 요약해 둔 것이고, 여기서는 구조·학습·평가까지 폄 |
| [Apple의 2026 AI 플랫폼](../industry/2026-06-08-apple-ai-platform-for-ios.md) | 같은 날 발표의 플랫폼 쪽. "Apple 모델을 쓴다"에서 "아무 모델이나 쓴다"로 옮겨간 축과, 그 기본값 자리에 놓이는 모델이 무엇인지 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md) | 5.1.2(i)의 갈림길이 "개인정보가 어디로 나가느냐"인데, 다섯 모델이 전부 온디바이스 또는 PCC라는 진술이 그 판단의 근거가 됨 |
| [Claude Fable 5.1](./2026-09-01-claude-fable-5-1.md) | 같은 `models/` 계층의 릴리스 문서. 둘 다 자체 발표 벤치마크라 방향만 볼 것이라는 조건이 같음 |

## 짚어야 할 것

- **전부 Apple 자체 인하우스 사람 평가임.** 외부 벤치마크나 공개 리더보드 수치가 하나도 없음
- **원문이 약속한 기술 리포트를 확인하지 못했음.** 본문에 "we look forward to sharing more details ... in a technical report later this summer"라고 적혀 있는데, 2026-09-16 기준 Apple ML Research에서 확인되는 최신 기술 리포트는 **2025년판**임. 2026년판은 검색으로도 찾지 못했음
- **컨텍스트 윈도우 크기와 토크나이저 서술이 이 글에 없음.** [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md)의 4K·32K는 프레임워크 문서에서 온 값이지 이 글의 값이 아님. 두 출처를 섞어 인용하면 안 됨
- **AFM 3 Core Advanced를 앱이 직접 고를 수 있는지 확인하지 못했음.** 이 글은 모델 계층만 서술하고 `SystemLanguageModel`이 Core와 Core Advanced 중 무엇에 연결되는지, 선택 API가 있는지를 말하지 않음. 프레임워크 문서 쪽에서 별도로 확인해야 함
- **PCC 를 Google Cloud 로 확장했다면서 그 보증이 어떻게 유지되는지는 없음.** 확장했다는 사실과 "exclusively on ... Private Cloud Compute" 라는 결론이 같은 글에 나란히 있는데, 남의 인프라 위에서 PCC 의 검증 가능성이 어떤 근거로 성립하는지는 서술되지 않음
- **어느 기기가 Core Advanced를 돌리는지도 없음.** 플래시 상주 설계라 저장공간·메모리 요구가 기기마다 갈릴 텐데 기기 목록이나 최소 사양이 제시되지 않음
- 다섯 모델과 실제 기능(Siri, 사진 편집, Image Playground)의 연결은 서술돼 있지만, **개발자가 호출할 수 있는 범위와 OS 기능 전용 범위의 경계는 이 글로 갈라지지 않음**
- 원문 본문에 독자나 도구를 향한 지시문·명령문은 없었음

## 유효기간

**2026-09-16 확인 기준**임. 이 글 자체는 발표 문서라 문언이 크게 바뀌지 않지만 하단에 갱신 표시가 붙는 방식이라 재방문할 값어치가 있음(실제로 2026-09-09 갱신이 있었음). 다시 볼 때 확인할 것 둘. **2026년판 기술 리포트가 나왔는지**, 그리고 **AFM 3 Core Advanced의 개발자 접근 경로가 프레임워크 문서에 생겼는지**. 후자는 iOS 27 정식 출시와 함께 갈릴 가능성이 큼. **iOS 27 과 Xcode 27 은 2026-09-14 에 정식 출시됐음.** 그 시점의 확인은 [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md)에 정리돼 있고, 그 문서도 컨텍스트 수치와 프레임워크 가용성 표시는 재확인하지 못했다고 적어 둠. 즉 이 문서의 유보 둘은 그쪽에서도 아직 안 풀림.
