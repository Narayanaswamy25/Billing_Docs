# Drawbacks Review

Reviewed 2026-10-31 against the full docs set (89 files, about 4,900 lines) after the shared-database, access, advanced-logic, change-history and backup updates. **This is a review of the design on paper.** Nothing is built, tested or validated with users, so every item is a risk or cost found by reading, not a measured defect.

Severity: **Critical** (could stop the project or the business), **High** (large cost or risk), **Medium**, **Low**. "Owner" means the decision or action sits with you.

## Where the project stands (the numbers behind the review)
- 89 documents, **no code, no prototype, no user test**.
- Decisions in the log: 27 proposed, 12 accepted, 2 open, 1 hypothesis. Findings marked "needs owner": 16.
- About 50 legal or tax statements marked **Verify**.
- Team size, budget, dates, prices and pilot customers are **unknown**.

## A. Scope and delivery (the biggest category)
| # | Drawback | Why it matters | Severity | What to do |
| --- | --- | --- | --- | --- |
| A1 | **Release 1 is very large.** It combines GST billing and e-invoice, offline Counter with numbering blocks, shared-database isolation, 15-verb access control, entitlements and plans, approvals, advanced inventory, restocking and reminders, returns, reports, change history and backups. | Larger than most products' first three releases; no dates exist; risk of never shipping or shipping late and thin | **Critical** | Cut release 1 to a thin slice (see section I), ship, then add layers |
| A2 | **Team size, skills and budget are unknown.** | The roadmap cannot be dated; the design assumes security, DevOps, Flutter, Node, MongoDB and tax expertise | **Critical** | Answer question 8 in the decision log; size the plan to the team |
| A3 | **Decision debt:** 27 proposals and 16 owner items are unsettled; many docs say "proposed". | Developers cannot start safely; docs may be reversed later | High | Decide in one sitting; mark Accepted or changed |
| A4 | **Nothing is validated with real shop staff.** Three-panel layout, icon rail, one-button forms, drafts and plain words are hypotheses (D-010). | The product may be slower or harder than a competitor's simple POS | High | Paper or clickable test with five shops before building more |
| A5 | **Over-design risk:** many documents describe features whose value is not yet proven (restock maths, reminder catalogue, farm flows, time-travel restore). | Effort goes to design, not to the first customer | Medium | Mark each feature M (must) or L (later); build M only |
| A6 | **Documentation drift:** four sections were written one after another and patched by script; wording can still disagree somewhere; some source documents were never seen (data dictionary, original role document, Flutter architecture document, the architecture phase diagram). | Hidden contradictions; the field-level data dictionary is **missing**, so tables and fields are not yet specified | High | Add a doc-lint pass, supply the missing sources, keep one change log |
| A7 | **Many numbers are placeholders** (discount caps, lockout times, retention, plan tiers, backup windows, SLAs). | Wrong defaults may annoy customers or create risk | Medium | Owner and first customers set real values |

