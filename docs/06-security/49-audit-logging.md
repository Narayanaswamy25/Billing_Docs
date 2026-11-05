# Audit and Activity Logging

> Field-level change tracking, revisions, revert, recycle bin, restore points and backups are designed in [05-logic/history-and-backup/](../05-logic/history-and-backup/00-README.md). This page covers the log set and its integrity.

## Logs and what each holds
| Log | Where | Holds | Visible to |
| --- | --- | --- | --- |
| Company audit log | `audit_log` collection (stamped with `companyId`) | Who, what, before and after values, device, branch; sign-in results, role/override changes, exports, denied requests, support-mode actions, reveals of masked values | Company admin (exportable with Export verb; the export is logged) |
| Platform audit log | `platform_audit` | Every Console action: module switches, plan changes, support grants, staff actions, key events | Platform staff by role |
| Activity (per user) | Same audit data | Records the person added, changed, approved, posted, removed | Admin: everyone; manager: own team; same field rules (hidden values not shown in before/after); a link opens only if the viewer may see the record, otherwise "A record you cannot see" |
| Security and breach registers | Platform database | Findings, severity, due date; incidents | Security |

## Integrity
Append-only, never editable through the application. Each entry stores the SHA-256 of itself and of the previous entry; the last hash of each day is copied to a write-once store in another account. Logs ship off the server within seconds; nobody who runs servers can delete them; access to logs is logged. Posted documents carry a chained document hash.

## Must always be logged
Reprints (with user), voids and cancels, price overrides, number voids with reason, manual "UPI received" marks, approvals and delegations, manager PIN approvals, masked-value reveals, every access change (who changed which role, person or switch, before and after), exports, support sessions, denied-attempt counts per user.

## Alerts (and response)
| Event | Response |
| --- | --- |
| Repeated failed sign-ins for an account or address | Alert, temporary lock |
| Admin signs in from a new country or device | Alert admin and security |
| Role, override or security setting changes | Alert for admin-level changes |
| Large export or unusual read volume | Alert; fresh second step required |
| Many permission-denied replies | Alert: probing or a mis-set role |
| Any cross-company error | Page security at once; should never happen |
| Key-service spike or failed unwrap | Page |
| Support mode started | Notify company admin and security |
| New API key, webhook address or changed provider credentials | Notify company admin |
| Production role or schema change | Alert |
| Failed backup/restore or deleted backups | Page |
| Unusual outgoing destinations | Alert |

## Retention
Platform logs ≥180 days rolling (CERT-In); company audit log per company setting, at least one year for processing logs (DPDP). Closed-year ledgers and logs move to object storage after retention rather than new collections.
