# Entitlements and Plans

An **entitlement** is what a company may use. It is decided by the super admin from the plan the company pays for, plus add-ons, plus per-company switches, filtered by payment state. The company admin works only inside it.

## 1. Catalog (release 1)
Modules match the UI icons. A **feature** is a switch inside a module. *Core* items are always on.

| Module | Core? | Features (switchable) |
| --- | --- | --- |
| Dashboard | Core | cards per available module |
| People | Core | `users_roles`, `activity`, `audit_log` |
| Settings | Core | `business_rules`, `numbering`, `tax_setup`, `print_layouts`, `approvals`, `integrations` |
| Sales | No | `quotation`, `sales_order`, `invoice`, `credit_debit_note`, `receipts`, `customers`, `recurring_invoices` |
| Counter (POS) | No (needs Sales) | `pos`, `pos_offline`, `shifts_cash`, `held_bills`, `returns_at_counter` |
| Purchases | No | `purchase_order`, `goods_receipt`, `purchase_bill`, `suppliers`, `debit_note` |
| Stock | No | `items`, `batches_serials`, `transfers`, `counts`, `price_lists` |
| Accounts | No | `journal_ledger`, `bank_reconciliation`, `tax_registers` |
| GST and compliance | No (needs Sales, Accounts tax) | `gst`, `e_invoice`, `e_way_bill`, `tds_tcs`, `gst_returns_data` |
| HR | No | `hr_employees`, `attendance`, `leave` (`payroll` reserved, off in release 1) |
| Reports | No | one switch per report group: sales, purchases, stock, accounts, tax, receivables, control |
| Messaging | No | `whatsapp`, `sms`, `email_templates` |
| Packs | No | `retail`, `restaurant`, `services`, `wholesale` (later releases) |
Feature keys follow the architecture doc; add keys here when a module is built.

### Features added by the advanced logic design
| Module | New features |
| --- | --- |
| Stock | `uom_conversions`, `reservations`, `landed_cost`, `repack`, `wastage`, `reorder_planning`, `lots_expiry` |
| Sales | `price_lists_advanced`, `schemes`, `delivery_challan`, `sale_on_approval`, `dunning`, `sales_returns` |
| Purchases | `purchase_returns`, `supplier_claims`, `supplier_scorecard` |
| Farm and trade | `farm_lots`, `weighment`, `rate_board`, `account_sale`, `crop_costing` |
| Settings and compliance | `reminders`, `compliance_rules`, `licence_tracking`, `mrp_guard`, `msme_dues` |
| History and backup | `change_history` (core), `revert`, `restore_points`, `restore_as_of`, `recycle_bin`, `backups_self_service`, `company_restore` |
Business profiles (advanced/01) preselect these.

## 2. Plans, add-ons and examples
A **plan** is a named set of features and limits. An **add-on** is a bundle sold on top. Prices are not decided; the structure is.
| Part | Meaning |
| --- | --- |
| Plan | Features, limits (users, branches, invoices per month, storage, message credits), billing cycle |
| Add-on | Extra features or limit increases (for example HR, E-invoice, Accounts, Restaurant pack, extra users) |
| Agreement | The company's actual plan + add-ons + dates + overrides; the record the super admin edits |
| Business-type preset | Which plan and switches a new company starts with (retail, restaurant, services, wholesale) |
Examples from your role document, as presets only: *Retail shop, billing only* (Sales, Counter, Stock); *Restaurant* (Sales, Counter, Stock, Purchases, restaurant pack); *Wholesale with payroll* (all modules; payroll stays reserved until it is built).

