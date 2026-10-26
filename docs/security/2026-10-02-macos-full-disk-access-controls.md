---
title: macOS Full Disk Access 제한 예고, Apple 이 권한을 좁히는 근거로 AI 에이전트를 댐
source: https://developer.apple.com/news/?id=p6zjojqw
author: Apple
published: 2026-10-02
collected: 2026-10-03
tags: [apple, agent-security, permissions, privacy, macos, coding-agent]
---

출처: [Updates to Full Disk Access in macOS](https://developer.apple.com/news/?id=p6zjojqw) (2026-10-02) · [Change Privacy & Security settings on Mac](https://support.apple.com/guide/mac-help/change-privacy-security-settings-on-mac-mchl211c911f/mac) (macOS 사용 설명서, Full Disk Access 절) (둘 다 **2026-10-03 확인**)

## 요약

Apple 이 **2026-10-02** 개발자 공지로 macOS 의 **Full Disk Access(FDA) 에 추가 통제를 넣겠다**고 예고함. 분량은 두 단락이고 그 안에 세 가지가 들어 있음. 첫째, FDA 는 백업 앱을 돌게 하려고 **다른 권한 통제를 사실상 우회하는 권한**이라는 성격 규정. 둘째, 일부 개발자가 이것을 사용자가 충분히 이해하지 못한 상태에서 **파일·메일·메시지·브라우징 기록까지 노출하는 방식으로 쓰고 있다**는 지적. 셋째, 근거로 댄 것이 **AI 에이전트**임. 원문 문장이 "As AI agents become increasingly capable and autonomous, the risks associated with this level of access will grow substantially" 임. 앞으로는 **"very explicit user action"** 을 거쳐야만 이 권한을 줄 수 있게 만들겠다고 적음.

**이 공지에 없는 것이 더 중요함.** 시점, 어느 OS 버전, 통제의 형태, 이미 부여된 권한을 어떻게 다루는지가 **전부 안 적혀 있음.** 즉 지금 할 수 있는 것은 설계 전제를 바꾸는 것까지고, 코드를 고치는 단계는 아님. 그리고 이 저장소 기준으로 이 공지의 값어치는 따로 있음. [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md)에 정리한 대로 Apple 자신이 한 달 반 전에 **코드 서명된 에이전트에게 디렉터리 트리 권한을 장기간 주는 모델**을 IDE 에 넣었고 그것을 감시하는 계층은 **켜야 도는 것**임. 같은 회사가 OS 권한 쪽에서는 반대 방향으로 조이겠다고 말한 것이라, 둘 중 어느 쪽이 에이전트 파일 접근의 기준선이 되는지가 아직 안 정해진 상태로 보는 편이 맞음.

## Apple 이 말한 것과 말하지 않은 것

공지 전문이 두 단락이라 그대로 갈라 둘 수 있음.

| 항목 | 공지가 적은 것 |
|---|---|
| FDA 의 성격 | "Full Disk Access largely sidesteps these controls in order to allow backup apps to function properly on the Mac" |
| 문제 진단 | 일부 개발자가 "in ways that could put users at risk" 로 쓰고 있고, 노출 범위가 "files, mail, messages, and even browsing history" 임 |
| 통신 앱의 추가 피해 | 사용자가 **대화한 상대방의 프라이버시까지** 함께 깨진다고 적음 |
| 변경 방향 | "additional controls" 를 넣어 "very explicit user action" 없이는 부여되지 않게 함 |
| 근거 | AI 에이전트가 더 유능하고 자율적이 될수록 이 수준의 접근이 만드는 위험이 "grow substantially" |

반대로 **안 적힌 것**이 이것들임. 적용 시점과 OS 버전, "additional controls" 가 UI 변경인지 엔타이틀먼트 심사인지 배포 경로별 제약인지, 기존에 켜져 있는 FDA 가 유지되는지 다시 묻는지, 그리고 **문제가 된 사용 방식의 구체적인 예**. 이 공지는 개발자 뉴스 항목이고 문서나 릴리스 노트가 아니므로 API 수준의 서술이 아예 없음.

## FDA 가 실제로 열어 주는 범위

사용자 쪽 문구가 범위를 가장 정확히 말해 줌. macOS 사용 설명서의 Full Disk Access 절임.

> "Allow apps to access all files on your computer, including data from other apps (for example, Mail, Messages, Safari, and Home), data from Time Machine backups, and certain administrative settings for all users on this Mac."

여기서 읽어야 할 것은 "all files" 가 아니라 뒤쪽임. **다른 앱의 데이터**, **Time Machine 백업**, 그리고 **이 Mac 의 모든 사용자에 걸친 일부 관리 설정**까지가 한 스위치에 묶여 있음. 권한 하나가 앱 경계와 사용자 경계를 동시에 넘음. 공지가 통신 상대방의 프라이버시를 따로 언급한 이유가 이 구조에 있음. 메시지 앱의 데이터를 읽는 것은 그 기기 사용자 한 명의 동의로 끝나는 일이 아님.

이 저장소가 반복해 적은 형태가 여기 그대로 있음. [OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](./2026-08-07-openai-hugging-face-agent-incident.md)에서 **파일 쓰기 권한 하나**가 에이전트끼리 비공식 채널을 만드는 표면이 됐음. 권한의 이름이 좁아 보이는 것과 그 권한이 실제로 닿는 범위가 좁은 것은 다른 문제임.

**확인할 수 없었음.** 앱이 FDA 를 코드로 요청하는 경로가 있는지, 배포 방식(Mac App Store, Developer ID, 직접 배포)에 따라 취급이 갈리는지는 위 두 출처로는 확인되지 않음. 사용 설명서가 보여 주는 것은 사용자가 시스템 설정의 개인정보 보호 및 보안에서 앱을 목록에 넣는 경로까지임. 개발자 문서 쪽 서술은 이 공지가 가리키는 변경이 실제 문서로 내려온 뒤에 다시 봐야 함.

## 이 공지가 이 저장소의 결론과 만나는 지점

[사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md)가 내린 결론이 이것임. **승인 정확도를 올리려 하지 말고 위험한 것을 승인 대상에서 빼는 편이 싸고 확실함.** Apple 의 이번 조치는 방향이 같은데 층이 하나 아래임. 사람이 FDA 를 켤지 말지 판단하는 정확도를 올리려는 대신, 켜지는 경로 자체를 좁히겠다는 것임.

다만 **"very explicit user action" 은 그 결론과 반쯤만 겹침.** 더 강한 확인 절차는 여전히 사람의 판단을 요구하는 설계이고, 위 문서가 측정한 것은 바로 그 판단의 정확도가 생각보다 낮다는 것이었음. 확인 단계를 늘리면 잘못 켜는 비율은 줄지만 0 이 되지는 않음. 이 조치를 "FDA 를 받은 앱은 이제 안전하다" 로 읽으면 안 되는 이유가 그것임.

반대 방향의 긴장도 같이 봐야 함. Xcode 27 쪽 모델을 옆에 놓으면 이럼.

| 표면 | 지금 서 있는 형태 |
|---|---|
| macOS 의 FDA | 범위가 넓고, Apple 이 **부여 경로를 좁히겠다**고 예고함 (시점 미정) |
| Xcode 27 의 에이전트 파일 접근 | 코드 서명된 에이전트에게 **디렉터리 트리 권한을 장기간** 주고, 무인용으로 `--unsafe-always-allow-all-agents` 가 있음 |
| Xcode 27 의 감시 계층 | 에이전트와 그 자식 프로세스까지 감시하지만 **Coding Intelligence 설정에서 켜야** 돎 |

세 줄이 한 기기 안에서 겹침. **이번 공지가 Xcode 의 코딩 에이전트나 `xcrun mcp-server` 를 범위에 포함하는지는 공지에 없음.** 포함하지 않는다고 적힌 것도 아님. 문언만으로는 FDA 라는 특정 권한에 대한 이야기로 읽는 것이 맞고, 에이전트 파일 접근 전반에 대한 정책 선언으로 확대 해석할 근거는 이 문서에 없음.

## 앱을 내는 쪽에서 지금 할 것

공지에 시점이 없으므로 코드 변경은 아직 할 일이 아님. 전제를 바꿔 두는 것까지가 지금 할 수 있는 것임.

1. **FDA 에 기대는 Mac 앱이 있으면 부여율이 떨어지는 쪽으로 가정함.** 백업, 동기화, 파일 인덱싱 계열이 여기 걸림. 권한이 없을 때 기능이 통째로 죽는 구조인지, 사용자가 고른 폴더만으로 축소 동작이 가능한 구조인지가 갈림길이 됨
2. **지금 FDA 를 쓰고 있다면 실제로 읽는 경로를 목록으로 만들어 둠.** 공지가 지목한 것은 권한 자체가 아니라 사용자가 이해하지 못한 범위까지 읽는 사용 방식임. 좁은 권한으로 대체 가능한 경로가 섞여 있는지는 그 목록이 있어야 보임
3. **Mac 에서 코딩 에이전트를 돌리고 있으면 Coding Intelligence 의 파일시스템 감시 계층을 켬.** 이것은 이번 공지와 무관하게 이미 할 수 있는 일이고, 켜지 않으면 없는 것과 같음
4. 개발자 공지는 문서로 내려오기 전의 예고이므로, 설계를 바꾸는 판단은 **릴리스 노트나 프레임워크 문서에 통제의 형태가 적힌 뒤**에 함

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Xcode 27의 에이전트 표면](../agents/2026-08-26-xcode-27-agent-surface.md) | 코드 서명된 에이전트의 장기 디렉터리 권한과 켜야 도는 감시 계층. 같은 기기에서 이번 조치와 반대 방향으로 서 있음 |
| [사람이 에이전트 명령 승인에서 위협 3건 중 1건을 놓친다](./2026-08-05-agent-approval-miss-rates.md) | 승인 정확도를 올리는 대신 승인 대상을 줄인다는 결론. Apple 이 OS 권한 층에서 같은 선택을 함 |
| [OpenAI 학습용 에이전트가 Hugging Face를 공격한 사고](./2026-08-07-openai-hugging-face-agent-incident.md) | 좁아 보이는 파일 권한 하나가 실제로는 넓은 표면이 되는 형태 |
| [iOS 27 · Xcode 27 정식 출시](../industry/2026-09-14-ios-27-ai-apis-shipped.md) | 같은 사이클의 개발자 공지 묶음. 제출 요건과 Rosetta 타임라인은 그쪽에 정리돼 있음 |
| [앱에 AI를 붙일 때 걸리는 App Store 심사 조항](../industry/2026-08-21-app-store-ai-review-rules.md) | 사용자 동의의 형식을 Apple 이 어디까지 요구하는지. 약관에 묻어두면 공개로 안 쳐준다는 그쪽 조항과 "very explicit user action" 이 같은 태도임 |

