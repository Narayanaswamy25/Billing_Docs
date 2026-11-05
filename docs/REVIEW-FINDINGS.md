# Review Findings: problems found in the source specs

Reviewed on 2026-10-31. Sources: *System Architecture and Design* (28 sections), *Front-End UI Specification*, *What Each Role Sees*, *Access and Visibility Workflow*. Where two sources disagree, this file records which one the docs follow. Items marked **Needs owner** are proposals waiting for a decision (see [01-architecture/29-decision-log.md](01-architecture/29-decision-log.md)).

Status key: **Resolved** (docs already follow the fix), **Needs owner** (proposed, not settled), **Verify** (outside fact not confirmed).

## A. Contradictions between the specs

| ID | Sev | Finding | Resolution in these docs | Status |
| --- | --- | --- | --- | --- |
| F-01 | High | **Client stack.** Architecture picks React + Vite, service worker, IndexedDB, HttpOnly cookies, a local print agent. The UI spec's last sections build everything in Flutter (`app_shell`, `ui_kit`, `go_router`, `Bloc`) and targets Windows, Mac, Android and tablets with thermal printers. | Docs follow Flutter for all clients (UI spec is later and more detailed; it targets hardware that Flutter reaches natively). Knock-on changes: offline store becomes a local DB (Isar or Drift) instead of IndexedDB; auth uses bearer tokens in secure storage instead of cookies; shared Zod schemas become OpenAPI-generated Dart models; the print agent becomes native printer plugins. Backend is unchanged. | **Resolved** (D-001: Flutter, owner 2026-10-31) |
| F-02 | High | **Three permission-key formats and two verb lists.** Architecture: `sales.invoice` + actions `view/create/edit/cancel/print`. UI manifest: `stock.items.view`. UI router example: `stock.see`. UI spec fixes 14 verbs; architecture lists 5. | One format: `<module>.<screen>.<verb>`, lowercase, the 14 verbs, scope stored separately. Architecture `create`→`add`, `edit`→`change`. `see`/`edit`/`full`/`own` from the role doc are retired. See [06-security/22-authentication-authorization.md](06-security/22-authentication-authorization.md). | Resolved |
| F-03 | High | **Four checks vs five checks.** Architecture says four (module, role, override, record scope). Access doc says five. | Five checks, in this order: module on → role has verb → no personal override denies → record in scope → field visible. | Resolved |
| F-04 | High | **The example invoice number breaks its own rule.** UI spec says numbers must fit the portal's 16-character limit, then gives `INV/BLR/26-27/0001`, which is **18 characters**. | Examples now fit: `BLR/26-27/00001` (15). Document type is carried by the series, not the prefix. The format editor must count characters including separators. See [05-logic/18-business-logic.md](05-logic/18-business-logic.md). | Resolved |
| F-05 | High | **Offline numbering model differs.** Architecture: each terminal has its own series, number taken by atomic increment at post. UI spec: devices get blocks of numbers from one series. The first is server-assigned at post, the second is device-assigned before post. | Docs use **blocks from the branch series, issued by the server** (UI spec), because it keeps one series per branch and per year. Per-device series stays as the fallback if an auditor objects to out-of-order numbers. Legal basis recorded in 18-business-logic: GST Rule 46(b) allows multiple series and does not require date order. The gap-and-void process applies to both. | **Resolved** (D-005: blocks of 50; owner 2026-10-31) |
| F-06 | Med | **Payroll scope.** Role doc gives the HR officer "Start payroll run", payroll dashboard cards and Accountant payroll access. UI spec and the architecture phases defer payroll. | Payroll is out of release 1. Role defaults drop payroll rows and cards; the data model reserves space. | Resolved |
| F-07 | Med | **Rail lists differ.** Role doc: Manager has no HR/People; Accountant has no Stock or Settings; Billing officer has no Reports; Super admin has 5 icons. UI spec: Manager gets HR (team) and People (Activity); Accountant gets Stock and Settings; Billing officer gets Reports; Super admin has 6. | UI spec wins. Rails are generated from the module manifest, so the role doc tables must be regenerated, not hand-edited. | Resolved |
| F-08 | Med | **Stock employee entries.** Access doc: entry posts at once and is flagged for review, by company setting. UI spec: saved entries are locked and corrected by Request correction → manager approval → reversing entry. | Both hold: posting is immediate (setting), correction always goes through the reversing-entry path. | Resolved |
| F-09 | Med | **Posted-document wording.** Architecture strategy `PostedDocumentPolicy` has ReversalOnly / EditWithinWindow / CancelWithApproval. UI always says "Changes need a credit note". | Default is ReversalOnly. The message text follows the company's policy; the other two policies need their own copy. | Resolved |
| F-10 | Low | **Module naming.** "Inventory" (access doc, architecture) vs "Stock" (UI). | Module key `stock`; "inventory" is a search keyword and the backend package name. | Resolved |
| F-11 | Low | **Drafts.** UI: forms autosave drafts on the device and "closing the app keeps it". Security: drafts are encrypted per user and cleared on sign-out unless kept. Neither says whether drafts sync to the server. | Device-only in release 1. Consequence: a draft cannot be resumed on another device. | **Needs owner** (D-009) |
| F-12 | Low | **Missing collection.** `invitations` has a TTL index in the architecture but is not in the 65-collection budget. | Counted inside `users` (type field) in [04-data/14-data-model.md](04-data/14-data-model.md). | Resolved |
| F-13 | Low | **No verb for managing users.** Access doc lets staff with "manage users" invite users at or below their own role. | Modelled as `people.users.add` / `people.users.change` plus the no-grant-above-own guard. | Resolved |

