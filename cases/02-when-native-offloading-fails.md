# When Platform-Native Offloading Fails

**Question asked** · Can we move page rendering back onto the platform and cut the database load?
**Answer after three attempts** · No, and here is exactly why
**Deliverable** · A briefing that turned an open-ended infrastructure cost into a decision with known options

## Context

Over 90% of meaningful page traffic — the four main category pages, every subcategory page, and all 1,067 brand pages — was being served through our own backend and database rather than by the platform.

That is inverted. A hosted commerce platform is supposed to render its own catalog pages. Every page that goes through our backend is database load we pay for, latency we own, and an outage we are responsible for.

So the obvious question came up, repeatedly: why not just let the platform do it?

## The three attempts

### Attempt 1 — Route only the large brands through our backend

**Idea.** Six brands have 5,000+ products each; the platform's native listing struggles at that size. The other 1,061 brands are small. Send only the big six to our backend and let the platform handle the rest — an estimated 90% reduction in backend traffic.

**Result: failed.**

**Why.** The rule requires the page to know, at render time, how many products the brand has. The platform does not report that count reliably during template rendering — the value is subject to lazy evaluation and does not resolve to an accurate number in that context. Small brands were therefore misclassified as large and routed to our backend anyway.

The decision predicate was not available at the point the decision had to be made. The result was all 1,067 brand pages on our backend — the exact state we were trying to escape.

### Attempt 2 — Move subcategory pages to the platform's native search widget

**Idea.** Category pages (women/men/kids × clothing/shoes/bags) are the highest-traffic templates. Render them with the platform's built-in search and filtering, and they never touch our backend.

**Result: structurally impossible.**

**Why.** Our category taxonomy lives in a product metafield. The platform's search widget can only build filter options from a specific set of supported metafield data types, and ours is not one of them. The page renders — but with no category filter and no gender filter, because the widget cannot turn our taxonomy into facets.

A category page without category filtering is not a category page. Falling back to it for some pages and not others would have meant two different filtering experiences on the same site, so all subcategory pages stayed on our backend for consistency.

### Attempt 3 — Backfill product metafields to make native search viable

**Idea.** Fill in the platform-side product data completely and correctly, so native search has accurate facets to work with, laying groundwork for attempts 1 and 2 to be retried.

**Result: the backfill succeeded — 235,848 of 235,961 products, a 0.04% failure rate. The offloading did not.**

**Why.** The blocker in attempt 2 was the metafield *type*, not the metafield *data*. Complete and accurate values in an unsupported type are still an unsupported type. The backfill delivered real value — catalog data consistency, and the option preserved should the platform ever support the type — but it did not move a single page off our backend.

## What I concluded

**The predicate has to exist where the decision is made.** Attempt 1 was a sound strategy defeated entirely by *where* the information was available. This is a general failure mode with hosted platforms: the data exists, the API exposes it, and it is still not accessible at the one point in the rendering lifecycle where you need it.

**Platform extension points constrain by type, not just by capability.** Attempt 2 failed on a schema detail, not on a feature gap. "The platform supports metafield filtering" is true and was not sufficient. Verifying that *our specific* data type is supported would have cost an hour at the start and saved the whole attempt.

**Documenting failures is the deliverable.** By the end there was a briefing, written for the client rather than for engineers, that laid out each attempt, why it failed, where traffic actually goes, and what the remaining options are — five incremental improvements to try first, with a database instance upgrade as the fallback if they prove insufficient.

That document is worth more than a fourth attempt would have been. "Why is our infrastructure bill this shape" went from an open question that resurfaces every quarter to a settled one with a documented decision tree.

**A negative result stops repeated work.** Without the write-up, someone proposes native offloading again in six months and the cycle repeats. With it, the conversation starts from what is already known.

## Honest assessment

The right time to have discovered the attempt 2 blocker was before building toward it. Checking whether the platform's search widget could consume our specific metafield type was a small, cheap verification available at the very beginning, and doing it late cost real work.

The lesson generalizes past this project: when a plan depends on a platform capability, verify the capability against *your* data, at the *exact* extension point, before anything else is built on the assumption.

---

## 한국어 요약

**질문** — 페이지 렌더링을 플랫폼으로 되돌려서 DB 부하를 줄일 수 있는가?
**3번의 시도 끝의 답** — 아니오, 그리고 정확히 왜 안 되는지
**결과물** — 열려 있던 인프라 비용 문제를 **선택지가 명확한 결정**으로 바꾼 고객사 브리핑

의미 있는 페이지 트래픽의 90% 이상 — 메인 카테고리 4종, 모든 하위 카테고리, 브랜드 페이지 1,067개 전부 — 이 플랫폼이 아니라 자체 백엔드와 DB를 거치고 있었습니다. **뒤집힌 상태입니다.** 호스팅형 커머스 플랫폼은 자기 카탈로그 페이지를 자기가 렌더해야 합니다. 우리 백엔드를 거치는 모든 페이지는 우리가 지불하는 DB 부하, 우리가 소유한 지연, 우리 책임인 장애입니다. 그래서 당연한 질문이 반복해서 올라왔습니다 — 그냥 플랫폼이 하게 두면 안 되나?