## 3. Resolution (how the server decides, in order)
1. Start with the plan's features.
2. Add the add-ons' features.
3. Apply per-company **overrides** set by the super admin: `on` adds a feature (even if the plan lacks it, for example a trial), `off` removes one. Overrides can have an end date.
4. Apply **dependencies** (section 4): a feature needs its module and its required features; if a requirement is off, the feature is off too.
5. Apply the **payment-state mask** (section 6).
6. Add the core items.
7. Result = the company's entitlement set. Store it as a snapshot with an **entitlements version**; cache it per company; refresh on change by a Redis broadcast, with a 60-second backstop.
Everything downstream (rail, role editor rows, dashboard cards, search, reports, jobs, routes) is generated from this set. A route whose module is off returns `MODULE_OFF` and the app shows "This page is not available".

## 4. Dependencies
| Needs | Because |
| --- | --- |
| Counter needs Sales (`invoice`) and Stock `items` | Bills are invoices and move stock |
| `e_invoice` needs `gst` and `invoice` | IRN is attached to an invoice |
| `e_way_bill` needs `gst` and `invoice` or `sales_order` | Transport document |
| `goods_receipt` needs `purchase_order` | Document chain |
| `bank_reconciliation`, `tax_registers` need `journal_ledger` | They post to the books |
| `leave` needs `hr_employees` | Employee record |
| Any report group needs its module | Data source |
When the super admin turns a feature off, the screen shows what else turns off with it ("Turning off Sales also turns off Counter, E-invoice and Sales reports") and asks to confirm. Turning a feature on is refused until its requirements are on, or offers to switch them on together.

## 5. What each side sees
| Who | Sees |
| --- | --- |
| Super admin | Every module and feature, the resolved set, who overrode what and when, usage against limits, payment state |
| Company admin | Only the resolved set. No "locked" or "upgrade" teasers for items that are off, because the product never shows what the company does not have. The admin sees plan name, limits and usage in Settings, and an Ask button to contact the provider |
| Everyone else | What their role allows inside that set |

## 6. Payment states and what they allow
Lengths of the periods are for the owner to set.
| State | Entitlements | User experience |
| --- | --- | --- |
| Trial | Plan or trial overrides, with an end date | Banner with days left |
| Active | Full | Normal |
| Past due | Full | Banner to the admin |
| Grace | Full | Prominent banner for all admins and managers; days left shown |
| Read-only | View, print, export, send existing documents; no new or changed records; **Counter keeps issuing bills offline for a set time** so a shop is never stuck at the till | "Your plan is overdue. You can view and print. Contact us." |
| Blocked | No access except the sign-in and contact screens | "This page is not available. Contact us." |
| Retired | Offboarding only: data export, then deletion after notice | — |
Moving back to Active restores everything instantly; nothing was deleted at any step.

## 7. Limits and seats
| Limit | Enforced | Behaviour at 80% / 100% |
| --- | --- | --- |
| Users | When a user is **invited** (the seat is reserved; unaccepted invitations expire after 72 hours and release it) | Admin notice at 80%; at 100% the Add user button explains "Plan limit reached. Ask your admin." |
| Branches | When a branch is added | Same |
| Invoices per month | At draft creation online; **offline Counter bills are always accepted and counted**, never refused | Notice at 80%; at 100% new invoice drafts in the Workspace explain the limit; overage is billed (owner decides) |
| Storage, message credits | When used | Notice, then block of the specific action |
Disabled users do not use a seat (a setting). Limits count in a usage meter updated by workers, except user and branch counts which are checked inside the same transaction as the add.

## 8. Changes over time
- **Immediate:** the super admin flips a switch; takes effect at the next refresh.
- **Scheduled:** a change with an effective date (for example a plan change at renewal) is stored and applied by a daily job; the plan-change review lists what will switch off and what stays readable.
- **Switch off keeps data;** roles keep their dormant settings; switching on restores them with a "Review them" notice to the admin.
- **Every change** is logged in the platform log with who, which company, before and after, and reason.

## 9. Data (platform database)
`plans`, `plan_features` (inside `plans`), `modules` (catalog), `agreements` (plan, add-ons, overrides, dates), `company_entitlements` (resolved snapshot and version), `subscriptions`, `usage_meters`, `platform_audit`. See [14-data-model.md](../../04-data/14-data-model.md).
