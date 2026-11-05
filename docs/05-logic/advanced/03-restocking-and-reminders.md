# Restocking and Reminders

## Part A. Restocking

### 1. Inputs per item (and per location when needed)
| Input | Source | Notes |
| --- | --- | --- |
| Average daily demand (ADD) | Sales in the last N days (default 28), excluding days the item was out of stock, adding back nothing for returns (net of returns) | Rolling; recalculated nightly into `stock_balances.stats` |
| Demand variability (σd) | Standard deviation of daily demand over the same window | For safety stock |
| Seasonal index | Same period last year ÷ yearly average, only when 12+ months of history exist; otherwise 1.0 | Optional switch per item |
| Lead time (LT) | Per supplier-item; **learned** from actual receipts (moving average of order date → receipt date); owner can override | Falls back to a default |
| Review period (R) | Days between checks (default 1, because the scan runs daily) | |
| Service level | A items 95%, B 90%, C 85% (settings); z = 1.65, 1.28, 1.04 | |
| MOQ, pack size | Supplier-item | Orders rounded up to pack size, at least MOQ |
| Shelf life | Item | Caps order size for perishables |
| Max stock | Item or default (days of cover) | Caps overstock |

### 2. Formulas (kept simple on purpose)
- **Safety stock** = z × σd × √LT (or the manual value if set).
- **Reorder point (ROP)** = ADD × seasonal index × (LT + R) + safety stock.
- **Target stock** = ADD × seasonal index × (LT + R + coverage days) + safety stock. Coverage days default 7 (store) or 14 (wholesale).
- **Position** = Available + Incoming − Reserved-not-yet-counted (reservations are already subtracted from Available).
- **Suggested order** = target − position, rounded **up** to pack size, at least MOQ, never above max stock.
- **Perishable cap:** suggestion ≤ ADD × (shelf life ÷ 2), so stock can be sold within half its shelf life.
- **Days of cover** = Available ÷ (ADD × seasonal index); **Expected stock-out date** = today + days of cover.
- Every suggestion shows its working in plain words: "You sell about 12 a day (last 28 days). Supplier takes 5 days. Safety 18. Reorder at 78. You have 64 and 0 coming."

### 3. Item status (shown as a chip and used by reminders)
| Status | Rule |
| --- | --- |
| OK | Position above ROP |
| Low | Position ≤ ROP |
| Critical | Available ≤ safety stock, or days of cover < lead time |
| Out | Available ≤ 0 and the item sold in the last 30 days |
| Overstock | Position > max stock, or days of cover > 3× target cover |
| Slow-moving | No sale for N days (default 45) |
| Dead | No sale for 90+ days |
| Expiring | A batch inside its alert window (default the larger of 7 days or 20% of shelf life) |
| Expired | A batch past expiry still in stock |

### 4. The Restock list (a Stock screen, view `stock.reorder.view`)
Grouped by preferred supplier, newest risk first. Each row: item, status, available, incoming, days of cover, suggested quantity, last price and best price (visible only to roles with view cost), expected stock-out date, "why". Actions:
- **Create PO draft** for a supplier group (needs `purchases.orders.add`): lines prefilled, price from last or best price, expected date = today + lead time. The PO is **never sent automatically**; an optional nightly job drafts POs (company setting) and notifies the buyer.
- **Edit quantity**, **Snooze** (1, 3, 7 days or until stock arrives), **Ignore** with a reason (discontinued, seasonal, not needed), **Already ordered** (link to a PO).
- Items with no supplier show "Choose a supplier".
- Stock employees see the list **without prices**; they can flag "running low" from the shelf, which raises the item's status for the buyer to review.

### 5. Receiving against the suggestion
Goods receipt compares PO, delivery and bill: shortage, excess (over-receipt rule), damage (goes to Damaged or Supplier return). Receipt updates actual lead time, supplier delivered-in-full percentage and price history. A shortage re-opens the suggestion.

### 6. Overstock, slow and dead stock
Reports and reminders suggest: stop reordering, run a price offer (store), return to supplier before expiry or within the supplier's return window, transfer to a busier branch, or write off. Farm lots use ageing instead (see 06).

## Part B. The reminder engine

### 7. Concepts
| Concept | Meaning |
| --- | --- |
| **Reminder rule** | Trigger or schedule + condition + recipients + channel + frequency + auto-resolve condition |
| **Reminder instance** | One fired reminder for one thing (an item, an invoice, a licence) with a state |
| **State** | Open → Seen → Snoozed → Done (auto or by person) → Escalated or Expired |
| **Digest** | Many low-severity reminders grouped into one message at a set time (default 9:00 local) |
Rules live in `templates_rules` (per company, editable in Settings → Reminders, `settings.reminders`); instances live in `notifications`. Defaults come from the business profile; the admin can switch any rule off (except legal-deadline ones for the admin, per the notification rules).

