# Events High-Volume Search Redesign

## Standup summary

- Implemented the backend portion of the Events high-volume search redesign in the Event module.
- Updated `SearchEventsHandler` to ignore the page-level global filter for Events search, detect requests with no search criteria, and return a limited recent/default result set of 15 rather than run a full unfiltered search.
- Kept the existing criteria-based dynamic search path unchanged, including brand-filter logic and the query framework.
- Added the `GetRecentEventsForUserAsync(Guid userId, int limit, CancellationToken ct)` repository contract.
- Implemented recent-event retrieval in `EventSummaryRepository`: defaults to the current season (`IsCurrentSeason`), orders by start date descending, applies the requested limit, and populates brand data.
- Updated `SearchEventsHandlerTests` for the new behavior and corrected syntax from a nested test insertion.
- Added coverage for the no-criteria/global-only path.
- Ran the targeted `SearchEventsHandlerTests`: 8/8 passed.

## Refactor

### 1) Search orchestration moved and solidified in `EventSearchService`

**Refactor**
- Centralized fallback logic in `EventSearchService.SearchAsync(...)` with `UseRecentEventsFallback`.
- Kept the handler thin (`SearchEventsHandler`) and delegated orchestration to the service.

**Decision**
- Preserve the CQRS thin-handler pattern and place fallback/business decisions in the service.

**Why**
- Cleaner separation of concerns.
- Easier to test fallback behavior independently.

### 2) Prevent full-load default behavior

**Refactor**
- In the fallback-enabled path:
  - `request.Filters.GlobalFilter = null`
  - If no usable column criteria exists, return recent items instead of running the dynamic query.

**Decision**
- “No usable criteria” should never execute a broad search.

**Why**
- Directly addresses the performance issue caused by loading too much by default.

### 3) Recent-items fallback implementation and safeguards

**Refactor**
- Added and used a recent-audit-driven fallback pipeline:
  - Read recent audited event IDs for the current user
  - Intersect with current-season active events
  - Return a capped result set (`DefaultRecentItemsLimit = 15`)
- Added pagination behavior for fallback responses, including paged and non-paged awareness.

**Decision**
- Use user-specific audit history plus season eligibility, not generic recent events.
- Keep hard limits to control operational cost:
  - `AuditCandidateBatchSize = 500`
  - `MaxAuditCandidateBatches = 10`

**Why**
- Predictable query cost.
- Relevant results for the signed-in user.

### 4) Determinism and duplicate handling in fallback

**Refactor**
- Added deduplication by `EventId` in the fallback assembly step while preserving first-seen audit recency order.

**Decision**
- Duplicate audit hits should not duplicate output events.

**Why**
- Prevents confusing UX and wasted slots in the capped fallback list.

### 5) Brand filter criteria guard fixes

**Refactor**
- Split brand handling by payload shape:
  - Array payload: only special array logic applies.
  - Scalar payload: supported scalar operators with usable values count as criteria.
- Kept brand alias normalization to `BrandSearchText`.

**Decision**
- Treat scalar and array semantics differently.
- Avoid false “no criteria” classification for valid scalar operators like `startsWith`, `endsWith`, and `notEquals`.

**Why**
- Fixed incorrect fallback triggering for valid brand filters.

### 6) Malformed-value bypass gap closure

**Refactor**
- Added operator-aware “usable value” validation in the criteria check:
  - Range operators such as `between` and `betweenInclusive` now require a valid 2-value range payload.
  - Invalid but non-empty values no longer count as criteria.

**Decision**
- Criteria detection must align with operator shape expectations, not just a non-empty check.

**Why**
- Prevents malformed filters from bypassing fallback and hitting the broad query path.

### 7) Architecture boundary correction

**Refactor**
- Moved contracts:
  - `IRecentAuditEntityReader`
  - `RecentAuditEntityRecord`
- Relocated them from `Shared.AuditTrail` to `Shared.Domain.Contracts`.
- Kept the implementation (`RecentAuditEntityReader`) and DI registration in `Shared.AuditTrail`.