## B. Gaps and risks

| ID | Sev | Finding | Action | Status |
| --- | --- | --- | --- | --- |
| F-14 | High | **Cell sizing ignores indexes.** Architecture divides 10,000 by collections per company (65 → ~150 companies). It also says each collection has about four indexes, and each index is its own file. That is roughly 65 + 260 ≈ 325 files per company, so the real limit may be several times lower. | Size cells by files (collections + indexes), then load-test. Re-open question 3 (pooled tenancy for small companies). The 10,000-collection figure itself was not re-confirmed by search. | **Needs owner** (D-003) |
| F-15 | High | **E-invoice 30-day reporting limit is not in the specs.** Since 1 April 2025, taxpayers with aggregate turnover of ₹10 crore or more cannot report an invoice to the portal more than 30 days after its date; the ₹5 crore mandate threshold is separate. Sources disagree on whether the 30-day rule starts at ₹5 or ₹10 crore. | Add a "Pending e-invoice ageing" alert and a company setting for the turnover band; tax advisor confirms. See [03-backend/27-third-party-integrations.md](03-backend/27-third-party-integrations.md). | **Verify** |
| F-16 | Med | **E-invoice cancellation.** Sources agree on a 24-hour window after IRN generation; one adds that cancellation is blocked if an e-way bill is linked. | Countdown in the UI is kept; add the e-way-bill condition to the rule. | **Verify** |
| F-17 | Med | **GSTR and portal rules change often.** Both docs already say the tax team must confirm thresholds. | Keep all thresholds and windows as dated settings, never constants. | Resolved |
| F-18 | Med | **Offline permission staleness** is left open in the access doc ("decide how long a device may run on an old list"). | Proposed default: 24 hours, then the Counter locks to the PIN screen and refuses new bills until it syncs. | **Needs owner** (D-008) |
| F-19 | Low | **Tech claims not confirmed by search.** (a) Node.js 26 is scheduled to enter Active LTS on 28 Oct 2026 (end of life Apr 2029). Confirm on nodejs.org that the promotion has happened. (b) NestJS 12.0.0 exists; the claim that the Node adapter is recommended from 12.3 and default in 13 was **not found**. (c) MongoDB 9.0 is out, but MongoDB's own page says Community and Enterprise availability is "coming soon", so self-managed deployments may be on 8.x. | Pin versions in [01-architecture/04-tech-stack.md](01-architecture/04-tech-stack.md) from what is installable on the chosen platform; use the stock Express or Fastify adapter until the Node adapter claim is verified. | **Verify** |
| F-20 | Low | **DPDP dates** match: rules notified 13 Nov 2025, operational duties (including breach intimation) from 13 May 2027, intimation without delay then a detailed report within 72 hours. Rule numbers differ between summaries. | Cite the gazette text, not summaries, in legal docs. | Resolved |
| F-21 | Low | **Prisma vs raw driver.** The owner's other product (AllMyNote) uses NestJS + Prisma + MongoDB; this product uses Express (D-021). This architecture requires the raw driver with a database per company. | Keep the raw driver; do not copy the Prisma setup. See ADR D-002. | Resolved |
| F-22 | Med | **Blocks of 50 can run out offline.** A device that uses all 50 numbers cannot issue another numbered invoice until it syncs. | **Resolved by the owner (2026-10-31):** 50 means 50 bills offline even with the server down, which is enough. Warning at 10 left and a stop at 0 stay as a safeguard; block size is a company setting. | Resolved |
| F-23 | Med | **Manager and People access.** The role document hides People from the Manager; the owner says a manager adds employee email accounts. | Manager gets `people.users.view/add/change` (Team scope, assignable roles only); rail shows People for the Manager. See D-022. | **Needs owner** (confirm the default list) |
| F-24 | Med | **"Super admin has entire access" vs privacy design.** The architecture keeps company records out of the super admin's reach except by support grant. | Kept as designed (D-023): full control of the service, data access by grant only. | **Needs owner** |

