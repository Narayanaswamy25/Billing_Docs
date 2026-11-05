# Assumptions, Constraints and Out of Scope

## Assumptions
| Assumption | If wrong |
| --- | --- |
| India first; businesses under GST | Other countries need their own tax and payroll packs |
| Service run by the owner for clients; no on-premise installs | On-premise needs separate packaging and updates |
| All companies share collections in a cell, isolated in software (D-039) | A client needing physical separation gets a dedicated cell (D-043) |
| The same email can belong to two companies | Login directory makes email globally unique |
| Backend is Node.js + Express + MongoDB Atlas; clients are Flutter (D-001, D-012, D-021) | Decided by the owner; changing it means revisiting the decision log |
| Panel is 25% (owner wrote 5, 50, 70) | At 50% the body would be 45% |
| "Vector hashing" means encryption with an IV plus hashing, not AI vector search | Tell the owner if vector search was meant |

## Constraints
- Budget, deadline and team size: **not given**. The roadmap therefore has no dates.
- GST thresholds and windows change; a tax advisor confirms them before build.
- A lawyer reviews privacy notices, consent flow and processor terms before launch.

## Out of scope for release 1
| Item | Decision | Reason |
| --- | --- | --- |
| Payroll (PF, ESI, professional tax, payslips) | Later | A product in itself, legal filings |
| Return filing to the GST portal | Later (data is prepared) | Needs a government partner and certification |
| Manufacturing, bill of materials | Out | Different screens and costing |
| Service jobs and AMC | Out | Different document flow |
| Online store sync | Out | Needs stock reservation rules |
| Loyalty points | Out | Can be added at the counter later |
| Multi-currency | Out unless exports planned | Touches every amount field |
| Offline Workspace | Out (drafts built so stock counting can be added) | Keeps back office simple |
| ISO 27001 / SOC 2 certification | After launch, when customers ask | Policies written to match now |
| Tally / Vyapar / Zoho import mappings | Not specified | Needs its own document |

When a module is out, its icon is not drawn; the rail never shows "Coming soon".

## Must not do
- Show a role anything the super admin or admin did not give it.
- Store card numbers (gateway token only) or a full Aadhaar number.
- Log customer names, phones, GSTINs or amounts.
- Let platform staff read company records without a company grant.
