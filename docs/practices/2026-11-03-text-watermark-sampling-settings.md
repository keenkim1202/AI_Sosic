---
title: Claude 텍스트 워터마크, 샘플링 파라미터를 건드리면 조용히 빠지는 자리
source: https://platform.claude.com/docs/en/build-with-claude/text-watermark
author: Anthropic
published: 2026-11-03
collected: 2026-11-06
tags: [claude, anthropic, compliance, disclosure, policy, app-store, provenance]
---

출처: [Text watermark (플랫폼 문서)](https://platform.claude.com/docs/en/build-with-claude/text-watermark) · [릴리스 노트 2026-11-03](https://platform.claude.com/docs/en/release-notes/api) · [How Claude marks AI-generated content (지원 문서)](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content) · [How Claude's text watermarking works (뉴스룸, 2026-08-14)](https://www.anthropic.com/news/claude-text-watermark) · [Claude Content Checker](https://claude.com/check-files) (전부 **2026-11-06 확인**)

## 요약

**2026-11-03 릴리스 노트가 Claude Mythos 5, Opus 4.7, Sonnet 4.6 의 텍스트에도 워터마크가 들어간다고 적었음.** 여기까지는 코드에 아무것도 안 하면 되는 변경임. 플랫폼 문서가 **"의미·품질·가독성을 바꾸지 않고, 토큰이나 숨은 문자를 더하지 않으며, 요청이나 응답 처리를 고칠 필요가 없다"**고 못 박음.

손이 가는 자리는 한 줄뿐인데 그 한 줄이 조용함. **Sonnet 4.6 에서 `temperature`·`top_p`·`top_k` 중 하나라도 비기본값으로 두면 그 응답에 워터마크가 안 붙을 수 있음.** 에러도 없고 응답 필드도 없음. 그리고 **스스로 확인할 방법이 없음.** 텍스트 검출은 private preview 이고, 무료 Content Checker 는 파일만 본다고 자기 페이지에 적어 둠.

그래서 이 문서의 결론은 짧음. 워터마크에 기대는 설계라면 **세 파라미터를 아예 보내지 말 것.** 반대로 워터마크를 근거로 "이 텍스트는 Claude 가 썼다"를 판정하는 기능은 만들지 말 것. 1차 출처들이 전부 그렇게 못 쓴다고 적어 뒀음.

## 어느 모델에 무엇이 붙는지

지원 문서가 모델별 표를 두는데, 셀을 평평하게 늘어놓는 형태라 네 줄은 체크가 어느 열인지 복원하지 못했음. 확인된 것만 적음.

| 모델 | 확인된 것 |
|---|---|
| Fable 5.1, Fable 5, Mythos 5.1, Opus 5.5, Opus 5, Opus 4.8, Sonnet 5.5, Sonnet 5, Haiku 5.5 | 1차 표면 텍스트, 클라우드 파트너 텍스트, 파일 Content Credentials 셋 다 체크 |
| Mythos 5, Opus 4.7 | 같은 셋이지만 클라우드 파트너 텍스트에 `*` 가 붙음 |
| Sonnet 4.6 | 1차 표면과 클라우드 파트너 텍스트에 `†`, 클라우드 쪽에는 `*` 도 같이 붙음 |
| Opus 4.6, Opus 4.5, Sonnet 4.5, Haiku 4.5 | 체크가 하나씩만 있고 **어느 열인지 확인할 수 없었음** |

각주 둘이 표 아래에 있고 문언은 이것임.

> "*Rollout on Amazon Bedrock will complete by November 10."

> "† For Sonnet 4.6, outputs may not reliably carry the text watermark when `temperature`, `top_p`, or `top_k` is set to a non-default value."

표식 방식은 둘로 갈림. **텍스트는 생성 과정에 박히는 통계적 워터마크**이고, **파일은 C2PA Content Credentials 서명**임. 후자는 코드 실행 툴이 만든 PNG·JPEG 같은 파일에 붙고, 플랫폼이 Claude 의 파일 생성을 제공하는 곳에서만 해당됨. 즉 **Messages API 로 텍스트만 받는 앱에는 C2PA 쪽이 아예 안 걸림.**

적용 범위는 1차 표면(Claude Platform, Claude, Claude Code, Claude Cowork, Claude Tag)과 클라우드 파트너(AWS, Google Cloud, Microsoft Foundry) 양쪽임. 기준선도 문서가 적어 둠.

> "Claude models launched in the EU on or after August 2, 2026 will support machine-readable marking at launch."

날짜가 EU AI Act 쪽 시점이라는 것이 여기서 읽힘. 그 이전 모델에는 소급해서 붙이는 중이고, 그래서 위 표가 모델마다 상태가 다른 모양이 됨.

## 왜 샘플링 파라미터가 걸리는가

뉴스룸 글이 구조를 설명함. 워터마크는 텍스트에 무엇을 **덧붙이는** 것이 아니고, **토큰을 고를 때 쓰는 무작위성의 출처를 바꾸는** 것임.

> "watermarking uses the key and a few words that come before to settle what word the model should pick"

그리고 같은 글이 **"Nothing is added to the text and there are no hidden characters;"** 와 **"Watermarking doesn't require extra tokens, and will not be more expensive;"** 로 비용·형태 영향을 부정함. 방식은 Google DeepMind 의 SynthID-Text 계열이라고 밝힘.

여기까지 읽으면 샘플링 파라미터가 왜 걸리는지가 바로 나옴. 세 파라미터는 **그 선택 분포를 직접 건드리는 값들**임. 플랫폼 문서의 지시가 그래서 "조심해서 쓰라"가 아니라 아예 빼라는 쪽임.

> "If you rely on the watermark, omit all three parameters from your requests, including any that a framework or client library sets for you."

**뒷구절이 실무에서 더 중요함.** 직접 안 넣었어도 래퍼가 넣어 주는 경우가 흔하기 때문임. Swift 쪽에서 자체 클라이언트를 만들어 `temperature` 를 기본 상수로 채워 두는 구조라면, 코드 어디에도 "0.7" 을 쓴 기억이 없는데 매 요청에 실려 나감. 요청 바디를 한 번 찍어 보고 세 키가 없는지 확인하는 것이 유일한 점검 방법임.

## 400 으로 막는 모델과 조용히 빠지는 모델

이 저장소에는 같은 세 파라미터가 **에러로 막히는** 사례가 이미 있음. [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) 은 `temperature`·`top_p`·`top_k` 비기본값을 Fable 5 와 동일하게 금지함. 보내면 요청이 실패하므로 배포 전에 드러남.

Sonnet 4.6 은 반대임. **같은 파라미터가 허용되고, 대신 워터마크가 빠짐.** 요청은 200 이고 응답 품질도 그대로라서 아무 신호가 없음.

| 모델 | 비기본값 샘플링을 보냈을 때 | 언제 알게 되는가 |
|---|---|---|
| Fable 5.1, Fable 5 | 400 으로 거절 | 첫 호출에서 바로 |
| Sonnet 4.6 | 통과. 워터마크가 안 붙을 수 있음 | **자력으로는 알 수 없음** |

모델 하나를 갈아끼우면 같은 코드가 "에러 나는 구성"에서 "조용히 표식 없는 구성"으로 바뀐다는 뜻임. 라우터나 폴백을 두고 모델을 섞어 쓰는 쪽이면 이쪽이 더 성가신 조합임.

## 앱에서 실제로 걸리는 것

**첫째, 출력 형태에는 영향이 없음.** 토큰도 문자도 안 늘어나므로 글자 수 계산, 말풍선 레이아웃, 클립보드 복사, 로컬 저장 전부 그대로임. 이 변경 때문에 고칠 UI 는 없음.

**둘째, 검증은 못 함.** 두 경로가 이렇게 갈림.

| 도구 | 보는 것 | 접근 |
|---|---|---|
| Claude Content Checker | 파일의 Content Credential. 페이지가 **"The tool does not check text."** 라고 명시 | 무료 |
| Detection API | 텍스트 워터마크 | **private preview.** 규제기관·연구자·자기 출력을 확인하는 기업 같은 적격 조직 대상 |

지원 문서가 **"We plan to expand access to the detection API over time."** 까지만 적음. 즉 **"우리 앱이 내보내는 텍스트에 워터마크가 실제로 들어 있나"를 지금 확인할 방법이 개인 개발자에게는 없음.** 할 수 있는 것은 세 파라미터를 안 보내는 것뿐이고, 그게 맞게 동작하는지는 믿는 수밖에 없음.

**셋째, 워터마크는 AI 고지를 대신하지 않음.** 사람 눈에 안 보이는 표식이므로, [Anthropic 2026 사용 정책](../industry/2026-10-08-anthropic-usage-policy-2026.md)이 요구하는 **"대화 시작 시점 또는 제품 UI"** 의 AI 고지와는 다른 층위임. [App Store 심사 5.1.2(i)](../industry/2026-08-21-app-store-ai-review-rules.md)의 서드파티 전송 공개·동의도 마찬가지로 UI 에서 해야 하는 일임. 워터마크가 붙는다고 줄어드는 의무가 하나도 없음.

**넷째, 반대 방향의 기대를 걸면 안 됨.** 사용자 제출물이 AI 생성인지 가리는 기능을 워터마크로 만들 수 없음. 지원 문서가 한계를 직접 적음.

- 표식이 검출돼도 결론이 아님: **"A detected mark provides a signal that content was processed by Claude, but is not fully conclusive."** 교정·번역·요약만 맡긴 경우도 같은 표식이 붙고, 그 뒤에 내용이 바뀌었을 수도 있음
- 표식이 없어도 결론이 아님: **"Lack of a detected mark doesn't mean the content wasn't AI-generated or processed."** 이유로 아주 짧은 문단, 형식 변환·재저장·스크린샷으로 메타데이터가 떨어진 파일, 그리고 비기본값 샘플링이 열거됨
- 뉴스룸 글은 **"A watermark can only determine that Claude was likely involved"**, **"it can't tell whether the text was written by a different AI"** 로 더 좁힘. 가벼운 편집은 대개 살아남지만 전면 재작성은 지운다고 적었고, 사실 위주 문단·교정·코드 출력처럼 Claude 가 고를 여지가 적은 텍스트에서는 신호가 희박해짐

iOS 앱에 넣을 만한 기능으로 바꿔 읽으면, **"AI 작성 여부 판정"은 지금 만들 수 있는 기능이 아니고, 만들어도 1차 출처가 근거로 못 쓴다고 적어 둔 상태임.**

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude Fable 5.1](../models/2026-09-01-claude-fable-5-1.md) | 그 문서의 "콘텐츠 출처 표시" 한 줄이 여기서 모델 표와 샘플링 단서로 펼쳐짐. 비기본값 샘플링을 400 으로 막는 쪽과 조용히 빠지는 쪽의 대비 |
| [Anthropic 2026 사용 정책 개정](../industry/2026-10-08-anthropic-usage-policy-2026.md) | AI 고지 의무. 워터마크는 사용자에게 안 보이므로 고지를 대신하지 못함 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md) | 5.1.2(i) 의 공개와 동의. 출처 표식과 심사 공개는 서로 다른 일 |
| [Foundation Models 허용 사용 요건](../industry/2026-10-19-foundation-models-acceptable-use.md) | 금지 항목에 워터마크·콘텐츠 크리덴셜 제거가 이름으로 들어 있음. 그 조항은 Apple 프레임워크 사용 요건이라 Claude 출력에 직접 걸리는 것은 아니지만, 두 백엔드를 같이 쓰는 앱이 파일을 재인코딩하면 그쪽에서 먼저 걸림 |
| [Models API의 line 과 capabilities](./2026-10-01-models-api-line-and-capabilities.md) | 워터마크 지원 여부도 `capabilities` 에 없음. "API 가 안 알려주는 모델별 제약" 목록이 하나 늘어남 |
| [거절 응답의 청구와 fallback](./2026-09-24-refusal-billing-and-fallback.md) | 모델을 갈아타는 경로. 폴백으로 모델이 바뀌면 워터마크 조건도 같이 바뀜 |

