# Known Issues and Technical Debt

Nothing is built, so this file tracks **known spec debt** and decisions deferred on purpose. Detailed analysis: [REVIEW-FINDINGS.md](../REVIEW-FINDINGS.md).

## Open spec problems
| ID | Summary | Needed to close |
| --- | --- | --- |
| F-11 | Drafts device-only | Owner, D-009 |
| — | Cell company cap and sharding trigger not yet measured | Load test (54), D-039 |
| F-15, F-16 | E-invoice 30-day window and cancel condition | Tax advisor |
| F-18 | Offline permission staleness | Owner, D-008 |
| F-19 | Node 26 LTS status, MongoDB version on the chosen Atlas tier | Check official pages when pinning |
| — | Role document still uses Edit/Full/Own and an older rail | Regenerate from the manifest |
| — | Architecture phase diagram not reconciled with the UI build order | Compare and record |

## Deferred on purpose (release 1)
Payroll, GST portal filing, manufacturing, service jobs and AMC, online-store sync, loyalty, multi-currency, offline Workspace, Tally/Vyapar/Zoho import mappings, ISO 27001 / SOC 2.

## Debt to watch once code exists
- Shortcuts taken in posting, numbering or permission code are recorded here with a reason, owner and removal condition.
- Any index added outside the budget; any collection beyond the 66.
- Any constant that should be a dated setting.
- Any hand-written permission key that bypassed the manifest.
- Any place the client decides access without the server's answer.

Format for new entries: `ID | what | why accepted | risk | remove by`.
