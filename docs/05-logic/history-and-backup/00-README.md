# Change History, Undo and Backup: overview

**What the owner asked for:** if an employee changes anything, the system records **who changed what and when**, and the admin can **undo the change**. Backups must protect against mistakes and disasters. This folder is that design.

## The four promises
1. **Every change is recorded.** Who (name, role, device, branch), what (field by field, before and after), when, why (a reason where it matters), and from where (screen, Counter, import, job, support).
2. **Nothing is ever erased from the history.** Undoing a change adds a new entry that says "restored by Meena"; the original change stays visible.
3. **Mistakes can be undone at the right level.**
   - *A person's own slip:* "Undo" within 5 minutes, no approval.
   - *Someone else's change:* the admin (or a manager where allowed) restores the earlier version, whole or field by field.
   - *A batch of changes:* undo the whole batch (a bad import or bulk price change).
   - *A posted invoice or ledger entry:* these are never edited or rolled back; a guided **reversal** is used (cancel, credit note, reversing entry), because tax law and the books depend on it.
   - *Deleted items:* restore from the recycle bin.
   - *Many masters at once:* "restore as of a date" with a preview.
4. **Backups exist in layers**, from instant self-service undo to provider-assisted disaster recovery, and are tested every month.

## How it fits the existing design
| Existing piece | What this adds |
| --- | --- |
| `audit_log` (append-only, hash-chained) | Becomes the **change event** log with field-level before/after, reasons, change sets, revert links |
| Soft delete, `version` field, Undo bar (10 s) | Recycle bin, full revision history, 5-minute self-undo |
| Posted documents immutable | Guided reversal flows, never "revert" |
| Role editor "Reset to template" | Revert any past version of a role |
| Import undo (24 h) | One case of the general **change set** revert |
| Backups (Atlas PITR, per-company exports) | User-facing **Backups** screen, restore points before risky actions, restore-as-of, monthly restore tests |
| 14 verbs | A 15th verb, **`revert`** (D-048) |
| Tenancy (shared database) | All history and backups are company-stamped and isolated like any other data |

## Documents
| # | Document | Answers |
| --- | --- | --- |
| 01 | [Change tracking](01-change-tracking.md) | What is recorded, event model, sensitive-change alerts, who sees history, retention, privacy |
| 02 | [Revert and restore](02-revert-and-restore.md) | Rules by record class, flows, blockers, posted-document reversals, restore as of a date, UI |
| 03 | [Backups and restore points](03-backup-and-restore-points.md) | Backup layers, schedules, retention, the Backups screen, disaster recovery, guardrails, verification |
| 04 | [History API, data and tests](04-history-api-data-tests.md) | Endpoints, errors, collections, permission keys, jobs, test cases |

## Decisions
D-048 the `revert` verb · D-049 change events plus revisions · D-050 history never deleted, posted documents reversed not reverted · D-051 five-minute self-undo · D-052 backup layers and restore-as-of · D-053 staff are told changes are recorded · D-054 sensitive-change alerts. See [29-decision-log.md](../../01-architecture/29-decision-log.md).
