# Breaking Up the Files Everyone Edits

**Symptom** · Most merge conflicts and most regressions traced to two files
**Cause** · Those files had become where all filter behaviour lived, regardless of which feature it belonged to
**Fix** · A rule about where new code goes, plus an escape hatch for when touching the old code is too risky

## Context

The theme's product listing and search filtering had accumulated into two very large scripts — one for curated search filtering, one for the main storefront listing filter. Every filter-adjacent feature ended up in one of them, because that is where the existing filter code was.

With several people working on the theme in parallel, those two files were where the work collided.

## Problem

**Conflicts were constant and semantically ugly.** Two people adding unrelated filter features to the same 2,000-line file don't get a clean textual conflict — they get an auto-merge that produces code where two features silently interfere.

**Side effects were untraceable.** When every filter behaviour shares a file, it also tends to share state and event handlers. A change to gender toggling would break wishlist rendering, and the connection between the two was not visible from either feature's perspective.

**The blast radius punished the wrong people.** Someone shipping a small, well-scoped feature had to touch the most dangerous file in the codebase, and would be the one to discover a regression in someone else's feature.

## Approach

The rule that fixed it was deliberately blunt, because subtle rules don't survive contact with deadlines.

**Do not add more than a screenful of new code to the existing large files.** Anything beyond that becomes its own module.

**Split on single responsibility *plus* independent activation condition.** Both halves matter. A module should do one thing, and it should be possible to say precisely when it is active — this page type only, this feature flag only, these four shops only. The second condition is what keeps a module from quietly becoming general again: if it only loads where it applies, there is no temptation to put unrelated things in it.

**One entry point per module, with a named global.** Each module is a self-contained IIFE in its own asset file, exposing its surface on a single named global. Loading is explicit — the section or snippet that needs the behaviour includes the script. This is not the most elegant module system available. It is one where you can open a page, look at what it loaded, and find the file.

**A documented escape hatch for high-risk edits.** When a change would require modifying shared logic, the sanctioned move is to write the correction in a new file — a DOM adjustment, a capture-phase handler that intercepts before the legacy path runs — rather than editing the shared file directly.

This is, in isolation, worse code. Two files now cooperate where one would do. It was still the right call under the actual constraint: parallel work on a live storefront where a merge conflict in shared filter logic costs more than a layer of indirection. The alternative — serialize everyone behind one person's refactor of the shared file — costs more still.

## What made it work

**The rule was numeric.** "Keep files small" is advice nobody applies to their own change. "More than a screenful goes in a new file" is a threshold you either crossed or didn't.

**Splitting was incremental, not a rewrite.** The large files were never rewritten. They stopped growing, and new behaviour landed beside them. Existing code was extracted only when it was being changed anyway.

**Activation conditions were written down per module.** Knowing a module is page-type-scoped or shop-scoped is what lets the next person decide whether their new behaviour belongs in it — which is the decision that determines whether the split holds or slowly reverses.

## Honest assessment

The theme now has more files, and some behaviours are spread across two of them — a legacy path plus a corrective module. Reading a feature end to end can require opening both.

That is a real cost, and it is the cost that was chosen. Merge conflicts in shared filter logic were producing regressions on a live storefront; indirection produces confusion. Confusion is cheaper, and unlike a bad auto-merge, it is visible before it ships.

---

## 한국어 요약

**증상** — 머지 충돌과 회귀 버그 대부분이 파일 2개에서 나옴
**원인** — 어느 기능에 속하든 필터 동작은 전부 그 두 파일로 들어가 있었음
**해법** — 새 코드를 어디에 둘지 정하는 규칙, 그리고 기존 코드를 건드리기 너무 위험할 때 쓰는 우회로

테마의 상품 목록·검색 필터가 아주 큰 스크립트 두 개로 쌓여 있었습니다. 하나는 큐레이션 검색 필터, 하나는 메인 스토어프론트 목록 필터. 필터와 조금이라도 관련된 기능은 전부 둘 중 하나로 들어갔습니다. 기존 필터 코드가 거기 있으니까요. 여러 사람이 나란히 테마를 작업하다 보니 이 두 파일에서 일이 부딪혔습니다.

**어려웠던 지점**

