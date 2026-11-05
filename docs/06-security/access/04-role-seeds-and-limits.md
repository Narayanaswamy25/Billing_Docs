# Role Seeds and Limits

Starting permissions for the seven company roles. These are **templates**: when a company is created, each is copied into that company's `roles` collection; the admin edits the copy. Platform updates to templates never overwrite a company's edits (the role shows "Modified from template" and can Reset). A role only receives a permission if the module and feature are in the company's entitlements.

Verb codes as in [03-permission-catalog.md](03-permission-catalog.md). Scope in brackets: (All) (Branch) (Team) (Own). Everything not listed is **no access**. Preset names: Full control = all applicable verbs; Custom = anything else.

## Company admin: Full control (All) on everything the company has
Every screen in the catalog, every applicable verb, scope All, all field modes `show` except PAN/Aadhaar/bank (`mask + reveal`). Limits: unlimited. Cannot exceed entitlements; cannot remove the last admin; cannot change own role.

## Manager (Branch)
| Screens | Verbs |
| --- | --- |
| dashboard.cards | V |
| sales.quotes, sales.orders | V A C R Ap Ca Pr S E Cv |
| sales.invoices | V A C R P Ap Ca Pr Rp S E Cv Vc |
| sales.receipts | V A C P Ca Pr Rp S E |
| **sales.returns** | V A C P Ap Pr Rp S E (takes returns, restock or damaged) |
| sales.customers | V A C E |
| pos.bills, pos.held, pos.shifts | V (bills); V A R (held); V A C Ap E (shifts) |
| pos.terminals | V A C (admin may remove) |
| purchases.orders, purchases.bills | V A C R P Ap Ca Pr S E Cv Vc |
| purchases.receipts, purchases.suppliers | V A C P Pr E; V A C E |
| stock.items | V A C E Vc |
| **stock.adjustments, stock.corrections, stock.counts, stock.transfers** | V A P Ap E (updates inventory and approves corrections and counts) |
| stock.price_lists, stock.locations | V A C |
| accounts.ledger, accounts.tax | V (read only) |
| hr.attendance, hr.leave (Team) | V Ap E |
| **people.users** (Team) | V A C — only roles the admin lets managers create (default Billing officer, Salesperson, Stock employee) |
| people.activity (Team) | V |
| reports.sales, reports.purchases, reports.stock, reports.receivables | V E |
| Fields | Cost and margin `show`; PAN/bank `mask`; customer phone/email `show` |
| Limits | maxDiscountPercent 25 · maxRefundAmount ₹10,000 · priceOverride up to 10% · belowCostSale needs approval · backdateDays 2 · creditLimitOverride needs approval · cashDifferenceTolerance ₹100 · maxStockAdjustment ₹5,000 |
No access to settings, roles, billing plan, payroll, integrations.

## Accountant (All branches they are given)
| Screens | Verbs |
| --- | --- |
| dashboard.cards | V |
| sales.invoices, sales.returns, sales.receipts, sales.orders, sales.quotes | V Pr S E (read, print, export; no create) |
| sales.customers | V E (phone and email masked) |
| purchases.* | V Pr E Vc |
| stock.items | V E Vc |
| accounts.journal | V A C R P Ap Ca Pr E |
| accounts.ledger | V Pr E |
| accounts.bank | V A C P E I |
| accounts.tax | V P E |
| reports.* (all groups except hr) | V E |
| settings.tax, settings.numbering, settings.rules | V |
| Fields | Cost and margin `show`; customer phone `mask`; PAN/bank `mask` |
| Limits | backdateDays any date in an open period; others N/A |
Technical accounting words are kept for this role.

## Billing officer (Branch)
| Screens | Verbs |
| --- | --- |
| sales.quotes, sales.orders | V A C R Pr S Cv |
| sales.invoices | V A C R P Pr Rp S Cv (sending includes e-invoice) |
| sales.receipts | V A C P Pr S |
| sales.returns | V A (draft only; posting needs a manager) |
| sales.customers | V A C |
| stock.items | V (selling price and quantity; no cost) |
| reports.sales | V |
| Fields | Cost and margin `hide`; customer phone `show` |
| Limits | maxDiscountPercent 10 · maxRefundAmount ₹0 (returns drafted only) · priceOverride none · backdateDays 0 |

