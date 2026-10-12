# Events High-Volume Search Redesign

## Standup summary

- Implemented the backend portion of the Events high-volume search redesign in the Event module.
- Updated `SearchEventsHandler` to ignore the page-level global filter for Events search, detect requests with no search criteria, and return a limited recent/default result set of 15 rather than running a full search.
- Kept the existing criteria-based dynamic search path unchanged, including brand-filter logic and the query framework.
- Added the `GetRecentEventsForUserAsync(Guid userId, int limit, CancellationToken ct)` repository contract.
- Implemented recent-event retrieval in `EventSummaryRepository`: defaults to the current season (`IsCurrentSeason`), orders by start date descending, applies the requested limit, and populates brand names.
- Updated `SearchEventsHandlerTests` for the new behavior and corrected syntax from a nested test insertion.
- Added coverage for the no-criteria/global-only path.
- Ran the targeted `SearchEventsHandlerTests`: 8/8 passed.

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

## Standup phrasing

> I completed backend safeguards for Events high-volume search. The search handler now short-circuits no-criteria requests to a limited recent/default dataset and avoids full-list retrieval. The criteria-based search path remains intact. I implemented repository support for recent/current-season retrieval and updated unit tests; `SearchEventsHandlerTests` are all passing. Next, I’ll clean the diff, run broader Event tests, and prepare PR notes including frontend follow-up requirements.
