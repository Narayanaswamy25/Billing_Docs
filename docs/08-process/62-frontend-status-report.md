# Frontend Status Report

Audit date: 2026-11-05. Repository: `Billing_frontend` (Flutter, Dart pub workspace with melos). Method: ran the project gate and every package's tests, read `docs/roadmap.json`, `docs/GAPS.md`, the code layout, and compared with `02-frontend/` and the backend contract. Companion: [61-backend-status-report.md](61-backend-status-report.md).

## 1. Verdict

The foundations are solid and the core daily loop works against fakes: sign in, items, customers, suppliers, stores, invoices (post, cancel, print, credit note, payment), stock on hand and adjustments, purchase bills, people and roles, three reports, dashboard, system pages. That is **21 of 47 planned tasks (45%)**. Nothing has been run against a live backend or on a device. The Counter (offline POS) is still the generated app template. The Console has sign-in, company list and suspend/resume only.

## 2. Gates run today (2026-11-05)

| Gate | Result |
| --- | --- |
| `dart run tool/verify.dart` (forbidden imports, backend contract, hardcoded values, analyze errors and warnings) | **Passed** |
| Tests across 17 packages and 3 apps | **All pass, 0 failures**, about 150 tests |
| Response shapes against the real backend (2026-11-05, backend test harness) | **Matched for every endpoint the app calls**, no mismatch (details in section 5a). Screens in a browser or on a device: never |
| Run on a phone, tablet or desktop build | Never (device-only parts are untested) |
| `flutter analyze` infos | Not checked by the gate (errors and warnings only) |

Tests by package: core 3, domain_shared 5, permissions 2, api_client 7, local_store 7, sync 21, session 8, app_shell 13, workspace 8, counter 1, console 6, stock 12, sales 19, reports 9, purchases 13, people 8, dashboard 8. **No tests**: `ui_kit`, `l10n`, `lookups`.

The contract file `api/openapi.json` is byte-identical in content to the backend's (669 paths).

## 3. What is done

| Area | State |
| --- | --- |
| Architecture | Layer rules enforced by `tool/check_deps.dart`; features never import each other; pickers go through `lookups`. Declarative `ModuleDefinition`s; access resolver hides modules and screens the person cannot use (absent, not greyed) |
| Money | Integer paise with `formatRupees`; a script rejects raw colours, spacing, text and double money |
| API client | Dio, single-flight token refresh, errors mapped to `Result`, idempotency keys on writes, `if-match` on updates |
| Session | Sign-in, secure token store, forgot/reset/accept-invite, authenticator code, first-sign-in tips |
| Stock | Items, stores/warehouses and units, stock levels, adjustments, low-stock list |
| Sales | Customers, invoice list and new invoice (draft then post), cancel, on-device PDF print and share, credit notes, payments, receivables ageing |
| Purchases | Suppliers, purchase bills |
| People | Users and roles list |
| Reports | Sales summary, profit and loss, stock valuation, receivables |
| Dashboard | Setup checklist, approvals waiting, my tasks, money to collect |
| System pages | Not available, session expired |
| Console | Platform sign-in, company list, suspend/resume |
| Counter foundation | Encrypted local store (SQLCipher) and sync outbox (idempotency keys, per-document order, 50-number block ledger), unit-tested |
| Tooling | `verify`, `check_contract`, `check_hardcoded`, `endpoint_brief`, `next_step`, roadmap with evidence files, `AUTONOMY.md`, `GAPS.md` |

## 4. What is not done

### 4.1 Remaining roadmap tasks (26 of 47)

| Group | Tasks not built |
| --- | --- |
| Foundations | x1 `/me` and plain-words My access |
| Sales | s5 quotes and sales orders; s6 receipts list and customer statement |
| Stock depth | q2 stock counts and transfers; q3 batches, expiry, serials; q4 price lists |
| Payments | r2 reminders and cheques; a1 journal and ledger; a2 bank view and reconciliation |
| GST | e1 e-invoice status, retry, e-way bill; a3 tax registers and GSTR data |
| Purchases | u3 purchase orders, goods receipts, bill matching; u4 approvals inbox and rules; u5 roles list, access editor, Activity tab; h1 HR staff, attendance, leave |
| Settings and admin | g2 branches, numbering, tax, print, rules; g3 integrations; g4 backups, recycle bin, restore as of a date; x2 import |
| Console | c2 create company, module switches, offboarding; c3 plans, subscriptions, features, usage; c4 support access, platform log, staff |
| Counter | k2 offline billing (scan, bill, offline, shift); k3 held bills, customers, returns; k4 shift and sync screens |
| Quality | i1 accessibility pass and second language |

### 4.2 Reach into the backend

The frontend calls about 35 of the backend's 669 paths (under 6%). Modules with **no screen at all**: accounting (76 paths), HR/payroll (59), India GST (47), packs (47), CRM (36), messaging (40), promotions (19), compliance (13), integrations (14), privacy (14), POS beyond sync, support (5). The docs call some of these "later" (payroll, GST portal filing, loyalty, manufacturing), but accounting, GST and bank screens are in release 1.

### 4.3 Dashboard gaps
Only 4 of the documented cards exist. Missing: Alerts section, sales today, stock alerts, payments due, attendance today, latest access changes, items to post, bank items, GST due. The approvals card is read-only until u4.

