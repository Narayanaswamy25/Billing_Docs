# Selling and Pricing

## 1. Document types (chosen by tax status, customer and goods)
| Document | When | Notes |
| --- | --- | --- |
| Tax invoice | Registered seller, taxable supply | Full Rule 46 content; 16-character number; HSN digits by turnover (4 up to ₹5 crore on B2B, 6 above; B2C may omit digit count up to ₹5 crore, per Notification 78/2020 summaries); e-invoice for B2B above the threshold |
| Bill of supply | Composition seller, exempt or nil-rated supply, unregistered farmer's exempt produce (wording by status) | No tax line; wording set by tax status (verify) |
| Invoice-cum-bill of supply | Mixed taxable and exempt lines | |
| Retail invoice or cash memo | B2C counter sale | Simplified layout; tax shown or included |
| Consolidated day-end invoice | Optional for very small B2C bills (the ₹200 rule, conditions to verify) | Individual receipts printed, one summary invoice posted |
| Delivery challan | Goods moved without sale (approval, job work, own transfer, consignment) or sale delivery before invoice | Links to the later invoice; time limit setting |
| Receipt voucher / refund voucher | Advance received / advance returned | Tax on advances depends on goods vs services (setting, advisor) |
| Payment voucher, procurement slip | Buying from an unregistered supplier, farmers | In place of a supplier invoice |
| Credit note, debit note | Adjust a posted invoice | See 05 |
| Quote, proforma, order | Before sale | Convert chain with reservations |
| Account sale | Commission agent to farmer | See 06 |

## 2. Price resolution (one order, always shown on the line)
1. **Manual override**, only within the role's `priceOverride` limit; reason required above a threshold.
2. **Promotion or scheme** (buy X get Y, bill-value offer, coupon), within validity dates.
3. **Customer contract price** (special price for a customer or group).
4. **Price list** for the customer group, by quantity slab, with validity dates.
5. **Item default price** (or **daily rate card** for commodities; see 06).
Then: apply discounts (line, then bill), compute tax, round off as its own line. The line's **price source** is stored so an auditor can see why a price was used.
- **Floor and ceiling:** never below cost unless approved; for **MRP-governed items never above printed MRP** (see 3).
- **Price mode:** tax-inclusive (B2C default) or tax-exclusive (B2B default), per customer type and overridable per document.
- **Trade discounts** (on the invoice) vs **post-sale discounts** (later credit notes) are different documents with different tax effects; the rules for the latter are dated settings (verify, since the law on post-sale discounts is reported as amended).

## 3. MRP guard
Items flagged MRP-governed (packaged goods with a printed MRP) cannot be sold above MRP. The Counter blocks and says "Price is above MRP." The admin can switch the flag off for loose goods, services and mixed items. Selling above printed MRP is treated as a violation under the packaged-commodity rules and has been penalised by consumer commissions (sources in 07). MRP change on a batch is recorded on the batch, because different batches can carry different printed MRPs.

## 4. B2C fast path (Counter)
Scan → price (price list, MRP check) → FEFO batch → pay (one tap per mode) → print → next. No customer needed; optional mobile with consent for the receipt by message.

## 5. B2B path (Workspace)
Quote → **Order** (credit check, reserve stock, promise date) → pick and pack → **Delivery challan** or dispatch → **Invoice** (e-invoice when in scope) → e-way bill when the consignment passes the value threshold (₹50,000 default, state and goods vary; verify) → payment terms.
- **Credit check:** available credit = limit − outstanding − open orders. Warn, block or Approve per rule. Overdue invoices beyond N days stop new orders ("credit hold") until paid or approved.
- **Partial delivery and invoicing** track ordered, delivered, invoiced; backorder rules in 02.
- **Customer-specific terms:** payment days, early-payment discount (for example 2% if paid in 10 days), interest on overdue (setting), price list, MSME flag, TDS awareness flag.
- **Delivery:** transporter, vehicle, route; proof of delivery optional; dispatch list per route.
- **Sale on approval or consignment:** goods go on a delivery challan to the **With customer** location; the customer approves (invoice) or returns within the time limit; the system reminds before the limit (invoice issuance time for such goods is limited by law; verify).

## 6. Collections (dunning)
Ageing buckets 0–30, 31–60, 61–90, 90+. Reminder ladder at due −3, due, +3, +7, +15, +30 days through consented channels; each step uses an editable template, logs who and when, and stops when paid. The +30 step notifies the manager and can place a credit hold. **Statements** monthly or on demand. Part payments auto-allocate oldest first (setting: choose invoices). Advances are held on the customer and offered on the next invoice. Cheques follow Received → Deposited → Cleared or Bounced; a bounce reopens the invoice, adds bank charges, and starts the dishonour timeline reminder (07).

## 7. Payments
| Control | Default | Law note |
| --- | --- | --- |
| UPI | QR on Pay screen, wait for confirmation; manual "Received" needs a reason and is flagged | Payment system rules apply to the provider, not us |
| Card | Token only | No card numbers stored |
| Round-off | Own line, company rule | |
| Refund mode | Original mode first | See 05 |

## 8. Weighed and measured sales
Scale input fills quantity; tare supported; price per kg or per unit; rounding rule; unit shown on the line. Weighing scales used for trade must be verified and stamped under the legal-metrology rules, so each scale has a "next verification due" date with a reminder (07). Packaged sales use net content from the item.

## 9. Price boards for commodities
Items marked **daily rate** use a rate card entered each morning (per commodity, grade and unit, with effective time). Documents capture the rate used. Edits during the day create a new rate with a time, never an overwrite.

## 10. Role limits applied here
`maxDiscountPercent`, `priceOverride`, `belowCostSale`, `backdateDays`, `creditLimitOverride`, `maxRefundAmount`; exceeding a limit blocks unless an approval rule routes it to a higher role (D-018).

## 11. Events
`order.confirmed` (reserve), `delivery.dispatched`, `invoice.posted`, `payment.received`, `credit.hold_set`, `reminder.dunning_sent`, `cheque.bounced`; consumers: stock, accounts, reminders, e-invoice, reports.