## B. Architecture
| # | Drawback | Why it matters | Severity | What to do |
| --- | --- | --- | --- | --- |
| B1 | **Shared database makes one bug dangerous.** A mistake in the data access layer could expose many companies at once. MongoDB has no built-in per-company row security. | The strongest isolation (a separate database) was given up; the whole defence is software we write | **Critical** | Build the data access layer first, test it hardest, keep the dedicated-cell option for big or regulated customers, run canaries in production |
| B2 | **The data access layer is a bottleneck in design and in code.** Rewriting `$lookup`, banning operators and validating every result limit what queries and reports can do and cost CPU. | Slower development of reports and analytics; performance budget pressure | High | Allow-list builders for reports; precomputed summaries; measure early |
| B3 | **Write amplification.** Every change writes the document, an audit event, a revision snapshot, an outbox event and (for posting) ledger rows, all in one transaction. | The 800 ms p95 post target and storage growth are at risk; costs rise with usage | High | Prototype posting with all writes; compact revisions; measure |
| B4 | **Hot spots in shared collections:** number series counters, stock balances of fast items, the outbox, the audit log. | Write conflicts and slow posts under load; one busy company can slow neighbours | High | Per-device number blocks, retries, per-company quotas, zones or dedicated cells |
| B5 | **A shared migration touches every company at once.** | A bad migration is a multi-company incident | High | Batches, canary companies, dry runs, pause and resume |
| B6 | **Restoring or deleting one company is hard in shared storage.** Backups contain every company together; restoring one needs a temporary cluster and an extract tool; deletion needs a purge job. | Slow recovery, extra tooling, and a backup-expiry gap for erasure | High | Build and rehearse the tools; state the gap in contracts |
| B7 | **Express means we build the framework.** Module loader, route registry, dependency wiring and permission middleware are our code. | More custom code, more bugs, harder onboarding than a structured framework | Medium | Keep the structure small and heavily tested; document it |
| B8 | **Flutter for every client** has weak spots: dense keyboard-first data entry, Flutter web, desktop accessibility, and printer, scanner and cash-drawer plugins that vary by device. | Hardware and web quality are risks for a counter product | High | Prototype printing and scanning on the real devices first; consider a web client for the back office later |
| B9 | **Local database choice is unsettled.** Isar's long-term maintenance should be checked; Drift is an alternative. | A dead local database library stalls the offline Counter | Medium | Spike both; choose with the Counter prototype |
| B10 | **Offline Counter is a distributed system.** Numbering blocks, idempotent sync, conflicts, permission snapshots and clock differences all interact. | Subtle bugs produce duplicate, missing or out-of-order invoices | High | Design tests first (offline suite); prototype sync with two devices |
| B11 | **Atlas dependence and cost.** Point-in-time restore, sharding, auditing and customer-managed keys depend on the plan tier; India region and tier availability were not confirmed. | Costs and features may differ from the design; lock-in | High | Confirm tier and region; build a cost model |
| B12 | **Many moving parts:** API, workers, key service, Redis, object storage, KMS, Atlas, monitoring, GSP, messaging and payment gateway. | Operations burden and many failure points | High | Start with managed services; automate; reduce parts for release 1 |
| B13 | **The 15-verb permission system is heavy for small shops** even with presets. | Admin confusion and mis-set roles | Medium | Presets, preview-as-role and save review are designed; test with real admins |

## C. Security and privacy
| # | Drawback | Why it matters | Severity | What to do |
| --- | --- | --- | --- | --- |
| C1 | **Business fields are not field-encrypted** (only restricted and personal ones). Anyone with full database access could read invoices and stock. | Protection relies on nobody having access | High | Strict break-glass, audits; offer Queryable Encryption or a dedicated cell for sensitive customers |
| C2 | **Central credential store.** One identity across companies means a platform-database breach exposes hashes for all of them. | Larger blast radius for account takeover | High | Argon2id plus pepper in the vault, strict cell, alerts; consider hardware keys for admins |
| C3 | **Mobile-only staff logins** use temporary passwords and no SMS codes. | Weaker sign-in and recovery for counter staff | Medium | Quick PIN per device, admin reset, add SMS OTP later |
| C4 | **Two-step apps are hard for some admins.** | Support load and lockouts | Medium | Recovery codes; guided setup; consider email codes as a fallback |
| C5 | **Admin recovery is manual** (two checks, second approver, 24-hour wait). | Slow for a stuck shop; needs staffed support | Medium | Staff the process; document it |
| C6 | **Crypto-shredding is partial** and backups keep plain business data until they expire. | Erasure promises must be worded carefully | Medium | Say it in the contract; limit retention |
| C7 | **Immutable history vs erasure.** Redacting names and values in old events and snapshots while keeping the hash chain intact is delicate. | Legal and technical edge cases | Medium | Lawyer review; build and test the redaction event |
| C8 | **Staff monitoring concerns.** Detailed change tracking and alerts can feel like surveillance. | HR, legal and trust issues | Low | Notice, policy template, show people their own activity |
| C9 | **Support grants and break-glass add human steps** that can be abused or delayed. | Slow incident response vs control | Medium | Two-person rules, logs, time limits |
| C10 | **Security is a lot of custom code** (data access layer, key service, step-up, offline grants). | One flaw has outsized impact; needs external review | High | Pen test before launch, code ownership, fuzzing |
| C11 | **Flutter plugin and package supply chain** (printing, storage, scanning). | Compromised or abandoned packages | Medium | Pin, review, scan, minimise |

