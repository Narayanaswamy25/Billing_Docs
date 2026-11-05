# Backup and Disaster Recovery

## Targets (proposals; agree with first customers)
| Target | Value |
| --- | --- |
| Data loss after a failure (RPO) | Up to 5 minutes (point-in-time recovery) |
| Restore a failed cell (RTO) | Within 1 hour |
| Restore one company | Rehearsed quarterly |
| Full cell restore | Rehearsed yearly, timed against the targets |

## Backups
| What | How |
| --- | --- |
| Each cell | Atlas Cloud Backup with point-in-time restore (confirm the tier includes it) |
| Platform database | Own backup, more often than cells |
| Files and exports | Object storage with versioning, copy in another region |
| Key vault and KMS | Backed up on their own inside the KMS provider, never beside the data; encrypted backups are useless without keys |
| Where | Separate account, separate credentials, own encryption key, immutable lock for the retention period; deleting a backup needs two people |

## User-facing layers and guardrails
Self-service layers (history, recycle bin, restore points, restore as of a date), the Backups screen, plan tiers and the **company-restore guardrails** (never remove issued tax documents, never reuse invoice numbers, maintenance mode, second approval) are in [05-logic/history-and-backup/03-backup-and-restore-points.md](../05-logic/history-and-backup/03-backup-and-restore-points.md). Monthly automated restore tests run per cell.

## Restore procedures
- **One company:** restore the snapshot to a new temporary cluster, extract that company's documents by `companyId` with the verified tool, import under a restore plan, reconcile, destroy the temporary cluster (tenancy/04, D-046); or import the company's latest per-company encrypted export.
- **Whole cell:** restore into a clean network, bring up replicas, re-point the registry, verify with the isolation and reconciliation jobs, rotate keys if compromise is suspected.
- **Key vault:** drill restore; two named people required for recovery, logged.
- **Move a company to another cell:** quarantine writes, copy documents by `companyId`, verify, flip the `tenants.cell` mapping, keep the old copy read-only for a rollback window, then purge it.

## Ransomware playbook
Isolate → check backups are intact and unlocked-only-by-us → restore into a clean network → rotate every key and credential → follow the incident clock in [47-privacy-compliance.md](../06-security/47-privacy-compliance.md).

## Drills
Quarterly: single-company restore. Yearly: full cell. Each drill logs date, who, time taken, result; failed drills are tracked as vulnerabilities.

## Reconciliation after restore
Stock balances vs stock ledger and journal totals vs lines (nightly job) must agree before a restored company is re-opened; numbering gaps are listed in the Gaps report.
