---
title: 연령등급 디스크립터에 virtual companion 이 들어옴, AI 대화 기능이 18+ 로 떨어지는 경로
source: https://developer.apple.com/news/?id=yxkh08us
author: Apple
published: 2026-11-02
collected: 2026-11-11
tags: [apple, ios, app-store, app-review, compliance, age-rating]
---

출처: [Upcoming: New age rating content descriptors in App Store Connect](https://developer.apple.com/news/?id=yxkh08us) (2026-11-02) · [Age ratings values and definitions](https://developer.apple.com/help/app-store-connect/reference/age-ratings/) (기존 디스크립터 목록과 전역 18+ 트리거 확인, 2026-11-11 기준) · [Set an app age rating](https://developer.apple.com/help/app-store-connect/manage-app-information/set-an-app-age-rating/) (문항 동작 확인, 2026-11-11 기준)

## 요약

App Store Connect 연령등급 문항에 **2026-11-16부터 디스크립터 둘이 추가**됨. `Shared Location` 과 `Dating and Relationships` 이고, AI 기능을 붙이는 쪽에 걸리는 것은 후자임. 정의 문장에 **`a virtual companion that engages a user in sexual or romantic content`** 가 들어가 있고, 이 디스크립터가 붙은 앱은 **최소 연령등급이 18+** 가 됨. 지금까지 연령등급 문항에는 **모델이 대화 상대인 경우를 지목하는 항목이 아예 없었음.** 사용자 간 통신을 묻는 `Messaging and Chat` 만 있었으니, 캐릭터 대화 기능은 문항 어디에도 걸리지 않고 지나갔다는 뜻임. 기한은 둘임. 업데이트를 제출하거나 문항을 바꿀 때 답해야 하고, **모든 앱이 2027-01-12까지** 선언을 끝내야 함. 그리고 이 18+ 는 **빈도 조건이 없음.** 전역 18+ 를 만드는 기존 항목은 `Gambling` 하나를 빼면 전부 `Frequent` 가 붙어 있음.

## 추가되는 디스크립터 둘

| 디스크립터 | 정의 (공지 원문) | 최소 등급에 미치는 영향 |
|---|---|---|
| **Shared Location** | "Sharing of real-world location with another person or group." | "Apps that include Shared Location may not have an impact on their minimum age rating in some regions." |
| **Dating and Relationships** | "Features that enable dating or relationship connections between real people and/or a virtual companion that engages a user in sexual or romantic content." | "Apps that include Dating and Relationships receive a minimum age rating of 18+." |

선언 시점도 공지가 직접 적음.

> "You must provide these declarations when submitting an app update or making changes to your age rating questionnaire."

> "All apps must have these declarations completed by January 12, 2027."

즉 11-16 이후 아무 업데이트나 올리는 순간 문항이 막아서고, 업데이트를 안 올려도 2027-01-12 가 하한임.

## 왜 AI 대화 기능에 걸리는가

정의 문장의 구조를 봐야 함. **`between real people and/or a virtual companion`** 임. `and/or` 라서 **사람 상대가 하나도 없어도 걸림.** 앱 안에 사람끼리 연결되는 기능이 전혀 없고 모델만 있어도 조건이 성립함.

걸리는 조건은 둘이 함께일 때임.

1. 대화 상대가 **가상 동반자(virtual companion)** 로 설계됐는가
2. 그것이 사용자를 **성적 또는 연애 콘텐츠로 끌어들이는가**

둘째가 실제 게이트임. 코드 리뷰를 돕는 어시스턴트나 요약 기능은 1번만 해당하거나 둘 다 아님. 반대로 **페르소나를 가진 캐릭터 챗, 롤플레이, "AI 친구" 포지셔닝**은 2번을 어떻게 막아뒀느냐로 답이 갈림. 온디바이스 Foundation Models 로 돌리든 서버 모델을 쓰든 문항은 구현을 묻지 않으므로 이 구분은 무관함.

여기서 주의할 게 하나 있음. **판단하는 쪽이 Apple 이 아니라 문항에 답하는 개발자임.** 이 저장소의 [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md)에 적어둔 **2.3.6** 이 바로 그 자리고, 지침은 연령등급을 잘못 매기면 규제 기관 조회로 이어질 수 있다고 적어둠. "모더레이션으로 성적·연애 콘텐츠를 차단했으니 해당 없음"으로 답하려면 그 가드레일이 말이 아니라 동작으로 서 있어야 함.

## 기존 디스크립터 지도에서 이 둘이 앉는 자리

참조 페이지의 디스크립터는 여섯 묶음임. **In-App Controls**(Parental Controls, Age Assurance), **Capabilities**(Unrestricted Web Access, User-Generated Content, Social Media, Social Media Disabled for Users Under 13, Messaging and Chat, Advertising), **Mature Themes**, **Medical or Wellness**, **Sexuality or Nudity**, **Violence**, **Chance-Based Activities**.

이 목록에서 AI 를 지목하는 항목은 하나도 없었음. 가장 가까운 것이 `Messaging and Chat` 인데 정의가 **"Direct communication between users"**, 즉 사용자 간 통신임. 모델이 상대인 대화는 사용자가 아니므로 이 항목으로는 안 걸림. `virtual companion` 이라는 말이 문항에 들어오는 것은 이번이 처음임.

전역 18+ 를 만드는 트리거를 2026-11-11 기준으로 대조하면 이렇게 됨.

| 디스크립터 | 18+ 가 되는 조건 |
|---|---|
| Alcohol, Tobacco, or Drug Use or References | `Frequent` |
| Sexual Content or Nudity | `Frequent` |
| Realistic Violence | `Frequent` |
| Simulated Gambling | `Frequent` |
| Gambling | 빈도 조건 없음 |
| **Dating and Relationships** | 빈도 조건 없음 (2026-11-16 추가 예정) |

여섯 중 넷이 `Frequent` 를 요구함. 즉 기존 구조는 "얼마나 자주 나오느냐"로 완충이 있었고, 성적 콘텐츠도 `Infrequent` 면 18+ 까지 가지 않았음. **`Dating and Relationships` 는 그 완충이 없음.** 기능이 있느냐 없느냐로 끝남.

⚠️ 더 위에 있는 칸도 같이 봐야 함. `Graphic Sexual Content and Nudity` 와 `Prolonged Graphic or Sadistic Realistic Violence` 는 18+ 가 아니라 **Unrated** 로 떨어지고, 참조 페이지는 **"An Unrated app can't be published on the App Store"** 라고 적음. 성인용 AI 롤플레이를 기획하면 18+ 가 상한이 아니라 중간 지점이라는 뜻임.

## 18+ 가 붙으면 달라지는 것, 그리고 확인하지 못한 것

두 도움말 페이지에서 확인된 것은 둘뿐임.

- **Kids 카테고리에 표시될 수 있는 것은 계산된 등급이 4+ 또는 9+ 인 앱임.** 18+ 는 당연히 제외
- EULA 의 최소 연령 요구가 Apple 이 계산한 등급보다 높으면 **등급을 수동으로 올려야 함.** 원문은 "you must override to a rating that adheres to the requirements"

반대로 **18+ 가 스토어 노출, Screen Time, Declared Age Range API 와 어떻게 맞물리는지는 이 공지와 두 도움말 페이지에 없음. 확인할 수 없었음.** 디스크립터 목록에 `Age Assurance`(선언 연령 범위 API·연령 추정·정부 ID 확인) 항목이 이미 있다는 것까지만 확인됨.

## 지역별로 갈리는 부분

브라질이 따로 적혀 있음. 현지 규제가 `Shared Location` 에 **A16**, `Dating and Relationships` 에 **A18** 을 요구함. 낮은 등급이 맞다고 보는 경우(공지가 드는 예는 부모와 자녀용 위치 공유) **브라질 법무공공안전부(MJSP)에 공식 등급을 신청**할 수 있고, 확인되면 그 등급이 브라질 App Store 에 표시됨.

`Shared Location` 쪽은 공지 자체가 지역에 따라 최소 등급에 영향이 없을 수 있다고 적었으므로, 전역 등급만 보고 지역 등급을 추정하면 안 됨. 참조 페이지에는 호주·브라질·한국 전용 등급표가 따로 있음.

## 지금 할 것

1. 대화형 AI 기능의 **페르소나와 허용 주제를 문서로 확정**함. "성적·연애 콘텐츠를 허용하는가"가 Yes/No 로 답해지는 상태여야 문항에 답할 수 있음
2. 금지로 답할 거면 **가드레일이 실제로 서는지 확인**함. 시스템 프롬프트에 한 줄 적어둔 수준이면 문항 답변과 앱 동작이 어긋남. 이 저장소의 [자기개선 에이전트 루프의 7가지 규칙](../practices/2026-08-10-self-improving-agent-loops.md)이 말하는 얕은 지표 문제가 그대로 걸림
3. **문항을 바꾸는 것 자체가 선언 제출 시점임.** 11-16 이후 등급 문항을 손대면 그 자리에서 답해야 하므로, 릴리스 일정과 묶어 미리 정리하는 쪽이 쌈
4. 18+ 를 받아들일지 기능을 좁힐지 정함. 대상 사용자층이 그 아래라면 기획 단계로 되돌아가는 결정이고, 2027-01-12 는 그 결정을 미룰 수 있는 한계임

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | **2.3.6** 연령등급 문항 정확성. 이 공지가 그 문항에 항목을 더하는 것임 |
| [App Review Guidelines 2026-10-16 개정](./2026-10-16-app-review-ncii-hidden-features.md) | 1.2 가 `AI content creation apps` 를 대상 유형으로 지목한 것과 같은 방향. 한쪽은 조항, 이쪽은 문항 |
| [iOS 27 · Xcode 27 정식 출시](./2026-09-14-ios-27-ai-apis-shipped.md) | 연령등급 문항에 소셜 미디어 표시가 추가된 변경. 문항이 늘어나는 흐름의 앞 항목 |
| [Anthropic 2026 사용 정책 개정](./2026-10-08-anthropic-usage-policy-2026.md) | 모델 제공자 쪽 약관. AI 고지 요구와 고위험 영역이 같은 기능에 겹쳐 걸림 |
| [Foundation Models 허용 사용 요건](./2026-10-19-foundation-models-acceptable-use.md) | 계약서 3.3.11(A) 의 금지 항목. 의존 유발 항목이 동반자형 기획과 정면으로 겹침 |

## 짚어야 할 것

- **공지는 AI 나 챗봇, 모델을 이름으로 부르지 않음.** AI 기능에 걸린다고 읽는 근거는 정의 문장의 `virtual companion` 한 구절뿐이고, 그 해석은 이 문서의 것임
- **`virtual companion` 의 범위를 Apple 이 더 정의한 문서를 찾지 못했음.** 캐릭터 대화 앱 중 어디까지가 "사용자를 성적·연애 콘텐츠로 끌어들이는" 것인지는 **확인할 수 없었음.** 심사 사례도 아직 없음
- **2026-11-11 기준 두 디스크립터는 도움말의 디스크립터 목록과 Age ratings 참조 표에 아직 없음.** 공지가 11-16 부터라고 적었으니 예정대로면 그 뒤에 들어오고, 정의 문구가 그때 달라질 수 있음
- 2차 보도 중 **"Apple 이 AI 동반자 앱을 데이팅 앱으로 분류한다"** 는 식의 서술이 돌아다니는데 **공지에는 그런 문장이 없음.** 디스크립터 선언과 앱 카테고리는 다른 층위고, 이 문서는 그 주장을 쓰지 않았음
- 전역 18+ 트리거 표는 2026-11-11 에 Age ratings 참조 페이지에서 확인한 것임. 같은 페이지의 **지역별 표는 브라질 항목만 대조했고 호주·한국 표는 안 봤음**
- 공지 본문에는 지시문이나 명령문 형태로 읽힐 문장이 없었음

## 유효기간

**2026-11-11 확인 기준이고 디스크립터 추가 전임.** 날짜(11-16 추가, 2027-01-12 완료)는 고정된 기록이라 낡지 않음. 낡는 것은 정의 문구와 트리거 표임. 다시 볼 때 확인할 것 셋. **11-16 이후 Age ratings 참조 페이지와 디스크립터 목록에 두 항목이 실제로 들어왔고 정의가 공지와 같은지**, **`Dating and Relationships` 가 전역 18+ 표에 빈도 조건 없이 올라왔는지**, 그리고 **`virtual companion` 범위를 설명하는 별도 문서나 심사 사례가 생겼는지**. 셋 다 위 두 도움말 페이지에서 항목 이름으로 검색하면 잡힘.