## HR officer (Branch)
| Screens | Verbs |
| --- | --- |
| dashboard.cards | V |
| hr.attendance, hr.leave | V A C Ap E (Ca for leave) |
| hr.staff | V A C R E I |
| reports.hr | V E |
| Fields | Salary and bank `show` once payroll exists; PAN/Aadhaar token `mask` |
| Limits | N/A |
Release 1: no payroll, no payslips.

## Salesperson (Own bills, Branch for items)
| Screens | Verbs |
| --- | --- |
| pos.bills (Own) | V A P Pr S (cancel and reprint need a manager) |
| pos.held (Own) | V A R |
| pos.shifts (Own) | V A C |
| sales.customers | V A (pick or quick-add a customer while billing) |
| **stock.items** | V — shows item, **quantity and selling price only**; no cost, no margin, no add or change |
| Not granted | sales.returns (admin can switch on; the Counter drawer then shows Returns), purchases, accounts, reports, people |
| Fields | Cost and margin `hide`; customer phone `show` |
| Limits | maxDiscountPercent 5 · maxRefundAmount ₹0 · priceOverride none · backdateDays 0 · cashDifferenceTolerance ₹50 |
Manager approval at the till: a manager enters their quick PIN; the approval is logged against the manager.

## Stock employee (Branch)
| Screens | Verbs |
| --- | --- |
| stock.items | V (quantity, unit, location; **no prices**) |
| stock.entries | V A |
| stock.corrections | V A (Request correction) |
| stock.counts | V A (enter counted quantities; no approval) |
| Not granted | Anything else, including Dashboard |
| Fields | All price and cost fields `hide` |
| Limits | maxStockAdjustment ₹0 (everything above goes to approval) |
Opens on the stock entry form, ready to scan.

## Who may create whom (default `assignableRoles`)
| Creator | Roles they may create |
| --- | --- |
| Company admin | All company roles |
| Manager | Billing officer, Salesperson, Stock employee |
| Others | None |
Stored on the creator's role so the admin edits it in the role editor.

## Notes on these defaults
- Limit values and the manager's list are placeholders for the owner (D-022, D-029); they are settings.
- Approve is never given by a preset except to Manager and Admin, and approvals only trigger when an admin sets a rule (D-018); until then, exceeding a limit is simply blocked.
- The role document's old "Edit/Full/Own" labels map as: See → V; Add → V A; Edit → V A C; Full → the verbs above for that role; Own → scope Own.

## Addendum: advanced logic permissions
| Role | Added permissions |
| --- | --- |
| Company admin | All new keys |
| Manager | `stock.reorder` V E; `stock.reservations` V A R Ap; `stock.conversions` V A P Ap; `stock.wastage` V A P Ap; `sales.challans` V A C P Pr S E; `purchases.returns` V A C P Ap Pr S; `purchases.claims` V A C; `settings.reminders` V |
| Accountant | `purchases.returns` V Pr E; `settings.compliance` V; `compliance.licences` V |
| Billing officer | `sales.challans` V A C P Pr S |
| HR officer | none |
| Salesperson | none |
| Stock employee | `stock.reorder` V (no prices); `stock.wastage` V A (draft only); `stock.reservations` V; `farm.weighment` V A P Pr and `farm.lots` V A for farm helpers |
| Farm owner | Company admin role in a single-person company |
Limit added: `maxWastageValue` (manager ₹2,000, stock employee ₹0 draft-only).

## Addendum: change history and revert
| Role | History and revert |
| --- | --- |
| Company admin | `revert` on every applicable screen; `settings.backups` V A C Ap E; all history; Activity for everyone |
| Manager | `revert` on `sales.quotes`, `sales.orders`, `sales.customers`, `stock.items`, `stock.price_lists`, `stock.locations`, `purchases.suppliers` (Branch scope, `maxRevertAgeDays` 30); history of their Team; `settings.backups` V |
| Accountant | History view for accounting records; no `revert` |
| HR officer | History view for HR records; no `revert` |
| Billing officer, Salesperson, Stock employee | Own changes in My activity and 5-minute self-undo; no `revert` |
Limits added: `maxRevertAgeDays` (manager 30, admin none) and `selfUndoMinutes` (everyone 5).
