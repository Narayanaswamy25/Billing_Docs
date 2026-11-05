# Runbook and Troubleshooting Guide

First steps for any incident: note the time and the company ID; check the status page and the cell dashboard; check recent releases and module switches; preserve logs. For a suspected breach go straight to [47-privacy-compliance.md](../06-security/47-privacy-compliance.md) (incident clock). Never edit production data by hand.

| Problem | Signs | First checks | Fix |
| --- | --- | --- | --- |
| Cell nearing limits | Memory, connections, storage or latency alarms | Which companies use the most; index size vs RAM | Raise the tier, shard, or move the heaviest company to a dedicated cell; tighten quotas |
| E-invoice portal outage | Failed e-invoice alert, many Pending | GSP adapter circuit breaker, portal status | Leave documents Pending; retries continue; tell companies; watch reporting-window ageing |
| Stuck queue | Queue age rising, workers idle | Worker health, Redis, per-company concurrency | Restart workers, replay dead letters after cause is fixed |
| Failed migration | Errors after a release, some companies on old shape | Migration batch log | Pause, fix, re-run the batch (idempotent); code supports the previous version; canary results first |
| **Suspected cross-company exposure** | Result-validator alert, canary probe hit, customer report | TDAL events, audit logs | Follow the exposure playbook in tenancy/04: quarantine the route, scope, notify, fix, add a regression test |
| Failed restore | Drill or real | Backup account access, key vault | Fall back to older point-in-time; escalate; log |
| Leaked key or credential | Alert, report, scan hit | Which key (company, API, provider) | Revoke, rotate, re-encrypt what was exposed; per-company keys limit blast radius |
| Suspected breach | Cross-company error, mass export, probing | Audit logs, denied counts | Declare incident, contain, follow clock |
| Duplicate or missing invoice numbers | Gaps report, auditor query | Number series, offline blocks, retired devices | Void unused block numbers with a reason; explain each gap in the log; never reuse a cancelled number |
| Counter shows "Offline, N bills waiting" for long | Sync dot amber/red | Connectivity, token expiry, permission age (24 h limit) | Sign in again, sync; resolve Conflicts (e.g. short stock) |
| Printer not reached | Banner on Counter | Printer power, paper size, driver | Try again; choose another printer; bill is already saved |
| User says "screen missing" | Support ticket | Module switch, role ticks, personal change, branch scope; My access page | Explain via Effective access ("Why can this person see this?"); admin edits role |
| Many "not available" replies for one user | Denied-attempt alert | Role mis-set vs probing | Fix role or investigate |
| Transaction abort rate high | Abort metric over 1% | Hot number series, hot stock balance | Per-terminal numbering, retry loop; reduce transaction length |
| Slow search or lists | Latency alerts | Query plans, missing index, scan | Add index with migration and plan test |
| Suspended company says shop stuck | Past-due flow | Billing state, Counter offline allowance | Contact; grace and offline allowance settings |

Add owner, last-reviewed date and escalation contact to each row when a team exists.
