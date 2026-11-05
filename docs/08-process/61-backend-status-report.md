# Backend Status Report

Audit date: 2026-11-05. Repository: `Billing_backend`. Method: ran the gates, read `docs/HANDOFF.md`, `docs/checklist/`, `docs/openapi.json`, source markers (`TODO`) and compared with this docs set. Companion: [62-frontend-status-report.md](62-frontend-status-report.md).

## 1. Verdict

The business logic is broadly built and well tested. Everything that needs real infrastructure (Redis, providers, KMS, cloud, load and penetration tests) is not done and never ran. The code does **not** follow four decisions in these docs on purpose (section 5). The backend is ready for a front end to build on, not for production.

## 2. Gates run today (2026-11-05)

| Gate | Result |
| --- | --- |
| `npm run typecheck` | Clean |
| `npm run lint` | Clean |
| `npm test` | **125 spec files, 811 tests, 0 failures** (206 s) |
| Real infrastructure (Atlas, Redis, Docker, CI, Terraform) | Never run |

Note: the test log prints a stack trace ("Unsupported state or unable to authenticate data", `envelope.ts`). The suite still passes, so it looks like a deliberate tamper-detection test. Worth confirming that the test asserts it, so it is not hiding a real decrypt failure.

## 3. What is done

| Area | State |
| --- | --- |
| Size | About 20,000 lines of source, 669 API paths in `docs/openapi.json` (328 distinct permission keys), 19 company modules plus the platform realm |
| Foundation | Tooling, kernel (context, permissions, audit, idempotency, outbox, CRUD layer, approvals, scheduler), money as Decimal128, encryption envelope, layer rules enforced by ESLint and `test/architecture/` |
| Tenancy | Database per company grouped into cells, `TenantRouter`, provisioning, per-company migrations (version 10), export and restore, isolation tests |
| Modules | identity, masters, tax, inventory (batches, serials, reservations, pick and pack, manufacturing), accounting (banking, cheques, loans, budgets, snapshots, year-end close), sales (quotes, orders, invoices, receipts, credit notes, wallet), pos, purchases (requests, quotes, over-receipt, landed cost), workflow, promotions, hr (including payroll), india (GSTR-1, 3B, e-invoice, e-way, TDS/TCS), compliance, messaging, crm, integrations, reporting, privacy, packs |
| Platform realm | Companies, plans, billing, support, breach register, platform audit |
| Worker | Outbox relay with change streams, automatic handlers, daily scheduled jobs, retries |
| Contracts | OpenAPI generated, with a test that fails on breaking changes. The copy in the frontend is identical to this one |
| History | Change history and revert (ADR 0017) |
| Docs in repo | 17 decision records, 8 runbooks, README per module |

## 4. What is not done

### 4.1 Needs infrastructure this environment does not have
1. **Redis wiring.** BullMQ queues (critical, default, bulk), shared rate-limit store, pub/sub for module-switch broadcast, server-sent events. Ports and in-memory versions exist. Rate limiting is per process, so it multiplies with each API instance.
2. **Real provider adapters.** GST Suvidha Provider (`NotConfiguredGsp`, so e-invoice fails loudly), SMS/email/WhatsApp (`NotConfiguredProvider`), payment gateway, bank feed, S3 storage (a local folder is used). The notifier is `MemoryNotifier`, so **invitation and reset emails are not delivered**.
3. **KMS/HSM adapter.** Only `LocalKms` exists; production refuses to start with it. Mutual TLS to the key service is written but never exercised. Re-encryption after a key rotation is not built.
4. **Load test and penetration test.** No performance figure in the docs is measured (`31-non-functional-requirements`, `54-scalability-plan`).
5. **Deployment.** Dockerfile, CI workflow (`.github/workflows/ci.yml`) and `infra/mongod.conf` were written and never executed. No Terraform or Kubernetes. No point-in-time backup, restore drills or status page.