### 8. Delivery controls (against alert fatigue)
- **Dedupe:** one open instance per thing; it updates instead of repeating.
- **Severity:** Info (digest), Warning (daily digest and badge), Critical (immediate in-app and push).
- **Quiet hours** per person; **daily cap** per person with a "more waiting" link.
- **Snooze** from the notification; **auto-resolve** when the cause is fixed (stock received, invoice paid, licence renewed).
- **Escalation:** unanswered Warning for 2 days goes to the manager; Critical after 4 hours.
- **Recipients by role and branch:** a branch manager sees their branch; stock employees see shelf tasks only; accountants see money and compliance.
- **Customer-facing reminders** (payment dues) go only through consented channels and approved templates (see 07, DPDP and messaging rules).

### 9. Reminder catalogue (defaults; every number is a setting)
**Stock and supply**
| Reminder | Trigger | Default recipients | Channel and timing |
| --- | --- | --- | --- |
| Low stock | Status Low | Buyer, manager | Daily digest 9:00 |
| Critical or out of stock | Status Critical or Out for an item sold in last 7 days | Buyer, manager, stock employee | Immediate |
| Restock list ready | Nightly scan found suggestions | Buyer | Digest |
| Expiring soon | Batch inside alert window | Stock employee, manager | Daily, 30/15/7 days by shelf life |
| Expired in stock | Past expiry | Manager | Daily until written off |
| Overstock, slow, dead stock | Status rules | Manager | Weekly |
| Count due | Cycle schedule | Stock employee | On the day |
| Transfer stuck | In transit longer than N days | Both branches | Daily |
| Open PO overdue | Expected date passed | Buyer | Daily |
| Return waiting for inspection | Quarantine over 2 days | Manager | Daily |
| Supplier credit note awaited | Purchase return or claim open over 15 days | Accountant | Weekly |
**Money**
| Reminder | Trigger | Recipients | Timing |
| --- | --- | --- | --- |
| Customer payment ladder | Due −3, due, +3, +7, +15, +30 days | Customer (consented), salesperson, accountant | Template message; the +30 step also tells the manager and can stop credit |
| Credit limit nearing | 90% of limit | Manager | Immediate |
| Supplier payment due | Due date by terms | Accountant | Due −3 and due day |
| **MSME supplier due** | Supplier flagged micro or small: due date = acceptance + agreed days, **max 45**, or 15 if no written agreement | Accountant | Due −10, −3, due day, then overdue interest estimate |
| **Input-credit 180-day** | Purchase bill with input credit still unpaid at day 150 and 170 | Accountant | Weekly until paid |
| Cheque to deposit or clearing | Cheque date or status | Accountant | Day before |
| **Cheque dishonoured** | Status Bounced | Accountant, admin | Timeline checklist from the date of information (verify with a lawyer; see 07) |
| Shift cash difference | Beyond tolerance at close | Manager | Immediate |
| Bank items to reconcile | Older than 7 days | Accountant | Weekly |
**Compliance**
| Reminder | Trigger | Recipients | Timing |
| --- | --- | --- | --- |
| GST return dates | Calendar settings (dated) | Accountant, admin | 5 days and 1 day before |
| Pending e-invoice ageing | Invoice date + window | Accountant | Before window closes |
| E-way bill expiring | Validity end | Dispatcher, manager | 4 hours before |
| TDS payment and return dates | Calendar | Accountant | Before due |
| Licence or registration expiry | FSSAI, drug, trader, commission, shop, weighing-scale verification | Admin | 60, 30, 7 days |
| Credit note last date | Section 34 date from invoice year (see 07) | Accountant | Before the date |
**Sales and customers (B2B)**
| Reminder | Trigger | Recipients |
| --- | --- | --- |
| Customer usually reorders | Customer's typical interval exceeded | Salesperson |
| Quote expiring, order pending dispatch | Dates | Salesperson, dispatcher |
**Farm and trader**
| Reminder | Trigger | Recipients |
| --- | --- | --- |
| Lot ageing | Stored longer than N days | Owner |
| Moisture or quality risk | Lot flagged | Owner |
| Buyer payment pending | Past agreed days | Owner |
| Advance recovery pending | Advance older than N days | Trader or agent |

### 10. Jobs
Scan jobs are listed in [39-background-jobs.md](../../03-backend/39-background-jobs.md): demand statistics (nightly), restock scan (hourly or nightly), expiry scan (daily), dues ladders (daily), compliance calendar (daily), licence scan (weekly), lead-time learning (on receipt). Each job writes instances idempotently (key = rule + thing + day).

### 11. Settings per company
Demand window, service levels per ABC class, coverage days, alert windows, digest time, quiet hours, escalation times, auto-draft PO on/off, which reminder rules are on, channels per rule, customer-message templates. Defaults by business profile.
