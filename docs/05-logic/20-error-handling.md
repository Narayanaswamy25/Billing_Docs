# Error Handling and Logging

## Error shape (every API error)
`{ code, message, fieldErrors?: [{field, message}], requestId }`. `code` is stable; the client maps it to text in the user's language. Unknown errors return a generic message and the request ID; detail only in logs. No stack traces in replies.

| Code | HTTP | Meaning | Client wording |
| --- | --- | --- | --- |
| PERMISSION_DENIED | 403 | A check failed | "This page is not available. Contact your admin." (same for module-off) |
| MODULE_OFF | 403 | Module not switched on | same wording |
| VALIDATION_FAILED | 422 | Field wrong; list included | Summary of fields at top of form |
| VERSION_CONFLICT | 409 | Record changed since read | "This record changed. Review the changes." Keep mine / Take theirs per field |
| STOCK_INSUFFICIENT | 409 | Negative-stock rule is Block | Plain words per rule |
| CREDIT_LIMIT_BLOCKED | 409 | Credit rule is Block | "Credit limit reached." (+ Approve if rule) |
| PERIOD_LOCKED | 409 | Date in locked period | "This period is locked." |
| RATE_LIMITED | 429 | Too many requests | "Slow down and try again." |
| (server) | 5xx | Infrastructure | "Something went wrong. Try again." + reference code |

Added for history and restore: `REVERT_BLOCKED` (409, with reason codes), `REVERT_NOT_ALLOWED` (403), `VERSION_CONFLICT` with per-field choice (409), `RETENTION_EXPIRED` (410), `RESTORE_CONFLICT` (409), `STEP_UP_REQUIRED` (401). See [history-and-backup/04](history-and-backup/04-history-api-data-tests.md).

Categories: business refusals (4xx), validation (422), access (401/403), conflict (409), transient (safe to retry), infrastructure (5xx). One exception filter maps them. Domain errors are classes with stable codes.

## Client states (same set and wording on every list and record screen)
Loading (grey placeholders, no spinner, layout does not jump) · Empty with Add ("No invoices yet. Use Add invoice to create the first one.") · Empty without Add ("No invoices yet.") · No match ("Nothing matches. Clear filters.") · Load failed ("Could not load invoices." Try again; keeps last data) · Server error · Not available · Posted record · Locked saved entry · Saving (button disabled, no double create) · Save conflict · Record locked by someone ("Priya is editing. You can read it.") · Waiting for approval · Portal down ("E-invoice pending. We will retry.") · Limit reached ("Plan limit reached. Ask your admin.") · Import errors · Undo · Offline Workspace ("No connection. Your draft is kept.", Save disabled) · Offline Counter ("Offline, 3 bills waiting", billing continues) · Access changed · Access removed mid-edit · Session expired ("You were signed out. Sign in to continue." Drafts kept) · App update required · Maintenance ("Back at 6:30 PM. Your drafts are safe.").

## Retries and resilience
Timeouts on every outside call, query, job and request. Retry only operations safe to repeat, with growing delay, jitter and a cap; never inside a transaction. Circuit breaker and per-provider queues and concurrency limits. Posting routes require `Idempotency-Key` (key + request hash + result stored under a unique index for a set time; same key with different body refused). Outbox delivery is at least once; consumers record handled event IDs. Events carry type, version, ID and time; consumers ignore unknown fields. Fail closed: if permission cache, tenant context, data access layer or key service fails, refuse.

## Logging
JSON lines: time, level, service, version, request ID, trace ID, company ID, user ID, route, duration, error code. Personal data and secrets masked or absent; **never** customer names, phones, GSTINs or amounts. Audit records are separate from logs. Adapter requests/replies logged with secrets removed in the company DB. Metrics: rate/errors/duration per route, queue depth and age, job duration, pool use, cache hit rate, transaction aborts, business counts. Traces: OpenTelemetry spans per use case, DB call and adapter call; trace ID travels inside queue jobs. Crash and usage reporting on from the first build with no personal data.
