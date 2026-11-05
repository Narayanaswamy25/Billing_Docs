# State Machines

## Common document lifecycle
`draft → confirmed → posted → closed`, and `cancelled` from allowed states. Status chips (8 named states, 4 colours, each with icon or word): **done** (Posted, Paid), **waiting** (Draft, Pending, Part paid), **needs attention** (Overdue, Failed), **cancelled**.

```
draft ──(approval needed)──► waiting for approval ──approve──► draft (ready to post)
  │                                   └──reject (reason)──► draft (reason shown)
  └──post──► posted ──fully settled/converted──► closed
                │
                └──cancel (window + permission)──► cancelled   (number kept, never reused)
```
Corrections after posting: credit note / debit note / reversing entry; never edit.

## Entities
| Entity | States and transitions |
| --- | --- |
| Invoice | draft → [waiting for approval] → posted → part paid → paid → closed; posted → overdue (by due date, derived); posted → cancelled (window) |
| Quote / Order | draft → sent/confirmed → partly converted → fully converted (closed) \| cancelled |
| Purchase order | draft → sent → partly received → received → billed → closed |
| E-invoice | Not required · Pending → Generated · Failed (retry → Generated) · Generated → Cancelled (24 h window, no linked e-way bill) |
| E-way bill | generated → updated → expired \| cancelled; expiring ones appear in Dashboard alerts |
| Cheque | Received → Deposited → Cleared \| Bounced (bounce reopens invoice) |
| Stock transfer | Sent → In transit → Received |
| Stock count | Open → Counted → Waiting for approval → Applied \| Rejected |
| Approval request | Pending → Approved \| Rejected \| Escalated \| Expired |
| Company | Requested → Active → Suspended → Retired; billing states Trial, Active, Past due, Grace, Suspended; offboarding Started → Exported → Deleted |
| User | Invited → Active → Disabled |
| Import | Uploaded → Checked → Run → (Undone within 24 h) |
| Counter bill (offline) | Created on device → Pending sync → Synced \| Conflict (flagged for manager) |
| Counter shift | Open → Closed (cash difference may need Approve) |
| Support grant | Requested → Active (time-limited) → Expired \| Revoked |

## Rules
- Number is assigned on the transition to **posted**.
- A document with unfinished children cannot be cancelled.
- The requester of an approval cannot approve it.
- An invoice posted while the portal is down stays Pending; a background retry runs; Failed shows the portal's reason and a Retry button; posting is never lost.
- A conflict from offline sync (e.g. stock short) accepts the bill (the sale happened) and flags it.

## Added by the advanced logic design
| Entity | States and transitions |
| --- | --- |
| Sale return | Requested → Inspected → Approved → Restocked \| Quarantined \| Scrapped \| Sent to supplier → Refunded or Credited → Closed; Rejected ends with a reason |
| Purchase return and claim | Drafted → Sent to supplier → Credit awaited → Credited \| Replaced \| Rejected → Closed |
| Reservation | Held → Partly filled → Filled \| Released \| Expired |
| Backorder line | Open → Allocated → Delivered |
| Restock suggestion | New → Seen → PO drafted \| Snoozed \| Ignored (reason) \| Resolved (stock arrived) |
| Reminder instance | Open → Seen → Snoozed → Done \| Escalated \| Expired |
| Delivery challan | Issued → Delivered → Converted to invoice \| Returned \| Lapsed (time limit) |
| Conversion (repack) | Drafted → Posted |
| Wastage | Drafted → Approved → Posted |
| Farm lot | Received or harvested → Stored → Partly sold → Sold out \| Written off |
| Account sale | Drafted → Posted → Paid |
| Weighment slip | Drafted → Posted |
