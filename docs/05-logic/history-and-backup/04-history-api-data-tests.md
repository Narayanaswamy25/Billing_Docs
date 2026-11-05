# History API, Data and Tests

## 1. Permission additions
**New verb `revert` (D-048)** (code **Rv**): restore an earlier version of an editable record or undo a change set. Requires `view` and `change` on the same screen. It is **not** in the "Can work" preset; "Full control" includes it. It does not apply to posted documents or ledgers (use `cancel` and returns flows).
| Applicable to | Keys |
| --- | --- |
| Sales | `sales.quotes`, `sales.orders`, `sales.customers` |
| Purchases | `purchases.orders` (drafts), `purchases.suppliers` |
| Stock | `stock.items`, `stock.price_lists`, `stock.locations`, `stock.counts` (draft sheets) |
| Accounts | `accounts.journal` (drafts only) |
| HR | `hr.staff`, `hr.attendance`, `hr.leave` |
| People | `people.users`, `people.roles` |
| Settings | `settings.branches`, `settings.rules`, `settings.numbering`, `settings.tax`, `settings.print`, `settings.integrations` (metadata only), `settings.approvals`, `settings.reminders`, `settings.compliance` |
| Farm | `farm.lots`, `farm.weighment` (drafts) |
| New screens | `settings.backups` V A C Ap E (view backups, request restore, restore as of date, download with `export`, approve company restore requests is a provider action); `settings.recycle_bin` V (restore uses `revert` on the record's screen) |
**Defaults:** Admin all of the above. Manager: `revert` on `sales.quotes`, `sales.orders`, `sales.customers`, `stock.items`, `stock.price_lists`, `stock.locations`, `purchases.suppliers`, Branch scope, `maxRevertAgeDays` 30. Others none. Everyone has **self-undo** (`selfUndoMinutes` 5) for their own changes without the verb. Added limits: `maxRevertAgeDays`, `selfUndoMinutes`.

## 2. Endpoints (company realm, under `/api/v1`)
| Method and path | Purpose | Permission |
| --- | --- | --- |
| `GET /{resource}/{id}/history` | Revisions and events of a record (cursor paging) | `view` on the record; field rules applied |
| `GET /history/revisions/{revisionId}` | One revision (for Compare) | same |
| `GET /history/compare?a=&b=` | Field differences between two revisions | same |
| `POST /{resource}/{id}/revert` | `{ toRevision, fields?: [..], reason, resolveConflicts?: {field: "keep"\|"restore"} }` | `revert` (or self-undo window) ; step-up for sensitive classes |
| `POST /{resource}/{id}/undo-last` | Self-undo of the caller's own last change | signed-in author, within 5 minutes |
| `GET /changesets/{id}` | A change set with its records | `people.activity.view` or the module's `view` |
| `POST /changesets/{id}/revert` | Revert all or chosen rows | `revert` on each affected screen; step-up |
| `GET /changes` | The Activity feed with filters (actor, module, action, sensitivity, date, record, change set) | `people.activity.view` (admin all, manager Team) |
| `GET /me/activity` | The caller's own changes (90 days) | signed-in |
| `GET /recycle-bin` · `POST /recycle-bin/{id}/restore` | List deleted records; restore | `revert` on the record's screen |
| `POST /restore-as-of/preview` | `{ types, asOf, branches }` → counts and diffs, blocked list | `settings.backups.change` (admin), step-up |
| `POST /restore-as-of/apply` | `{ previewId, reason }` → creates a restore point and a change set | same |
| `GET /backups` · `GET /backups/{id}` | Backup list and status | `settings.backups.view` |
| `POST /backups/on-demand` | Create a company backup now | `settings.backups.add`, rate limited |
| `GET /backups/{id}/download` | Signed, expiring link to the encrypted file | `settings.backups.export`, step-up |
| `POST /backups/restore-requests` | Ask the provider to restore (scope, time, mode) | `settings.backups.change`, step-up |
| `GET /restore-points` · `POST /restore-points/{id}/restore` | Automatic restore points | `settings.backups.view` · `.change` |
Posted-document corrections use the existing endpoints (`…/cancel`, `/sales/returns`, reversing journal) and write events with `reversalOf`.

## 3. Error codes added
| Code | HTTP | Meaning |
| --- | --- | --- |
| `REVERT_BLOCKED` | 409 | A blocker applies; body lists reason codes (`PERIOD_LOCKED`, `POSTED_DOCUMENT`, `EINVOICE_GENERATED`, `DOWNSTREAM_USE`, `UNIQUE_CONFLICT`, `ENTITLEMENT_OFF`, `AGE_LIMIT`, `FIELD_NOT_VISIBLE`) |
| `VERSION_CONFLICT` | 409 | Record changed since the revision; body lists newer changes for field-level choice |
| `REVERT_NOT_ALLOWED` | 403 | Caller lacks `revert`, scope or field visibility |
| `RETENTION_EXPIRED` | 410 | Revision thinned or archived |
| `RESTORE_CONFLICT` | 409 | Restore-as-of or company restore conflicts with locked or posted data |
| `STEP_UP_REQUIRED` | 401 | Fresh password or code needed |

## 4. Data
| Item | Where | Notes |
| --- | --- | --- |
| Change events | `audit_log` (existing, now with `changes[]`, `reason`, `changeSetId`, `revertOf`, `revisionId`, `source`, `device`, `clientAt`) | Indexes (all start with `companyId`): `{at:-1}`, `{actorId, at:-1}`, `{target.recordId, at:-1}`, `{action, at:-1}`, `{changeSetId}` |
| Revisions | **`record_revisions` (new collection; the budget becomes 66)** | `{companyId, collection, recordId, rev, at, actorId, changeSetId, eventId, class, snapshot, snapshotHash, revertOf?}`; indexes `{recordId, rev:-1}`, `{changeSetId}`, `{at:-1}`; snapshots hold ciphertext for encrypted fields |
| Recycle bin | The record itself with `deletedAt`, `deletedBy` (soft delete) plus its last revision | Archived after 90 days |
| Restore points | `record_revisions` + a manifest in `workflow` (type `restore_point`) | No full copies |
| Backup catalogue | `files` (type `backup`) per company; `backup_runs` in the platform database | Manifest, checksums, kind, retention class |
| Restore requests | `workflow` (type `restore_request`) and `recovery_cases`-style approvals in the platform database | Two-person approval |
| Files | Object storage versioning; versions listed through the file's history | |
Revisions and events are written by the **data access layer** in the same transaction as the change.

## 5. Jobs
| Job | When |
| --- | --- |
| Revision compaction (thin to monthly after 400 days) | Nightly |
| Recycle-bin archive after 90 days | Daily |
| Company backup export and manifest check | Nightly (per setting) |
| Backup verification restore test | Monthly per cell |
| Sensitive-change digest to admins and managers | Daily (immediate for Critical) |
| File version expiry | Daily |
| Retention enforcement | Weekly |
| Redaction of an erased person's personal values | On request |

## 6. Events recorded for this feature
`change.recorded`, `change.reverted`, `changeset.created`, `changeset.reverted`, `restore_point.created`, `restore_as_of.previewed`, `restore_as_of.applied`, `recycle_bin.restored`, `backup.created`, `backup.verified`, `backup.failed`, `backup.downloaded`, `restore.requested`, `restore.approved`, `restore.completed`, `history.viewed` (sensitive records), `selfundo.used`.

## 7. Tests (join the access and isolation suites; release gate)
| ID | Case | Expected |
| --- | --- | --- |
| T-26 | Any change to a class A or B record | One event and one revision in the same transaction; failed change leaves neither |
| T-27 | Event contents | Actor name and role as at that time, device, branch, before/after, reason where required |
| T-28 | Audit chain | Tampering with an event or revision breaks the hash check |
| T-29 | Salesperson edits an item's price | Manager receives the sensitive-change alert; History shows who and when |
| T-30 | Admin restores a previous version | New revision `revertOf` the old; original still visible; value restored |
| T-31 | Revert with a conflicting newer change | `VERSION_CONFLICT` with per-field choice; chosen fields restored |
| T-32 | Non-admin without `revert` tries to revert someone else | 403 `REVERT_NOT_ALLOWED` |
| T-33 | Author self-undo at 4 minutes and at 6 minutes | Works at 4; refused at 6 |
| T-34 | Revert of a posted invoice | Blocked; the menu offers Cancel or Credit note only |
| T-35 | Cancel inside the window; outside it | First works; second only credit note |
| T-36 | Role revert | Preview shows affected people; permission version rises; step-up required |
| T-37 | Delete then restore from recycle bin | Returns with history; unique conflict shown if barcode reused |
| T-38 | Bulk edit then change-set revert | Unchanged rows restored; changed rows listed |
| T-39 | Restore as of a date | Preview matches the apply result; restore point exists; posted documents and stock untouched |
| T-40 | Reverter cannot see a field | `FIELD_NOT_VISIBLE`; hidden values never shown in History |
| T-41 | Cross-company | Another company's history, revision, change set or backup is invisible (404) |
| T-42 | Offline Counter change synced late | Appears in the right order with device time; conflict flagged |
| T-43 | Company backup | Exists nightly; manifest and checksum valid; download needs step-up and is logged |
| T-44 | Monthly restore test | Temporary cluster restore passes counts, hashes, decryption, isolation and reconciliation |
| T-45 | Company restore guardrails | Numbers never reused; posted tax documents not removed; maintenance mode on; second approval needed |
| T-46 | Erasure request | Names and personal values redacted, hash chain intact, redaction event logged |
| T-47 | Settings revert | Creates a new dated version effective now; past versions and documents unaffected |
| T-48 | Number series revert | Refused if it would reuse numbers |