**Decision**
- Application layer consumes inward contracts only; infrastructure owns implementation only.

**Why**
- Fixed the outward dependency violation and aligned with the Clean Architecture rule.

### 8) API contract/documentation alignment

**Refactor**
- Updated XML remarks for `POST /api/events/search` to explicitly document:
  - Global filter discard in fallback-enabled behavior
  - No-criteria recent-items fallback
  - 15-item cap semantics and pagination behavior

**Decision**
- Generated API docs must match runtime behavior exceptions.

**Why**
- Prevents reviewer and consumer confusion, and avoids incorrect integration assumptions.

### 9) Test-suite refactor and additions

**Added/updated coverage**
- Criteria-path and fallback-path tests in `EventSearchServiceTests`:
  - Scalar brand operator criteria routing
  - Ten-batch query limit enforcement
  - Recent-items deduplication
  - Brand post-processing calls for paged and non-paged non-empty results
  - No populate call for empty results
  - Invalid range value fallback bypass fix
- Existing handler, repository, and infrastructure tests were adjusted where needed.

**Decision**
- Shift verification from older handler-centric assumptions to the service behavior contract.

**Why**
- Reduced regression risk across the exact failure modes found during review rounds.

## Key files

- `Modules/Event/Event.Application/Handlers/Search/SearchEventsHandler.cs`
- `Modules/Event/Event.Domain/RepositoryInterfaces/IEventSummaryRepository.cs`
- `Modules/Event/Event.Infrastructure/Repositories/EventSummaryRepository.cs`
- `ModulesTests/EventTests/Event.ApplicationTest/Handlers/SearchEventsHandlerTests.cs`
- `EventSearchRequestMapper.cs` was also modified in the working tree; confirm that change is intentional before including it in the PR.

## Debugging notes

- Many reported errors were cascading `MSB4181` errors caused by one root issue.
- The architecture rule identified during debugging: Domain must not reference Application.
- Test failures were due to the behavior change (no criteria now returns recent items), not unrelated instability.
- When handler behavior changes, old tests may need explicit filters to continue exercising the original criteria-based path.
- Keep test method braces clean; a nested `[Fact]` caused multiple syntax errors.

## Next steps

1. Clean up the diff and ensure only ticket-related files are included; remove lockfiles or other unrelated changes.
2. Build the Event infrastructure and Event application test projects.
3. Run targeted and broader Event tests.
4. Verify the requirements:
   - No criteria returns only a limited recent/default result set.
   - Criteria uses the normal filtered search.
   - The Events endpoint ignores the global filter on the backend.
5. Prepare PR notes and call out the frontend follow-up:
   - Hide the global search control.
   - Require an explicit Search click or Enter.
   - Show guidance explaining that criteria are required.
   - Default the UI to recent items/current season.

## Technical breakdown

### Problem

The previous default fallback path used offset-based paging in a loop. Because offset pagination on deduped audit entities reprocessed earlier data repeatedly, each additional call could rescan prior rows, increasing DB work significantly in high-history users.

### New execution model

1. Read a bounded recent-audit candidate window once (<= 5000).
2. Process candidates in-memory in 500-record slices.
3. For each slice, query current-season eligible events.
4. Stop early once 15 fallback events are collected.
5. Preserve deterministic ordering + dedupe by event ID.

### Why this is better

- Eliminates repeated audit-reader invocations and offset rescans.
- Keeps strict upper bounds (5000 candidates, max 10 eligibility slices).
- Maintains existing fallback semantics (15 cap, current-season only, recency order).

### Cost profile (worst-case)

- Before: up to 10 audit-reader calls + up to 10 season checks.
- After: 1 audit-reader call + up to 10 season checks.

So we reduced the most expensive repeated part (audit-reader loop) without changing user-facing behavior.

## Standup phrasing

> I completed backend safeguards for Events high-volume search. The search handler now short-circuits no-criteria requests to a limited recent/default dataset and avoids full-list retrieval. The criteria-based search path remains unchanged, and the targeted handler tests are passing.
