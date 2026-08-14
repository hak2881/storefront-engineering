# AJAX Navigation on a Theme That Assumes Page Loads

**Constraint** · The base theme is built around full page navigation
**Requirement** · Filtering and listing updates without reloads, on the highest-traffic pages
**Difficulty** · Everything that was implicit in a page load becomes state you have to manage

## Context

Product listing and filtering are where this storefront's traffic lives. Full page reloads on every facet change are slow, lose scroll position, and make multi-select filtering feel broken.

So listing updates happen over AJAX. The base theme was not designed for that, and every convenience a page load used to provide for free became something to implement.

## Problem

**A page load resets everything, and that was load-bearing.** Filter panel open or closed, which facets are applied, scroll position, which cards have rendered — all of it was implicitly correct after a reload because it was rebuilt from the URL. Replace the reload with a partial update and every one of those becomes state that can drift.

**The back button has opinions.** A customer who applies three filters and presses back expects to lose one filter, not to leave the page. Without history entries per filter change, back exits the listing entirely — which reads as the site losing their work.

**Deep links must reproduce the state.** A filtered listing that can be reached by clicking but not by pasting a URL is broken for sharing, for search engines, and for the customer who bookmarked it.

**Partial updates leave stale DOM behind.** Replacing a listing without replacing everything that depended on it produces elements from the previous state that are still visible and still bound to old handlers.

**Filter semantics are not obvious.** Multi-select within a facet, interaction between facets, what "clear all" clears, and whether a gender toggle is a filter or a context switch — each of these is a product decision that has to be settled before it can be implemented, and each one changes what the state has to hold.

## Approach

**The URL is the state, and everything else derives from it.** Applied facets live in query parameters. The listing renders from the URL, whether that URL arrived by navigation, by a filter click, by a back button, or by a paste. This is the decision that makes deep linking and history work as consequences rather than as separate features.

**Filter changes push history entries.** Each meaningful state change is a history entry, so back walks through filter states instead of leaving the page.

**View state is separated from filter state.** Whether the filter panel is open is not a filter. Conflating the two puts UI state in the URL, where it gets shared and bookmarked and comes back to a different viewport that wanted a different default. Panel state persists across in-page navigation without becoming part of the shareable URL.

**Gender selection was defined as a context switch, not a facet.** It changes what set of products and what set of categories are in scope, rather than narrowing the current set. Once that was settled it went into its own module with its own policy, rather than living as one more branch in the filter code — which also kept it out of the shared files that [case 01](01-breaking-up-shared-files.md) was trying to shrink.

**Rendering after an update is owned by one place per component.** Product cards, wishlist markers, and related sections each re-render through a single entry point after a listing update, rather than each feature patching the DOM where it happens to notice a change. Stale-element bugs come from having several partial updaters; the fix is to have one per thing being rendered.

**Verified visually, state by state.** Filter open, filter closed, multi-select applied, clear-all, arriving from another listing, arriving from a deep link — each combination captured and compared. Filter state bugs are combinatorial and mostly invisible in code review; they are found by producing the state and looking at it.

## Outcome

- Listing and filter updates without page reloads on the highest-traffic templates, with scroll position preserved
- Back button walks filter history rather than exiting the listing
- Any filtered state is reachable and reproducible from its URL
- Gender scoping isolated as its own module with an explicit policy, instead of a branch inside shared filter logic

---

## 한국어 요약

**제약** — 베이스 테마가 전체 페이지 이동을 전제로 만들어져 있음
**요구** — 트래픽이 가장 많은 페이지에서 리로드 없는 필터링·목록 갱신
**난점** — 페이지 로드가 암묵적으로 해주던 모든 것이 **직접 관리해야 할 상태**가 됨

**어려웠던 지점**

