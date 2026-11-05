# Roadmap and Task Breakdown

No dates: team size, budget and pilot customers are unknown (question 8). Order is fixed by the UI spec's build order; backend work is added per step. A step starts only when the previous step's gate passes. Status key: `[ ]` not started, `[~]` in progress, `[x]` done.

## Step 0: Settle decisions (before any code)
- [x] D-001 client stack: Flutter (owner, 2026-10-31)
- [x] D-021 backend: Node.js + Express + MongoDB (owner, 2026-10-31)
- [x] D-039 shared database with strict isolation (owner); [ ] cell sizing method and trigger for sharding (54)
- [x] D-005 offline numbering: blocks of 50, next block only after sync (owner); legal basis recorded (GST Rule 46(b)); accountant review stays in the pre-launch checklist
- [x] D-007 payroll out of release 1
- [x] D-012 MongoDB Atlas; [ ] D-014 cloud and region
- [x] D-013 route: GSP adapter; [ ] choose the GSP vendor
- [x] D-006 panel is 25% (owner)
- [ ] Align the role document to the 15 verbs (see REVIEW-FINDINGS F-02, F-06, F-07)
**Gate:** every Proposed decision is Accepted or changed.

## Step 1: Foundations
(Includes change events, revisions and the revert verb written by the data access layer in the same transaction; history/backup design: 05-logic/history-and-backup.)
Client: manifest, shell, rail, permission gate, pane widths, design tokens. Backend: kernel (context, audit, numbering, outbox, settings), tenancy (tenant context, **tenant data access layer**, registry, provisioning, migration runner, key service, isolation test suite and canary probes: tenancy/01 and 05), identity (users, roles, sessions, sign-in, PIN, two-step), security package (envelopes, hashing, key-service client), catalog (modules and switches), permission engine with the five checks, identities and memberships, entitlements resolver, registered Counter terminals, offline grant (access/01, 02, 07). 15 verbs, 4 scopes, access editor with presets and preview, permission test matrix from day one, isolation suite.
**Gate:** matrix and isolation suites pass; a module switch removes everything for a company.

## Step 2: Getting started (add: business profile setup wizard and profile presets, advanced/01)
Masters (parties, items, units, price lists, branches, warehouses), import with undo, opening balances, "Who works here?" setup, + New, Dashboard checklist.
**Gate:** first bill in under 3 minutes in a timed test.

## Step 3: Sales
(Add: guided reversal flows for posted documents, self-undo, recycle bin.)
Document flow, numbering (16-character rule, blocks), posting transaction, print and share, the Counter (scan-first, offline, sync, shift), opening on the task, silent drafts, undo, plain words, Counter speed pass. (Swap with Step 6 if the first customer needs e-invoicing from day one.)
**Gate:** offline suite, numbering suite, speed targets, five-staff test.

## Step 4: Payments
Payment modes, split/part payment, credit limit, ageing, reminders, cheques, Bank view and reconciliation.

## Step 5: Stock depth (expanded)
Inventory engine (advanced/02): states and location types, units and conversions, FEFO, reservations and backorders, landed cost, repack, counts by ABC, wastage; restocking maths, Restock list and PO drafts; reminder engine and catalogue (advanced/03); returns engine with dispositions and supplier claims (advanced/05).
Batches, serials, valuation, transfers, counts, adjustments, Request correction.

## Step 6: GST compliance
GSP adapter, e-invoice states and retry, e-way bill, reverse charge, TDS/TCS, tax registers, GSTR data, pending-ageing alert. **Gate:** accountant signs off sample invoices per tax case.

## Step 7: Purchases and approvals
Purchase documents, approval engine, notifications, My access, mid-session rules, Activity tab (needs the `team` field first, D-016), HR attendance and leave.

## Step 7b: Release 1.5 profiles
Trader and Farm: weighment, lots, rate board, account sale, Simple mode (advanced/06).

## Step 8: Finish (add: Backups screen, restore points, restore as of a date, monthly restore tests, company restore process)
Reports, Console additions (subscriptions, support access, offboarding, platform log), security hardening, visual polish, launch checks (pen test, restore drill, pre-launch checklist in 23-security-policy).

## Later (not release 1)
Payroll, GST portal filing, restaurant and wholesale packs beyond the first pack, customer portal, online store, loyalty, manufacturing, multi-currency, offline Workspace.

## Merge note
The architecture doc also defines seven phases with gates (diagram not readable as text). Compare them with the steps above and record differences in the decision log.
