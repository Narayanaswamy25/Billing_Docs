# Risk Register

Likelihood and impact: H / M / L. Owner column is blank until a team exists.

| ID | Risk | L | I | Mitigation | Owner |
| --- | --- | --- | --- | --- | --- |
| R-01 | Client stack conflict (React vs Flutter) | H | H | Closed: Flutter chosen (D-001); keep the architecture doc's React sections marked superseded | |
| R-02 | One company sees another's data (more likely in a shared database than before) | M | Critical | Mandatory TDAL, company from token only, per-company keys, scoped caches/queues/files, result validator, canary probes, isolation tests as release gate, kill switches (tenancy/*) | |
| R-03 | One heavy company degrades neighbours in a shared cell | M | M | Per-company quotas, query limits, workload separation, dedicated cell option (D-043) | |
| R-04 | Duplicate or gapped invoice numbers (offline, failed posts) | M | H | Number only at post, server-issued blocks, idempotency keys, void-and-log process, gaps report | |
| R-05 | E-invoice/GSP outage or reporting-window breach | H | M | Pending status, retries, ageing alert, advisor-confirmed rules, dated settings | |
| R-06 | Payroll scope creep delays release 1 | M | H | Payroll out of release 1 (D-007) | |
| R-07 | Tax rules change after build | H | M | Thresholds as dated settings; accountant checks sample invoices per tax case | |
| R-08 | Role matrix too complex for small shops | M | M | Presets, Who works here setup, Preview as role, test with five staff | |
| R-09 | Key loss or KMS misuse makes data unreadable or exposed | L | Critical | Two-person recovery, key backup in KMS, rotation and destruction rehearsed | |
| R-10 | Breach reporting clocks missed | L | H | Named roles, rehearsed plan; CERT-In 6 h, DPDP intimation without delay + 72 h report (from 13 May 2027) | |
| R-11 | Thermal printer/scanner variety breaks counters | M | M | Device test list; bill is saved before printing; "Printer not reached" recovery | |
| R-12 | Stale offline permissions let a removed user sell | L | M | 24 h limit then lock (D-008) | |
| R-13 | Unverified tech claims (Node 26 LTS status, MongoDB 9 on the chosen Atlas tier) | M | L | Pin to installable versions; see F-19 | |
| R-14 | Large company overloads a shared cell | M | M | Per-company rate limits, query time limits, move to dedicated cell | |
| R-15 | No team size or deadline known | H | M | Roadmap ordered, not dated; set dates after question 8 is answered | |
| R-16 | A Counter uses all 50 numbers offline and must sync before billing again | L | L | Owner judged 50 enough; warn at 10 left; company block size; Held bills (D-005) | |
| R-17 | Revert or restore used wrongly or maliciously (for example hiding a fraud, or rolling back a permission) | M | H | Reason required, step-up, notifications to the editor and admin, revert is itself an audited sensitive event, age limits for non-admins, approvals for sensitive classes | |
| R-18 | A company restore removes issued tax documents or reuses invoice numbers | L | Critical | Restore guardrails: never remove issued documents, never reuse numbers, accountant sign-off, maintenance mode, diff preview (history/03) | |
| R-19 | UPI, WhatsApp and SMS costs are uncertain or changing (reported UPI merchant fee, per-message WhatsApp pricing, DLT lead time) | M | M | Release 1 avoids a payment gateway (D-055); utility messages only; DLT registration started early; adapters swappable | |
| R-20 | The original Isar local database is reported unmaintained | M | M | Spike Drift and the Isar fork on the Counter; abstract storage behind a repository (D-056) | |
| R-21 | No public e-invoice provider prices; vendor lock-in and outages | M | M | Three quotes, EInvoicePort adapter, Pending status (D-060) | |
