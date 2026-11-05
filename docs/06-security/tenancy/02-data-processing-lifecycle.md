# Data Processing and Lifecycle

How a piece of data is handled from the moment it is typed, scanned or imported until it is deleted, who can touch it at each step, and which outside services ever see it. Each company is the owner of its data and decides its purposes; the product processes it only to deliver the service to that company.

## 1. Processing principles (D-042)
1. **Purpose limitation:** company data is used only to run that company's billing, stock, accounts, HR and reports. No secondary use, no sale, no advertising, no cross-company analysis.
2. **Minimisation:** collect what a screen needs; optional fields stay optional; Aadhaar only as a token or last four digits; card numbers never.
3. **Isolation by default:** every step runs inside one company's context (see 01).
4. **Least visibility:** a person sees only what their role, scope and field rules allow; staff of the platform see none by default.
5. **Encrypt by class:** transport always; storage always; sensitive fields with per-company keys (03).
6. **No training or sharing:** company data is never used to train models, build shared catalogues or benchmark other companies. Product analytics use **counts and timings only**, with no content.
7. **Test with synthetic data:** development and staging never hold real company data (masked or generated only).
8. **Everything attributable:** reads of sensitive fields, exports, reveals, support access and administrative changes are logged.
9. **Deletion is real:** purge on exit with verification (04); erasure requests handled within the law's limits (47).
10. **Region:** data resides in an Indian region unless a company chooses otherwise.

## 2. The lifecycle, step by step
| Stage | What happens | Controls |
| --- | --- | --- |
| **1. Capture** | A person types or scans on a Flutter app, uploads a file, imports a sheet, or a partner API or webhook sends data | Client-side validation (usability only); screens ask only needed fields; device storage encrypted; imports go through a validate-and-fix step with an error file |
| **2. Transport** | App ↔ API over TLS 1.2+; API ↔ MongoDB and Redis over TLS; workers ↔ providers over TLS | HSTS on web; certificate validation; short-lived tokens; request IDs; replay protection on webhooks |
| **3. Entry** | Request passes the middleware chain: authenticate → tenant context → module on → permission → idempotency → schema validation | Unknown fields rejected; company from the token only; rate limits per company and user |
| **4. Processing** | A use case runs in memory under the company context; pure domain code calculates tax, prices, stock; repositories read and write through the TDAL | No cross-company caches or globals; money in Decimal128; no personal data in logs; time limits on every query |
| **5. Storage** | Documents in shared collections stamped with `companyId`; files in object storage under the company path | Disk encryption; per-company field encryption for restricted and personal fields; hashes for passwords and tokens; blind indexes for exact search |
| **6. Caches, queues, temporary files** | Redis caches, BullMQ jobs, temporary export or PDF files | Company-prefixed keys; short TTLs; jobs carry IDs not documents; temporary files in a per-job folder, encrypted, deleted after use or at expiry |
| **7. Presentation** | Lists, records, dashboards, prints | Field rules (hidden, masked); only records in scope; masked values revealed only with the reveal setting (logged) |
| **8. Sharing out** | Invoices by WhatsApp, SMS, email or PDF; e-invoice to the GSP; payments through the gateway; webhooks to the company's systems; exports | Only the fields needed for the purpose; customer messages only with consent and approved templates; expiring links; every send logged on the record |
| **9. Analysis** | Reports, dashboards, restock suggestions, reminders | Computed per company by workers into per-company summaries; never merged with other companies; reports respect field rules and scope |
| **10. Retention** | Records kept for statutory periods (proposed default 8 years, setting); audit logs per policy | Retention rules per record type; archive to encrypted object storage after the period; legal holds possible |
| **11. Deletion** | A person's erasure request, a record's end of retention, or the company leaving | Erasure or pseudonymisation of personal data where the law allows; **company purge** removes all documents, files, caches and keys (04); backups expire on schedule; a deletion certificate is issued |
| **12. Incident** | Suspected exposure | Playbook in 04; reporting clocks in 47 |

## 3. Who can touch the data
| Party | Access to company data | How controlled |
| --- | --- | --- |
| Company users | What their role, scope, branches and field rules allow | Five checks on every request |
| Company admin | Everything inside the company's entitlements, including audit and role editing | Step-up for sensitive actions |
| Platform super admin and staff | **None** by default; entitlements, usage, health only | D-023; separate realm; hardware second step |
| Support person | Read-only, only inside a company-approved, time-limited grant | Grant, banner, double logging, field masking |
| Engineers and operators | **None** to production data in normal work; break-glass only, approved by a second owner, time-limited, read-only, logged | Bastion, tenant-filtered viewer, session recording of commands |
| Automated services | Only through the TDAL under a rebuilt company context; the purge, the outbox relay and the migration runner are the only components allowed to work outside one company, and they are reviewed, logged and tested | Code ownership, tests, alerts |
| Database provider (Atlas, cloud) | Infrastructure access under their contract; data encrypted at rest; customer-managed keys if the tier offers them | Contract, region, audit reports |
| Other companies | **Never** | Layers 1 to 10 |

