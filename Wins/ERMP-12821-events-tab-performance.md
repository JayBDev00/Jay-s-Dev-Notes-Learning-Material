# Win: ERMP-12821 — Events Tab Performance (my first feature, end to end)

**Started:** 10/12/2026 | **Closed:** 11/07/2026 (about 4 weeks, ticket to production)

## The problem

The Events tab loaded every event as soon as it opened. That caused performance problems after launch and would get worse as historical events accumulate.

## What I shipped

- Criteria-based search that requires at least one field.
- Default filter set to the current season.
- Explicit Search button click required before a query runs.
- Page-level global search removed from the Events page.
- Recently Viewed Events: the last 10 events a user opened.
- Existing UI result filters kept in place.
- Work in both repos: backend services (PR #5472) and frontend (PR #6294).

See `Week-1-October-2026/standup-summary.md` for the original backend implementation notes.

## Full lifecycle (first time through all of it)

Dev -> code review -> QA -> UAT -> Staging -> Production. Signed off by QA (Donald Mol), the BA (Jessica Lomba), and the business (Jason Winegard), who smoke-tested in Prod.

## What went wrong and how I fixed it

| Issue | Cause | Fix |
|---|---|---|
| Recently Viewed failed QA | I built the last 10 *search results*; the requirement was the last 10 *events opened* | Reworked after talking with the team, QA approved on retest |
| Competition Type became a text field | Business noticed it lost its dropdown | Competition Search fix, PR #5702: exact-match dropdown filters, event-level grouping/pagination, regression tests (52/52 passing) |
| Brand filter matched on Event Name | Backend query criteria too broad | PR #5815, verified with screenshots in Staging |

## Learnings

- Confirm what a user story means before building. The Recently Viewed mismatch cost a rework cycle.
- Write down deployment dependencies. PR #6206 had to ship first or the new search inputs stayed hidden, so I added a deploy checklist.
- Check what a release contains before telling testers a fix is in. A hotfix went out without my PR.
- Show proof (screenshots) when saying something is fixed.
- Defaults have side effects: defaulting to the current season raised the question of how to find past seasons.

## Open follow-up

Business needs a way to find competitions from past seasons. A Seasons tab was suggested, and it would also be needed on Registrations and probably other pages.
