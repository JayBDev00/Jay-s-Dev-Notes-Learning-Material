# ERMP-16554 — TBD Event Dates Implementation Notes

## Frontend implementation

Event creation now supports TBD dates:

- `eventTBD` is managed by React Hook Form rather than separate local state.
- When TBD is checked:
	- Start/End Date fields are disabled.
	- Date-required markers are hidden.
	- Date validation errors are cleared.
	- Start/end date validation is skipped.
- When TBD is unchecked, normal date validation resumes.

### Create payload behavior

```json
{
	"isDateTBD": true,
	"eventStartDate": null,
	"eventEndDate": null
}
```

This also overrides previously entered dates, not only empty dates.

## Shared UI fix

`EventsDateField` had a hidden native `input[type="date"]` used by the calendar picker. Its visible controls respected `disabled`, but the native input did not.

The native input now receives the `disabled` prop:

```tsx
<input
	// existing props
	disabled={disabled}
	type="date"
/>
```

This prevents native browser min/max validation from blocking Continue when a user:

1. Enters valid dates.
2. Moves Start Date later than End Date.
3. Selects TBD.

## Backend implementation

Backend now accepts nullable event dates in create/update requests.

Validation rules:

- `IsDateTBD === true` means both dates must be `null`.
- `IsDateTBD === false` means both dates are required and must not be default values.
- `NumberOfDays` is not calculated for TBD/default-date events.

Current limitation: persistence/read DTOs still normalize TBD dates to concrete/default values rather than exposing nullable dates end-to-end. A broader nullable-persistence change would require review of integrations, UKG, search, sorting, and downstream date consumers.

## Tests

Added/updated coverage:

- Undated TBD event payload sends nullable dates.
- TBD event with previously populated dates still sends nullable dates.
- Shared date field test verifies its disabled native picker is excluded from browser form validation.

Validation completed:

```powershell
pnpm --filter=@vrst/ermp typecheck
pnpm --filter=@vrst/ermp exec vitest run "src/app/utils/api/events-mgmt/__tests__/events.payload.test.ts"
```

ERMP payload suite: **13/13 passing**.

The shared UI test could not run locally because the install is missing `@rolldown/binding-win32-x64-msvc`; this is an environment dependency issue, not a test failure.
