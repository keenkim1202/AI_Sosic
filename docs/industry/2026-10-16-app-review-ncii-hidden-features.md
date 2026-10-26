---
title: App Review Guidelines 2026-10-16 개정, AI 콘텐츠 생성 앱과 숨은 기능
source: https://developer.apple.com/news/?id=vvzx6cul
author: Apple
published: 2026-10-16
collected: 2026-10-17
tags: [apple, ios, app-store, app-review, compliance, content-moderation]
---

출처: [Strengthening App Review Guidelines against nonconsensual intimate imagery and bait-and-switch apps](https://developer.apple.com/news/?id=vvzx6cul) (2026-10-16) · [Updated App Review Guidelines are now available](https://developer.apple.com/news/?id=92th7e5i) (2026-10-16, 개정 항목 목록) · [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) (조항 전문 확인, 2026-10-17 기준)

## 요약

App Review Guidelines가 **2026-10-16에 개정**됐고, 이 저장소에 적어둔 "개정이 나오면 조항 번호로 대조하라"는 재확인 시점이 여기임. AI 기능을 붙이는 쪽에 걸리는 변경은 둘임. **1.2의 적용 대상에 `AI content creation apps`가 이름으로 들어가고** 모더레이션 요구를 안 지키면 **Developer Code of Conduct 위반**으로 번질 수 있다는 문장이 붙었음. 그리고 숨은·휴면·미문서화 기능 금지가 메타데이터 조항(2.3.1)에서 떨어져 나와 **2.5.19 신설 조항**이 됐고, 위반 시 **앱 제거에 더해 개발자 계정 해지**가 명시됨. 함께 신설된 **2.3.14**는 크리에이티브 애셋에 **검증 불가능한 제품 주장**을 담지 말라고 적음. AI 기능 홍보 문구가 여기 걸리는 자리임. Apple은 같은 공지에서 작년에 안전 문제가 있는 앱 **15만 건 이상**을 차단하고 사기·악용으로 개발자 계정 **19만 3천 개 이상**을 해지했다고 밝힘(Apple 자체 집계).

## 1.2, AI 콘텐츠 생성 앱이 조항 안에 이름으로 들어감

현재 1.2의 첫 문단 전문임.

> "Apps that enable users to create, generate, upload, share, or otherwise contribute content present particular challenges, including intellectual property infringement, nonconsensual intimate imagery, and anonymous bullying. This includes social media, user-generated content platforms, and AI content creation apps."

요구사항 넷은 그대로임. 게시 전 **부적절 콘텐츠 필터**, **신고 수단과 적시 대응**, **악성 사용자 차단**, **공개 연락처**.

달라진 것은 적용 범위를 읽는 방식임. 이 저장소의 [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md)은 네이티브 생성 기능의 1.2 해당 여부를 **"다른 사용자에게 닿는 콘텐츠가 생기느냐"**로 판단하라고 적어뒀음. 그 기준은 조항 문언에서 끌어낸 추론이었는데, 지금 문언은 **`create, generate`** 를 `upload, share` 와 같은 줄에 두고 **AI 콘텐츠 생성 앱을 대상 유형으로 지목**함. 즉 생성물이 앱 밖으로 나가지 않는 설계도 대상으로 읽힐 여지가 생김.

⚠️ 다만 **조항이 "공유되지 않는 생성물도 포함된다"고 적은 것은 아님.** 문언은 앱 유형을 지목하는 데서 멈춤. 보수적으로 읽으면 사용자가 이미지나 텍스트를 생성하는 기능이 있으면 필터·신고·차단·연락처 넷을 설계에 넣는 쪽이고, 그 판단은 이 문서의 것임.

마지막 문단에 책임 귀속이 한 칸 올라갔음.

> "Developers who fail to implement appropriate safeguards or fail to remediate identified violations after notice may be considered in violation of the Developer Code of Conduct."

Apple 공지가 이 항목을 **"1.2: Clarified app content and app types that are subject to this guideline, and potential Developer Code of Conduct violation"**으로 요약했으므로, Code of Conduct 문장은 이번 개정분임. 실무 차이는 **제재 경로**임. 조항 위반은 앱이 내려가고 끝나지만 Code of Conduct 위반은 **개발자 프로그램 제거**로 이어지는 경로임. 조항 자체도 그 뒤에 "Egregious or repeated behavior" 는 즉시 제거 사유라고 적어둠.

생성형 기능에 대고 번역하면 이렇게 됨. 신고 버튼이 없는 이미지 생성 앱은 조항 미준수가 아니라 **대응 체계 부재**로 분류될 수 있고, 통보를 받고도 고치지 않은 기록이 남으면 그때부터는 앱 하나의 문제가 아님.

## 2.5.19, 숨은 기능 금지가 신설 조항으로 분가함

조항 전문임.

> "Don't include any hidden, dormant, or undocumented features in your app; your app's functionality should be clear to end users and App Review. Attempting to hide undocumented features in your app binary, such as through code obfuscation or other methods, is grounds for removal of your app from the App Store and termination of your developer account."

Apple 공지는 이것을 **"formerly covered in Guideline 2.3.1"**, 즉 오래된 정책을 조항으로 떼어낸 것이라고 적음. 2.3.1(a)는 지금도 **새 기능을 Notes for Review에 구체적으로 적고 심사가 접근할 수 있게 하라**는 요구로 남아 있음. 둘이 이렇게 갈림.

| 조항 | 묻는 것 | 제재 |
|---|---|---|
| **2.3.1(a)** | 제출한 빌드의 기능을 심사에 **설명했는가** | 앱 제거 또는 대체 배포 차단, 계정 해지 |
| **2.5.19** | 바이너리에 **숨은·휴면·미문서화 기능이 있는가** | 앱 제거, 개발자 계정 해지 |

AI 기능을 붙이는 쪽에서 이 조항이 걸리는 자리는 **원격 구성으로 켜는 기능**임. 모델 전환, 프롬프트 교체, 서버 플래그로 여는 베타 기능은 전부 "심사 시점에 제출한 빌드에서 꺼져 있는 코드"임. 공지 본문이 금지 대상으로 설명하는 것은 **심사 때 목적을 다르게 말하고 승인 후에 해로운 기능을 켜는 bait-and-switch**지만, 조항 문언은 `dormant` 를 그냥 금지함.

⚠️ **문서화되고 심사가 접근할 수 있는 기능 플래그가 2.5.19에 해당하는지는 문언으로 갈리지 않음. 확인할 수 없었음.** 갈림길은 "숨겼는가"인데 조항이 그 판단 기준을 적지 않았고, 조항이 예시로 드는 것은 코드 난독화임. 지금 할 수 있는 것은 셋임.

1. 원격으로 켜는 AI 기능 목록을 만들고, 각각이 **제출 빌드에서 어떤 상태인지** 적어둠
2. 켜질 수 있는 기능을 **Notes for Review에 구체적으로 적음**. 2.3.1(a)가 "generic descriptions will be rejected"라고 적어둠
3. 심사 중에 플래그를 바꾸지 않음. 승인 후에 처음 켜지는 경로가 있으면 그 빌드로 심사를 다시 받는 쪽이 쌈

[Prompt Caching In Agents](../agents/2026-07-22-prompt-caching-in-agents.md) 쪽 설계와 겹치는 지점이 하나 있음. 시스템 프롬프트나 툴 목록을 서버에서 내려받아 조립하는 구조는 캐시 prefix를 안정화하려고 쓰는 패턴이기도 한데, 심사 관점에서는 **앱 기능이 바이너리 밖에서 정의되는 구조**로 보임. 2.5.2(번들 자체 완결성)와 2.5.19를 같이 놓고 봐야 함.

## 2.3.14, 크리에이티브 애셋에 들어가는 AI 주장

메타데이터 쪽 신설 조항임. 제품 페이지 헤더와 검색 결과에 쓰는 애셋을 다룸.

> "Creative assets must pertain to your app's content and functionality or overarching brand, follow standard metadata guidelines, and be appropriate for all audiences. They should not include inappropriate content, references to other apps with an intent to mislead, unverifiable product claims, or specific or discounted pricing information."

AI 기능을 홍보할 때 걸리는 것은 **`unverifiable product claims`** 임. 모델 품질이나 정확도를 수치로 적은 애셋이 여기 해당할 수 있고, 온디바이스 모델의 출력 품질은 기기·언어·프롬프트에 따라 갈려서 재현 가능한 근거를 대기가 어려움. 애셋이 **`pertain to your app's content and functionality`** 여야 한다는 조건도 있으니, 생성 결과 예시를 애셋에 쓸 계획이면 앱이 실제로 그 수준을 낸다는 전제가 붙음.

## 섹션 5와 대체 배포

인신매매·아동 착취 콘텐츠를 담은 앱에 대해 **App Store 제거에 더해 대체 배포 차단**이 명시됨. EU의 대체 배포 경로를 쓰는 앱에만 추가로 걸리는 항목이고 AI와 직접 관계는 없음. 다만 제재가 **스토어 단위가 아니라 배포 경로 전체 단위로 적히기 시작했다**는 신호로 읽을 만함. 2.5.19의 "termination of your developer account"도 같은 방향임.

## AI와 무관한 나머지 개정

- **3.1.1(a)**: Music Streaming Services Entitlements 불릿이 StoreKit External Purchase Link Entitlements 쪽으로 합쳐짐
- **Before You Submit / After You Submit**: 문서 링크 추가

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](./2026-08-21-app-store-ai-review-rules.md) | 같은 지침의 2026-08-21 스냅샷. 5.1.2(i)·4.7·2.5.2 서술은 이번 개정 항목에 없으므로 그대로 유효함. 1.2 적용 범위 판단과 2.5.2 주변만 이 문서로 갱신됨 |
| [iOS 27 · Xcode 27 정식 출시](./2026-09-14-ios-27-ai-apis-shipped.md) | 연령등급 문항에 소셜 미디어 기능 표시가 추가된 변경을 그쪽에서 다룸. 1.2 대상 유형 확대와 같은 방향 |
| [Foundation Models 실전](../practices/2026-08-21-foundation-models-in-practice.md) | 온디바이스로만 돌려서 5.1.2(i)를 피하는 설계가 1.2는 피하지 못한다는 것이 이번 개정으로 더 분명해짐 |

