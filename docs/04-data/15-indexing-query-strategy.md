# Indexing and Query Strategy

The index list itself is in [14-data-model.md](14-data-model.md). This file is the policy.

## Rules
1. **One index per query pattern**, equality fields first, then sort, then range (ESR). Partial indexes keep open and unpaid lists small; indexes skip soft-deleted rows.
2. **Index budget:** about four indexes per collection, **each starting with `companyId`** (write speed and memory; the old file-count limit no longer applies). Adding an index needs a query-plan test and a migration; drop unused ones after the weekly slow-query review.
3. **No collection scan on hot collections.** The 30 hottest queries have plan tests; CI fails on a scan or too many documents examined per document returned.
4. **Paging:** keyset on an indexed field, never skip. Max page size set; totals optional.
5. **Projection always:** lists return only the listed fields.
6. **Time limit on every query** (`maxTimeMS`, per route; e.g. 2000 ms for detail reads).
7. **Scope filter on every query** (branches, Own/Team/Branch/All) built by the repository; the filter fields are part of the index design (e.g. branch first for branch-scoped lists).
8. **No joins on hot paths.** Documents hold snapshots; reports read summary collections written by workers; secondaries serve reports where seconds of lag are fine.
9. **Aggregations** start with a match and a sort an index serves.
10. **Bulk operations** for batches; transactions short, retried on transient errors, hot documents avoided.
11. **Majority write concern** for postings; default for ordinary reads.
12. **Index builds** are schema changes: run outside business hours, one company at a time; watch for aborted transactions.

## Expected query patterns
| Pattern | Collection | Index |
| --- | --- | --- |
| Find a document by number | sales_documents | type, series, number (unique) |
| A customer's history and statement | sales_documents | partyId, businessDate |
| Overdue / ageing lists | sales_documents | status, dueDate (partial: open credit) |
| Stock card, valuation | stock_ledger | itemId, warehouseId, at |
| Conditional stock decrement | stock_balances | itemId, warehouseId, batchId (unique) |
| Ledger, trial balance | journal_entries | lines.accountId, businessDate |
| Counter and panel search | items, parties | lower-case search key; barcode and tax number unique |
| Attendance per person per day | attendance | employeeId, date (unique) |
| Outbox relay | outbox | status, createdAt |
| Session and invitation cleanup | sessions, users | expiresAt (TTL) |
| Idempotent posting | sales_documents / pos_sync_log | idempotency key (unique) |
| Exact private-field search | parties, employees | blind-index field |

## Review cadence
Weekly slow-query review; quarterly index audit (unused, duplicate, oversized); index size and memory per cell watched.