| F-25 | High | **Per-company credentials vs the company switcher.** The architecture stores users and passwords inside each company database, yet the UI spec gives multi-company people (accountants) a company picker. Same email would mean separate passwords and two-step per company. | One identity with memberships (D-025). | **Needs owner** |
| F-26 | High | **Offline Counter vs short-lived tokens.** Nothing said how a person signs in or is authorised offline after the access token expires. | Offline grant, 24 hours, conflict flagging (D-028). | **Needs owner** |
| F-27 | Med | **Staff without email.** The brief assumes an email for every account; many counter and stock staff have none. | Email or mobile login with temporary password; no SMS in release 1 (D-026). | **Needs owner** |
| F-28 | High | **No recovery for a locked-out or departed admin.** | Verified, two-person, waiting-period recovery (D-029). | **Needs owner** |
| F-29 | Med | **Seats and invitations.** A manager can invite people, but nothing stopped them exceeding the plan or inviting in parallel. | Seat reserved inside the invite transaction; invitation expires after 72 hours and releases it. | Resolved |
| F-30 | Med | **No permission catalog.** The 14 verbs were defined but not which apply to which screen. | access/03-permission-catalog.md. | Resolved |
| F-31 | Med | **No numeric role limits** (discount, refund, backdating, price override). | Limits per role in access/03 and 04 with placeholder values for the owner. | **Needs owner** (values) |
| F-32 | Med | **Counter registration missing.** Number blocks and sync need to know which device is asking. | Registered terminals (D-027). | **Needs owner** |
| F-33 | Low | **"Reveal permission" is not one of the 14 verbs.** | It is a per-field mode (`mask + reveal`) on the role. | Resolved |
| F-34 | Med | **Tick-level role seeds were missing** (only module-level). | access/04-role-seeds-and-limits.md. | Resolved |

| F-35 | High | **Only retail and wholesale were designed.** Farmers, traders and commission agents (weighment, lots, account sale, no-tax bills) were missing. | Business profiles and farm and trader flows (advanced/01, 06). | **Needs owner** (phasing, D-032) |
| F-36 | High | **Inventory logic was thin:** no stock states, units and conversions, reservations, FEFO, landed cost, repack, cycle counts, wastage, traceability. | Inventory engine (advanced/02). | Resolved in docs |
| F-37 | Med | **No reminder engine** beyond a list of alerts: no dedupe, snooze, escalation, digest, consent rules, or restock maths. | Restocking and reminder design (advanced/03). | Resolved in docs |
| F-38 | Med | **Returns were one paragraph:** no inspection, disposition, original-cost restock, supplier claims, fraud controls. | Returns and reverse flows (advanced/05). | Resolved in docs |
| F-39 | High | **Indian Acts were not mapped** and several numbers in the earlier docs had no legal source; the Income-tax Act 2025 is reported to renumber sections from 1 April 2026. | Compliance map with sources and a Verify column; rules stored by subject, not section number (advanced/07). | **Needs advisor** |
| F-40 | — | *Withdrawn by the owner on 2026-10-31 (farm offline and extra languages removed from scope).* | — | Closed |
| F-41 | Low | **TCS on sale of goods** is switched off from 1 April 2025 per sources, but the GST docs listed "TDS and TCS" without that note. | TCS on goods defaults off; purchase-side TDS stays (07). | Resolved in docs |

