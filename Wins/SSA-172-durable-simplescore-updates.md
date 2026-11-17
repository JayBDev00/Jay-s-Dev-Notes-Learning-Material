# Win: SSA-172 — Durable SimpleScore Self-Updates

**Type:** Scoring App reliability improvement | **Implemented:** November 2026 | **Scope:** server and deployment workflow

**Ticket opened:** [TODO: ticket opened date] | **Merged/deployed:** [TODO: merged/deployed status and date] | **Sign-off:** [TODO: who signed off] | **Impact:** [TODO: before/after metric or who benefited]

## What I delivered

- Added a persisted `auto_update_job` record so update state survives a browser refresh or process restart.
- Used a PostgreSQL advisory lock so only one PM2 worker can start an update.
- Changed update start to return `202 Accepted` with a durable job payload and added a status endpoint.
- Added queued-job recovery on API startup.
- Marked an update successful only once the replacement API worker reconnects to the configured database.
- Staged and validated artifacts before activation, retaining the previous release directory for recovery.

## Evidence

- Server: PR #158
- Validation: 23 focused controller tests, focused ESLint, and `git diff --check` passed.

## Learnings

- Long-running deployment work should be a durable job, not browser state or one synchronous request.
- Mutual exclusion must work across workers; a database-backed lock is stronger than an in-memory guard.
- Completion should mean a meaningful health check, not merely that a process was started.
- Safe activation includes a recovery path: validate before switching and retain the prior release until the replacement is healthy.

## Follow-up awareness

The matching client needs to handle the asynchronous `202` response rather than treating the update as immediately complete.
