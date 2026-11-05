# Change Tracking

## 1. What is tracked (classes)
Each record belongs to a **class** that decides how it is stored and whether it can be reverted.
| Class | Examples | What is stored | Can be reverted? |
| --- | --- | --- | --- |
| **A. Editable master data and settings** | Items, customers, suppliers, price lists, locations, units, branches, business rules, numbering formats, tax setup, print layouts, approval and reminder rules, compliance rules, HR staff records, rate board, farm lots before sale | Change event **and** full revision snapshot per meaningful save | Yes (`revert`) |
| **B. Drafts and unposted documents** | Quotes, orders, draft invoices, draft POs, draft journals, draft stock documents | Change event; one revision per editing session (autosaves collapse) | Yes: author self-undo for 5 minutes, otherwise `revert` |
| **C. Posted documents** | Invoices, receipts, returns, bills, posted journals, stock documents, payments, shift closes | Events only (posted, printed, reprinted, sent, cancelled, e-invoice states); document body is immutable | **No.** Use a guided reversal (see 02) |
| **D. Ledgers** | Stock ledger, journal lines | Append-only; every row already carries who, when, source | **No.** Use reversing entries |
| **E. Deletions** | Soft delete of class A or B records | Event with the last snapshot | Yes: restore from the recycle bin |
| **F. Bulk operations** | Imports, bulk edit, bulk price update, auto-drafted POs, repack | One **change set** with a manifest and per-record revisions | Yes: revert the whole set or chosen rows |
| **G. Access and security** | Roles, personal changes, user status, branches, API keys, support grants, device approvals | Event and revision (secrets as ciphertext only) | Roles, users, personal changes: yes (admin, step-up). Keys and grants: revoke, not revert |
| **H. Files** | Attachments, logos, ID proofs | Event; object versions kept | Yes: restore a previous version or an undeleted file within the retention period |
| **I. Audit and platform logs** | Audit events, platform log | Append-only, hash-chained | Never |

## 2. The change event (one per action)
| Field | Meaning |
| --- | --- |
| `eventId`, `companyId` | Identity and company stamp (tenancy rules apply) |
| `at` / `clientAt` | Server time (trusted) and device time (for offline Counter work) |
| `actor` | User ID, **name and role as they were at that time**, level; or `system` (job) or `support` (grant ID) |
| `onBehalfOf` | Approver, manager PIN holder, or delegate when an approval or PIN was used |
| `action` | created, changed, removed, restored, reverted, posted, cancelled, printed, reprinted, sent, exported, revealed, approved, rejected, imported, bulk_changed, role_changed, signed_in, … |
| `target` | Collection, record ID, human label ("Item: Basmati 5 kg"), branch |
| `changes[]` | Per field: label, before, after, sensitivity, shown or masked (ciphertext kept for encrypted fields) |
| `reason` | Required for sensitive actions and for every revert |
| `source` | Workspace, Counter, API key, import, job, support session |
| `device`, `session`, `network` | Device ID and name, session ID, network address (kept for security review; visible to admins and security only) |
| `changeSetId` | Groups events from one action (an import, a bulk edit, a revert of many) |
| `revertOf`, `revisionId` | Link to the event being undone; link to the snapshot |
| `hash`, `prevHash` | Tamper-evident chain (daily anchor in a write-once store) |
Events are written **in the same transaction as the change** by the data layer, so there is never a change without its event, and a failed change leaves no event.

## 3. Revisions (the snapshots that make undo possible)
- A revision is the **whole record** after a change (class A, B, G, H). Encrypted fields stay as ciphertext in the snapshot, so history never exposes restricted data.
- **Autosaves collapse:** while someone is editing a draft the system keeps one revision per editing session, not one per keystroke.
- **Compaction:** every revision is kept at full detail for 400 days; after that one per month is kept until the retention end. Change events (the field-by-field diffs) are never thinned.
- **Size control:** very large documents (many lines) store a line diff plus a full snapshot every N revisions.
- Revisions are numbered per record (`rev` 1, 2, 3…) and verified with a hash.

## 4. Who sees history
| Viewer | Sees |
| --- | --- |
| Any person | Their own changes in **Me → My activity** (last 90 days) |
| Anyone with `view` on a record | That record's **History tab**, limited to what the field rules let them see (a value hidden from you is never shown in before/after; masked stays masked) |
| Manager | Their Team's changes across the modules they can view |
| Admin | Everyone's changes; the full **Activity** feed with filters (person, module, date, action, record, change set, sensitivity) and export (Export verb, logged, step-up) |
| Accountant, HR officer | History of records in their modules |
| Platform staff | None, except inside a company-approved support grant |
Opening the history of sensitive records (roles, bank details, salary data) is itself logged.

## 5. Sensitive changes: highlighted and alerted (D-054)
Defaults (each is a setting; thresholds are the company's):
| Sensitive change | Default alert | To |
| --- | --- | --- |
| Selling price changed by more than 10%, or cost price changed | Warning, digest | Admin, manager |
| Item sold below cost, or discount above the person's limit | Immediate | Manager |
| Stock adjustment or wastage above the role limit | Warning | Manager |
| Record or master **deleted** | Warning, digest | Admin |
| Bank details, GSTIN, credit limit, payment terms changed | Warning | Admin, accountant |
| Role, permission, branch, user status changed | Immediate | Admin |
| Settings and rules (tax setup, numbering, approvals, compliance) changed | Immediate | Admin |
| Backdated entry, reprint, manual "UPI received", shift cash difference | Warning | Manager |
| Bulk edit, import, price-list overwrite | Immediate | Admin |
| Reveal of masked values, large export | Immediate | Admin |
| Any **revert** | Warning | Original editor and admin |
They flow through the reminder engine (dedupe, digest, snooze, escalation). The admin Dashboard shows a **Changes to review** card (counts by type for the day) that opens the filtered Activity feed.

## 6. Offline Counter and late changes
Counter work creates events on the device with `clientAt` and the device ID. At sync the server stamps `at`. The feed sorts by server time and shows device time beside it. If a change from the device conflicts with a newer server change, the record keeps the server version, the device change is saved as an event marked **Conflict**, and the manager sees it (same rule as other offline conflicts).

## 7. Privacy and staff notice (D-053)
- Staff are told that their changes are recorded: a line in the first-sign-in tips and on **My access** ("Changes you make are recorded with your name"). The company's employee notice and policy come from the company (template provided; the lawyer reviews).
- Records of who did what are processed for security, accuracy and legal reasons by the company as the data owner.
- **Erasure:** when a person's data must be erased, their name in events becomes "Former user" with the ID kept, personal values in old snapshots are replaced by tombstones, and a **redaction event** is added to the hash chain, so the chain stays verifiable. What may be kept follows statutory rules (47).
- Event data is company-stamped and isolated like every other collection (tenancy rules).

## 8. Retention
| Data | Default retention |
| --- | --- |
| Change events | At least 1 year; default 8 years (setting; the longest statutory period) |
| Revisions | Full detail 400 days; one per month afterward until the retention end |
| Recycle bin | 90 days, then the record is archived (kept in history, no longer restorable with one click) |
| File versions | 30 days for deleted or replaced files; longer by plan |
| Security events | 1 year minimum; platform log 180 days minimum rolling |