## 짚어야 할 것

- **이전 문언과의 축조 대조는 하지 못했음.** 1.2에서 `AI content creation apps` 라는 문구가 이번에 들어간 것인지 전부터 있던 것인지는 **확인할 수 없었음.** Apple 공지가 "app types that are subject to this guideline"을 명확히 했다고 적었고 이 저장소의 2026-08-21 문서는 그 문장을 인용하지 않았다는 것까지가 확인된 것임. Code of Conduct 문장은 공지가 개정 항목으로 지목했으므로 이번 것임
- **공지 본문과 조항 문언의 범위가 다름.** 공지는 nudification 앱과 bait-and-switch를 겨냥한다고 설명하지만, 조항은 `AI content creation apps` 와 `dormant` 처럼 더 넓은 말로 적혀 있음. 심사에서 어디까지 적용되는지는 문언만으로 알 수 없음
- **15만·19만 3천은 Apple 자체 집계이고 기준 기간이 "Last year"로만 적혀 있음.** 역년인지 회계연도인지 공지가 밝히지 않음
- **번역본은 한 달 내 제공 예정임.** 한국어판 문언으로 대조할 수 있는 시점은 2026-11 중임
- 심사 실무는 조항 문언보다 리뷰어 판단으로 갈리는 부분이 있음. 이 문서는 개정 문언과 공지만 정리한 것이고 리젝 사례를 모은 게 아님

## 유효기간

**2026-10-17 확인 기준**임. 개정 항목 목록과 날짜는 고정된 기록이라 낡지 않지만, 조항 문언은 상시 개정됨. 다시 볼 때 확인할 것 둘. **2.5.19가 원격 기능 플래그에 어떻게 적용되는지 밝히는 문서나 사례가 생겼는지**, 그리고 **한국어 번역본이 올라왔는지**(2026-11 예정). 그사이 개정은 [개발자 뉴스](https://developer.apple.com/news/)에서 조항 번호로 확인하는 편이 나음.
