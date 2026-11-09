# Win: ERMP-16554 — Salesforce-Camps Dates TBD Fix

**Type:** ERMP Defect | **Opened:** 10/13/2026 | **QA approved:** 10/21/2026 (about 8 days) | **Status:** Closed

Implementation details: see `Week-2-October-2026/16554-implementation.md`.

## The problem

Events marked as "Dates TBD" were not working properly when created.

## What I did

- Moved `eventTBD` into React Hook Form and, when TBD is checked, disabled the date fields, cleared date validation, and sent `null` dates in the create payload (even if dates had been entered before).
- Found and fixed a shared component bug: `EventsDateField` had a hidden native date input that ignored `disabled`, so browser min/max validation could still block Continue.
- Updated the backend to accept nullable dates with clear validation rules (TBD means both dates null; otherwise both required).
- Added payload tests (13/13 passing).

## Proactive catch

While QA was testing, I noticed an admin upcoming events table on login that the patch did not cover, and I patched it before anyone reported it.

## Learnings

- A bug in a shared component can hide behind a form that looks correct. The visible controls respected `disabled`, but the hidden native input did not.
- Check for other places that use the same data before calling a fix done.
- Know and write down a fix's limits. Persistence and read DTOs still normalize TBD dates to default values, so going fully nullable would need review of integrations, UKG, search, sorting, and other date consumers.

## Open follow-up

Fully nullable date persistence end to end (see limitation above).