## 짚어야 할 것

- **플랫폼 문서와 지원 문서가 어긋남.** 플랫폼 문서는 샘플링 단서를 **Sonnet 4.6 에만** 적지만, 지원 문서의 한계 목록은 **"It was generated by Claude Opus 4.6 or Claude Sonnet 4.6 with a non-default `temperature`, `top_p`, or `top_k`."** 로 둘을 적음. 어느 쪽이 최신인지는 확인할 수 없었음. **넓은 쪽(둘 다)으로 가정하는 편이 안전함**
- **기본값을 명시적으로 보내는 것이 "비기본값"에 해당하는지 확인할 수 없었음.** 문서가 "omit all three" 라고만 적고 값으로 가르는 문언을 안 둠. 비워 두는 쪽이 안전함
- **모델 표의 네 줄은 열을 복원하지 못했음.** Opus 4.6, Opus 4.5, Sonnet 4.5, Haiku 4.5 가 1차 표면만인지 클라우드 파트너까지인지는 **확인할 수 없었음**
- **검출 정확도나 오탐률 수치가 어느 1차 출처에도 없음.** 뉴스룸 글이 **"Detecting a watermark also doesn't work well on small samples"** 로 짧은 표본에서 약하다고만 적음. 숫자로 임계값을 잡을 근거가 없음
- **비용 영향이 없다는 것은 뉴스룸 글의 서술임.** 청구 데이터로 대조하지는 못했음
- **릴리스 노트가 거론한 Mythos 5, Opus 4.7, Sonnet 4.6 은 이 저장소에 문서가 없는 모델임.** 이번에 확인한 것은 표와 각주까지이고, 해당 모델들의 다른 제약은 대조하지 않았음
- **Detection API 의 적격 조건은 문서가 예시로만 적음.** 규제기관·연구자·자기 출력을 확인하는 기업이 예시인데, 신청 심사 기준이나 처리 기간은 확인할 수 없었음

## 유효기간

**2026-11-06 확인 기준**임. 모델 표는 모델이 추가될 때마다 늘어나므로 위 표를 고치지 말고 지원 문서에서 새로 받을 것. 다시 볼 때 확인할 것 셋. **Bedrock 롤아웃이 2026-11-10 에 끝나 `*` 각주가 사라졌는지**(사라지면 클라우드 파트너 경로의 유보가 풀림), **텍스트 검출이 private preview 를 벗어났는지**(벗어나면 자기 출력을 자력으로 점검할 수 있게 되므로 이 문서의 "검증은 못 함" 절이 통째로 바뀜), 그리고 **플랫폼 문서와 지원 문서의 샘플링 단서 범위가 한쪽으로 정리됐는지.** 시작점은 [릴리스 노트](https://platform.claude.com/docs/en/release-notes/api)에서 watermark 항목이 더 붙었는지임.
