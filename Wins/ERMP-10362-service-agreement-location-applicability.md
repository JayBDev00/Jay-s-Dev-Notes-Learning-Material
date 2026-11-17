# Win: ERMP-10362 — Service Agreement Location Applicability

**Type:** ERMP Finance feature | **Implemented:** November 2026 | **Scope:** frontend, services, and database migrations

**Ticket opened:** [TODO: ticket opened date] | **Merged/deployed:** [TODO: merged/deployed status and date] | **Sign-off:** [TODO: who signed off] | **Impact:** [TODO: before/after metric or who benefited]

## What I delivered

- Built normalized facility and venue applicability for service agreements, with additive Flyway migration support.
- Defined availability precedence: event override, event-specific agreement, session venue, session facility, event facility, then global agreement.
- Added session-aware agreement lookup and showed users why an agreement applies.
- Replaced the generic planned-accrual path with an agreement-backed endpoint. The server owns eligibility, rate, UOM, estimate, validation, and audit values.
- Added active-location validation and guards against deletion of locations referenced by service agreements.
- Kept the ad-hoc accrual flow while adding loading, error, no-match, and authorized next-step states for the agreement flow.

## Evidence

- Services: PR #6081
- Frontend: PR #6820
- Database migration: PR #1299
- Validation: 19 focused accrual-tab tests, frontend typecheck, and `git diff --check` passed.

## Learnings

- Financial and contractual authority belongs on the server; the UI should not set agreement-backed values.
- An explicit precedence order makes matching rules testable and explainable.
- Cross-stack delivery needs a rollout plan: database migration and API contracts must exist before the UI enables a workflow.
- Preserve existing data during contract evolution. If an older update response omits applicability data, do not accidentally clear known restrictions.

## Leadership and collaboration

- **Nov. 4, Finance meeting:** Presented the fixed service-accrual work, including that many reported 400 errors were fixed and the work was moving to QA with UAT next. Asked Finance for context because the 1036X tickets described what broke but not the desired outcome, and asked Sal for a one-on-one knowledge transfer.
- **Nov. 5, working session with Sal:** Asked how variance is displayed, whether staffing and travel are included, where the data originates, whether UKG is integrated, and whether categories come from the GL account list. Asked Sal to annotate the worksheet and share a copy.
- **Nov. 6, Event Financials Q&A:** Helped align on integration ownership and the event hierarchy with the group. Proposed manual spreadsheet upload as the MVP because bad input data could make a live API integration worse; the group moved integration discussion out of scope. Asked for a clear definition of done on each ticket and raised the audit-trail need before the work reached a point of no return.

## Knowledge transfer

The sessions with Sal gave me the context to ask better questions about variance, staffing, travel, UKG, the GL-driven category list, and vendor verification. I also asked for an annotated SAP worksheet to support follow-up research.

- [TODO: what Sal explained about the current financial workflow]
- [TODO: what the annotated worksheet clarified]
- [TODO: link to Sal's worksheet or related research]

## Follow-up awareness

Region-level applicability was intentionally excluded because there was no reliable structured region source.
