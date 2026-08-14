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

**증상** — 머지 충돌과 회귀 버그 대부분이 파일 2개로 추적됨
**원인** — 그 파일들이 어느 기능에 속하든 모든 필터 동작이 사는 곳이 되어 있었음
**해법** — 새 코드가 어디로 가는지에 대한 규칙 + 기존 코드를 건드리기 너무 위험할 때의 우회로

테마의 상품 목록·검색 필터가 아주 큰 스크립트 2개(큐레이션 검색 필터, 메인 스토어프론트 목록 필터)로 쌓여 있었습니다. 필터 관련 기능이 전부 둘 중 하나로 들어갔습니다. **기존 필터 코드가 거기 있었기 때문**입니다. 여러 사람이 병렬로 테마를 작업하면서, 이 두 파일이 충돌 지점이 됐습니다.

**어려웠던 지점**

- **충돌이 상시적이고 의미론적으로 고약했습니다.** 2,000줄짜리 한 파일에 서로 무관한 필터 기능을 각자 추가하면 깔끔한 텍스트 충돌이 나는 게 아니라, **두 기능이 조용히 간섭하는 코드로 auto-merge**가 됩니다.
- **사이드이펙트를 추적할 수 없었습니다.** 모든 필터 동작이 파일을 공유하면 상태와 이벤트 핸들러도 공유하게 됩니다. 성별 토글을 바꾸면 위시리스트 렌더링이 깨지는데, 그 연결이 **어느 기능 관점에서도 안 보였습니다.**
- **폭발 반경이 엉뚱한 사람을 벌줬습니다.** 작고 잘 정의된 기능을 내는 사람이 코드베이스에서 가장 위험한 파일을 건드려야 했고, 남의 기능의 회귀 버그를 발견하는 것도 그 사람이었습니다.

**접근** — 규칙을 일부러 무디게 만들었습니다. 섬세한 규칙은 마감과 만나면 살아남지 못하기 때문입니다.

- **기존 대형 파일에 화면 한 장 이상의 새 코드를 추가하지 않습니다.** 그걸 넘으면 별도 모듈이 됩니다.
- **분리 기준은 단일 책임 *그리고* 독립 활성화 조건.** 양쪽 다 중요합니다. 모듈은 한 가지 일을 해야 하고, **언제 활성화되는지 정확히 말할 수 있어야** 합니다 — 이 페이지 타입에서만, 이 플래그에서만, 이 4개 샵에서만. 두 번째 조건이 모듈이 다시 조용히 범용화되는 걸 막습니다. **적용되는 곳에서만 로드되면 무관한 걸 집어넣을 유혹이 없습니다.**
- **모듈당 진입점 하나, 이름 붙은 전역 하나.** 각 모듈은 자체 에셋 파일의 독립 IIFE이고 표면을 하나의 명명된 전역으로 노출합니다. 로딩은 명시적입니다 — 그 동작이 필요한 섹션/스니펫이 스크립트를 포함합니다. 가장 우아한 모듈 시스템은 아닙니다. 하지만 **페이지를 열고, 뭐가 로드됐는지 보고, 파일을 찾을 수 있는** 시스템입니다.
- **고위험 수정에는 문서화된 우회로.** 공유 로직 수정이 필요한 변경은, 공유 파일을 직접 고치는 대신 **새 파일에 보정 로직**(DOM 조정, 레거시 경로보다 먼저 가로채는 capture phase 핸들러)을 작성하는 것이 공인된 방식입니다.

이것만 떼어 보면 더 나쁜 코드입니다. 하나면 될 일에 파일 두 개가 협력합니다. 그래도 **실제 제약 아래서는 옳은 선택**이었습니다 — 운영 중인 스토어프론트에서의 병렬 작업, 그리고 공유 필터 로직의 머지 충돌이 간접층 하나보다 비싼 상황. 대안(공유 파일을 한 사람이 리팩터링할 때까지 모두를 직렬화)은 더 비쌉니다.

**작동하게 만든 것**

- **규칙이 숫자였습니다.** "파일을 작게 유지하라"는 아무도 자기 변경에는 적용하지 않는 조언입니다. "화면 한 장 넘으면 새 파일"은 넘었거나 안 넘었거나입니다.
- **분리는 재작성이 아니라 점진적이었습니다.** 대형 파일을 다시 쓰지 않았습니다. **자라기를 멈췄고**, 새 동작이 그 옆에 붙었습니다. 기존 코드는 어차피 바꿔야 할 때만 추출했습니다.
- **활성화 조건을 모듈별로 적어뒀습니다.** 어떤 모듈이 페이지 타입 한정인지 샵 한정인지 아는 것이, 다음 사람이 자기 새 동작을 거기 넣을지 판단하는 근거입니다. 그 판단이 분리가 유지되느냐 서서히 되돌아가느냐를 결정합니다.

**솔직한 평가** — 이제 파일이 더 많고, 일부 동작은 레거시 경로 + 보정 모듈 두 곳에 걸쳐 있습니다. 기능 하나를 처음부터 끝까지 읽으려면 둘 다 열어야 할 수 있습니다. 실재하는 비용이고, **선택한 비용**입니다. 공유 필터 로직의 머지 충돌은 운영 스토어프론트에 회귀 버그를 만들고 있었고, 간접층은 혼란을 만듭니다. **혼란이 더 쌉니다.** 그리고 잘못된 auto-merge와 달리, 배포 전에 보입니다.