## D. Product, UX and domain
| # | Drawback | Why it matters | Severity | What to do |
| --- | --- | --- | --- | --- |
| D1 | **"For every business" is a stretch.** Store and wholesale are designed; trader, farm, pharmacy, restaurant and services are very different. | Jack-of-all-trades risk; thin quality everywhere | High | Win store and wholesale first; add packs after proof |
| D2 | **Advanced inventory may overwhelm small shops** (states, locations, reservations, landed cost, repack). | Violates the "first bill in 3 minutes" and "7 choices per screen" goals | High | Profile presets hide complexity; test with kirana owners |
| D3 | **Restock suggestions use simple averages.** Seasonal, promotional or new items may be mis-ordered; early data is thin. | Customers lose trust after bad suggestions | Medium | Show the reasoning, allow overrides, start with alerts before auto-draft |
| D4 | **Reminders can annoy** (alert fatigue) and **customer messages** need consent, templates and have cost. | Ignored alerts, legal risk, bills | Medium | Digest, caps, snooze; decide the messaging provider and costs |
| D5 | **Farm and trader flows are unvalidated**, vary by state, and rely on commodity and market-law rules that differ. | Rework or wrong documents | High | Interview farmers and mandi traders; build after the first release |
| D6 | **Only three languages** at launch (English, Hindi, Tamil). | Slows adoption in other states | Medium | Plan language packs; keep strings short |
| D7 | **No payroll in release 1**, thin HR. | Wholesale and larger customers expect it | Medium | Say so clearly; plan the release |
| D8 | **No GST return filing, no Tally, Vyapar or Zoho import**, no self-service signup, no customer portal. | Competitors have these; onboarding friction | High | Decide which are needed to win first customers |
| D9 | **Restaurant and services packs are not designed**, yet "all types of business" was a goal. | Positioning gap | Medium | Be honest in marketing; schedule |
| D10 | **Multi-currency, exports and inter-company transactions are out.** | Exporters and group companies cannot use it | Medium | State the limits |
| D11 | **Revert is easy to misunderstand.** People may expect "undo anything", including posted invoices. | Support load; frustration | Medium | Clear wording, guided reversal flows |
| D12 | **Time-travel restore cannot guarantee cross-record consistency** (items, prices, drafts, stock). | Surprising results | Medium | Preview, restore points, limit to masters |

## E. Legal and compliance
| # | Drawback | Why it matters | Severity | What to do |
| --- | --- | --- | --- | --- |
| E1 | **About 50 legal statements are unverified**, many from blogs and news, not statutes. | Wrong defaults could cause wrong invoices or penalties | **Critical** | Tax advisor and lawyer review every Verify row before launch |
| E2 | **Laws change** (e-invoice thresholds, return rules, Income-tax Act renumbering reported for 2026). | Constant maintenance and liability | High | Dated settings, a compliance owner, release notes for tax changes |
| E3 | **Product liability for tax correctness.** A calculation or document error is the customer's problem and ours. | Reputation and legal exposure | High | Accountant-checked test cases, clear terms, support process |
| E4 | **Role under data-protection law is unclear** (provider as processor, company as controller) and no legal text exists. | Contracts and notices missing | High | Lawyer drafts terms, privacy notice and processing agreement |
| E5 | **GSP vendor not chosen**; portal downtime and rate limits are external. | E-invoice failures hurt customers | High | Choose early; keep the adapter swappable; Pending status |
| E6 | **Out-of-order invoice numbers from blocks** rely on commentary, not a legal opinion. | Audit disagreement risk | Medium | Accountant review; per-device series as fallback |
| E7 | **State-specific rules** (market laws, shop laws) multiply. | Maintenance burden | Medium | Settings by state; start with a few states |

