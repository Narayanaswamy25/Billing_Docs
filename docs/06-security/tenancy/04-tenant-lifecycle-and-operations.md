# Company Lifecycle and Operations in a Shared Database

"Tenant" and "company" mean the same here.

## 1. Provisioning (seconds, not minutes)
Idempotent by company ID:
1. Create the registry row in `tenants` with status `Provisioning` and the cell (default cell, or the dedicated one).
2. Create the company's keys in the key service (company master key and field, blind-index, file and export keys).
3. Insert seed documents, all stamped with `companyId`: roles from templates, chart of accounts, number series, tax and statutory settings (dated), default business rules, reminder rules, compliance rules, profile presets, lookups.
4. Create the first admin invitation (identity and membership in the platform database).
5. Compute entitlements and store the snapshot and version.
6. Set status `Ready`; until then the company cannot sign in. Audit entry.
No database or collection is created; indexes already exist. A failed provisioning is safe to repeat and to roll back (delete only documents with that `companyId`, then the registry row).

## 2. Schema migrations (all companies share one schema)
- One schema version for the whole cluster; each document may carry `schemaVersion` where a shape changed.
- **Expand then contract:** new code reads both shapes; a migration moves data; a later release drops the old shape; rollback is a redeploy.
- Migrations run **in company batches** (for example 20 companies at a time, oldest-first by ID), in the **migration runner** (a reviewed platform component), **canary companies first** (internal and pilot), with a stop-on-error threshold. Each migration is idempotent and runs twice on a copy in CI.
- Index builds are rolling builds on the replica set outside peak hours, one collection at a time; every new index starts with `companyId`.
- Risk specific to sharing: a bad migration touches all companies at once. Mitigations are canary batches, dry-run counts, per-batch verification queries, and the ability to pause and resume.
- Feature flags can be targeted to companies and cells (separate from entitlements).

## 3. Backup and restore
**What exists:** Atlas continuous backup with point-in-time restore of the **whole cluster** (confirm the tier); nightly **per-company encrypted exports** to object storage under the company's export key (setting, default on for paying companies); a separate backup account with immutable retention.
**Restore one company** (the common request):
1. Put the company in maintenance (quarantine, read-only message).
2. Restore the cluster snapshot to a **new temporary cluster** isolated from production.
3. Run the **extract tool** (platform component): copy every document with that `companyId` from each collection, verify counts and checksums against the manifest, and decrypt-check a sample.
4. Import into production under a **restore plan**: replace or merge by document ID and version, record what changed; uniqueness is per company so there are no cross-company clashes.
5. Reconcile (stock balances vs ledger, journals vs lines, number series and gaps), lift maintenance, log the restore, notify the admin, destroy the temporary cluster.
**Faster alternative:** import the company's latest per-company export, then replay from the outbox or audit trail if available.
**Whole cell:** restore into a clean network, verify with isolation and reconciliation jobs, rotate keys if compromise is suspected.
**Drills:** one-company restore quarterly; whole-cell yearly; timings recorded.

## 4. Export for the company
A background job writes each collection's documents for that `companyId` (plus files) to object storage, encrypted with the export key, with a manifest of counts; a signed, expiring link goes to an admin (step-up required). Used for portability and offboarding.

## 5. Offboarding and purge (D-047)
1. **Notice period** starts; company can export; plan limits drop to read-only at the end.
2. **Freeze:** status `Retiring`; no sign-in except the admin export page.
3. **Final export** delivered and acknowledged.
4. **Purge job** (platform context, two-person approval to start): for each collection, delete documents with the company's `companyId` in batches with a throttle; delete files under `tenants/{companyId}/`; clear Redis keys `t:{companyId}:*`; cancel queued jobs; remove the company's rows in platform collections that hold company-specific data (entitlements, usage detail); keep only what the law requires (invoices to the company, tax records of the provider) as a minimal record.
5. **Verify:** counts of documents with that `companyId` are zero in every collection, in a second pass by an independent job; sampled raw reads confirm.
6. **Destroy keys** (after verification and a delay for any dispute); log the key IDs destroyed.
7. **Deletion certificate** to the company: date, scope, collections cleared, and the **date by which the last backup containing their data expires** (backups cannot be selectively purged).
8. Registry row marked `Retired` with no data.

## 6. Moving a company to a dedicated cell (or between cells)
Steps: announce a maintenance window → quarantine writes → copy all documents with the `companyId` to the target cluster (extract tool) → verify counts, checksums and a business reconciliation → flip the `tenants.cell` mapping → lift quarantine → keep the old copy read-only for a short rollback window → purge from the old cell (as in section 5 steps 4 to 5). Keys can move with it or be re-wrapped under the dedicated root key.

## 7. Break-glass access
For a real production problem that needs eyes on data:
1. Request with reason, company, time (default one hour), scope (collections), read-only.
2. A second owner approves; the company admin is notified (and can refuse if their contract says so).
3. Access is through the **support viewer** (a tool built on the TDAL for that one company), via the bastion, with recorded commands.
4. Everything is logged in the platform audit and the company's audit log; the grant ends automatically; a post-access review is filed.
There are no standing database logins for people.

## 8. Handling a suspected cross-company exposure
| Step | Action | Who |
| --- | --- | --- |
| 1 Detect | Result-validator alert, canary probe hit, customer report, pen-test finding | Automated, security |
| 2 Contain | Quarantine the route or module (kill switch); set global read-only if writes are suspect; rotate service credentials | Technical lead, within minutes |
| 3 Scope | Use per-company audit logs, query logs and canary markers to list **which companies and fields** were exposed and for how long | Security |
| 4 Preserve | Snapshot logs and affected data; keep chain of custody | Security |
| 5 Fix | Patch the TDAL or route; add the regression test to the isolation suite; verify in staging with the canary test | Engineering |
| 6 Notify | Affected companies without delay; CERT-In within 6 hours of noticing a reportable incident; Data Protection Board intimation without delay and detailed report within 72 hours where personal data is involved (duties from 13 May 2027); affected people as the law requires | Privacy lead, account owner |
| 7 Recover | Rotate affected companies' keys; re-encrypt; lift containment after tests pass | Technical lead |
| 8 Learn | Written review, owner and date for each fix; update the threat table | Incident lead |
The plan is rehearsed twice a year with a seeded isolation bug.

## 9. Fairness and capacity in a shared cell
Per-company limits on requests, concurrent jobs, export size and query time; bulk and report work on lower-priority queues and secondary nodes; metrics per company for the heaviest users; automatic alert when one company exceeds a share of a cell's CPU, memory, connections or storage; move the heaviest to a dedicated cell (section 6).