### 4.2 Can be built here, still open
| Module | Open items |
| --- | --- |
| sales | Delivery notes, debit notes, print/PDF of documents, refunds to a customer who already paid (credit note refuses to exceed what is owed), POS bills not covered by the discount approval rule |
| accounting | Linking a cleared cheque to the invoice or bill, accrued loan interest, more snapshot reports |
| purchases | Request for quotation document, approval limits by amount |
| hr | Accrued leave balances, surcharge in TDS projection, previous-employer income |
| india | GSTR-2B download (needs provider), reverse charge, interest and late fee, tying TCS to the receipt |
| messaging, crm | Real provider adapter, WhatsApp template approval, follow-up task from a low rating |
| packs | Table transfer, bill split, route-wise delivery, wholesale schemes inside invoices |
| reporting | Scheduled reports, dashboards from summary collections, Excel and PDF export (13 report endpoints exist) |
| privacy | Purpose limits enforced across modules |
| inventory | Expiry push notification, serial reservations, reversing a production run |
| platform | Per-company limit on running jobs, build the app from module manifests (the list in `src/app/modules.ts` is hand-written), platform host check, alert rules, daily copy of the audit hash to write-once storage, graceful-shutdown readiness drain |
| quality | Validators generated from Zod (hand-written today), OpenAPI generated from Zod, contract check against the last release, CSRF (only needed if sessions move to cookies) |
| history (ADR 0017) | Change sets and bulk revert, restore as of a date, restore points, backups screen, revision compaction, recycle-bin archive job, step-up for sensitive reverts |

## 5. Differences from this docs set (code vs docs)

The owner chose to keep the code stack and align behaviour (ADR 0017). These remain open. The docs have not been updated to say so, which is itself a problem: `CLAUDE.md` here still tells readers the backend is Express with shared collections.

| # | Docs say | Code does | Impact |
| --- | --- | --- | --- |
| B-1 | Express (D-021) | NestJS 12 on Fastify | Docs wrong or code wrong; pick one and record it |
| B-2 | Shared database with `companyId` on every document (D-039), `packages/data-access` | One MongoDB database per company in cells (ADR 0001) | Whole `06-security/tenancy` set describes a different design |
| B-3 | Permission keys `<module>.<screen>.<verb>`, 15 verbs | `module.feature:action` (for example `accounting.accounts:view`); verbs include create, edit, delete, revert, view | Frontend follows the code. The permission catalog in `access/03` does not match |
| B-4 | Payroll out of release 1 (D-007) | Payroll built in `hr` | Scope creep; harmless but it needs legal review before use |
| B-5 | Five-check order and field-level visibility per role | Not fully implemented (`FIELD_NOT_VISIBLE`, `AGE_LIMIT` blocks missing) | Security-model gap |
| B-6 | Identity with memberships and company switcher (D-025), step-up authentication | Not built | A person in two companies cannot switch |
| B-7 | Registered Counter terminals and offline number blocks of 50 (D-005, D-027) | Not built on the server | **Blocks the offline Counter** (frontend task k2) |
| B-8 | Business profiles (D-032), Trader and Farm flows | Not built | Release 1.5 per docs |
| B-9 | "Update required" signal for old clients (`46-api-versioning-changelog`) | Nothing sent | Frontend page cannot work |
| B-10 | Company maintenance mode with read-only banner | No state or error code | Frontend page cannot work |
| B-11 | Handoff said "239 paths, 347 operations" | The generated file has **669 paths, 800 operations** | **Fixed 2026-11-05** in `HANDOFF.md` |
| B-12 | Handoff said there is no front end and Node 26 is missing | Frontend exists in a sibling repo; Node 26.13.0 is under `~/.nvm` | **Fixed 2026-11-05** in `HANDOFF.md` and `AGENTS.md` |

## 6. Errors, risks and bugs found

No failing test and no compile error. These are risks and weak spots found by reading.