**시도 1 — 큰 브랜드만 자체 백엔드로**

상품 5,000개 이상인 6개 브랜드만 자체 백엔드로 보내고 나머지 1,061개는 플랫폼이 처리 (백엔드 트래픽 90% 절감 예상).

**결과: 실패.** 이 규칙은 **렌더 시점에** 해당 브랜드의 상품 수를 알아야 성립합니다. 그런데 플랫폼은 템플릿 렌더링 중에 그 개수를 신뢰성 있게 알려주지 않습니다 — 지연 평가 대상이라 그 맥락에서 정확한 값으로 확정되지 않습니다. 그래서 작은 브랜드가 큰 브랜드로 오분류되어 결국 자체 백엔드로 떨어졌습니다. **판단 근거가 판단이 필요한 지점에 존재하지 않았습니다.** 결과는 브랜드 페이지 1,067개 전부 자체 백엔드 — 벗어나려던 바로 그 상태.

**시도 2 — 하위 카테고리 페이지를 플랫폼 네이티브 검색 위젯으로**

**결과: 구조적으로 불가능.** 카테고리 분류가 상품 메타필드에 있는데, 플랫폼 검색 위젯은 **특정 지원 데이터 타입**의 메타필드로만 필터 옵션을 만들 수 있고 우리 타입은 거기 없었습니다. 페이지는 뜨지만 카테고리 필터도 성별 필터도 나오지 않습니다. **필터가 없는 카테고리 페이지는 카테고리 페이지가 아닙니다.** 일부만 그렇게 하면 같은 사이트에서 필터링 경험이 두 가지가 되므로, 일관성을 위해 하위 카테고리 전부 자체 백엔드에 남겼습니다.

**시도 3 — 네이티브 검색이 가능하도록 상품 메타필드 백필**

**결과: 백필은 성공 (235,961개 중 235,848개, 실패율 0.04%). 부하 이전은 실패.** 시도 2의 병목은 메타필드 **데이터**가 아니라 **타입**이었습니다. 지원되지 않는 타입에 값이 완벽하게 채워져 있어도 여전히 지원되지 않는 타입입니다. 백필 자체는 실질 가치(카탈로그 데이터 정합성, 향후 플랫폼이 해당 타입을 지원할 경우의 옵션 확보)를 남겼지만, **페이지 한 장도 백엔드에서 떼어내지 못했습니다.**

**결론**

- **판단 근거는 판단이 일어나는 곳에 있어야 합니다.** 시도 1은 전략은 타당했고 오로지 *정보가 어디서 접근 가능한가*에 의해 무너졌습니다. 호스팅형 플랫폼의 일반적 실패 모드입니다 — 데이터는 존재하고 API도 노출하는데, **정작 필요한 렌더링 생애주기의 그 한 지점에서는 접근할 수 없습니다.**
- **플랫폼 확장 지점은 기능이 아니라 타입으로 제약합니다.** 시도 2는 기능 부재가 아니라 스키마 세부사항에서 실패했습니다. "플랫폼이 메타필드 필터링을 지원한다"는 참이었고 충분하지 않았습니다. **우리의 그 타입**이 지원되는지 확인하는 건 초반에 한 시간이면 될 일이었고, 그랬으면 시도 전체를 아꼈을 겁니다.
- **실패를 문서화하는 것이 결과물입니다.** 최종적으로, 엔지니어가 아니라 고객사를 위해 쓴 브리핑이 남았습니다 — 각 시도와 실패 원인, 트래픽이 실제로 어디로 가는지, 남은 선택지(먼저 시도할 단기 개선안 5가지, 부족하면 DB 인스턴스 업그레이드). **4번째 시도보다 이 문서가 더 값어치 있습니다.** "인프라 비용이 왜 이 모양인가"가 분기마다 재소환되는 열린 질문에서 **결정 트리가 문서화된 닫힌 질문**이 됐습니다.
- **부정적 결과는 반복 작업을 멈춥니다.** 기록이 없으면 6개월 뒤 누가 네이티브 전환을 다시 제안하고 사이클이 반복됩니다.

**솔직한 평가** — 시도 2의 병목은 그쪽으로 작업을 쌓기 **전에** 발견했어야 합니다. 플랫폼 검색 위젯이 우리의 그 메타필드 타입을 소화할 수 있는지 확인하는 건 맨 처음에 가능한 작고 싼 검증이었고, 늦게 한 대가로 실제 공수를 지불했습니다. 이 교훈은 이 프로젝트 밖으로도 일반화됩니다 — **계획이 플랫폼 기능에 의존한다면, 그 가정 위에 뭔가를 쌓기 전에 *우리 데이터*로, *정확히 그 확장 지점*에서 검증하십시오.**
