# Security Policy

No system can promise it cannot be breached. Goals: an attacker must beat several independent layers; a breach of one company never reaches another; a stolen database or backup is unreadable without keys held elsewhere; attempts are seen and answered quickly.

## Data classes
| Class | Examples | Protection |
| --- | --- | --- |
| Restricted | Passwords, portal/gateway credentials, API secrets, bank accounts, person PAN, ID numbers | Passwords hashed (never encrypted); others field-encrypted with a company key; never in logs. Aadhaar never stored whole (last four or a token) |
| Personal | Mobile, email, address of individual customers; employee personal details; salary components | Field-encrypted; exact search via blind index |
| Business | Invoices, stock, ledgers, names, business GSTIN | Encrypted in transit and at rest; not field-encrypted (search, reports need them) |
| Public | Portal catalog | TLS and integrity |
Card numbers are never stored; gateway token only.

## Encryption
- **In transit:** TLS 1.2+ on every hop including API↔MongoDB and API↔Redis; HSTS on web.
- **At rest:** Atlas encrypts storage at rest; use customer-managed keys through the chosen cloud's KMS where the tier supports it (confirm for the tier). Backups are encrypted by Atlas and by their own key where offered.
- **Field level:** AES-256-GCM with a fresh random IV every time, stored in an envelope (key ID, IV, ciphertext, tag); associated data = company ID, collection, field name, record ID so ciphertext cannot be moved. Rotate data keys at least yearly (random 96-bit IVs: about four billion encryptions per key). Use MongoDB in-use encryption **or** application envelope encryption, not both.
- **Files:** per-company file key; SHA-256 checksum stored.
- **Backups:** own key, separate account and credentials, immutable.

## Hashing
Argon2id (≥19 MiB memory, 2 iterations, parallelism 1, per-password salt, pepper in the vault; rehash at sign-in when settings improve). Reset, invitation, refresh tokens and API keys: random 256-bit shown once, SHA-256 stored. Searchable private fields: HMAC-SHA-256 per-company key, normalized input. Audit log and posted documents: SHA-256 chained. Never MD5, SHA-1, or plain SHA-256 for passwords.

