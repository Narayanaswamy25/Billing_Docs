# Business Logic and Function Specifications

> Core posting, numbering and tax rules. The deeper logic for inventory, restocking, pricing, returns, reminders, farm and trader flows, and Indian Acts is in [advanced/00-README.md](advanced/00-README.md); where they differ, the advanced folder is newer.

Pure rules live in the domain layer and are unit-tested without a database. This file specifies the key functions; per-function edge-case tables grow with each module.

## PostInvoice (the central use case)
**Purpose:** make a draft invoice final in one atomic step.
**Inputs:** invoice ID, request context (user, company, clock), idempotency key.
**Output:** the posted invoice (view model).
**Steps (one unit of work, snapshot read concern, majority write concern):**
1. Load the draft; resolve the company rules once.
2. `priceInvoice(draft, rules)` (pure): line prices → discounts → tax split (CGST+SGST or IGST by place of supply) → rounding as its own line.
3. Issue stock through the stock service using the negative-stock policy (Block: conditional decrement so two cashiers cannot sell the last unit).
4. Take the next number from the series (atomic increment; an aborted transaction rolls the counter back).
5. `posted = priced.post(number, now)` (pure); save document.
6. Post the journal entry (debit = credit enforced).
7. Write the `invoice.posted` outbox event; commit.
**Edge cases:** repeated idempotency key → first result returned; same key with a different body → refused; write conflict → retry whole transaction; unknown commit result → retry commit only; stock short and policy Block → `STOCK_INSUFFICIENT`; credit limit and policy Block → `CREDIT_LIMIT_BLOCKED`; locked period → `PERIOD_LOCKED`; approval rule fires → state "Waiting for approval", cannot post; no network call to another service inside the transaction.
**After commit (worker):** e-invoice generation, messages, loyalty (later), reporting summaries.

## Numbering
- One series per document type, per branch, per financial year. The number is given **at post**, never at draft save; drafts show "New invoice (draft)".
- **Maximum 16 characters** including separators; allowed characters letters, digits, `/`, `-`. Format is a Settings rule with an effective date; the editor counts the longest possible result.
- Valid examples (≤16): `BLR/26-27/00001` (15), `B2/26-27/000001` (15). The old example `INV/BLR/26-27/0001` is 18 characters and is **invalid**.
- Year change: new series on 1 April; Settings shows the next number and can schedule the reset.
- Failed post → the next attempt gets the same number. Cancelled documents keep their number and show Cancelled; never reused.
- **Offline (accepted D-005): blocks of 50.** The server gives a Counter device 50 consecutive numbers from its branch series (`POST /pos/number-blocks`). The device uses them strictly in order. When all 50 are used, the device must send those bills to the server and finish the sync; only then does the server issue the next 50. The server posts the bills in number order and records the block as Complete. Block size is a company setting, default 50.
- **Block size and exhaustion (owner decision, 2026-10-31):** 50 numbers lets a Counter issue 50 bills fully offline, even if the server is down; the owner judged this enough. Block size is a company setting, default 50. The Counter still warns at 10 left ("10 bills left. Connect to sync.") and at 0 shows "Sync to continue billing", because a valid tax invoice needs a unique serial number and there is no safe temporary one. Held bills cover the rare case.
- **Unused numbers:** a device that is retired or lost with numbers left gets them marked Void by an admin; the gap is logged with a reason and listed in the Gaps report.
- **Legal basis for blocks (checked 2026-10-31 against the CBIC invoice-rules page and practitioner commentary):** GST Rule 46(b) asks for a consecutive serial number, in **one or multiple series**, using letters, numerals, `-` and `/`, up to 16 characters, **unique for the financial year**. It does not say invoices must be issued in strict date order, and it allows several series. So issuing numbers from per-branch blocks is allowed. What must hold: every number in a series is used on an invoice, cancelled (kept and shown Cancelled), or Voided with a logged reason; the invoice date is the date the bill was made on the device, not the sync date; the Gaps report explains every gap. That gaps are tolerated if explained is practitioner guidance, not statute text. Fallback if an auditor objects to out-of-order numbers: a separate series per device (allowed by "multiple series"), e.g. `B2/26-27/000001`. The company accountant's review in the pre-launch checklist is the final confirmation; it does not block building.

