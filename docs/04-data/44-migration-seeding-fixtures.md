# Data Migration, Seeding and Fixtures

## Schema migrations
- One schema version for the whole cluster; documents may carry `schemaVersion`; the platform registry records the cluster version.
- A release runs one migration in **company batches**, canary and internal companies first, with pause, resume and a stop-on-error threshold (see tenancy/04).
- **Expand then contract:** new code reads both shapes → migration moves data → a later release drops the old shape. Code supports the current and previous schema version, so a batch not yet migrated still works and rollback is a redeploy.
- Every migration is idempotent and is run twice on a copy of a sample company in CI; the second run must change nothing.
- Index builds are migrations: rolling, outside business hours, one collection at a time; each starts with `companyId`.
- Repairs are reviewed scripts in `tools/`, logged; nobody edits production data by hand.

## Seed data when a company is created
Provisioning (idempotent by company ID) seeds:
| Seed | Contents | Status |
| --- | --- | --- |
| Roles | Templates: Company admin, Manager, Accountant, Billing officer, HR officer, Salesperson, Stock employee, with tick-level verbs per module | Module-level defaults in 22; **tick-level seed file not written** |
| Business-type preset | Which modules start switched on (retail, restaurant, services, wholesale); new company starts with Sales and Stock | Not written |
| Chart of accounts | Default accounts and `account_mapping` for automatic posting | **Needs accountant** |
| Number series | One per document type, branch and financial year; format with effective date, ≤16 characters | Format examples in 18 |
| Tax and statutory settings | GST rates, HSN/SAC rules, e-invoice and e-way bill thresholds, TDS/TCS sections as dated settings | **Needs tax advisor** |
| Default business rules | Valuation, negative stock, tax mode, discount order, posted-document policy (ReversalOnly), credit rule, posting mode | Options in 10 |
| Lookups | Units, reasons (damage, loss, found, expired, other; return, price change, error), payment methods | Partial |
| Templates | Message and print layouts per language | Needs wording (43) |
| Setup checklist | Three to five steps with Skip; sample data on request | In 19 |

## Opening data for a customer
Import templates (Excel/CSV) for parties, items, opening stock and opening balances; debit = credit check before posting the opening; go-live date.

## Fixtures for tests
- Two-company fixture (companies A and B, different branches, one user per role each) used by isolation and permission-matrix tests.
- Reference company for load tests: defined profile (users, invoices per day, lines per invoice, stock movements per day, attachments); generator scripts create synthetic companies up to and past the planned cell count.
- Tax-case invoice set: same state, other state, reverse charge, exempt, TDS (accountant-checked).
- Fake GSP and fake payment gateway in `packages/testing`.
- Sample data for the "sample data on request" onboarding option; clearly marked and removable.
- Clock fixture to freeze time.
