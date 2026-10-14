Branch Snapshot
•	Branch: ERMP12821
•	HEAD: 15d42c4d9
•	Tracking: origin/ERMP12821 (in sync)
•	Working tree: clean
•	Merge-base vs develop: f5ee27b1aab72541d496e44d4dd5c532fa0c882a
Delta vs develop
•	20 commits
•	17 files changed
•	+1344 / -499 lines
Commits on this branch (oldest → newest)
1.	8cf7c9997 ERMP12821: prevent default full event load; add criteria-first and recent-items fallback
2.	b7f4580b9 ERMP-12821: prevent default full event load and add user-specific recent fallback
3.	626785767 finalize criteria-first fallback comments
4.	608a82335 enforce criteria search with recent-events fallback
5.	b8ed4e88f address review feedback
6.	982f7fe30 harden fallback metadata + deterministic ordering
7.	e359e0e25 fix invalid brand array handling + pagination slicing
8.	3cc439b3e address review findings
9.	80dae34e2 empty season config fix candidate
10.	930e5d3be bound recent-event audit scan
11.	21144290a follow-up fixes
12.	a10f2dc6a bound audit reader scan + duplicate-history regression
13.	493c64fab bound audit scan budget + fallback batch iteration
14.	699a98d39 criteria guard hardening
15.	39d0ac0c5 finalize criteria guard + audit bound
16.	ac4348e83 fix brand scalar criteria guard + dedupe fallback events
17.	954d6f87a move recent-audit contract to Shared.Domain
18.	5857fcecf document search endpoint behavior exceptions
19.	72ecc14f4 add brand population regression tests
20.	15d42c4d9 operator-aware value validation (prevents malformed-value bypass)
Functional Outcome (net)
•	No default full event load on events search entry path.
•	Global filter nullified in fallback-enabled endpoint flow.
•	No-criteria requests return recent current-season events.
•	Recent fallback bounded (batch limits) and deduplicated.
•	Brand filter handling corrected for scalar vs array behavior.
•	Malformed non-empty range values no longer bypass fallback.
•	API XML remarks now reflect real fallback semantics.
Architecture Outcome (net)
•	Recent-audit reader contract moved inward:
•	Shared.Domain.Contracts.IRecentAuditEntityReader
•	Shared.Domain.Contracts.RecentAuditEntityRecord
•	EF implementation remains in Shared.AuditTrail.
Test Coverage Added/Updated
•	EventSearchServiceTests expanded for:
•	scalar brand criteria path,
•	ten-batch guard,
•	fallback deduplication,
•	brand population post-processing (paged/non-paged/empty),
•	invalid range value fallback.
•	Infra tests for recent audit reader and repository behavior updated.
If needed, I can also generate a PR-ready changelog grouped by user story (US-01..US-05).