## Posted documents and corrections
Policy `ReversalOnly` (default): a posted invoice shows "Posted. Changes need a credit note." with Create credit note (roles with `sales.returns.add`). Credit note starts from the invoice (lines and tax prefilled), needs a reason (return, price change, error), cannot exceed the invoice. Debit note: same shape on purchases and for extra sales charges. Return restores stock only if "Restock" is chosen; damaged goods go to a Damaged location. Stock corrections never edit history: Request correction → manager Approve → reversing entry + new entry, original stays visible with the link.

## Document flow
Sales: Quote → Order → Invoice → Return/Credit note. Purchases: PO → Goods received → Bill → Payment → Debit note. Each step is a Convert that copies lines and links records.
- Partial conversion: shows ordered, already converted, this time; source stays open until fully converted.
- Over-conversion: blocked, or needs Approve (company choice).
- Editing a source after part conversion: limited to unfinished lines.
- A document with linked children cannot be cancelled until the children are.

## Tax and GST
CGST/SGST vs IGST from place of supply; per-rate split on the summary; reverse-charge checkbox on purchase bills; optional TDS lines and TCS lines for categories that still use them (rate, section, amount); **TCS on sale of goods ended 1 April 2025 per sources and defaults off**, buyer-side TDS on purchases stays (advanced/07); exempt, nil-rated and composition flags change which lines are drawn; HSN summary; GSTR-1, 3B summary and 2B reconciliation (match, mismatch, missing). All rates and thresholds are dated settings. Filing to the portal is out of release 1 (data prepared and exported).

## Payments and credit
Modes: cash, UPI, card, bank transfer, cheque, wallet, store credit (each can map to its own account). Split payment: Pay is disabled until lines equal the total or the balance is marked Credit. Part payment chip "Part paid". Advance receipts sit on the customer and are offered next time. Credit limit: warn, block or Approve (rule). Ageing buckets 0–30, 31–60, 61–90, 90+. Cheque states Received → Deposited → Cleared | Bounced (bounce reopens the invoice). Refunds always link to a credit note or return, by original mode where possible. Round-off is its own line. UPI at the counter: QR, wait for confirmation; manual "Received" needs a reason and is flagged in the shift report.

## Stock
Negative stock: Block, Warn, Allow. Valuation chosen once (weighted average or FIFO; changing needs a support request). Tracking per item: none, batch+expiry, serial (each serial scanned). Reorder level badge on the Stock icon. Expired batches cannot be sold; near-expiry in alerts. Transfers: Sent → In transit → Received (out of sender at Sent, into receiver only at Received). Counts: expected vs counted; differences need Approve before changing stock. Adjustments carry a reason (damage, loss, found, expired, other).

## Approvals
Rule = trigger + threshold + approver role (e.g. discount > 10% → Branch manager). Triggers: discount, price below cost, purchase over amount, credit limit exceeded, count variance, backdated entry, large write-off. Requester can open but not post. Reject needs a reason and returns the record to draft. Delegation with start/end dates (decisions logged under both names). Escalation after a set time. Self-approval never allowed. Counter: manager approves with quick PIN, logged against the manager.

## Import and opening balances
Upload → map columns → check (bad GSTIN, unknown unit, duplicate barcode) → fix (error file with Reason column, or in grid) → run (valid rows import, rest wait; summary added/updated/skipped) → undo for 24 h if no record has been used in a document. Opening balances need debit = credit; after the go-live date earlier-dated documents need Approve.

## Pending e-invoice ageing (new, F-15)
Each posted invoice for a company in the e-invoice band tracks `einvoice.status` and age since invoice date. Alert before the reporting window closes (30 days for taxpayers with aggregate turnover of ₹10 crore or more, per current sources; the tax advisor confirms the band and window as a dated setting). Cancel countdown: 24 hours from IRN generation, and not after an e-way bill is linked (one source; **verify**).
- **Offline B2B bills and e-invoice:** an offline bill that needs an IRN gets it after sync; the reporting-window clock runs from the invoice date, not the sync date. Retail B2C counter bills are, per current guidance, not part of e-invoicing (verify with the tax advisor).
