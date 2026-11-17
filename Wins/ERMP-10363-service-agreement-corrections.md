# Win: ERMP-10363 — Service Agreement Corrections and Accrual Contracts

**Type:** ERMP Finance feature | **Implemented:** November 2026 | **Scope:** frontend, services, and database migrations

**Ticket opened:** [TODO: ticket opened date] | **Merged/deployed:** [TODO: merged/deployed status and date] | **Sign-off:** [TODO: who signed off] | **Impact:** [TODO: before/after metric or who benefited]

## What I delivered

- Added relationship-manager metadata, notes, audit information, commitment quantities, and accrual types to service agreements.
- Used canonical UserProfile GUIDs and validated that selected relationship managers and accrual types are active.
- Added an administrator-only unlock-for-correction workflow with a required reason and durable audit entry.
- Made Active and Expired agreement documents immutable while preserving read/download access.
- Added explicit agreement-line accrual creation with duplicate protection.
- Kept legacy `MinQuantity` compatibility while introducing the clearer commitment-quantity contract.
- Delivered ordered migrations: additive schema changes, idempotent data backfill, then required-data enforcement.

## Evidence

- Services: PR #6083
- Frontend: PR #6826
- Database migrations: PR #1300
- Validation: Finance API build, 512 application tests, 223 API tests, frontend typecheck, and four focused helper tests passed.

## Learnings

- Corrections are a workflow: authorization, reason, audit record, and state transition all matter.
- Safer schema changes follow a sequence: add, backfill, enforce.
- Compatibility layers let a contract evolve without forcing an all-at-once consumer migration.
- Cross-module safeguards prevent later destructive actions from invalidating a financial agreement.

## Leadership and collaboration

- **Nov. 4, Finance meeting:** Presented the fixed service-accrual work and asked Finance for the business context missing from the 1036X ticket descriptions. Asked Sal for a one-on-one knowledge transfer.
- **Nov. 6, Event Financials Q&A:** Asked the group to update tickets with clear definitions of done and pushed for requirements before building. Raised an audit-trail ticket early, added Sam as a business approver on tickets, asked to join reporting meetings, and proposed rewriting tickets before the demo; Nora agreed.
- **Nov. 12, working session:** Explained the current ad hoc accrual flow, cleaner UI, and unit-of-measure fields. Proposed a preview that asks whether to override the accrual so planners retain their buffers; the group accepted it. Asked what should appear on the service-line screen so it becomes acceptance criteria, and suggested ticket cleanup, research, and a show-and-tell the following week.

## Knowledge transfer

The sessions with Sal gave me the context to ask better questions about variance, staffing, travel, UKG, the GL-driven category list, and vendor verification. I also asked for an annotated SAP worksheet to support follow-up research.

- [TODO: what Sal explained about the current financial workflow]
- [TODO: what the annotated worksheet clarified]
- [TODO: link to Sal's worksheet or related research]

## Deployment note

Run the normal Flyway pipeline before enabling fields that depend on the new database contract.
