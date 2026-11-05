# Testing Strategy

A build is not released until the permission matrix passes with no exceptions.

## Pyramid
Many domain tests (pure functions: money, tax, rounding, costing, number formats; thousands of quick cases incl. rounding and negative stock) → API tests per module against a real MongoDB replica set in a container → one isolation suite (two companies, every route) → a few end-to-end paths that must never break (invoice, offline Counter, e-invoice against a fake GSP, import).

## Suites
| Test | What it covers | Gate |
| --- | --- | --- |
| Permission matrix | Every role: each icon, tab, field, button, badge, count, search result and quick-jump entry, checked against the server, with a second user in another branch | Any failure blocks release |
| Isolation | The full suite in 06-security/tenancy/05: TDAL unit and fuzz tests, generated per-repository two-company tests, canary markers across API, exports, logs, Redis, queues and storage, static checks, production canary probes | Any success of a cross-company read or write fails the build |
| Golden screenshots | One image per role per width (599, 600, 1023, 1024, 1280, 1440, 1920), light and dark, from the manifest | Diff fails |
| Counter devices | Low-end Android phone, tablet in portrait, Windows PC with 80 mm thermal printer and scanner, Mac | Manual, per release |
| Offline | Airplane mode through a full shift, then sync, with two counters selling the last unit | No duplicates |
| Numbering | Failed posts, parallel posts, offline blocks: no duplicates, every gap explained; separator-inclusive 16-character check | Fail on any |
| Compliance | Sample invoices per tax case (same state, other state, reverse charge, exempt, TDS) checked by an accountant | Sign-off |
| Import | 50,000-row files and files with every kind of error | Pass |
| Accessibility | Screen-reader pass and largest text size every release | Pass |
| Performance | Speed targets on the lowest supported phone; quick load test on PRs touching posting, full one nightly | Over budget fails |
| Query plans | The 30 hottest queries use an index | Collection scan fails |
| Encryption | Read raw documents: no restricted field plain, IVs never repeat, changed byte fails the tag, key rotation keeps old data readable | Any plain field fails |
| Configuration scan | Security headers, cookie flags, open ports on every address | Missing setting fails |
| Contract | OpenAPI vs last release, event schemas, adapters against fakes | Breaking change without `/v2` fails |
| Migration | Run twice on a sample company | Second run must change nothing |
| Security | Dependency audit, secret scan, static analysis, container scan, licence check | High finding fails |
| Soak | 24-hour run, steady memory | Before release |

History, undo, revert and backup tests T-26 to T-48 in [history-and-backup/04](../05-logic/history-and-backup/04-history-api-data-tests.md) also join the gate. Access tests T-01 to T-25 in [access/08-threats-and-tests.md](../06-security/access/08-threats-and-tests.md) join the matrix and isolation suites.

## Gates on every pull request
Strict types, lint (module boundaries, banned imports, size and complexity limits), format, domain coverage floor (start 90% of lines on domain folders), API tests, isolation, contract checks, query plans, security scans, reproducible build, code-owner review.

## Sample test cases (start list)
- Refuses to post when stock is short and the rule is Block.
- Takes the tax out of a tax-inclusive price at 18 percent (118.00 → total 118.00, tax 18.00).
- A repeated idempotency key returns the first result; same key with different body is refused.
- A role without `sales.invoices.add` never sees Add invoice and gets 403 from the API.
- A hidden field is absent from list, detail, search, export and notification.
- A 17-character number format is rejected by the format editor.
- A Stock employee with view+add sees Request correction, not Edit, on a saved entry.
- Switching a module off removes its rail icon, editor rows, cards and search results.