- **충돌이 상시적이었고, 내용도 고약했습니다.** 2,000줄짜리 한 파일에 서로 무관한 필터 기능을 각자 추가하면 깔끔한 텍스트 충돌이 나지 않습니다. 두 기능이 조용히 간섭하는 코드로 auto-merge 돼버립니다.
- **사이드이펙트를 추적할 수 없었습니다.** 필터 동작이 전부 한 파일에 있으면 상태와 이벤트 핸들러까지 같이 쓰게 됩니다. 성별 토글을 손봤는데 위시리스트 렌더링이 깨지고, 그 연결고리는 어느 기능 쪽에서 봐도 보이지 않았습니다.
- **폭발 반경이 엉뚱한 사람에게 떨어졌습니다.** 작고 범위가 분명한 기능을 내는 사람이 코드베이스에서 가장 위험한 파일을 건드려야 했고, 남이 만든 회귀 버그를 발견하는 것도 그 사람이었습니다.

**접근** — 규칙은 일부러 무디게 잡았습니다. 섬세한 규칙은 마감 앞에서 살아남지 못하니까요.

- **기존 대형 파일에는 화면 한 장이 넘는 새 코드를 넣지 않습니다.** 그걸 넘어가면 별도 모듈로 뺍니다.
- **분리 기준은 단일 책임 *그리고* 독립적인 활성화 조건.** 둘 다 있어야 합니다. 모듈은 한 가지 일만 해야 하고, 언제 켜지는지를 정확히 말할 수 있어야 합니다 — 이 페이지 타입에서만, 이 플래그가 켜졌을 때만, 이 네 개 샵에서만. 두 번째 조건이 모듈이 슬그머니 다시 범용으로 커지는 걸 막습니다. 적용되는 곳에서만 로드된다면 무관한 걸 끼워 넣을 이유가 없습니다.
- **모듈당 진입점 하나, 이름 붙인 전역 하나.** 각 모듈은 자기 에셋 파일 안의 독립 IIFE이고, 바깥에 드러내는 건 명명된 전역 하나뿐입니다. 로딩도 명시적입니다. 그 동작이 필요한 섹션이나 스니펫이 직접 스크립트를 포함합니다. 세상에서 가장 우아한 모듈 시스템은 아닙니다. 다만 페이지를 열어서 뭐가 로드됐는지 보고 파일을 찾아갈 수는 있습니다.
- **위험한 수정에는 문서로 정해둔 우회로.** 공유 로직을 고쳐야 하는 변경이라면, 공유 파일을 직접 건드리는 대신 새 파일에 보정 로직을 쓰는 게 공인된 방식입니다. DOM을 나중에 조정하거나, 레거시 경로보다 먼저 가로채는 capture phase 핸들러를 두는 식으로요.

떼어놓고 보면 더 나쁜 코드입니다. 파일 하나면 될 일을 두 개가 나눠 합니다. 그래도 실제 제약 아래서는 이쪽이 맞았습니다. 운영 중인 스토어프론트를 여러 사람이 동시에 손대는 상황이고, 공유 필터 로직에서 나는 머지 충돌은 간접층 하나보다 비쌉니다. 대안은 한 사람이 공유 파일을 리팩터링할 때까지 나머지가 줄 서서 기다리는 것인데, 그건 더 비쌉니다.

**작동하게 만든 것**

- **규칙이 숫자였습니다.** "파일을 작게 유지하라"는 아무도 자기 변경에는 적용하지 않는 조언입니다. "화면 한 장 넘으면 새 파일"은 넘었거나 안 넘었거나 둘 중 하나입니다.
- **재작성이 아니라 점진적으로 갈랐습니다.** 대형 파일을 다시 쓰지는 않았습니다. 자라기를 멈췄고, 새 동작이 그 옆에 붙었습니다. 기존 코드는 어차피 손대야 할 때만 꺼냈습니다.
- **활성화 조건을 모듈별로 적어뒀습니다.** 이 모듈이 특정 페이지 타입 전용인지 특정 샵 전용인지 알아야, 다음 사람이 자기 코드를 거기 넣을지 말지 판단할 수 있습니다. 그 판단이 쌓여서 분리가 유지되거나 서서히 원래대로 돌아갑니다.

**솔직한 평가** — 이제 파일 수가 늘었고, 어떤 동작은 레거시 경로와 보정 모듈 두 곳에 나뉘어 있습니다. 기능 하나를 끝까지 읽으려면 두 파일을 다 열어야 할 때도 있습니다. 실재하는 비용이고, 알고 택한 비용입니다. 공유 필터 로직의 머지 충돌은 운영 중인 스토어프론트에 회귀 버그를 만들고 있었고, 간접층이 만드는 건 혼란입니다. 혼란 쪽이 쌉니다. 게다가 잘못된 auto-merge와 달리 배포 전에 눈에 띕니다.
