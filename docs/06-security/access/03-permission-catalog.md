# Permission Catalog

The registry of every `<module>.<screen>.<verb>` key. A key exists only where the verb makes sense for the screen. The role editor, the Flutter manifest and the permission middleware are all generated from this catalog, so a key not listed here cannot be granted.

Verb codes: **V** view · **A** add · **C** change · **R** remove (drafts) · **P** post · **Ap** approve · **Ca** cancel · **Pr** print · **Rp** reprint · **S** send · **E** export · **Cv** convert · **Vc** view_cost · **I** import · **Rv** revert (D-048). Dependencies: every verb needs V; P needs A; Rv needs C.

## Workspace and Counter
| Key prefix | Applicable verbs | Notes |
| --- | --- | --- |
| `dashboard.cards` | V | One card per available module |
| `sales.quotes` | V A C R Pr S E Cv | Convert to order or invoice |
| `sales.orders` | V A C R Ap Ca Pr S E Cv | |
| `sales.invoices` | V A C R P Ap Ca Pr Rp S E Cv Vc | `Vc` shows margin; `S` includes sending to the e-invoice portal |
| `sales.receipts` | V A C R P Ca Pr Rp S E | |
| `sales.returns` | V A C R P Ap Ca Pr Rp S E | Return, credit note, restock |
| `sales.customers` | V A C R E I | `R` = deactivate if no history |
| `pos.bills` | V A P Ca Pr Rp S | The Counter's bill; scope Own for salespeople |
| `pos.held` | V A R | Held bills |
| `pos.shifts` | V A C Ap E | Open, close, cash difference approval |
| `pos.terminals` | V A C R | Register and revoke Counter devices |
| `purchases.orders` | V A C R P Ap Ca Pr S E Cv Vc | |
| `purchases.receipts` | V A C R P Ca Pr E | Goods received ("Items received") |
| `purchases.bills` | V A C R P Ap Ca Pr E Vc | |
| `purchases.suppliers` | V A C R E I | |
| `stock.items` | V A C R E I Vc | Quantity, selling price; cost needs `Vc` |
| `stock.entries` | V A | "Add stock entry" |
| `stock.corrections` | V A Ap | Request correction and its approval |
| `stock.counts` | V A C P Ap E | "Stock check" |
| `stock.transfers` | V A C P Ap Ca Pr | Sent, In transit, Received |
| `stock.adjustments` | V A P Ap E | Always with a reason |
| `stock.price_lists` | V A C R E I | |
| `stock.locations` | V A C R | |
| `accounts.journal` | V A C R P Ap Ca Pr E | "Money entries" in plain words |
| `accounts.ledger` | V Pr E | "Account history" |
| `accounts.bank` | V A C P E I | Reconciliation, cheques |
| `accounts.tax` | V P E | Registers, GSTR data; `P` locks a period |
| `hr.attendance` | V A C Ap E | |
| `hr.leave` | V A C Ap Ca E | |
| `hr.staff` | V A C R E I | Salary and bank fields follow field rules (payroll later) |
| `people.users` | V A C E | `C` = change branches, disable, issue credentials within limits |
| `people.roles` | V A C R E | Admin only by default |
| `people.activity` | V E | |
| `reports.<group>` | V E | Groups: sales, purchases, stock, accounts, tax, receivables, control, hr |
| `settings.branches` | V A C R | |
| `settings.rules` | V A C | Business rules with effective dates |
| `settings.numbering` | V A C | |
| `settings.tax` | V A C | |
| `settings.print` | V A C | |
| `settings.integrations` | V A C R | API keys need step-up |
| `settings.approvals` | V A C R | Approval rules |
### Added by the advanced logic design
| Key prefix | Applicable verbs | Notes |
| --- | --- | --- |
| `stock.reorder` | V E | Restock list; creating a PO uses `purchases.orders.add`; stock employees see it without prices |
| `stock.reservations` | V A R Ap | Reserve and release stock for orders |
| `stock.conversions` | V A P Ap | Repack and combine |
| `stock.wastage` | V A P Ap | Spoilage, breakage, shrinkage |
| `sales.challans` | V A C R P Pr S E Cv | Delivery challan |
| `purchases.returns` | V A C R P Ap Pr S E | Purchase returns and debit notes |
| `purchases.claims` | V A C | Supplier claims |
| `farm.lots` | V A C R E | Harvest and arrival lots |
| `farm.weighment` | V A P Pr | Weighment slips |
| `farm.account_sales` | V A C P Ap Pr S E | Commission agent account sale |
| `settings.reminders` | V A C R | Reminder rules |
| `settings.compliance` | V A C | Legal rule settings, licence numbers |
| `compliance.licences` | V A C R | Licence register and scale verification |

Not in this catalog: Console permissions (platform realm, see 06) and payroll (reserved).

## The revert verb
Applies only to editable records, drafts, settings, roles, users and HR or farm drafts; never to posted documents or ledgers. Keys and defaults: [05-logic/history-and-backup/04-history-api-data-tests.md](../../05-logic/history-and-backup/04-history-api-data-tests.md). Added screens: `settings.backups` (V A C Ap E), `settings.recycle_bin` (V).

## Scope
Set per role and module (or per row): **Own** (records the person created), **Team** (people who report to them, direct and indirect), **Branch** (their branches), **All**. The effective scope is the role's scope ∩ the person's branches. Scope applies to lists, search, counts, badges, reports and exports.

## Field rules (separate from verbs)
Each sensitive field has a mode **per role**: `show`, `mask`, `hide`, or `mask + reveal`. **Reveal is a field mode, not a verb** (the earlier text called it a permission; it is a per-field flag on the role). A reveal shows the full value for about 15 seconds, logs who revealed what, and asks for step-up for the most sensitive fields.
| Field | Default shown to | Default for others |
| --- | --- | --- |
| Cost price, supplier price | Roles with `Vc` | hide |
| Margin and profit | Roles with `Vc` on the module | hide |
| Customer phone and email | Admin, Manager, Billing officer, Salesperson | Accountant mask; others hide |
| PAN, Aadhaar token, bank account number | mask for all | admin may grant reveal |
| Salary and bank details (payroll later) | Admin, HR officer | hide |
| User list and role settings | Admin (Manager sees users in Team only) | hide |
A hidden field is not drawn, listed, searched, exported or sent in an API reply, and is never required for that role.

## Business limits (numbers that cap what a role can do alone)
Per role, edited by the admin; defaults in 04. If an action exceeds a limit it is **blocked**, unless an approval rule routes it to a higher role (approvals are off until the admin sets a rule, D-018).
| Limit | Meaning |
| --- | --- |
| `maxDiscountPercent` | Largest discount on a line or bill |
| `maxRefundAmount` | Largest return or credit note without approval |
| `priceOverride` | none, up to X%, any |
| `belowCostSale` | block, needs approval, allowed |
| `backdateDays` | How far back a document date may be |
| `creditLimitOverride` | no, needs approval, yes |
| `cashDifferenceTolerance` | Shift cash difference allowed before approval |
| `maxStockAdjustment` | Largest quantity or value adjustment without approval |

## Special actions mapped to verbs
| Action | Required |
| --- | --- |
| Reset someone's password link | `people.users.change`, target within the caller's Team and a lower level, step-up |
| Reset someone's two-step | Admin only, step-up |
| Unlock an account | `people.users.change` (Team) |
| Register a Counter | `pos.terminals.add` |
| Approve a stock correction | `stock.corrections.approve` |
| Retry e-invoice | `sales.invoices.send` |
| Open cash drawer | `pos.shifts.change` (manager) |
| Export a whole table | `.export` plus step-up |
