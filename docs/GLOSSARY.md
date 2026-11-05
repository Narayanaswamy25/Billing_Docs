# Glossary

| Term | Meaning |
| --- | --- |
| Console | The super admin's app: companies, plans, features, support access, platform log, platform staff. |
| Workspace | The company back-office app for admin, manager, accountant, billing officer, HR officer, stock employee. |
| Counter | The salesperson's fast billing app. Own layout, same permission rules, works offline. |
| Rail / Panel / Body | The three screen areas: icon strip (5%), list column (25%), detail area (70%). |
| Module | A switchable product area (Sales, Purchases, Stock, Accounts, HR, People, Reports, Settings; Dashboard, People and Settings are always on). |
| Feature | A switch inside a module (e.g. payroll inside HR). |
| Manifest | The one declaration per module (icon, screens, verbs, dashboard cards, routes, jobs). Rails, editor rows, cards and role tables are generated from it. |
| Verb | One of 15 permissions: view, add, change, remove, post, approve, cancel, print, reprint, send, export, convert, view_cost, import, revert. |
| Scope | Which records a verb covers: Own, Team, Branch, All. A setting, not a verb. |
| Preset | A named bundle of verbs: No access, Just view, Can work, Full control; anything else is Custom. |
| Personal change | One added or removed permission for one user, shown with a badge. |
| Field rule | Hidden (not drawn, listed, searched or exported) or Masked (last digits only) per field and role. |
| Cell | One MongoDB (Atlas) cluster holding the shared collections of many companies. |
| Tenant context | The company, user and session for the current request, built only from the signed token and read-only for the rest of the request. |
| Posting | Making a document final in one transaction: document, stock rows, journal, number, outbox event. |
| Outbox | Events written in the same transaction as the change, then relayed to queues. |
| IRN | Invoice Reference Number issued by the e-invoice portal. |
| GSP | GST Suvidha Provider; the partner that connects the app to the e-invoice and e-way bill portals. |
| E-way bill | Transport document required for movement of goods above a value. |
| GSTIN / HSN / SAC | Tax registration number / goods code / services code. |
| CGST, SGST, IGST | Central, state and integrated GST; split depends on place of supply. |
| TDS / TCS | Tax deducted / collected at source. |
| Reverse charge | Tax liability moves from supplier to buyer. |
| Credit note / Debit note | Documents that reduce / increase an already posted invoice. UI plain words: "Refund note". |
| Block (numbers) | A range of invoice numbers given to a Counter device for offline use. |
| IV | Initialization vector: random value stored beside ciphertext; never reused with the same key. |
| Blind index | HMAC of a normalized value stored beside an encrypted field, so exact-match search works without decrypting. |
| Support mode | Time-limited, logged, read-only platform access granted by a company admin. |
| DPDP | India's Digital Personal Data Protection Act and Rules 2025. |
| CERT-In | Indian computer emergency team; incident reporting within 6 hours. |
| Entitlement | What a company may use: plan defaults plus add-ons plus the super admin's per-module and per-feature switches. |
| Assignable roles | The roles a creator (admin or manager) is allowed to give to a new user. |
| Business profile | Setup choice (Store, Wholesale, Trader, Farm…) that presets features, documents, reminders and legal checks. |
| ADD | Average daily demand used for reorder maths. |
| Reorder point (ROP) | Stock level at which a new order should be placed: demand during lead time plus safety stock. |
| Safety stock | Extra stock held for demand and supplier variation. |
| Lead time | Days from ordering to receiving. |
| MOQ, pack size | Minimum order quantity; the multiple a supplier sells in. |
| Days of cover | Available stock divided by daily demand. |
| FEFO / FIFO | First-expiry-first-out / first-in-first-out picking. |
| Landed cost | Purchase cost plus freight, duty and handling. |
| Disposition | What happens to a returned item: restock, damaged, quarantine, supplier return, scrap. |
| Lot | A harvest or arrival batch with crop, grade and weight data. |
| Kata slip | Weighment slip. |
| Account sale (patti) | Statement from a commission agent to a farmer: sales less commission and charges. |
| Bill of supply | Document with no tax line, used for exempt supplies and composition sellers (wording by status). |
| Delivery challan | Document for goods moved without a sale. |
| MRP | Maximum retail price printed on packaged goods. |
| TDAL | Tenant Data Access Layer: the only code allowed to talk to the database driver; adds `companyId` to every operation and validates results. |
| Company ID (`companyId`) | Immutable identifier stamped on every company-owned document and used to prefix indexes, cache keys and file paths. |
| Quarantine | Switch that blocks a company or route while keeping data untouched. |
| Canary company | Synthetic company with unique markers used to detect any cross-company leak. |
| Purge | Verified physical deletion of all of a company's data on exit. |
| Dedicated cell | A separate cluster (and keys, region) for one company that needs physical separation. |
| Change event | One audit entry per action: who, role, device, time, field-by-field before and after, reason, source. |
| Revision | A saved snapshot of an editable record after a change; what a revert restores. |
| Revert | Restoring an earlier version of an editable record as a new revision; history stays. |
| Reversal | Correcting a posted document with a linked document (cancel, credit note, reversing entry) instead of editing it. |
| Change set | The group of changes made by one action (import, bulk edit, revert). |
| Restore point | Automatic safety copy taken before a risky action. |
| Recycle bin | Deleted records kept for 90 days so they can be restored. |
| Restore as of a date | Wizard that sets many masters back to how they were at a chosen time, with a preview. |
