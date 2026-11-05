# Privacy and Compliance

Not legal advice: a lawyer confirms this against the contracts and the gazette text before launch. Summaries of the DPDP Rules disagree on rule numbers, so cite the gazette, not summaries.

## What applies
| Rule | Asks | How the design answers |
| --- | --- | --- |
| CERT-In directions (28 Apr 2022) | Report cyber incidents within 6 hours; keep logs of all systems for a rolling 180 days; sync clocks to NIC/NPL; name a point of contact | Incident clock below; logs in an India region; time sync; named contact |
| DPDP Act and Rules 2025 (Rules notified 13 Nov 2025) | Notice and consent, safeguards (encryption, masking, tokens), breach intimation, logs kept at least a year, guardian consent for children. Operational duties, including breach intimation, from **13 May 2027** | Consent and breach records; encryption and masking; processing logs ≥ 1 year |
| Payment card rules | Do not hold card numbers unless fully certified | Gateway token only |
| GST and accounting records | Records intact and available | Immutable posting, document hashes, backups, retention settings |
| ISO 27001 / SOC 2 | Often requested by larger customers | Policies written to match; certify after launch on request |

## Data handling
- Host in an India region unless a client chooses otherwise.
- Retention period per record type; export of all company data on request; deletion on exit after a notice period; company key destroyed after retention.
- Consent records, purpose limits, masking in screens; export and erasure requests handled as tasks (privacy module).
- Logs and crash reports never contain customer names, phones, GSTINs or amounts.
- Company audit log kept as long as the company sets (processing logs at least one year); platform logs at least 180 days rolling.
- Billing records may have statutory retention periods that override an erasure request; decide the periods with the accountant and lawyer.

## Incident clock
| Step | When | Who |
| --- | --- | --- |
| Triage and declare | Within 1 hour of an alert that looks real | On-call engineer |
| Preserve evidence | At once | Technical lead |
| Contain (revoke sessions, rotate keys, quarantine the company or route) | At once, in parallel | Technical lead |
| Report to CERT-In | Within 6 hours of noticing a reportable incident | Named point of contact |
| Inform affected companies | As the contract says, as soon as facts are clear | Account owner |
| Personal data breach: Board and affected people | Intimation without delay; detailed report within 72 hours (duties apply from 13 May 2027) | Privacy lead |
| Recover and learn | Restore from clean backups; written review with owners | Incident lead |
Plan is rehearsed twice a year; contacts tested.

## Open items
Terms of service and privacy policy drafts (checklist item 48) need a lawyer and are not written here.