| ID | Severity | Finding | Where | Suggested fix |
| --- | --- | --- | --- | --- |
| BE-1 | High | The email lookup (blind index) key is **derived from the password pepper** instead of coming from the key vault. One leaked secret exposes both | `src/app/build-deps.ts:55` (TODO key-service) | Fetch the key from the key service before any real user data |
| BE-2 | High | Invitations and password-reset messages go to `MemoryNotifier`, so in a real deployment **nobody receives them**. Frontend sign-up, invite and forgot-password flows cannot work end to end | `build-deps.ts:37` | Email adapter, or a documented dev-only mode |
| BE-3 | High | E-invoicing and all messaging fail by design until a provider is chosen (D-013 open) | `build-deps.ts:39-41` | Choose the vendor; keep the port |
| BE-4 | Medium | Rate limiting is in process memory. Two API instances give double the allowed rate, which weakens the brute-force limit on sign-in | `ARCHITECTURE`, checklist 02 | Redis store behind the same port |
| BE-5 | Medium | Storage is a local folder. Exports and PDFs vanish with the container | `build-deps.ts:44` | S3-compatible adapter |
| BE-6 | Medium | `CSRF` not applicable while tokens are bearer; any move to cookie sessions must add it first | checklist 05 | Keep as a recorded constraint |
| BE-7 | Medium | Statutory data (GST rules, payroll slabs, TDS) are defaults a chartered accountant must confirm | HANDOFF "Known limits" | Add to the pre-launch checklist as owned items |
| BE-8 | Medium | Invoices are never edited; credit refuses amounts above what is owed, so a customer who overpaid cannot be refunded in cash | sales | Build refunds and debit notes |
| BE-9 | Low | The module list is hand-written, so a new module can be built but left out of the app by mistake | `src/app/modules.ts` | Build from manifests; there is a cycle check already |
| BE-10 | Low | Environment traps: tests need a downloaded `mongod`; they run one file at a time; Prettier width 220 reflows files and silently drops scripted edits | HANDOFF | Documented only |
| BE-11 | Low | The repo has two commits ("first commit") and `CLAUDE.md` and `graphify-out/` are untracked | git | Commit; add `graphify-out/` to `.gitignore` if it should stay local |
| BE-12 | Info | `.env` holds a real Atlas string; the cluster also holds the user's other project's database (`allinone_dev`). Tests never touch it | HANDOFF | Keep it out of logs; consider a separate cluster or user for this project |

## 7. What the frontend needs from the backend

| Need | Status |
| --- | --- |
| Response shapes in OpenAPI | **Missing**: the contract has routes, permissions and request bodies, not responses. The frontend reads controllers and e2e specs. A live check on 2026-11-05 found every shape the frontend uses correct, but nothing keeps it that way, so this stays the biggest contract risk |
| Update-required signal, maintenance state | Missing (B-9, B-10) |
| Offline Counter support (terminal registration, number blocks) | Missing (B-7). `/api/v1/pos/sync` exists |
| Cross-module "home card" data (sales today, low stock, payments due) | Each needs separate calls today; no aggregated dashboard endpoint |
| Excel/PDF export for reports | Missing (frontend builds invoice PDFs on the device) |
| Real email delivery | Missing (BE-2) |

## 7a. Live shape check (2026-11-05)

Every endpoint the frontend calls was run on the test harness and the responses matched the Flutter parsers (list in the frontend report, section 5a). Observation for the backend: a new company has no roles, so the user invite flow needs seeded default roles (docs `access/04-role-seeds-and-limits.md`).

## 8. Recommended order

1. Record B-1 to B-3 as accepted deviations in the decision log (stale handoff numbers B-11 and B-12 are fixed; the backend edits are not committed yet).
2. Add response schemas to OpenAPI (generated from Zod) and a contract test. This makes the frontend work reliable.
3. Close BE-1 and BE-2 (key vault index key, email adapter).
4. Server pieces the frontend is waiting on: update-required header, maintenance state, terminal registration with number blocks.
5. Redis queues and shared rate limit, then S3 storage.
6. Sales gaps: delivery notes, debit notes, refunds, document PDF.
7. Load test, penetration test, deployment dry run, Terraform.

## Re-check 2026-11-05 (second run)

`npm run typecheck` and `npm run lint` clean; `npm test` 125 files, 811 tests, 0 failures. Logged stack traces ("unable to authenticate data" in `envelope.ts`) during the run come from the tamper-detection tests and are expected. Atlas and `.env` were not used. Nothing new found missing; the open items in the sections above still stand (response schemas in OpenAPI, update-required and maintenance signals, terminal registration, email adapter, Redis and provider adapters).

### Update 2026-11-05: update-required and maintenance signals built
New `ClientPolicyGuard` (`src/app/client-policy.guard.ts`): a request whose `X-App-Version` is below `MIN_CLIENT_VERSION` gets 426 `UPDATE_REQUIRED` (details carry the minimum); with `MAINTENANCE_MODE=true` every write gets 503 `MAINTENANCE` with `Retry-After` while reads and `/health` still work. A request with no version header is never refused for its version. Both are settings, also changeable at runtime through `deps.clientPolicy`. Covered by `client-policy.e2e.spec.ts` and `client-version.spec.ts`. **Not built:** per-company maintenance (restore, move) and an operator endpoint to flip the flags without a restart.