## 5. Errors and bugs

| ID | Severity | Finding | Where |
| --- | --- | --- | --- |
| FE-0 | Info | A new company has **no roles**, so the invite form has no role to choose until roles exist | live check |
| FE-1 | None now | No compile error, analyze error, warning or failing test | verify and tests above |
| FE-2 | Fixed | Reports package did not compile earlier today (missing `alsoViewableWith`, three permission keys, ARB strings, fakes). Fixed the same day, with cubit tests | `docs/GAPS.md` |
| FE-3 | Medium (was High) | Response shapes are now **checked against a real backend run** (section 5a). Remaining: no screen has been rendered in a browser or on a device; the dashboard has not seen non-empty tasks | GAPS "Unproven" |
| FE-4 | High | The contract has no response shapes, so wrong field names pass every check | `api/openapi.json`, see backend BE report section 7 |
| FE-5 | Medium | Update-required page and maintenance page: classes exist (`UpdateRequiredFailure`) but nothing produces them; backend sends no signal | GAPS "Blocked by backend" |
| FE-6 | Medium | First-sign-in tips are skipped when the sign-in token has no user id. The wording (D-053) is a first draft, a lawyer must review | GAPS follow-ups |
| FE-7 | Medium | Dashboard approvals card needs `workflow.tasks:view`, so an approver without it sees no approvals | GAPS follow-ups |
| FE-8 | Medium | Console has no tips store and relies on a platform session built separately; staff screens, create company and support access are missing, so it cannot yet onboard a customer | `apps/console` |
| FE-9 | Medium | **Counter app is the Very Good template** (`AppBlocObserver`, "Add cross-flavor configuration here", about 300 lines). The sync library under it is real but nothing in the app uses it yet | `apps/counter` |
| FE-10 | Medium | Sync outbox and encrypted store are device-only, so SQLCipher behaviour on real Android and iOS is unproven | GAPS |
| FE-11 | Low | `ui_kit`, `l10n`, `lookups` have no tests. `ui_kit` is the base of every screen | packages |
| FE-12 | Low | Only English; ARB is the single string source, second language is task i1 | `packages/l10n` |
| FE-13 | Low | Roadmap evidence is only "file exists", so "done" does not prove it compiles. The gate must run first (it did, today, and passed) | `roadmap.json` note |
| FE-14 | Low | `flutter analyze` info-level lints are ignored by the gate | `tool/verify.dart` |
| FE-15 | Process (commit done, remote not) | **38 changed or untracked paths with a single commit** ("first commit"). Dashboard, system pages, all `tool/*` scripts, `GAPS.md` and `AUTONOMY.md` are uncommitted. One bad command loses a day's work | git |
| FE-16 | Process | The dashboard was checked by its author only, with no separate checker and no calibration | GAPS |
| FE-17 | Info | Frontend docs `07` (folder structure), `08` (design system tokens, wireframes), `09` (packages), `36` (component inventory) are still Draft, so the code is ahead of its own docs | `02-frontend/` |

## 5a. Live shape check (2026-11-05)

A temporary spec on the backend's test harness (real app, temporary MongoDB; Atlas was not used; the spec was deleted afterwards) called every endpoint the repositories use and compared the JSON with the Dart parsers. Checked and matching: items, customers/suppliers, warehouses, branches, units; stock adjustments and balances; invoice create, post, list, get, cancel; receipts; credit notes; purchase bills and payments; the four reports; users, roles, `/me/permissions`, workflow tasks and requests; sign-in and platform sign-in (TOTP required); company suspend and resume. Error bodies match the mapper (`code`, `message`, `fields`, `requestId`). Not covered: rendering, the dashboard with real tasks, the console list with real companies.

## 6. Code versus docs

| Docs | Code | Note |
| --- | --- | --- |
| Permission keys `<module>.<screen>.<verb>` with 15 verbs | `module.feature:action`, generated from the backend's `x-permission` | Follows the backend. Update the docs |
| Accessibility WCAG 2.2 AA (`37`) | No accessibility pass or tests yet | Task i1 |
| Second language (`38`) | English only | Task i1 |
| Counter works offline for a full shift (D-005, D-027) | Library only, and the backend lacks terminal registration | Blocked by backend B-7 |
| Drawer: Counter drafts device-only (F-11) | Not built | Waiting on owner |

## 7. Recommended order

1. ~~Commit~~ done 2026-11-05 (commit `1a24ef1`). Still needed: push to a remote (FE-15).
2. ~~First live run for shapes~~ done 2026-11-05, no mismatches. Next: run the Workspace app in Chrome against a local backend with seeded data (sign in, one invoice, one report) to catch rendering and flow problems (FE-3).
3. Ask the backend to publish response schemas (FE-4); then check them in `check_contract`.
4. Release-1 core still missing: s5, s6, u3, u4, u5, x1, then accounting (a1 to a3) and settings (g2).
5. Console c2 to c4, so a company can be created without scripts.
6. Counter k2 to k4 once the backend has terminal registration and number blocks.
7. Add tests for `ui_kit`; run the accessibility pass (i1) while there are still few screens.
8. Update docs 07, 08, 09 and 36 from the code.