- **페이지 로드는 모든 걸 리셋하고, 그게 구조를 떠받치고 있었습니다.** 필터 패널 열림/닫힘, 적용된 facet, 스크롤 위치, 렌더된 카드 — 전부 리로드 후에는 URL로부터 재구성되기 때문에 암묵적으로 맞았습니다. 리로드를 부분 갱신으로 바꾸는 순간 이 모두가 **어긋날 수 있는 상태**가 됩니다.
- **뒤로 가기에는 사용자의 기대가 있습니다.** 필터 3개를 적용하고 뒤로 가기를 누른 고객은 필터 하나가 풀리기를 기대하지, 페이지를 떠나기를 기대하지 않습니다. 필터 변경마다 히스토리 항목이 없으면 뒤로 가기가 목록을 완전히 벗어나고, 이건 **사이트가 작업을 날린 것**으로 읽힙니다.
- **딥링크가 상태를 재현해야 합니다.** 클릭으로는 도달하는데 URL 붙여넣기로는 도달 못 하는 필터 목록은 공유·검색엔진·북마크 모두에서 깨진 것입니다.
- **부분 갱신은 낡은 DOM을 남깁니다.** 목록만 갈아끼우고 그에 의존하던 것들을 그대로 두면, 이전 상태의 요소가 여전히 보이고 여전히 옛 핸들러에 묶여 있습니다.
- **필터 의미론이 자명하지 않습니다.** facet 내 다중 선택, facet 간 상호작용, "전체 해제"가 무엇까지 해제하는지, 성별 토글이 필터인지 컨텍스트 전환인지 — 각각이 구현 전에 확정되어야 하는 제품 결정이고, 각각이 상태가 담아야 할 것을 바꿉니다.

**접근**

- **URL이 곧 상태이고 나머지는 전부 거기서 파생됩니다.** 적용된 facet은 쿼리 파라미터에 있습니다. 목록은 URL로부터 렌더되며, 그 URL이 내비게이션으로 왔든 필터 클릭으로 왔든 뒤로 가기로 왔든 붙여넣기로 왔든 같습니다. **딥링크와 히스토리가 별도 기능이 아니라 결과가 되게 하는 결정**입니다.
- **필터 변경은 히스토리 항목을 쌓습니다.** 뒤로 가기가 페이지를 떠나는 대신 필터 상태들을 되짚습니다.
- **뷰 상태와 필터 상태를 분리했습니다.** 필터 패널이 열려 있는지는 필터가 아닙니다. 둘을 합치면 UI 상태가 URL에 들어가고, 그게 공유·북마크되어 **다른 기본값을 원하는 다른 뷰포트**로 돌아옵니다. 패널 상태는 페이지 내 이동을 넘어 유지되지만 공유 URL의 일부가 되지는 않습니다.
- **성별 선택은 facet이 아니라 컨텍스트 전환으로 정의했습니다.** 현재 집합을 좁히는 게 아니라 **어떤 상품 집합과 어떤 카테고리 집합이 범위인지**를 바꿉니다. 이게 정리된 뒤 필터 코드의 분기 하나가 아니라 자체 정책을 가진 자체 모듈로 뺐고, 이는 [케이스 01](01-breaking-up-shared-files.md)이 줄이려던 공유 파일 바깥에 두는 효과도 있었습니다.
- **갱신 후 렌더링은 컴포넌트당 한 곳이 소유합니다.** 상품 카드·위시리스트 표시·연관 섹션이 각각 단일 진입점을 통해 다시 렌더됩니다. 기능마다 변화를 눈치챈 자리에서 DOM을 패치하지 않습니다. **낡은 요소 버그는 부분 갱신자가 여럿이라서 생기고, 해법은 렌더 대상마다 하나만 두는 것**입니다.
- **상태별로 시각 검증.** 필터 열림/닫힘, 다중 선택, 전체 해제, 다른 목록에서 진입, 딥링크로 진입 — 조합마다 캡처해서 비교했습니다. 필터 상태 버그는 조합적이고 코드 리뷰에서 대부분 안 보입니다. **상태를 만들어서 눈으로 봐야 찾힙니다.**

**결과**

- 트래픽 상위 템플릿에서 리로드 없는 목록·필터 갱신, 스크롤 위치 유지
- 뒤로 가기가 목록을 벗어나지 않고 필터 히스토리를 되짚음
- 모든 필터 상태가 URL로 도달·재현 가능
- 성별 범위 지정을 공유 필터 로직 내부 분기가 아니라 명시적 정책을 가진 독립 모듈로 분리