## Keys
Root key in a KMS/HSM in an India region (two-person policy changes, every use logged) → company key → data keys (stored encrypted in the platform database's key vault collection, readable only by the key service). Key recovery needs two named people. Leaving company: export, wait retention, destroy the company key. Test and dev use their own keys and masked data.

## Field protection map
| Collection | Encrypted | Hashed / blind-indexed |
| --- | --- | --- |
| identities (platform) | email, mobile, second-step secret | password (Argon2id + pepper); email and mobile blind indexes |
| users (company) | — | quick-PIN verifier per device (Argon2id); no password stored here |
| memberships (platform) | — | identity ↔ company link only; no personal data |
| parties (customers) | mobile, email, person PAN, contact mobiles/emails, birthday | blind index on mobile, email, PAN |
| parties (suppliers) | bank details, contact details, PAN | blind index on PAN |
| employees | address, emergency contact, bank, PAN, UAN, ESIC, Aadhaar token, DOB, ID proof | blind index on PAN, mobile |
| banking | account numbers | none |
| connections | client IDs, secrets, session tokens, webhook secrets | API keys as SHA-256 |
| consents | person details | blind index |
| tokens (reset, invite, refresh) | not stored | SHA-256 only |

## Company isolation (shared database)
See [tenancy/00-README.md](tenancy/00-README.md): ten layers (tenant context, mandatory data access layer, schema and index rules, credentials, scoped caches/queues/files/logs, API rules, platform operations, detection, kill switches, scale and fairness), per-company keys ([tenancy/03](tenancy/03-encryption-and-keys.md)), processing rules ([tenancy/02](tenancy/02-data-processing-lifecycle.md)) and continuous tests ([tenancy/05](tenancy/05-isolation-tests-and-assurance.md)).

## Database and infrastructure
**On MongoDB Atlas (D-012):** one project per environment; one cluster per cell plus one for the platform database; database users created per application and per job with least-privilege roles (companies share collections, so isolation is done by the tenant data access layer; see [tenancy/01](tenancy/01-isolation-architecture.md)); private endpoints or peering, no public access, IP access list for administration; TLS enforced; Atlas Cloud Backup with point-in-time restore; Atlas database auditing and alerts where the tier includes them (confirm); automatic client-side field encryption and Queryable Encryption are available on Atlas but the field map below uses application-level envelope encryption **or** the driver's in-use encryption, not both. Atlas limits transaction lifetime through its own setting rather than a server flag.

Generic controls (apply on any host):
Authentication always on; one DB user per application and per job (API, worker, migration, backup, monitoring, admin); private network, no public address; TLS node-to-node; server-side scripting off (`$where`, `$function`, `mapReduce` unused); named human access through a bastion with a second step, expiring and logged; auditing on Enterprise/Atlas or server logs on Community; monthly patching, faster for critical. Platform database in its own cell with stricter access. Cloud: separate accounts per environment, least-privilege IAM, no long-lived keys, secrets in the vault, minimal signed container images, default-deny network policies, SPF/DKIM/DMARC, NTP sync to NIC/NPL time servers (CERT-In), masked data only outside production, managed staff devices.

## Application security (OWASP Top 10:2025)
| Risk | Controls |
| --- | --- |
| Broken access control (incl. SSRF) | Deny by default; five checks on every route; scope filter on every query; two-company and other-user tests; outgoing calls only to an allow-list, internal/metadata addresses blocked, re-checked after name lookup |
| Misconfiguration | Infrastructure as code, hardened images, config validated at start, one security-header module, automated scan each release |
| Supply chain | Lock files, reviewed new packages, SBOM, signed images, private mirror, waiting period for brand-new releases |
| Cryptographic failures | This document; no home-made crypto |
| Injection | Every input parsed by a schema naming its fields; never pass a request object to a query; no `$where`; escape HTML in PDF templates; neutralise spreadsheet cells starting `= + - @` on export |
| Insecure design | Short threat model per module, abuse cases, design review for money/keys/access |
| Authentication failures | See 22; breached-password check; new session ID at sign-in; refresh reuse detection |
| Integrity failures | Signed builds checked at deploy, signed webhooks, hash-chained audit, immutable posted documents, JSON only with schema checks |
| Logging failures | Alerts in [49-audit-logging.md](49-audit-logging.md) |
| Exceptional conditions | Fail closed: if the permission cache, tenant context, data access layer or key service fails, refuse the request; no stack traces; failure paths tested |

Client/API settings: strict CSP (web), HSTS, no framing, nosniff, strict referrer, restrictive permissions policy, origin allow-list, no-store on personal/financial replies, upload size/type limits with content check and virus scan, files served from a separate address. Mobile/desktop clients: encrypted local DB, tokens in secure storage, certificate pinning considered, local data limited to what the Counter needs and cleared at sign-out except unsent bills.

## MongoDB-specific
API accepts only fields its schemas name (no `$ne`/`$where` injection); repositories build queries; controllers never pass request objects.

## Fix targets
Critical 2 days · High 7 · Medium 30 · Low next release. Outside penetration test before launch and yearly; vulnerability disclosure page; a security champion per module team.

## Pre-launch checklist
- [ ] Cross-company isolation tests pass on every route
- [ ] Penetration test fixed and retested
- [ ] Single-company restore rehearsed
- [ ] Load test at planned cell size
- [ ] Accountant has checked tax settings; lawyer has reviewed notice, consent, processor terms
- [ ] Incident runbook and CERT-In / privacy clocks rehearsed
- [ ] A copy of the database read without keys shows no plain sensitive field
- [ ] Key rotation and key destruction rehearsed; key vault restore drilled
- [ ] Backups unreadable without the vault
- [ ] Security headers and cookie flags verified on every address
