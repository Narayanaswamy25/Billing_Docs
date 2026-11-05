# Returns and Reverse Flows

## 1. Sale return (customer to us)
**State flow:** Requested → Inspected → Approved → Restocked / Scrapped / Sent back → Refunded or Credited → Closed.
Quick path at the Counter collapses these into one screen for small, resellable items.

| Step | Rules |
| --- | --- |
| Find the sale | Scan the receipt or search by invoice, customer or batch. The return references the original invoice line, batch or serial. Return without an invoice needs approval, is limited by `maxRefundAmount`, and is valued at the current average cost |
| Quantity | Cannot exceed sold − already returned. Price and tax are those on the original line |
| Reason | Wrong item, defective, damaged in transit, expired, customer changed mind, price correction, excess supply, quality complaint |
| Condition and **disposition** | Resellable → Sellable location. Damaged → Damaged location. Expired → write-off. Needs inspection → **Quarantine**. Return to supplier → Supplier return location. Scrap. Food and medicine default to Quarantine, since they are rarely resellable |
| Time window | Per item class (setting). Beyond the window needs a manager |
| Stock | Restock at the **original sale cost**; batch and expiry kept; serial items re-enter by serial |
| Money | Refund by original mode (cash up to limits, UPI or card reversal, bank transfer), or store credit, or reduce the customer's outstanding. Never more than paid. Refund approval by role limit |
| Tax | A **credit note** (B2B) or return note with tax reversal (B2C, as the company's tax status requires) linked to the invoice; it is reported in the return period of its date; a last date to issue applies (the 30 November rule; see 07). The buyer's input credit reverses on their side |
| Documents | Return slip, credit note, refund voucher; all numbered at post |
| Exchange | One screen: returned lines and new sale lines net to one payment or refund |

## 2. Purchase return and supplier claims (us to supplier)
- **At receipt:** shortage, excess, damaged, wrong item, near-expiry: each line is accepted, rejected to **Supplier return**, or accepted-with-claim (price or quantity difference).
- **Later:** defective, expired within the supplier's return terms, quality complaints. A **Purchase return** document moves stock out of the Supplier return location at original cost and posts a **debit note** (tax adjusted as the law requires).
- **Claim tracking:** each return or claim shows what the supplier agreed to (credit note, replacement, refund) and a reminder if the supplier credit note is not received within N days (default 15). Evidence (photos, delivery note) attaches to the record.
- **Supplier scorecard:** delivered-in-full, on-time, damage and return rates; feeds lead time and reorder choices.

## 3. Replacement and warranty
Replacement is a return line plus an issue line without payment; serial and batch kept; warranty period from the original sale date; repair is out of scope (services pack).

## 4. Customer account adjustments
Credit note not refunded stays as **customer credit** (usable on the next invoice, expires per setting). Write-off of small balances needs approval and a reason. Bad debts after the legal and policy steps are a journal with approval.

## 5. After a return, what moves
| Record | Change |
| --- | --- |
| Stock ledger | Sale return receipt into the chosen location, at original cost; or write-off or supplier-return movement |
| Receivable or cash | Credit note reduces receivable; refund reduces cash or bank |
| Reorder | Restocked quantity returns to Available; if the item went Out and returns, the restock suggestion shrinks automatically |
| Reports | Return rate by item, supplier, salesperson and day; reasons mix |
| Reminders | Return awaiting inspection, supplier credit awaited, credit note last date |

## 6. Fraud and error controls
- Return without receipt, returns near shift close, refunds to a mode different from the original, repeated returns by the same person or item, returns by the same salesperson who made the sale: each is a flag on the **Control report** and can require approval.
- Refund above `maxRefundAmount` needs a manager PIN at the Counter.
- Returned serials are checked against sold serials.
- Cash refunds count in the shift reconciliation.
- Audit entries record who, when, reason and approver.

## 7. State machine (added to 21)
`Requested → Inspected → Approved → (Restocked | Quarantined | Scrapped | Sent to supplier) → Refunded/Credited → Closed`; `Rejected` ends with a reason; a posted credit note is never edited.

## 8. Business-type differences
| Profile | Difference |
| --- | --- |
| Store | Fast path, small values, store credit common |
| Wholesale | Return request, inspection, credit note, often replacement, transporter pickup, supplier claims |
| Trader | Rejected lot at weighment or after sale: price adjustment or return; account sale adjusted |
| Farm | Buyer rejects quality: price reduction or lot return; no tax documents when exempt |
