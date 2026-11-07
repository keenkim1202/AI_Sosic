---
title: IP allowlisting, 키 없이 앱에서 직접 부르는 경로와 정면으로 부딪히는 통제
source: https://platform.claude.com/docs/en/manage-claude/ip-allowlist
author: Anthropic
published: 2026-11-05
collected: 2026-11-07
tags: [agent-security, claude, anthropic, permissions, gateway, compliance]
---

출처: [IP allowlisting](https://platform.claude.com/docs/en/manage-claude/ip-allowlist) · [API 릴리스 노트 2026-11-05](https://platform.claude.com/docs/en/release-notes/api) (둘 다 **2026-11-07 확인**)

## 요약

**2026-11-05 에 조직 단위 IP allowlist 가 베타로 붙었음.** 목록에 없는 주소에서 온 요청은 거부되고, Messages API 는 **403 `permission_error`** 와 함께 `Your IP address isn't in this organization's allowed range.` 를 돌려줌. 목록은 IPv4·IPv6 단일 주소나 CIDR 을 받고 **조직당 1,000개**까지, 범위는 **IPv4 는 `/8`, IPv6 는 `/32` 보다 넓게** 잡을 수 없음. 설정은 Console 의 Identity and access 페이지이고 조직의 admin·owner 만 건드릴 수 있음.

목적이 한 줄로 적혀 있음. 유출된 키를 **못 쓰게 만드는 게 아니라 쓸 수 있는 자리를 좁히는 것**임.

> "IP allowlisting protects your organization by only allowing covered requests to come from specific IP ranges. This means that if an API key is leaked, it is much harder for an attacker to use the key."

문서 자신이 이것만으로 끝내지 말라고 먼저 적음. **"An allowlist limits where a key works, but not for how long."** 그래서 짧은 키 만료를 같이 걸거나 Workload Identity Federation 으로 옮겨 **정적 키를 아예 두지 않는 쪽이 더 안전하다**고 못 박음. 이 저장소의 [Usage and Cost Analytics API](../practices/2026-10-29-usage-cost-analytics-api.md)에서도 같은 권고가 나왔으니, allowlist 는 WIF 로 못 간 자리에 임시로 세우는 울타리로 읽는 편이 맞음.

**이 저장소에 가장 크게 걸리는 지점은 기능이 아니라 구조임.** 목록은 **조직 전체**를 덮고(`This list of ranges covers your entire organization.`) 공인 주소만 적을 수 있음. 그런데 [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md)의 App Attest 경로는 요청이 **사용자 기기에서 Anthropic 으로 직접** 감. 사용자 기기의 주소는 열거할 수 있는 집합이 아님. 즉 백엔드만 allowlist 로 묶고 앱은 예외로 두는 구성이 **설정 단위 자체로 불가능함.**

## 덮는 범위

| 덮는 것 | 붙는 조건 |
|---|---|
| Claude Console UI | 접근 자체가 제한됨 |
| Claude API 의 Messages · Message Batches · Token Counting · Models · Files · Skills | **API 키 또는 WIF 토큰**으로 부를 때 |
| Managed Agents 의 Agents · Sessions · Environments · Vaults · Memory stores | 같음. 단 **엔드포인트까지만** |
| Admin API | 이 조직만 덮는 키일 때 |
| Compliance API | 같음 |

덮지 않는다고 명시된 것이 둘 묶음임. 하나는 **클라우드 경유 전부**(Claude in Amazon Bedrock, Claude Platform on AWS, Claude Models on Google Cloud, Claude in Microsoft Foundry). 다른 하나는 **Managed Agents 런타임의 네트워킹**임.

> "It doesn't apply to any networking in the Claude Managed Agents runtime."

클라우드 경유가 빠지는 모양이 눈에 익어야 함. App Attest 도 같은 넷에서 안 됨. **키 없이 부르는 경로와 그 키를 지키는 통제가 둘 다 1차 API 에만 있다**는 뜻이고, 조직 요건이 Bedrock 경유라면 이 둘을 동시에 잃음.

Managed Agents 쪽은 방향이 반대인 통제와 짝지어 읽어야 함. allowlist 는 **들어오는 요청의 출처**를 보고, [에이전트 웹 egress 통제 3층](./2026-10-07-managed-agents-web-egress.md)의 `allowed_hosts` 는 **샌드박스가 나가는 목적지**를 봄. 목록이 둘이고 서로를 덮지 않음. 에이전트를 만드는 호출은 allowlist 에 걸리고, 그 에이전트가 세션 중에 어디로 나가는지는 allowlist 가 모름.

## App Attest 경로에서 무엇을 결정해야 하나

[Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md)가 정리한 인증 선택지가 셋이었음. `.appAttest` 는 앱에 키도 백엔드도 없고, `.proxied` 는 자기 백엔드가 키를 붙이고, `.apiKey` 는 개발 전용임. allowlist 를 켜면 셋의 값이 이렇게 갈림.

- **`.proxied` 가 유일하게 이득을 봄.** 릴레이 서버의 egress 주소는 고정이고 열거할 수 있음. 키가 유출돼도 그 주소 밖에서는 안 돎
- **`.appAttest` 는 켤 수 없음.** 사용자 기기는 모바일 데이터와 집 와이파이에서 들어오고 주소가 통지 없이 바뀜. 공유 프록시 주소를 적어 억지로 통과시키는 길이 있긴 한데, 문서가 그 선택의 대가를 직접 적어 뒀음: "If you add a shared proxy address, note that an attacker with a leaked key or a stolen sign-in could potentially use it through the proxy."
- **`.apiKey` 는 영향이 없음.** 어차피 개발 중에만 쓰는 것이고 개발 머신 주소를 적으면 됨

그래서 결론이 이렇게 됨. **allowlist 를 조직 정책으로 켜기로 하면 App Attest 가 지우려던 백엔드가 되살아남.** App Attest 의 값어치가 "키를 숨길 서버를 안 세워도 된다"였는데, allowlist 의 값어치는 "서버가 있고 그 서버의 주소가 고정이다"에서 나옴. 둘이 같은 가정을 반대로 깔고 있음.

한 가지 **갈라낼 수 없는 것**이 남음. allowlist 가 덮는 조건을 문서는 "with an API key or a Workload Identity Federation token" 으로 적음. App Attest 가 기기에 발급하는 것은 **1시간 수명의 액세스 토큰**이고 둘 중 어느 이름도 아님. 그 토큰이 covered 에 들어가는지 아닌지가 페이지에 안 적혀 있어서 두 갈래가 남는데, **어느 쪽이든 결론은 같음**. 덮인다면 allowlist 를 켜는 순간 앱이 403 으로 죽고, 덮이지 않는다면 allowlist 를 켜도 그 경로에는 보호가 없음. 앱에서 직접 부르는 트래픽을 **키 유출 통제의 사각지대로 보고 설계하는 쪽**이 안전함. 이 판단에 쓸 수 있는 유일한 확인된 사실은 그 토큰이 **워크스페이스 스코프이고 Messages API 호출만 인가한다**는 것뿐임.

## 켜는 절차에서 실제로 걸리는 것

세 단계인데 함정이 2단계와 3단계에 몰려 있음.

- **저장할 때 검사하는 것은 내 현재 주소 하나뿐임.** 문서 문언이 이럼: "But it does not check your servers, your teammates, or the address you might connect from tomorrow." 데이터센터, CI, 오피스, VPN, 프록시, 자체 호스팅 샌드박스의 워커를 **공인 IPv4 와 IPv6 양쪽으로** 미리 세어 적어야 함
- **사설 범위를 적으면 안 됨.** 적을 것은 트래픽이 내 네트워크를 **떠나는** 주소임. `10.0.0.0/8` 같은 것을 적어도 아무것도 통과하지 않음
- **적용에 최대 15분 걸림.** 그리고 문서가 직접 경고하는 자리가 **일·주 단위로만 도는 잡**임. 켠 직후 15분을 지켜봐도 주 1회 배치는 며칠 뒤에 403 으로 터짐
- **잠금 탈출구가 기본 권장이지만 공짜가 아님.** "Allow admins to access this settings page from any IP address" 를 켜 두라고 권하면서, 그 대가로 **admin 세션이 털리면 공격자가 원격에서 목록을 고칠 수 있다**고 같은 문단에 적음. 그래서 Compliance API 를 세워 두면 Activity Feed 에 목록 변경이 남으니 감사하라고 이어 붙임. 즉 이 토글은 **가용성과 변경 감사를 교환하는 선택**이고, 감사 쪽을 안 세우면 한쪽만 내준 것이 됨

완전히 잠겼을 때의 마지막 수단은 Anthropic 지원에 allowlist 를 꺼 달라고 요청하는 것임. 혼자 쓰는 조직에서 탈출구 토글을 끄는 것은 **지원 티켓이 복구 경로가 된다**는 뜻이므로 권할 만하지 않음.

## 이 저장소의 다른 문서와 겹치는 지점

| 문서 | 겹치는 지점 |
|---|---|
| [Claude for Foundation Models](../agents/2026-10-25-claude-for-foundation-models.md) | App Attest 의 키 없는 직접 호출과 이 통제가 반대 가정을 깔고 있음. `.proxied` 로 돌아가야 하는 이유가 하나 늘었음 |
| [User Profiles, 최종 사용자 귀속](../practices/2026-10-20-user-profiles-end-customer-attribution.md) | 둘 다 폭발 반경을 줄이는 장치인데 줄이는 대상이 다름. 프로필은 **집행 조치**의 반경, allowlist 는 **유출된 키**의 반경 |
| [에이전트 웹 egress 통제 3층](./2026-10-07-managed-agents-web-egress.md) | 같은 Managed Agents 에 걸리는 목록인데 방향이 반대임. 입구와 출구가 각자 목록을 들고 서로를 안 덮음 |
| [Usage and Cost Analytics API](../practices/2026-10-29-usage-cost-analytics-api.md) | 거기서 나온 "장기 키를 두지 말고 WIF 로" 권고와 같은 문단이 이 페이지에도 있음. allowlist 는 그 권고의 대체물이 아니라 보조물로 적혀 있음 |
| [Claude를 제품마다 어떻게 격리하는가](./2026-05-25-containing-claude-across-products.md) | 거기서 도메인 allowlist 가 유출을 막아주지 못한 사례가 나왔음. 주소 목록도 같은 성질이라 **반경 축소이고 차단이 아님** |

## 짚어야 할 것

- **베타이고 문서가 그렇게 적어 둠.** "Settings and behavior can change during the beta." 위의 숫자(1,000개, `/8`·`/32`, 15분)는 베타 기준이므로 켜기 전에 페이지를 다시 볼 것
- **직접 켜 보지 않았음.** 이 문서는 공식 문서 페이지와 릴리스 노트만 읽은 것이고, 403 응답 모양이나 15분 전파를 재현하지 않았음
- **App Attest 토큰의 covered 여부를 확인할 수 없었음.** 위에서 두 갈래로 적은 자리임. 페이지의 covered 목록이 인증 수단을 "API key or a Workload Identity Federation token" 둘로만 적고 앱 발급 토큰을 다루지 않음
- **Usage and Cost Analytics API 가 덮이는지 확인할 수 없었음.** covered 목록은 organization API 를 Admin API 와 Compliance API 둘로만 적는데, analytics 엔드포인트는 Admin API 키로 부르는 `/v1/organizations/...` 경로임. 같은 묶음으로 세는지 아닌지가 페이지에 없음
- **이 페이지의 절반은 이 저장소 범위 밖임.** Claude Enterprise(claude.ai) 조직용 절이 분량의 상당 부분이고, 거기 적힌 것은 사내 사용자 접근 통제임. 여기서는 Claude Platform 쪽, 즉 **내가 쓰는 API 키를 지키는 용도**만 정리했음
- 문서가 권하는 순서가 allowlist 를 **1순위로 두지 않는다**는 점을 흐리지 말 것. 1순위는 정적 키를 없애는 것이고 allowlist 는 그다음임

## 유효기간

**2026-11-07 확인 기준이고 기능은 공개 이틀째의 베타임.** 다시 볼 때 확인할 것 셋. **covered 목록에 message threads 나 analytics 같은 신규 엔드포인트가 이름으로 올라갔는지**(지금 목록은 2026-11-05 시점의 엔드포인트 집합이라 뒤에 붙는 것들이 자동으로 덮인다고 볼 근거가 없음), **App Attest 로 발급된 토큰의 취급이 문서에 적혔는지**(이 문서에서 유일하게 갈라내지 못한 자리임), 그리고 **베타 문언과 함께 숫자 제한이 바뀌었는지**. Bedrock·Google Cloud·Microsoft Foundry 로 확대되는지도 같이 볼 자리인데, 그 경로는 키 관리 주체가 클라우드 쪽이라 allowlist 가 Anthropic 쪽 기능으로 오지 않을 가능성이 더 큼.
