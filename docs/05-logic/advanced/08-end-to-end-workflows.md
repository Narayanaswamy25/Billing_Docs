# End-to-End Workflows by Business Type

Each flow lists steps, the documents made, the reminders that fire, and the permissions involved. Details of each engine are in 02 to 06.

## A. Store (B2C retail)
1. **Buy:** purchase order (or restock suggestion → PO draft) → goods received (check shortage, damage, expiry, MRP) → purchase bill (landed cost) → payment (supplier dues reminders; MSME rule if the supplier is micro or small).
2. **Shelve:** receipt puts stock in Sellable (damaged to Damaged). Batch and expiry captured for food.
3. **Sell:** scan → price (MRP guard) → FEFO batch → pay → print. Retail invoice; FSSAI number printed for food.
4. **Return:** receipt scan → condition and disposition → restock at original cost, or Quarantine → refund by original mode or store credit → return note.
5. **Replenish:** nightly demand statistics → status Low/Critical → **Restock list** → PO draft → loop to 1.
6. **Watch:** expiry reminders, dead-stock report, cycle counts, shift close with cash difference.
**Permissions:** salesperson `pos.bills`, `stock.items.view`; stock employee `stock.entries`, `stock.reorder.view`; manager returns, adjustments, approvals.

## B. Wholesale or distributor (B2B)
1. **Quote** (price list, scheme) → **Order** (credit check, reserve stock, promise date; backorder if short).
2. **Pick and pack** (FEFO by batch) → **Delivery challan** or dispatch (e-way bill above the value limit; route and vehicle).
3. **Invoice** (tax split by place of supply; e-invoice when in scope; e-way bill) → payment terms and early-payment discount.
4. **Collect:** ageing, dunning ladder, statement, cheque tracking, credit hold at +30.
5. **Return:** return request → inspection → credit note (before its last date) → restock or supplier claim.
6. **Buy:** reorder from supplier groups; GRN with three-way match; MSME dues; 180-day input-credit reminders; TDS awareness.
7. **Month end:** GST return data, TDS dates, stock valuation, ageing, MSME unpaid list.
**Permissions:** billing officer drafts and posts invoices; manager approves discounts and credit exceptions; accountant tax and bank; stock employee picks and receives.

## C. Trader or commission agent (mandi)
1. Farmer arrives → **weighment slip** (gross, tare, net, moisture, grade) → lot created in the farmer's name.
2. **Rate** from the rate board or negotiated → **auction or sale** to a buyer → buyer invoice or bill (tax per commodity) → buyer dues tracked.
3. **Account sale** to the farmer: gross value − commission − market fee − hamali, transport, weighing − advances = net payable.
4. **Pay the farmer** (cash, UPI, bank); advance recovery; farmer ledger.
5. **Reminders:** buyer payment ladder, farmer payable pending, licence renewal, scale verification.
6. **Controls:** price deviation approval, duplicate slip check, farmer-paid-before-buyer exposure limit.

## D. Farm or producer
1. **Plan and spend:** record inputs (seed, fertiliser, labour, fuel) against crop and plot.
2. **Harvest:** lot per crop, variety, grade, plot; weighment; storage location; shrinkage.
3. **Store:** lot ageing reminders; moisture flags.
4. **Sell:** to mandi or trader (weighment slip + bill of supply or sale receipt) or direct to consumers (cash memo); price from the rate board; payment by mode.
5. **Settle:** deductions if sold via an agent (account sale received and matched); advances recovered.
6. **Review:** crop-wise cost, revenue, profit per quintal and per acre; unsold stock; dues from buyers.
Runs in Simple mode.

## E. The restocking loop (all profiles with stock)
```
Sales ledger ──► nightly demand stats (ADD, σ, seasonal)
                      │
Stock position ◄── receipts, returns, adjustments, reservations
                      ▼
          status per item (OK / Low / Critical / Out / Overstock / Expiring)
                      ▼
 Reminder engine ──► digest or immediate notice ──► Restock list (grouped by supplier)
                      ▼
 Buyer reviews ──► PO draft ──► manager approval (if rule) ──► send to supplier
                      ▼
 Receipt (shortage? damage?) ──► updates lead time, supplier scorecard, stock ──► loop
```

## F. The return loop
```
Customer return ► find sale ► condition ► disposition ─┬─► Sellable (original cost)
                                                        ├─► Damaged ► claim supplier / write-off
                                                        ├─► Quarantine ► inspect ► (one of the above)
                                                        └─► Supplier return ► debit note ► credit awaited (reminder)
            ► refund / credit ► credit note (by its last date) ► reports and fraud flags
```

## G. Money loop (receivables and payables)
Invoice → due date by terms → ladder reminders (customer) → payment (allocate oldest first) → bank reconciliation. Bill → due date by terms (MSME cap) → reminders (accountant) → payment (cash warning) → 180-day watch.

## H. Daily and month-end checklists (reminders feed these)
**Daily:** open the shift; restock list; expiring batches; payments due; e-invoice failures; close the shift.
**Weekly:** slow and dead stock; supplier credit awaited; bank items.
**Month end:** GST data and dates, TDS, MSME dues, stock valuation, counts due, licence expiries.
