# Database Schema / Data Model

Field-level dictionary (every table's variables) is **not written yet**: the *Business Rules and Data Dictionary* document referenced by the architecture was not supplied. This file fixes conventions and the collection layout; per-field detail goes in `04-data/data-dictionary/<module>.md` as each module is built.

## Conventions
| Topic | Rule |
| --- | --- |
| Keys | ObjectId `_id`. **Every company-owned document has an immutable `companyId` (ObjectId)** as its first field; `$jsonSchema` requires it; platform collections have their own keys |
| Indexes | **Every index on a company collection starts with `companyId`**, including unique ones (uniqueness is per company); no cross-company unique constraints |
| Tenancy | Companies share collections (D-039); the data access layer adds `companyId` to every operation |
| Money, quantity, rates | Decimal128 only; one money service applies the company rounding rule |
| Dates | UTC timestamps plus a `businessDate` (plain date in the company's time zone) for period locks and reports |
| Audit fields | `createdAt, createdBy, updatedAt, updatedBy, version`; updates send the version read; mismatch → 409 |
| Delete | Soft delete (`deletedAt, deletedBy`); indexes skip deleted rows; parties with history are deactivated, not deleted |
| Lines | Embedded in the document header with a settable maximum line count; ledgers, journals and logs are never embedded |
| Snapshots | Posted documents copy party name/address/tax number, item name, price and tax rate, plus links to masters |
| Validation | Every collection has a `$jsonSchema` validator generated from the API schemas |
| Status | One lifecycle on every document: draft → confirmed → posted → closed; cancelled (see [21-state-machines.md](../05-logic/21-state-machines.md)) |
| Encrypted fields | Envelope `{keyId, iv, ciphertext, tag}` plus a blind-index sibling where exact search is needed |

## Collection budget (66 in total for the whole cluster, not per company)
Related documents share a collection with a `type` field. Keep indexes to about four per collection to protect write speed and memory; the old per-company file-count limit no longer applies (see [54-scalability-plan.md](../01-architecture/54-scalability-plan.md)).

| Area | Collections | # |
| --- | --- | --- |
| Core | users (includes invitations, with TTL index), roles, sessions, settings, audit_log, **record_revisions**, number_series, files, outbox, notifications, workflow | 11 |
| Masters | parties, items, price_lists, taxes, lookups, branches, warehouses, custom_fields | 8 |
| Sales | sales_documents (quotation, order, delivery, invoice, notes, recurring templates), receipts, wallet_entries | 3 |
| POS | pos_terminals, pos_shifts, pos_sync_log | 3 |
| Purchases | purchase_documents, supplier_payments | 2 |
| Inventory | stock_ledger, stock_balances, batches_serials, cost_layers, stock_documents, production | 6 |
| Accounting | accounts, account_mapping, journal_entries, expenses, banking, cheques, assets_loans_budgets | 7 |
| Promotions and CRM | offers, coupons_giftcards, loyalty_ledger, leads_activities | 4 |
| HR | employees, attendance, leave, payroll (reserved; D-007), hr_settings | 5 |
| India and compliance | gst_returns, tax_entries, einvoice_records, eway_bills, portal_log | 5 |
| Industry packs | retail, restaurant_orders, restaurant_floor, services, wholesale_routes, wholesale_schemes | 6 |
| Messaging and integrations | message_log, templates_rules, connections, webhook_log | 4 |
| Privacy and reports | consents, report_definitions | 2 |
| **Total** | | **66** |
Collections for out-of-scope modules (production, services, loyalty) are created only when the module is switched on.

## Platform database
companies, tenants (company, cell, status including Quarantined, schema version, provisioning status, key references), cells (name, connection reference, company count, size and load limits, status), **identities** (login, Argon2id hash, encrypted two-step secret, recovery codes, status), **memberships** (identity ↔ company ↔ company user; replaces `login_directory`), modules, plans, agreements (plan, add-ons, overrides, dates), **company_entitlements** (resolved set and version), company_modules, platform_staff, platform_roles, platform_audit, support_grants, recovery_cases, usage_meters, subscriptions, key vault (wrapped data keys), security and breach registers. See [access/](../06-security/access/00-README.md).

Company side (inside the 66-collection budget): `users` holds profile, role, branches, `reportsTo`, personal changes, limits, seat status and invitations (no password); `roles` holds permissions, scopes, field modes, limits and `assignableRoles`; `sessions` holds sessions and the device list; `pos_terminals` holds registered Counters, their keys and number blocks.

## Where the advanced logic lives (no new collections; the 66 hold)
| Need | Stored in |
| --- | --- |
| Item units, MRP, reorder parameters, compliance flags | `items` |
| Demand statistics, reserved, incoming | `stock_balances.stats` and fields (rolling aggregates written by workers) |
| Reservations, conversions, wastage, counts, weighment slips | `stock_documents` (type field) |
| Lots, batches, serials | `batches_serials` |
| Sale returns, challans, account sales, quotes | `sales_documents` (type field) |
| Purchase returns, claims | `purchase_documents` (type field) |
| Reminder rules | `templates_rules` |
| Reminder instances | `notifications` |
| Compliance rules, licence numbers, rate board | `settings`, `lookups` |
| Scale register, licence register | `settings` (typed entries) |

## Ledger rules
- **stock_ledger** is append-only; a mistake is a reversing row. **stock_balances** holds quantity and value per item, warehouse and batch, updated in the same transaction and rebuildable from the ledger. Weighted average on the balance; FIFO uses `cost_layers`.
- **journal_entries** holds the lines; the service refuses debit ≠ credit. Reports read monthly balance snapshots plus lines after the last snapshot.
- **audit_log** is append-only and now holds field-level change events; **record_revisions** holds full snapshots of editable records (see [history-and-backup/04](../05-logic/history-and-backup/04-history-api-data-tests.md)). Both are written in the same transaction as the change.
- **Number blocks** (50 per device by default) are stored in `number_series` as block entries: device, first and last number, issued time, status Issued, Used, Synced, Complete or Void. No extra collection.

## Key indexes
| Collection | Index | Serves |
| --- | --- | --- |
| sales_documents | type, series, number (unique) | Number lookup, no duplicates |
| sales_documents | partyId, businessDate | Customer history, statements |
| sales_documents | status, dueDate (partial, open credit only) | Overdue and ageing |
| stock_ledger | itemId, warehouseId, at | Stock card, valuation |
| stock_balances | itemId, warehouseId, batchId (unique) | Conditional update in posting |
| journal_entries | lines.accountId, businessDate | Ledger, trial balance |
| items, parties | lower-case search key; unique barcode and tax number | Counter search |
| attendance | employeeId, date (unique) | One row per person per day |
| outbox | status, createdAt | Relay |
| sessions, users(invitations) | expiresAt TTL | Cleanup |
| posting idempotency | idempotency key (unique) | Safe retries and repeat offline sync |
Rule: equality fields first, then sort, then range; partial indexes keep open lists small; no collection scans on hot paths (query-plan tests).

## Reports and growth
Dashboards and heavy reports read summary collections written by workers (and secondaries when seconds of delay are fine). Closed years move to object storage after retention.
