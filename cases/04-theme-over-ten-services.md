# A Theme in Front of Ten Services

**Setup** · Ten backend services behind one storefront theme
**Rule** · No page renders by fanning out to them
**Reason** · A storefront that is a distributed system fails like one

## Context

Behind this theme sit ten services: authentication, orders, membership, brand data, coupons, wishlist, profile, addresses, reviews, and search. Each exists because the platform has no adequate primitive for what it does.

Ten services is fine. Ten services *on the critical path of a page render* is not — that page's availability becomes the product of ten availabilities, and its latency becomes the maximum of ten latencies.

## Problem

**Everything wants to be on every page.** Membership tier affects pricing display. Wishlist state affects every product card. Coupon eligibility affects the cart badge. Left alone, a product listing page ends up calling five services before it can paint.

**Failure compounds silently.** With enough services on the render path, some service is always degraded, and the page is always slightly wrong in a way nobody can reproduce.

**Third-party SDKs add their own weight.** Social login, animation, carousels, and domestic identity verification each bring a script, and each has an opinion about when it initializes.

**Authentication is shared but the services are not.** A customer logs in once. Ten services need to know who they are, without ten separate session mechanisms.

## Approach

**Render the page first, then decorate it.** The page renders from platform data and the listing backend, and everything else arrives afterward. Wishlist markers, member pricing, and coupon state are applied to a page that is already painted and already usable. A degraded service means a missing badge, not a blank page.

**Coarse, slow-changing member state is projected into the platform, not fetched.** Tier and similar attributes are written onto the platform customer record when they change, so the theme reads them from the rendering context it already has. This is the same pattern as the ERP membership work in [commerce-backend-msa](https://github.com/hak2881/commerce-backend-msa) — cache the conclusion where the renderer already looks, and never let it be authoritative for a decision.

**One auth mechanism, one token.** A single sign-in produces a credential every service accepts. No service invents its own session, and adding an eleventh service does not add an eleventh login path.

**Per-component fetch boundaries, not per-page.** Each decorating component owns its own request, its own loading state, and its own failure state. There is no orchestration layer collecting responses from five services and rendering when all have returned — which would reintroduce exactly the coupling being avoided.

**Third-party SDKs load where they are used.** Animation, carousel, social, and verification scripts load in the sections that need them. On a theme with no bundler, this is also the only way to keep a page's script cost proportional to what the page actually does.

## Honest assessment

The service count is high for a storefront, and not all of it was designed — some services exist because a capability was needed before there was time to decide where it belonged. Brand data, search, and the listing backend in particular have overlapping responsibilities that a redesign would consolidate.

What holds the design together is not the number of services but the rule about the render path: the page must paint without them. That rule is what makes the count survivable, and it is the part worth keeping if the services were ever reorganized.

---

## 한국어 요약

**구성** — 스토어프론트 테마 하나 뒤에 백엔드 서비스 10개
**규칙** — 어떤 페이지도 그것들로 팬아웃해서 렌더하지 않는다
**이유** — 분산 시스템이 된 스토어프론트는 분산 시스템처럼 실패한다

이 테마 뒤에는 인증·주문·멤버십·브랜드·쿠폰·위시리스트·프로필·주소·리뷰·검색 10개 서비스가 있습니다. 플랫폼에 그 일을 할 만한 프리미티브가 없어서 하나씩 생긴 것들입니다. 서비스가 10개인 건 괜찮습니다. 문제는 그 10개가 페이지 렌더의 임계 경로에 있을 때입니다. 그러면 페이지 가용성은 열 개 가용성의 곱이 되고, 지연은 열 개 지연 중 최댓값이 됩니다.

**어려웠던 지점**

- **모든 게 모든 페이지에 있고 싶어 합니다.** 멤버십 등급은 가격 표시에, 위시리스트 상태는 모든 상품 카드에, 쿠폰 자격은 장바구니 뱃지에 영향을 줍니다. 놔두면 상품 목록 페이지 하나가 그려지기도 전에 서비스 다섯 개를 호출하게 됩니다.
- **장애가 조용히 누적됩니다.** 렌더 경로에 서비스가 충분히 많으면 항상 뭔가 하나는 degraded 상태입니다. 그래서 페이지는 아무도 재현하지 못하는 방식으로 늘 조금씩 틀립니다.
- **서드파티 SDK도 각자 무게를 더합니다.** 소셜 로그인, 애니메이션, 캐러셀, 본인인증이 저마다 스크립트를 들고 오고, 초기화 시점에 대한 의견도 저마다 다릅니다.
- **인증은 공유되지만 서비스는 아닙니다.** 고객은 한 번 로그인합니다. 서비스 10개가 그게 누구인지 알아야 하는데, 세션 메커니즘을 10개 두는 건 답이 아닙니다.

**접근**

- **페이지를 먼저 그리고, 그다음에 장식합니다.** 페이지는 플랫폼 데이터와 목록 백엔드로 렌더되고 나머지는 그 뒤에 도착합니다. 위시리스트 표시, 회원가, 쿠폰 상태는 이미 그려졌고 이미 쓸 수 있는 페이지 위에 얹힙니다. 서비스 하나가 아프면 뱃지가 안 뜨는 정도지, 빈 페이지가 뜨지는 않습니다.
- **성기고 느리게 바뀌는 회원 상태는 조회하지 않고 플랫폼에 투영합니다.** 등급 같은 속성은 바뀔 때 플랫폼 고객 레코드에 써두고, 테마는 이미 들고 있는 렌더링 컨텍스트에서 읽습니다. [commerce-backend-msa](https://github.com/hak2881/commerce-backend-msa)의 ERP 멤버십 작업과 같은 패턴입니다. 렌더러가 이미 보는 자리에 결론을 캐시해두되, 그 값을 판단의 근거로는 절대 쓰지 않습니다.
- **인증 메커니즘 하나, 토큰 하나.** 한 번 로그인하면 모든 서비스가 받아주는 자격증명이 나옵니다. 어떤 서비스도 자기 세션을 따로 만들지 않고, 11번째 서비스를 붙여도 11번째 로그인 경로가 생기지 않습니다.
- **fetch 경계는 페이지가 아니라 컴포넌트 단위.** 장식하는 컴포넌트가 각자 자기 요청과 로딩 상태, 실패 상태를 갖습니다. 서비스 다섯 개의 응답을 모아뒀다가 전부 돌아오면 렌더하는 오케스트레이션 계층은 두지 않았습니다. 그건 피하려던 결합을 그대로 다시 만드는 일입니다.
- **서드파티 SDK는 쓰이는 곳에서 로드합니다.** 번들러가 없는 테마에서는 페이지의 스크립트 비용을 그 페이지가 실제로 하는 일에 맞춰 유지하는 유일한 방법이기도 합니다.

**솔직한 평가** — 스토어프론트치고 서비스 개수가 많고, 전부 설계해서 나온 결과도 아닙니다. 어디에 둘지 정할 시간이 없는데 기능은 필요해서 생긴 서비스도 있습니다. 특히 브랜드, 검색, 목록 백엔드는 책임이 겹쳐서 다시 설계한다면 합쳐질 부분입니다.

이 설계를 지탱하는 건 서비스 개수가 아니라 렌더 경로에 대한 규칙입니다 — **페이지는 그것들 없이도 그려져야 한다.** 그 규칙이 있어서 개수가 견딜 만해지고, 서비스를 재편하더라도 이 규칙만은 남겨야 합니다.