| F-42 | High | **A shared database removes the database wall between companies.** One data-access bug could expose many companies, and the earlier design's strongest isolation control is gone. | Defence in depth: mandatory TDAL, schema and index rules, per-company keys, scoped caches, queues and files, detection, kill switches, canaries, isolation tests as a release gate (tenancy/01, 05). | **Accepted by owner**; risk R-02 raised |
| F-43 | Med | **MongoDB has no native row-level security by company.** Views that filter on database roles exist (MongoDB 7.0+) but need one database role per company, which does not scale. | The TDAL is the boundary (tenancy/01). | Resolved in design |
| F-44 | Med | **Deleting one company is harder** in shared collections and backups hold all companies together. | Purge job with verification, key destruction, backup-expiry date in the certificate (tenancy/04). | Resolved in design |
| F-45 | Med | **One schema migration touches all companies.** | Batches, canary companies, dry runs, pause and resume (tenancy/04). | Resolved in design |
| F-46 | Low | **Restore of one company needs a new method.** | Temporary-cluster extract and per-company exports (D-046). | Resolved in design |
| F-14 | — | *Superseded:* cell sizing by file count no longer applies; the file limit no longer scales with companies. Sizing is now by data volume, throughput and the heaviest companies (54). | — | Closed by D-039 |

| F-47 | High | **The audit log said who changed what, but nothing let the admin undo it.** Only drafts had a 10-second undo; there were no revisions, no recycle bin, no restore points. | Change events plus revisions, revert verb, self-undo, recycle bin, restore points, restore as of a date (history/01 to 03). | **Needs owner** (D-048 to D-054) |
| F-48 | Med | **"Undo" for posted documents must not be an edit.** Tax and ledger rules forbid it. | Guided reversal flows only (history/02, section 3). | Resolved in docs |
| F-49 | Med | **Immutable history vs the right to erasure.** | Redaction events keep the hash chain valid; names become "Former user" (history/01 section 7). | **Verify with a lawyer** |
| F-50 | Med | **Backups were provider-side only;** the admin could not see or control them, and a company restore could silently remove issued tax documents or reuse invoice numbers. | Backups screen, restore guardrails, verification (history/03). | Resolved in docs |
| F-51 | Low | **The 14-verb vocabulary had no way to express "undo others' changes".** | Added `revert` as verb 15. | **Needs owner** |

### B2. External review of the frontend specs (2026-11-01)

Strengths the reviewer confirmed, to keep: the manifest that generates rail, access editor, dashboard cards and golden tests; "absent, never greyed"; no feature-to-feature dependency; the server still enforces; concrete, testable accessibility rules.

**Inconsistencies between docs (fix now)**

| ID | Sev | Finding | Proposed fix | Status |
| --- | --- | --- | --- | --- |
| F-52 | Med | **Stock add key.** The manifest example in 06 uses `stock.entry.add`, but there is no entry screen (items, counts, locations…). It breaks the strict `<module>.<screen>.<verb>` rule. `import` and `/me` also do not fit the pattern. | Use a real screen id (for example `stock.items.add` or a named `adjustments` screen) and say how `import` and `me` are modelled. | **Needs owner** |
| F-53 | Med | **Manifest verbs.** 07 says `module.dart` holds "icon, screens, verbs, cards"; the 06 manifest has only `view` and `add`. The matrix editor needs every verb (change, post, approve, export, reveal, …). | Give `ScreenDefinition` a `verbs` list (or take it from the permission catalog, access/03) and update the example. | Open |
| F-54 | Low | **Matrix editor location.** 06 and 07 put it in `ui_kit`; 36 puts `MatrixEditor` in `company_workspace`. | Pick one (proposed: `ui_kit` for the widget, `company_workspace` for `AccessEditor`) and fix 36. | Open |
| F-55 | Low | **People location.** 06 puts People inside `company_workspace`; 07 has a separate `features/people`. | Follow 07 (a feature package); fix 06. | Open |
| F-56 | Low | **Role rail table header** in 08 says "before folding", yet Admin already lists Money and Team. | Make the header and the rows agree. | Open |
| F-57 | Low | **Panel limit.** "At most five stacked things" is broken when both company and branch pickers show (six). 08 and 36 list them differently. | Count pickers as one slot, or say six; align 08 and 36. | Open |
| F-58 | Low | **Typography.** "Two font weights" (08) conflicts with "bold first line" in list rows. | Say "semibold" in the row spec, or allow three weights. | Open |
| F-59 | Low | **Pane widths.** The draggable, remembered divider (240–400 px) is not reflected in `PaneWidths.forWidth`, which is a pure function of width. | Add a `panelOverride` argument, still pure and clamped; test it. | Open |

