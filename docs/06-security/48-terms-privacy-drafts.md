# Terms of Service and Privacy Policy: outline only

**This is not legal text.** A lawyer must draft and review the terms, privacy notice, consent flow and processor (data processing) terms before launch. This file lists what the product's behaviour requires them to say, so the lawyer starts from facts.

## Roles to define
The owner of the product is the service provider; each company is the party that decides why customer and employee data is processed; the product processes it on the company's behalf. Counsel to confirm how this maps to the DPDP terms (fiduciary / processor) and what contract terms are needed.

## Facts the documents must state
| Topic | Fact from the design |
| --- | --- |
| Data location | Hosted in an India region unless a client chooses otherwise |
| Isolation | Companies share infrastructure but their data is separated by a mandatory access layer, per-company encryption keys for sensitive fields, per-company storage paths and caches, and continuous isolation tests; platform staff cannot read company records unless the company grants time-limited, logged support access; a dedicated cluster is available for companies that need physical separation |
| Encryption | In transit and at rest; sensitive fields encrypted with a per-company key; keys held in a separate key system |
| Card data | Never stored; payment gateway token only |
| Aadhaar | Never stored whole |
| Logs | No customer names, phones, GSTINs or amounts in logs or crash reports; audit logs retained per policy |
| Retention and exit | Retention periods by record type; export of all data on request; deletion after a notice period; company key destroyed |
| Rights requests | Access, correction, erasure handled as tasks; statutory record-keeping may override erasure |
| Breach | CERT-In report within 6 hours; Board intimation without delay and detailed report within 72 hours (from 13 May 2027); affected companies informed per contract |
| Sub-processors | GSP, payment gateway, messaging providers, cloud, monitoring: listed, with exit plans |
| Availability and support | Targets in 31-non-functional-requirements (proposals, not promises) |
| Plans, limits, suspension | Past due → grace → read-only → blocked; data stays readable; Counter keeps billing offline for a set time |
| Acceptable use | No attempt to cross company boundaries; no storage of card numbers in free-text fields |
| Liability and tax accuracy | Tax rules are configurable settings; the company and its advisor remain responsible for tax decisions (lawyer to word) |

## Consent flow (product)
Consent records per person and purpose in `consents`; notice shown in the person's language; guardian consent where children's data is involved; withdrawal supported.

## To produce
Terms of service, privacy notice, data processing agreement, sub-processor list, cookie/storage notice (web build only), vulnerability disclosure page.
