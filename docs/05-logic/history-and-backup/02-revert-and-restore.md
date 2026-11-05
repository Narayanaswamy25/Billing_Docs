# Revert and Restore

## 1. Rules that always apply
1. **History is never deleted or edited.** A revert creates a new revision and event with `revertOf` pointing to the original. Reverting a revert (redo) is allowed.
2. **Who may revert:** the author can **undo their own change within 5 minutes** (`selfUndoMinutes`, D-051) on class A, B, F records they could change; anything else needs the `revert` verb on that screen, with the record in scope and its fields visible to the reverter.
3. **Reason required** for every revert except self-undo. **Step-up** (fresh password or code) for roles and permissions, user status, bank details, settings and rules, bulk and change-set reverts, restore-as-of, and restoring deletions of masters that have history.
4. **Preview first.** The screen shows the field differences and the effects ("3 open drafts use this item", "affects 4 people", "re-enables a credit hold").
5. **Version safety.** The revert runs in one transaction with an idempotency key and a version check. If the record changed since the revision being restored, the screen lists the newer changes and lets the person pick **field by field**: keep current or restore old.
6. **Never breaks the law or the books.** Posted documents, ledgers, filed periods and e-invoiced documents are outside revert (section 3).
7. **Quotas for non-admins:** `maxRevertAgeDays` (manager default 30) limits how old a change a non-admin may revert; admin has no age limit inside retention.
8. **Optional approval:** an approval rule may require admin approval when a non-admin reverts a sensitive class.
9. Every revert notifies the original editor and the admin, and is a sensitive event itself.

## 2. Revert by record class
| Record | What "revert" does | Notes and blockers |
| --- | --- | --- |
| **Item** | Restores fields (name, price, tax rate, units, reorder settings, flags) from a chosen revision | Posted documents keep their own snapshots, so history is untouched; open drafts will use the restored values; blocked if a unit conversion change would contradict stock already moved in the new unit |
| **Customer or supplier** | Restores contact, GSTIN, credit limit, terms, flags | Duplicate GSTIN or phone conflicts are shown; a credit limit restore can end or start a credit hold |
| **Price list or scheme** | Restores prices and validity | Affects future sales only |
| **Location, warehouse, branch** | Restores name and settings | Branch removal restore re-links users only if the admin confirms |
| **Role** | Restores permissions, scopes, field modes, limits and assignable roles of an earlier version | Shows how many people change; permission version rises; people see the change at next refresh; step-up |
| **User** | Restores role, branches, reports-to, status (re-enable after a disable) | Sessions stay revoked; a restored user signs in again; seat check applies; step-up |
| **Personal change** | Removes or restores one added or removed permission | Step-up |
| **Settings and rules** | Restores an earlier setting as a **new dated version effective now**; past effective dates and the documents posted under them are untouched | Number series formats cannot be restored to a value that would reuse numbers; step-up |
| **Draft document** | Restores the draft content (lines, prices, notes) | Cannot revert a draft that has been posted |
| **HR staff, attendance, leave** | Restores the record | Approved leave reverts by a reverse approval, logged |
| **Farm lots and weighment drafts** | Restores until the lot is sold from | Sold lots use correction entries |
| **File or attachment** | Restores a prior object version or an undeleted file | Within file retention |
| **Deleted record** | Restores from the **recycle bin** with its last revision | Unique values (barcode, GSTIN) must still be free; otherwise the conflict screen asks which to change |
| **Bulk edit or import (change set)** | Reverts all records in the set that are still unchanged since it ran; others are listed as conflicts for field-level choice; created records are removed only if unused | Imports keep the 24-hour whole-set rule; later reverts are per record |
| **Auto-drafted purchase orders** | Delete drafts not yet sent | Sent ones use the purchase-return flow |

