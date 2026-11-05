# Backups and Restore Points

Goal: no mistake, outage or attack should cost a company its data, and the admin can see and control what protects them. Backups work in **layers**, each with a different speed and reach.

## 1. The layers
| # | Layer | Protects against | Run by | Frequency | Kept | Restore speed |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | **Change history, self-undo and revert** | A person's mistake in editable data | The admin or the person | Always on | Per retention in 01 | Seconds |
| 2 | **Recycle bin** | Deleted records | Admin, manager | Always on | 90 days | Seconds |
| 3 | **Restore points** (automatic) | A risky action going wrong | The system | Before imports, bulk edits, role changes, period close, year-end, restore-as-of, plan downgrades | 35 days | Minutes |
| 4 | **Restore as of a date** (masters) | Many wrong edits across masters | Admin | On demand | Within revision retention | Minutes |
| 5 | **File versions and undelete** | Deleted or replaced attachments | Admin | Always on | 30 days (plan) | Seconds |
| 6 | **Company backups** (per-company encrypted exports) | Data loss for one company, portability, offboarding | The system | Nightly (setting), plus on-demand | Daily 35 days, monthly 12 months, yearly to the statutory period (plan) | Hours (provider-assisted) or download |
| 7 | **Cluster backup** (Atlas continuous backup with point-in-time restore) | Cluster failure, corruption, ransomware | The platform | Continuous | Window per plan (for example 7 to 35 days) | Hours |
| 8 | **Cross-region copy** | Loss of a region | The platform | Daily | As layer 7 | Hours to a day |
| 9 | **Device safety** | Counter device loss | The app | On every sync | Server holds everything synced | Instant after sync |
| 10 | **Verification** | A backup that does not restore | The platform | Monthly per cell, on every layer 6 export (checksum) | Reports kept 1 year | — |
Layers 1 to 5 are self-service. Layers 6 to 8 are the provider's responsibility and are shown to the admin on the Backups screen.

## 2. The Backups screen (Settings → Backups and restore)
- **Status panel:** last backup time, size, result (OK, Warning, Failed), last **verified** date, next scheduled run.
- **Backup list:** each company backup with date, size, kind (nightly, monthly, yearly, on-demand, pre-change), and actions: **Download** (encrypted file; step-up; the key is delivered separately; every download is logged), **Request restore**.
- **Restore points:** the automatic ones with the reason ("Before importing items.xlsx"), who triggered it, and **Restore to this point** (for masters) or **View what changed since**.
- **Restore as of a date:** the wizard from 02.
- **Recycle bin** and **File undelete** links.
- **Settings:** nightly backup on or off (for plans that allow it), time window, notifications for failure, and the retention shown from the plan.
- **Alerts:** a failed or missing backup tells the admin and the platform on-call; "backup not verified in 45 days" warns.

## 3. Pre-change restore points (automatic)
Created inside the same action, cheap because they store change-set manifests and the revisions that already exist, not full copies:
| Trigger | What is protected |
| --- | --- |
| Import or bulk edit | The records the action will touch |
| Role, permission or user bulk change | The roles and users involved |
| Period close or year-end | Settings and masters in effect; the closing journals are reversible as their own documents |
| Restore as of a date | The records about to be overwritten |
| Plan downgrade that switches things off | The entitlement snapshot and dormant settings |
| Price-list overwrite, tax-rate change | The affected lists and items |

## 4. Company-wide restore (provider-assisted; rare)
Used for corruption or a large wrong operation. It cannot be self-service because it touches every module, the books and tax records.
1. **Request:** the company admin asks on the Backups screen (step-up), choosing the backup or time and the scope (**selected modules** by default).
2. **Second approval:** the provider's support lead and a second owner approve; the company's **accountant sign-off** is recorded when books or documents are in scope.
3. **Guardrails** (always on):
   - Posted tax documents already sent to customers or the e-invoice portal are **never removed**. If the restore point is earlier than them, they are kept and re-attached, or the restore is limited to masters and drafts.
   - **Invoice numbers issued after the restore point are never reused:** the series counter becomes the highest number ever issued; gaps are listed and explained in the Gaps report.
   - Ledgers are not rewound silently: a company-wide books restore is done by importing missing entries and by reversing wrong ones, or by an explicit full-books restore with accountant sign-off and a documented bridging journal.
   - The company is placed in **maintenance** (read-only) for the duration; devices keep billing offline and sync after, with conflicts flagged.
4. **Pre-restore backup** is taken (layer 6 on-demand).
5. **Method:** restore the cluster snapshot to a temporary cluster, extract the company's documents by `companyId` (tooling in tenancy/04), verify counts and checksums, show a **diff preview** (records to add, replace, keep), apply under a restore plan (modes: add missing, replace selected, replace all), reconcile (stock vs ledger, journals vs lines, number series), then destroy the temporary cluster.
6. **After:** a restore report to the admin; audit events for every changed record; reminder to check Counters' sync.
Targets: acknowledgement within 1 hour in business hours; one company restored within the hours stated in the plan (proposed 4 to 8 hours); whole cell within 1 hour (31-non-functional-requirements).

## 5. Cluster disaster recovery (platform-wide)
Handled by 52-backup-disaster-recovery: restore the cluster from continuous backup or the cross-region copy, verify isolation and reconciliation, rotate keys if compromise is suspected. Companies are told through the status page and email; Counters keep billing offline.

## 6. Verification and trust
- **Every company export** carries a manifest (counts per collection, checksums, schema version) checked after upload.
- **Monthly restore test per cell:** restore the latest cluster backup to a temporary cluster; run integrity checks (counts, hashes, sample decryption with the company keys, isolation canaries, reconciliation jobs); record the result; any failure is a High finding.
- **Quarterly single-company** and **yearly whole-cell** drills with timings.
- **Immutability and separation:** backups in a separate account and credentials, immutable lock for the retention period, deletion needs two people, backup access logged.
- **Encryption:** backups carry Atlas encryption; company exports are encrypted with the company's export key; restoring needs the key service.

## 7. What each plan includes (structure only; values are the owner's)
| Feature | Basic | Standard | Plus |
| --- | --- | --- | --- |
| Change history on every record | Yes | Yes | Yes |
| Self-undo and revert (`revert`) | Yes | Yes | Yes |
| Recycle bin | 30 days | 90 days | 90 days |
| Restore points | Yes | Yes | Yes |
| Restore as of a date | — | Yes | Yes |
| Company backups kept | 7 daily | 35 daily + 12 monthly | 35 daily + 12 monthly + yearly to statutory |
| Self-service download of backups | — | Yes | Yes |
| Provider-assisted company restore | By request, paid | Included (limited per year) | Included |
| Cross-region copy | Platform default | Yes | Yes |
| Verification report visible to the admin | — | Yes | Yes |
Entitlement features: `change_history` (core), `revert`, `restore_points`, `restore_as_of`, `backups_self_service`, `company_restore`.

## 8. Counter and device backups
Unsynced bills live only on the device. The Counter shows "N bills waiting" and last sync time; at 10 numbers left it asks to sync; the device keeps its local database encrypted; **Export unsynced bills** (encrypted file) is available to a manager as a last resort before a device is wiped. Once synced, the server (layers 1 to 8) protects them.

## 9. Promises we make, and do not make
We promise: daily company backups on paid plans, tested restores, history of changes, reversal flows, and clear recovery steps. We do not promise: a one-click rewind of the whole company that would remove tax documents already issued, or recovery of data older than the retention window or deleted after retention. This wording goes in the service terms with the lawyer.
