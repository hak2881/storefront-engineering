# Storefront Engineering at Scale

What happens to a hosted commerce theme when the catalog is 1,067 brands, the traffic is real, and the platform's native features stop being able to serve the pages.

Case studies from a Korean multi-brand fashion and beauty marketplace: a heavily customized production theme fronting ten backend services, with no build step, running on a platform that was not designed for a catalog this shape.

These are write-ups, not source. Client code, credentials, and infrastructure identifiers are deliberately absent.

## The situation

| | |
|---|---|
| Brands in catalog | 1,067 |
| Largest brands | 5,000+ products each |
| Theme surface | ~147 sections, ~121 snippets, ~48 templates, ~172 stylesheets, ~74 scripts |
| Build step | None — vanilla ES6 and Web Components, served directly by the platform CLI |
| Backend services behind it | 10 (auth, order, membership, brand, coupons, wishlist, profile, address, review, search) |
| Share of page traffic served from own backend | 90%+ |

That last row is the whole story. A commerce theme is supposed to render from the platform's own data. This one renders from services I operate, on nearly every page that matters, because the platform could not be made to do it — and [case 02](cases/02-when-native-offloading-fails.md) is the record of the three serious attempts to prove otherwise.

## Cases

| # | Case | Core problem |
|---|---|---|
| 01 | [Breaking up the files everyone edits](cases/01-breaking-up-shared-files.md) | Two files were the source of most merge conflicts and most regressions |
| 02 | [When platform-native offloading fails](cases/02-when-native-offloading-fails.md) | Three attempts to move load back onto the platform, and why each failed |
| 03 | [AJAX navigation on a theme that assumes page loads](cases/03-ajax-navigation-state.md) | Filter and listing state that survives navigation the theme never anticipated |
| 04 | [A theme in front of ten services](cases/04-theme-over-ten-services.md) | Rendering member state without making the storefront a distributed system |

## Positions this work argued for

**Merge conflicts are an architecture signal.** When several people keep colliding in the same file, the file is doing too much. The fix is structural, not procedural.

**A negative result is a deliverable.** Three documented failed attempts at platform-native offloading are worth more to the client than a fourth attempt, because they turn "why is our infrastructure bill this high" into an answered question.

**No build step is a real constraint, and mostly a good one.** No bundler means no dependency graph to reason about, no build cache to invalidate, and a stylesheet or script that can be traced from the page to the file with no tooling. It costs module ergonomics. On a theme this large, edited by several people, the trace-ability was worth more.

**The storefront must not become a distributed system.** Ten services behind it, but no page renders by fanning out to ten of them.

## Stack

`Liquid` · `Vanilla JavaScript (ES6, Web Components)` · `CSS` · `GSAP` · `Swiper` · platform CLI · Kakao SDK · domestic identity verification

---

## 한국어 요약

카탈로그가 브랜드 1,067개이고, 트래픽이 실제로 있고, 플랫폼 네이티브 기능이 더 이상 페이지를 감당하지 못하게 됐을 때 호스팅형 커머스 테마에 일어나는 일들.

국내 멀티브랜드 패션·뷰티 마켓플레이스 사례입니다. 빌드 단계 없이 백엔드 서비스 10개 앞에 서 있는 대규모 커스텀 프로덕션 테마이며, 플랫폼은 애초에 이런 형태의 카탈로그를 상정하지 않았습니다.

| | |
|---|---|
| 카탈로그 브랜드 수 | 1,067개 |
| 최대 브랜드 | 각 5,000개 이상 상품 |
| 테마 규모 | 섹션 ~147 · 스니펫 ~121 · 템플릿 ~48 · 스타일시트 ~172 · 스크립트 ~74 |
| 빌드 단계 | 없음 — 바닐라 ES6 + 웹 컴포넌트, 플랫폼 CLI가 직접 서빙 |
| 뒤에 있는 백엔드 서비스 | 10개 (인증·주문·멤버십·브랜드·쿠폰·위시리스트·프로필·주소·리뷰·검색) |
| 자체 백엔드가 처리하는 페이지 트래픽 비중 | 90% 이상 |

마지막 줄이 전부입니다. 커머스 테마는 플랫폼 자체 데이터로 렌더되는 게 정상입니다. 이 테마는 **중요한 거의 모든 페이지에서** 제가 운영하는 서비스로 렌더합니다. 플랫폼으로는 안 됐기 때문이고, [케이스 02](cases/02-when-native-offloading-fails.md)가 그렇지 않음을 증명하려 한 세 번의 진지한 시도의 기록입니다.

| # | 케이스 | 핵심 문제 |
|---|---|---|
| 01 | 모두가 건드리는 파일 쪼개기 | 파일 2개가 머지 충돌과 회귀 버그의 대부분을 차지 |
| 02 | 플랫폼 네이티브 전환이 실패할 때 | 부하를 플랫폼으로 되돌리려는 3번의 시도와 각각의 실패 원인 |
| 03 | 페이지 로드를 전제한 테마 위의 AJAX 내비게이션 | 테마가 상정한 적 없는 내비게이션을 견디는 필터·목록 상태 |
| 04 | 서비스 10개 앞의 테마 | 스토어프론트를 분산 시스템으로 만들지 않으면서 회원 상태 렌더 |

**이 작업이 주장하는 입장**

- **머지 충돌은 아키텍처 신호입니다.** 여러 사람이 같은 파일에서 계속 부딪히면 그 파일이 너무 많은 일을 하고 있는 겁니다. 해법은 절차가 아니라 구조입니다.
- **부정적 결과도 결과물입니다.** 문서화된 3번의 실패한 네이티브 전환 시도는 4번째 시도보다 고객사에 더 가치 있습니다. "우리 인프라 비용이 왜 이렇게 높죠"를 **답이 있는 질문**으로 바꿔주기 때문입니다.
- **빌드 단계 없음은 진짜 제약이고, 대체로 좋은 제약입니다.** 번들러가 없으니 따질 의존성 그래프도, 무효화할 빌드 캐시도 없고, 스타일시트나 스크립트를 도구 없이 페이지에서 파일까지 추적할 수 있습니다. 모듈 편의성을 잃습니다. 여러 사람이 만지는 이 규모의 테마에서는 **추적 가능성이 더 값어치 있었습니다.**
- **스토어프론트가 분산 시스템이 되면 안 됩니다.** 뒤에 서비스가 10개 있지만, 어떤 페이지도 10개로 팬아웃해서 렌더하지 않습니다.