## 3. Things that are **reversed, not reverted** (posted and legal records)
The "Undo" menu on a posted record shows only the legal ways to correct it, with the reason field and the right verbs:
| Posted record | Available correction | Conditions |
| --- | --- | --- |
| Invoice | **Cancel** (needs `cancel`) | Inside the allowed window, no linked children; for an e-invoice, within 24 hours of IRN generation and no e-way bill linked (verify) |
| Invoice | **Credit note or return** (needs `sales.returns.add`) | Otherwise; credit note by its last allowed date (07-compliance) |
| Receipt or payment | Cancel or reverse with a refund | Bank and cheque states respected |
| Purchase bill | Cancel, or debit note | Same idea on the purchase side |
| Journal | **Reversing journal** (needs `accounts.journal.cancel`) | Period not locked |
| Stock document, transfer, count | **Reversing entry** or a correction approved by a manager | Original stays visible with its link |
| Shift close | **Reopen** by a manager the same day | Counter bills after the close need a new shift |
| Anything in a locked or filed period | Not allowed | `PERIOD_LOCKED`; needs the period reopened under approval |
These create new linked documents; the original is never changed.

## 4. Self-undo (5 minutes)
After a change the person sees "Saved. Undo" for 10 seconds (existing bar) and can still use **Undo my last change** in **Me → My activity** for 5 minutes. It restores the previous revision of their own change if nobody else has touched the record since. It never applies to posted records, security settings or deletions of records with history.

## 5. Restore as of a date (time travel for masters)
Admin only, step-up. Choose the data types (for example items and price lists), the date and time, and the branches. The system compares each record with its revision at that moment and shows: records that would change (with diffs), records created since (kept), records deleted since (restored), and **blocked** records (downstream use, locked periods). An automatic **restore point** is taken first; applying creates one change set that can itself be reverted. It never touches posted documents, ledgers or stock quantities.

## 6. Conflicts and blockers (reason codes)
`VERSION_CONFLICT` (record changed since; choose per field) · `PERIOD_LOCKED` · `POSTED_DOCUMENT` · `EINVOICE_GENERATED` · `DOWNSTREAM_USE` · `UNIQUE_CONFLICT` · `FIELD_NOT_VISIBLE` (reverter cannot see a field they would restore) · `ENTITLEMENT_OFF` (module switched off) · `RETENTION_EXPIRED` (revision thinned or archived) · `APPROVAL_REQUIRED` · `AGE_LIMIT` (older than the role's limit). Each has a plain-word message and, where possible, the way forward.

## 7. Screens and plain words
| Place | What it shows |
| --- | --- |
| Record **History** tab | Timeline: time, person (with role), action, one-line summary ("Selling price 120.00 → 135.00"), device and branch, reason; expand for fields; buttons **Compare**, **Restore this version**, **Undo this change** (shown only if permitted) |
| **Compare** | Two versions side by side with changed fields highlighted |
| **Restore dialog** | Preview of effects, per-field checkboxes, reason, step-up if needed, result with "Undo restore" for 10 seconds |
| **Activity** feed (People → user → Activity; admin: all) | Filters, sensitivity badges, **Revert** on each row where allowed, export |
| **Changes to review** (admin Dashboard card) | Counts of sensitive changes today; opens the feed |
| **Recycle bin** (Settings) | Deleted records with who and when; **Restore** |
| **Restore as of date** (Settings → Backups and restore) | The wizard in section 5 |
| **My activity** (Me) | The person's own changes and **Undo my last change** |
Plain words: "Undo this change", "Restore this version", "Restore deleted item". Accounting words appear for Accountants as elsewhere.

## 8. Worked examples
1. **A salesperson edits an item's price by mistake.** The manager's digest says "Price of Basmati 5 kg changed 120 → 135 by Ravi (Salesperson, Counter 2)". The manager opens the item, History, **Restore this version**, adds a reason, confirms. Posted invoices keep 135 where it was used; new sales use 120. The edit and the restore both stay in history.
2. **An employee deletes a customer.** The admin sees "Customer deleted" in Changes to review, opens the **Recycle bin**, restores it; the GSTIN is still free, so it comes back with its history.
3. **A wrong role change.** The admin opens the role History, restores yesterday's version after the preview says "affects 4 people"; step-up confirms.
4. **A bad bulk price update.** Open the change set, **Revert**; 118 rows restore, 2 changed since and are listed for a choice.
5. **A wrong invoice.** The menu offers Cancel (still inside the window) or Credit note; the invoice itself is untouched.