**Design risks**

| ID | Sev | Finding | Proposed fix | Status |
| --- | --- | --- | --- | --- |
| F-60 | High | **Two sources of truth for permissions.** The Flutter manifest generates access-editor rows; the backend has its own seed table (access/03, 04). They will drift. | Generate the manifest from a server-side or shared schema, or add a CI check that every key matches both ways. | **Needs owner** |
| F-61 | High | **Offline pricing and tax logic.** Counter works offline, so GST and price calculation must run on the device and match the server exactly. No shared package is in the list (09). | A pure-Dart `pricing_tax` package with test vectors shared with the backend, run in both CI pipelines. | **Needs owner** |
| F-62 | Med | **Cross-feature lookups.** Feature packages cannot depend on each other, yet an invoice line needs an item picker (stock) and a customer picker. The rule will be broken in week three. | A small shared `lookups` (or domain) package with picker widgets and read-only models. | **Needs owner** |
| F-63 | Med | **Routes use document numbers as IDs** (`INV-1042`). Numbers can repeat across branches or fiscal years, and offline bills have no final number until synced. | Stable ID in the route; the number is only a label. Fix the 06 example and 35. | Open |
| F-64 | High | **No platform matrix.** Nothing says which app targets Android, iOS, Windows, macOS or web. On Flutter web, screen-reader support, encrypted local storage, right-click menus and shortcuts are weaker, and the WCAG 2.2 AA goal depends on it. | Add a table (app × platform × supported features) to 06 and 04; decide web for Console and Workspace explicitly. | **Needs owner** |
| F-65 | High | **Thermal printing in Tamil and Hindi.** Most ESC/POS printers cannot shape Devanagari or Tamil, so those receipts need bitmap rendering. "A bill can print in Tamil" is a stated feature. | Decide in the Counter spike (bitmap vs printer code pages), not later. Note that extra languages were pulled from scope on 2026-10-31 (F-40); confirm whether Tamil and Hindi receipts remain. | **Needs owner** |
| F-66 | Med | **Multi-script search.** Names will be typed in English, Hindi and Tamil. No transliteration-tolerant search or collation is described. | Add to search design (03-backend/42) and i18n (38); decide on a normalised search field. | **Needs owner** |

**Smaller notes**

| ID | Sev | Finding | Proposed fix | Status |
| --- | --- | --- | --- | --- |
| F-67 | Low | **Golden test volume.** 8 roles × 7 widths × 2 themes explodes, and goldens drift between macOS and Linux CI. | Pin the CI environment; sample widths; add a long-text pass (Tamil, 200% text scale) at a few widths, where overflow actually happens. | Open |
| F-68 | Low | **`freezed` and "sealed + Equatable" overlap.** | Pick one and record it in 09 or the decision log. | Open |
| F-69 | Low | **About 20 packages is heavy for a small team.** | Start with the six `console_*` packages merged, or generate packages with Mason templates. | **Needs owner** |
| F-70 | Low | **Plain-words vs technical-words** doubles the ARB strings per language. | Keep keys technical; vary only display text. Note in 38. | Open |
| F-71 | Med | **Local drafts on sign-out or access revocation** may hold HR or payroll data. F-11/D-009 covers sign-out only in passing. | State the rule: clear drafts (or lock them) on sign-out and when the relevant permission is revoked on next sync. | **Needs owner** |

## C. What was improved in the docs

1. One vocabulary (15 verbs after D-048, 4 scopes, 4 presets) used in every file.
2. One permission-key format and a manifest that generates rails, editor rows, cards and role tables.
3. Corrected invoice-number examples and a character-count rule.
4. Payroll removed from release 1 consistently.
5. A single decision log carrying every open question with a recommendation.
6. A single build order: UI-spec steps 1–8, mapped to backend modules in [08-process/28-roadmap.md](08-process/28-roadmap.md).

## D. What is still unknown

- The architecture doc's seven-phase roadmap and the UI spec's eight-step build order were drawn or written separately. The phase diagram in the architecture file was not readable as text, so the roadmap here follows the UI spec's order and lists backend work per step. Compare with the original diagram.
- No team size, budget or dates were given, so the roadmap has no dates.