## 짚어야 할 것

- **예고이고 스펙이 아님.** 시점, OS 버전, 통제의 형태, 기존 부여분의 취급이 전부 공지에 없음. 이 문서가 확인한 것은 Apple 이 그렇게 하겠다고 말했다는 사실까지임
- **"additional controls" 가 무엇인지 추측하지 않았음.** UI 흐름 변경, 엔타이틀먼트, 심사, 배포 경로별 제약 중 어느 것인지 공지가 말하지 않음
- **Apple 이 문제 삼은 사용 방식의 구체적인 예가 공지에 없음.** 어떤 앱이 어떻게 썼는지, 사후 조치가 있었는지는 확인되지 않음
- **2차 보도만 있어 제외한 것이 있음.** 여러 매체가 이 발표의 배경으로 특정 상시 구동형 AI 에이전트 제품들을 이름으로 거론하는데, Apple 공지에는 제품 이름이 하나도 없음. 이 문서는 그 연결을 넣지 않았음
- **FDA 의 개발자 쪽 취급은 1차 출처로 확인하지 못했음.** 요청 API 의 존재 여부, 배포 방식별 차이, Endpoint Security 같은 FDA 를 요구하는 프레임워크들이 이번 변경에 어떻게 걸리는지는 공지와 사용 설명서 범위를 넘음. 개발자 포럼 답변은 1차 출처로 쓰지 않았음
- **원문에 지시문이나 명령문 형태의 문장은 없었음.** 두 단락 모두 정책 서술임
- 이 공지는 macOS 에 대한 것임. iOS 에 대응하는 권한이 있다는 서술은 공지에 없음

## 유효기간

**2026-10-03 확인 기준**이고, 공지 자체는 **2026-10-02** 날짜가 박힌 기록이라 문언은 낡지 않음. 낡는 것은 "아직 형태가 안 나왔다" 는 이 문서의 상태임. 다시 볼 때 확인할 것 셋. **통제의 형태가 릴리스 노트나 프레임워크 문서에 적혔는지**, **적용 OS 버전과 시점이 나왔는지**, 그리고 **범위가 Xcode 의 코딩 에이전트까지 닿는지**. 출발점은 [개발자 뉴스](https://developer.apple.com/news/)와 macOS 27.x 릴리스 노트임.
