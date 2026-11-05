# Product Requirements (PRD)

Priority: **M** must-have for release 1, **L** later release. IDs are stable; reference them in tasks and tests.

## Roles
Super admin, Company admin, Manager, Accountant, Billing officer, HR officer, Salesperson, Stock employee. Seeded as templates; the admin edits any of them inside the modules the super admin switched on. Platform staff roles: owner, support, finance, read-only.

## Requirements

| ID | Area | Requirement | Pri |
| --- | --- | --- | --- |
| P-01 | Platform | Super admin creates a company with business type preset, switches modules and single features, picks a plan, invites the admin. Every change is written to the platform log. | M |
| P-02 | Platform | Plans, subscriptions, usage meters, limits (users, branches, invoices per month, storage) with a notice at 80% and an explanation at 100%. | M |
| P-03 | Platform | Past-due → grace → read-only → blocked; Counter keeps billing offline for a set time. | M |
| P-04 | Platform | Support access only through a time-limited, logged grant from the company admin; banner "Viewing as support". | M |
| P-05 | Platform | Offboarding: export, notice period, key destruction, deletion log. | M |
| A-01 | Access | Admin edits roles per module and feature with presets, scope, hidden/masked fields, branches; sees a preview as the role and a save review before saving. | M |
| A-02 | Access | Personal changes per user with a badge and a list; Reset to template; platform template updates never overwrite company edits. | M |
| A-03 | Access | A switched-off module disappears everywhere (menu, editor, search, reports); switching back restores earlier settings. | M |
| A-04 | Access | No self-promotion; last admin cannot lose admin; admin cannot grant what they do not hold. | M |
| A-06 | Access | Managers add employee email accounts in roles the admin allows (default Billing officer, Salesperson, Stock employee), in their own branches; invitation by email; each person signs in with their own email. | M |
| A-07 | Access | Plan, add-ons and super admin switches define a company's entitlements; anything off is hidden everywhere; payment state moves a company Trial → Active → Past due → Grace → read-only → blocked. | M |
| A-05 | Access | Activity tab per user: admin sees all, manager sees own team, same field rules. | M |
| M-01 | Masters | Customers, suppliers, items, price lists with GSTIN check, duplicate detection, units and conversions, quick add inside an invoice, deactivate instead of delete. | M |
| S-01 | Sales | Quote → order → invoice → return/credit note via Convert; partial conversion; linked documents. | M |
| S-02 | Sales | Draft autosave; posting gives the number; posted documents locked; credit note needs a reason and cannot exceed the invoice. | M |
| S-03 | Sales | Payments: cash, UPI, card, bank transfer, cheque, wallet, store credit; split and part payment; advance; credit limit rule; ageing; reminders; refunds. | M |
| S-04 | Counter | Scan-first billing, held bills, returns, shift, one tap per payment method, offline with sync status, manager PIN approvals. | M |
| S-05 | Print | Thermal 58/80 mm, A4/A5, label sizes; preview; reprint marked Duplicate and logged; WhatsApp/email/SMS/PDF send; amount in words in English, Hindi, Tamil. | M |
| G-01 | GST | CGST/SGST/IGST by place of supply, e-invoice states (Not required, Pending, Generated, Failed, Cancelled) with retry, e-way bill, reverse charge, TDS/TCS, GSTR-1/3B/2B data. | M |
| G-02 | GST | Submit returns to the portal. | L |
| P-10 | Purchases | PO → goods received → bill → payment → debit note; reverse charge; over-receipt rule. | M |
| K-01 | Stock | Items, locations, batches/expiry, serials, reorder, transfers (Sent / In transit / Received), counts with approval, adjustments with reason, negative-stock setting, valuation (weighted average, FIFO). | M |
| K-02 | Stock | Request correction → manager approval → reversing entry for Add-only roles. | M |
| B-01 | Profiles | Business profiles (Store, Wholesale, Trader, Farm) set documents, features, reminders and compliance defaults at setup; changeable later inside entitlements. | M (Store, Wholesale); 1.5 (Trader, Farm) |
| K-03 | Stock | Units with conversions, decimals, weighed items; items with MRP, HSN, tax category. | M |
| K-04 | Stock | Stock states and location types, reservations, backorders, FEFO, batch and serial traceability, recall. | M |
| K-05 | Stock | Landed cost, repack and conversion, cycle counts, wastage and spoilage with approval. | M |
| RS-01 | Restock | Reorder planning from demand, lead time and safety stock with plain-word reasons; Restock list grouped by supplier; PO drafts; snooze and ignore with reasons. | M |
| RS-02 | Reminders | Reminder engine with rules, dedupe, digest, snooze, escalation, quiet hours; stock, money, compliance and sales reminders per the catalogue. | M |
| RT-01 | Returns | Sale returns with inspection, disposition, original-cost restock, refunds and credit notes; purchase returns, supplier claims and scorecard; fraud flags. | M |
| SP-01 | Pricing | Price resolution order, MRP guard, schemes and slabs, credit and dunning, delivery challan, sale on approval. | M |
| FM-01 | Farm | Weighment slips, harvest lots, daily rate board, bill of supply, input costs and crop profit; Simple mode. | 1.5 |
| FM-02 | Trader | Procurement slips, commission and market-fee settings, account sale to farmers, advances and farmer ledger. | 1.5 |
| C-01 | Compliance | Compliance rules as dated settings: MRP, MSME dues, input-credit 180-day watch, credit-note date, FSSAI number, licence and scale-verification reminders. | M |
| CH-01 | History | Every change recorded with who, role, device, branch, time, field-level before and after, reason and source; History tab on records; Activity feed with filters; "My activity"; sensitive-change alerts and an admin "Changes to review" card. | M |
| CH-02 | History | Undo and restore: 5-minute self-undo; `revert` for editable records, settings, roles and users; recycle bin; change-set revert; restore as of a date; guided reversal for posted documents; nothing erased from history. | M |
| BK-01 | Backup | Backups screen: restore points before risky actions, nightly company backups with download, provider-assisted company restore with guardrails, monthly restore tests, device safety. | M |
| AC-01 | Accounts | Journal, ledger, bank reconciliation, tax view, trial balance, P&L, balance sheet; debit = credit enforced. | M |
| H-01 | HR | Attendance, leave, staff records, self-service (Me). | M |
| H-02 | HR | Payroll, payslips, statutory deductions. | L |
| R-01 | Reports | The report list in the UI spec (sales, purchases, stock, accounts, tax, ageing, control) with scope-limited filters, background large exports, scheduled email, server-side field rules. | M |
| W-01 | Approvals | Rules by trigger, threshold, approver role; delegation, escalation, no self-approval; Counter PIN approval. | M |
| O-01 | Onboarding | Who works here? preset, Sales+Stock start, 3–5 step checklist, import with error file and 24-hour undo, opening balances with debit=credit check, go-live date. | M |
| N-01 | Notifications | Notification centre, per-person preferences, admin alerts cannot be disabled for admin. | M |
| X-01 | Platform | Companies with more than one login company get a company picker. | M |

## User stories (samples, one per role)
- *Stock employee:* I scan an item, type a quantity and tap Save and add next, so counting a shelf is one scan per item.
- *Salesperson:* I scan items, tap UPI, and the next bill is ready; if the internet drops I keep billing.
- *Manager:* I see approvals waiting first; I approve a discount in one tap.
- *Accountant:* I see items to post and bank lines to match; I keep technical words.
- *Admin:* I tick what a role may do, preview it, and see "4 changes affecting 4 people" before saving.

## Acceptance criteria (cross-cutting)
1. The permission matrix test passes for every role: icons, tabs, fields, buttons, badges, counts, search and quick-jump, with a second user in another branch proving nothing leaks.
2. First bill under 3 minutes in user testing with five real shop staff.
3. A repeat invoice or stock entry takes 5 taps or fewer.
4. No screen offers more than about 7 choices.