## 4. Where company data goes outside the database (sub-processors)
Final vendors are not chosen (D-013, D-014); the table shows what each category sees.
| Service | What it receives | What it must not receive | Notes |
| --- | --- | --- | --- |
| MongoDB Atlas (database) | All stored data, with sensitive fields already encrypted | — | Region India; private endpoints |
| Cloud hosting and object storage | Encrypted files, backups | Plain keys | Per-company file keys; separate backup account |
| KMS / HSM | Key wrap and unwrap requests | Data | Only the key service calls it |
| Redis (managed) | Cache entries and job metadata | Full documents, secrets in plain text | Short TTLs, company-prefixed keys |
| GSP and e-invoice portal | The invoice JSON for that company's own GSTIN | Other companies' data, internal notes | Per-company credentials from the vault |
| Payment gateway | Amount, reference, token | Card numbers (never held) | Signed callbacks |
| Messaging (WhatsApp, SMS, email) | Recipient, template, variables | Whole records | Consent check; templates approved; log on the record |
| Monitoring and error tracking | IDs, timings, error codes | Names, phones, GSTINs, amounts, field values | Scrubbers in the SDK; reviewed |
| Email service for invitations | Email address, link | — | SPF, DKIM, DMARC |
Each vendor needs a data processing contract, a listed sub-processor entry, a breach notice clause and an exit plan.

## 5. Data map (what is stored where)
| Data | Class | Stored in | Encryption | Retention | Visible to |
| --- | --- | --- | --- | --- | --- |
| Passwords, PIN verifiers | Restricted | `identities`, device secure store | Argon2id hash | Until changed or account removed | Nobody |
| Two-step secrets, recovery codes | Restricted | `identities` | Field-encrypted; codes hashed | Same | Nobody |
| Integration credentials, API secrets | Restricted | `connections` | Per-company field keys | Until revoked | Admin (masked) |
| Bank accounts, PAN, ID tokens | Restricted | parties, employees, banking | Per-company field keys; blind index where needed | Statutory | Roles with reveal |
| Mobile, email, address of people | Personal | parties, employees, users | Per-company field keys; blind index | Per policy | Roles by field rule |
| Invoices, stock, ledgers | Business | sales, stock, accounts collections | Disk and transport | Statutory (proposed 8 years) | By role and scope |
| Item, party names, business GSTIN | Business | masters | Disk and transport | Statutory | By role |
| Files and attachments | Business or personal | Object storage | Per-company file key | Per record | By role |
| Audit and activity logs | Business | `audit_log` | Disk; chained hash | Per policy (≥ 1 year) | Admin, manager (team) |
| Platform registry and entitlements | Platform | Platform database | Disk | While a company exists | Platform staff |
| Usage meters | Platform | Platform database | Disk | 2 years (setting) | Platform finance |
| Customer-facing messages sent | Business | `message_log` | Disk | Per policy | Admin, accountant |

## 6. Memory, device and temporary-data handling
- Secrets and keys live only in the key service, the vault and process memory for short periods; never in logs, environment dumps or error pages.
- Crash reports and traces pass through a scrubber that removes values; stack traces never reach clients.
- Temporary PDFs and exports live in a per-job directory on encrypted storage and are deleted when the job ends or after a short expiry.
- On devices, the local database and drafts are encrypted per user, keys held in the platform secure store, cleared at sign-out unless the person keeps them (D-009); a lost device is revoked remotely and its offline grant expires within 24 hours.
- Clipboard, screenshots and screen-sharing are not blocked by default; the admin may switch on screen-capture protection for sensitive screens where the OS supports it (setting).

## 7. Rights and requests (company is the owner; the product enables)
Access, correction, portability and erasure requests are handled as tasks inside the company: export of a person's data, pseudonymisation or erasure with exceptions for records the law requires, and a log. Company-wide export and purge follow 04. The legal text is for the company's lawyer (47, 48).

## 8. Processing of special flows
| Flow | Notes |
| --- | --- |
| Offline Counter | Bills created and stored on the registered device (encrypted); synced idempotently; permission snapshot signed and expiring in 24 hours |
| Imports | Files go to the company path, are scanned, validated and processed in a worker under the company context; error files are company-scoped and expire |
| Scheduled reports by email | Built per recipient with only what they may see |
| Owner with several companies | Each company processed in its own context and merged in the app (D-045) |
| Third-party webhooks | Verified signature; processed in the company context determined by the signing secret |
| Backups | Snapshots are encrypted and hold all companies' data together; restores always go to an isolated temporary cluster first (04) |
