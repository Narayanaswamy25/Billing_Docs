# Encryption and Keys in a Shared Database

In a shared database the data of many companies sits in the same files, so encryption has two jobs: protect against theft of storage or backups, and give each company its **own key** so a leak of one company's key never opens another's data.

## 1. Layers
| Layer | What | Notes |
| --- | --- | --- |
| In transit | TLS 1.2+ on every hop (app, API, database, Redis, providers) | HSTS on web |
| Storage at rest | Atlas storage encryption; customer-managed key through the chosen cloud KMS where the tier supports it | Protects a stolen disk or snapshot; does **not** separate companies |
| **Per-company field encryption** | Restricted and personal fields encrypted before they reach the database with that company's data keys | The separation between companies at the data level |
| Files | Each file encrypted with the company's file key before upload | A bucket-wide leak yields ciphertext |
| Backups and exports | Atlas backup encryption; per-company exports encrypted with that company's export key | |
| Devices | Encrypted local database per user | Keys in the platform secure store |

## 2. Key hierarchy
```
Root key (cloud KMS or HSM, India region; two-person policy changes; every use logged)
   └─ Company master key (KEK)  — one per company, wrapped by the root key, held in the platform key vault
        ├─ Field key       (restricted and personal fields)
        ├─ Blind-index key (HMAC for exact-match search)
        ├─ File key        (attachments and generated documents)
        └─ Export key      (exports and per-company backups)
```
- A **key service** (separate process, mutual TLS, only caller of the KMS) wraps and unwraps; it checks the caller's identity and that the **requested company equals the request's company context**, so a bug elsewhere cannot ask for another company's key.
- Unwrapped keys live in memory for minutes, never on disk, never in logs; unwrap counts per company are metered and alerted.
- Each key has an ID and version; ciphertext envelopes store `{companyId, keyId, version, iv, ciphertext, tag}`. The authenticated data binds **company, collection, field and record**, so ciphertext copied to another record or company fails to decrypt.
- Fresh random IV for every encryption; rotate a key before its usage count nears the random-IV limit (about four billion encryptions per key) and at least yearly.

## 3. What is encrypted per company, and what is not
| Encrypted with the company's keys | Plain at field level (protected by access control, disk and transport encryption) |
| --- | --- |
| Two-step secrets, integration credentials, API secrets, webhook secrets | Invoice, stock, ledger and report data |
| Bank account numbers, person PAN, ID tokens, UAN, ESIC | Item and party names, business GSTIN, addresses of businesses |
| Mobile, email, address, birthday of individual people | Quantities, prices, taxes |
| Employee personal and (later) payroll amounts | Document numbers, dates, statuses |
| Files (all) | Audit entries (but values of encrypted fields are stored encrypted) |
Why not encrypt everything: screens, indexes, sorting, totals and reports need to read business fields; encrypting them would slow the product and remove search. The consequence must be understood: **a person with unrestricted database access could read a company's plain business fields.** Mitigations are layers 1 to 10 and the rule that nobody has such access (D-023, 02). Companies that need stronger protection can use:
- **Queryable Encryption** on chosen fields (equality and range search stay possible; limits apply; Atlas feature; check the tier and version), or
- a **dedicated cell** (D-043) with its own cluster, keys and region, or
- customer-managed root keys on that cell.

## 4. Rotation, revocation and compromise
| Event | Action |
| --- | --- |
| Routine | Rotate field, file and export keys yearly: new version used for new writes; old versions kept for reads; background re-encryption optional |
| Suspected leak of one company's key | Rotate that company's keys only; re-encrypt its data; other companies unaffected |
| Suspected leak of the root key | Re-wrap all company master keys under a new root; rotate service credentials; incident playbook |
| Employee leaves | Rotate service credentials they could reach |
| Company leaves | Purge data (04), then destroy that company's keys |

## 5. Crypto-shredding: what it does and does not do
Destroying a company's keys makes its **encrypted** fields and files unrecoverable, including in old backups. It does **not** erase plain business fields in database snapshots that still exist. So deletion uses two steps: **purge** live data (04), then **destroy keys**; plain business fields in backups disappear only when those backups expire (state the maximum age to the company in writing, for example 35 days of point-in-time history, and in the deletion certificate).

## 6. Searching encrypted fields
Exact match through a blind index: HMAC-SHA-256 of the normalised value with the company's blind-index key, stored beside the ciphertext. Because the key is per company, the same phone number in two companies gives different indexes, so an index value cannot be used to link people across companies. Range and prefix search on encrypted fields are not offered.

## 7. Key custody
Root key recovery needs two named people; key vault and KMS are backed up inside the KMS provider, never beside the data; test and development use their own keys; dedicated-cell companies may hold their own root key. Every key action is in the platform audit log.

## 8. Tests
No restricted field appears in plain text in a raw collection read; IVs never repeat; a changed byte fails the tag; ciphertext moved to another record, field or company fails; rotation keeps old data readable; a company's key cannot unwrap another company's data; the key service refuses a request whose company differs from the context. These run in CI (05).
