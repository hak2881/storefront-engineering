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
**세 번의 시도 끝에 나온 답** — 안 됩니다. 그리고 왜 안 되는지는 정확히 압니다.
**결과물** — 열린 채로 남아 있던 인프라 비용 문제를 선택지가 분명한 결정으로 바꾼 고객사 브리핑

의미 있는 페이지 트래픽의 90% 이상이 플랫폼이 아니라 자체 백엔드와 DB를 거치고 있었습니다. 메인 카테고리 4종, 하위 카테고리 전부, 브랜드 페이지 1,067개 전부. 뒤집힌 상태입니다. 호스팅형 커머스 플랫폼이라면 자기 카탈로그 페이지는 자기가 렌더해야 합니다. 우리 백엔드를 거치는 페이지는 하나하나가 우리가 내는 DB 부하이고, 우리가 떠안은 지연이고, 우리 책임인 장애입니다. 그러니 뻔한 질문이 계속 올라왔습니다. 그냥 플랫폼이 하게 두면 안 되나?

**시도 1 — 큰 브랜드만 자체 백엔드로**

상품 5,000개가 넘는 6개 브랜드만 자체 백엔드로 보내고 나머지 1,061개는 플랫폼이 처리하게 한다. 백엔드 트래픽을 90%쯤 줄일 수 있다고 봤습니다.

**결과: 실패.** 이 규칙이 성립하려면 렌더 시점에 그 브랜드의 상품 수를 알아야 합니다. 그런데 플랫폼은 템플릿 렌더링 중에 그 숫자를 믿을 만하게 주지 않습니다. 지연 평가 대상이라 그 맥락에서는 정확한 값으로 확정되지 않습니다. 그래서 작은 브랜드가 큰 브랜드로 잘못 분류돼 결국 자체 백엔드로 떨어졌습니다. 판단 근거가 판단이 필요한 그 지점에 없었던 겁니다. 결과는 브랜드 페이지 1,067개 전부 자체 백엔드, 벗어나려던 바로 그 상태였습니다.

**시도 2 — 하위 카테고리 페이지를 플랫폼 네이티브 검색 위젯으로**

**결과: 구조적으로 불가능.** 카테고리 분류가 상품 메타필드에 들어 있는데, 플랫폼 검색 위젯은 지원하는 몇 가지 데이터 타입의 메타필드로만 필터 옵션을 만들 수 있고 우리 타입은 거기 없었습니다. 페이지는 뜹니다. 다만 카테고리 필터도 성별 필터도 나오지 않습니다. 필터가 없는 카테고리 페이지는 카테고리 페이지가 아닙니다. 일부 페이지만 이렇게 두면 같은 사이트 안에서 필터링 경험이 두 갈래로 갈리므로, 일관성을 위해 하위 카테고리는 전부 자체 백엔드에 남겼습니다.

**시도 3 — 네이티브 검색이 가능하도록 상품 메타필드 백필**

**결과: 백필은 성공했지만(235,961개 중 235,848개, 실패율 0.04%) 부하 이전은 실패.** 시도 2를 막은 건 메타필드의 데이터가 아니라 타입이었습니다. 지원되지 않는 타입은 값이 아무리 완벽하게 채워져 있어도 여전히 지원되지 않는 타입입니다. 백필 자체로 남은 것도 있습니다. 카탈로그 데이터 정합성이 좋아졌고, 나중에 플랫폼이 그 타입을 지원하면 쓸 수 있는 선택지가 생겼습니다. 다만 백엔드에서 떼어낸 페이지는 한 장도 없었습니다.

**결론**

- **판단 근거는 판단이 일어나는 곳에 있어야 합니다.** 시도 1은 전략 자체는 타당했고, 오직 그 정보를 *어디서* 읽을 수 있느냐 때문에 무너졌습니다. 호스팅형 플랫폼에서 흔한 실패 방식입니다. 데이터는 있고 API도 열려 있는데, 정작 필요한 렌더링 생애주기의 그 한 지점에서만 접근이 안 됩니다.
- **플랫폼 확장 지점은 기능이 아니라 타입으로 제약합니다.** 시도 2가 걸린 곳은 기능 부재가 아니라 스키마 세부사항이었습니다. "플랫폼이 메타필드 필터링을 지원한다"는 말은 참이었지만 그것만으로는 부족했습니다. 우리가 쓰는 *그* 타입이 지원되는지 확인하는 데는 처음에 한 시간이면 됐고, 그랬으면 시도 전체를 아꼈을 겁니다.
- **실패를 문서로 남기는 게 결과물입니다.** 마지막에 남은 건 엔지니어가 아니라 고객사를 위해 쓴 브리핑이었습니다. 각 시도와 실패 원인, 트래픽이 실제로 어디로 흐르는지, 남은 선택지는 무엇인지 — 먼저 시도해볼 단기 개선안 다섯 가지, 그걸로 부족하면 DB 인스턴스 업그레이드. 네 번째 시도를 하는 것보다 이 문서가 고객사에 도움이 됐습니다. "인프라 비용이 왜 이 모양인가"가 분기마다 다시 소환되는 열린 질문에서, 결정 트리가 문서화된 닫힌 질문이 됐으니까요.
- **안 된다는 결론은 같은 일을 반복하지 않게 합니다.** 기록이 없으면 6개월 뒤에 누군가 네이티브 전환을 다시 제안하고 같은 사이클이 돕니다.

**솔직한 평가** — 시도 2의 병목은 그쪽으로 작업을 쌓기 전에 찾았어야 합니다. 플랫폼 검색 위젯이 우리 메타필드 타입을 소화할 수 있는지 확인하는 건 맨 처음에 할 수 있는 작고 싼 검증이었는데, 늦게 한 대가로 실제 공수를 지불했습니다. 이 교훈은 이 프로젝트 밖에서도 통합니다. 계획이 플랫폼 기능에 기대고 있다면, 그 가정 위에 뭘 쌓기 전에 *우리 데이터*로 *정확히 그 확장 지점*에서 직접 확인해야 합니다.