## F. Operations, cost and business model
| # | Drawback | Why it matters | Severity | What to do |
| --- | --- | --- | --- | --- |
| F1 | **No cost model.** Atlas tiers, backups, key management, storage for revisions and exports, GSP fees, messaging, monitoring and support are not priced. | The product may not be profitable at small-shop prices | High | Build a per-company cost model before pricing |
| F2 | **Security and backup layers raise cost per company**, while small Indian businesses are price-sensitive. | Margins | High | Tiered plans; cheap defaults |
| F3 | **24-hour service expectation.** A counter that cannot bill loses money; incidents have reporting clocks (6 hours for cyber incidents). | Needs on-call staff and runbooks | High | Plan on-call; offline Counter mitigates |
| F4 | **Nightly per-company exports and monthly restore tests scale with the number of companies.** | Compute and storage grow | Medium | Incremental exports, sampling, plan limits |
| F5 | **Testing burden is huge:** isolation suite, permission matrix, golden screenshots (seven widths, two themes, every role), device lab, compliance samples, load tests. | Slow CI, high cost | High | Automate, prune, run heavy suites nightly |
| F6 | **Platform billing is thin:** subscriptions, invoices to companies and payment collection are outlined, not designed. | Cannot charge customers properly | Medium | Design it before launch |
| F7 | **Performance targets are unproven** (100 ms add-item on low-end Android, 1 s list, 300 ms search, 800 ms post). | May fail on real devices | High | Early prototype benchmarks |
| F8 | **Syncing large catalogues to devices** (lakhs of items) strains low-end phones. | Slow first sync, storage | Medium | Partial sync by branch and active items |
| F9 | **Single-person risk.** Two-person approvals and security owners assume at least two trusted people. | Key person dependence | Medium | Name a second owner or adviser |

## G. Missing or thin areas
Tally and competitor import mappings · GST return filing · payroll and statutory filings · customer portal · restaurant and services packs · self-service signup and onboarding · report builder · fixed assets, loans and budgets (listed in the architecture, not detailed here) · detailed pricing and billing flows · marketing and support processes · a full data dictionary · drawn wireframes and design tokens · accessibility testing plan beyond the checklist · documented customer-facing security whitepaper.

## H. The ten risks to act on first
1. **Cut release 1** to a slice you can finish (A1, A2).
2. **Prototype the three hard pieces:** the data access layer plus isolation tests (B1), posting with events and revisions under load (B3, F7), and offline sync with number blocks on two devices (B10).
3. **Prototype Flutter on the real hardware:** printer, scanner, cash drawer, low-end phone (B8, B9).
4. **Validate the screens with five shops** before building them all (A4, D2).
5. **Verify the legal rows** with a tax advisor and lawyer (E1, E2).
6. **Choose the GSP and the cloud region**, and confirm Atlas tiers (E5, B11).
7. **Build a cost model** and set prices (F1, F2).
8. **Settle the proposed decisions** in one pass (A3).
9. **Name the team and on-call plan** (A2, F3, F9).
10. **Add the missing source documents** (data dictionary, role document) so tables and fields can be specified (A6).

## I. A realistic first slice (suggested)
Store and Wholesale on the shared engines: sign-in and roles with presets; masters and import; posting and numbering; the Counter with offline blocks; payments and dues; basic inventory with batches and expiry; GST invoice with e-invoice through one GSP; basic reports; change history, self-undo and revert for masters; nightly company backups and a Backups screen. Everything else (advanced restock maths, reminders catalogue beyond the essentials, farm and trader, time-travel restore, payroll, packs) after the first customers.

## J. Update: products and services
The outside products, vendors and cost drivers are now listed in [01-architecture/products-and-services-needed.md](01-architecture/products-and-services-needed.md). It addresses B9 (local database: Drift proposed), B11 and F1 (cloud, Atlas tiers, cost drivers), E5 (GSP route and shortlist process) and D4 (messaging and DLT lead time). Prices are still unknown until quotes arrive.
