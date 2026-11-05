# Third-Party Integrations

Every outside service is a **port** (small interface) with one **adapter** per provider, so changing provider means writing an adapter, not touching modules.

| Service | Direction | Notes | Failure handling |
| --- | --- | --- | --- |
| GSP (GST Suvidha Provider) for e-invoice, e-way bill and GST data | Out, replies in | Accepted route (D-013). Register the chosen GSP on the e-invoice portal during setup; credentials and base URL come from the provider; test first in the portal's sandbox (einv-apisandbox.nic.in per vendor guides). **Vendor not chosen** | Pending status, retry with growing delay, dead-letter queue visible to platform staff, circuit breaker |
| Payment gateway | Out, callbacks in | Payment links, QR, settlement status; card data never stored | Signed callbacks, idempotent handling |
| WhatsApp, SMS, email | Out, delivery reports in | Approved templates, per-company sender, credits counted per plan | Retry, log every send on the record |
| Online store, aggregators | Both | **Out of release 1** | — |
| Bank feed, accounting export | In, out | Consent expiry tracked | Retry, re-consent notice |
| Customer systems | Out | Signed webhooks, public API | Retried, switched off after repeated failures with notice to admin |
| Hardware | Local | Thermal printer (ESC/POS), scanner, cash drawer via printer, optional weighing scale, card machine via provider | Bill saved before printing; "Printer not reached" with Try again and Choose another printer |

## Rules for every adapter
- Credentials from the key vault by reference, per company; never in logs.
- Timeout on every call, retry policy, circuit breaker, per-provider queue and concurrency limit (a slow GSP cannot starve payment callbacks).
- Requests and replies logged under the company (stamped with `companyId`) with secrets removed.
- Incoming callbacks: signature checked, answered quickly, work handed to a queue; callback address contains an unguessable per-company, per-provider ID.
- Outbound calls only to an allow-listed host set; internal and metadata addresses blocked.

## GST specifics to confirm with the tax advisor (all as dated settings)
| Item | Current understanding | Status |
| --- | --- | --- |
| Who needs e-invoices | Current guidance: B2B, SEZ and export invoices, not retail B2C counter bills | **Verify** with the tax advisor |
| E-invoice mandate threshold | Aggregate turnover above ₹5 crore in any year since FY 2017-18 | Sources agree; confirm |
| 30-day reporting limit | Taxpayers with turnover of ₹10 crore or more cannot report an invoice more than 30 days after its date (since 1 Apr 2025); some sources say ₹5 crore | **Verify** |
| E-invoice cancellation | Within 24 hours of IRN generation; afterwards use a credit note; one source adds not if an e-way bill is linked | **Verify** |
| Invoice number | Max 16 characters, letters/digits/`/`/`-` | GST rule; keep as a setting-checked limit |
| Posting while the portal is down | Allowed as Pending; some businesses must hold the IRN before goods move | D-019 |
| Return filing | Prepared and exported; submission later | Out of release 1 |

## GSP route notes (research 2026-10-31, vendor and consultant sources, not official)
- Smaller taxpayers normally use a GSP; direct integration with the e-invoice portal costs more to set up and maintain.
- A taxpayer already registered with a GSP for e-way bills need not register again for e-invoice.
- One source cites a limit of about 25 concurrent sessions per GSTIN per portal: confirm in the sandbox before sizing the worker concurrency per company.
- Keep the adapter behind the `EInvoicePort` interface so switching GSP vendor, or going direct later, means a new adapter only.
- Route options found: NIC IRP (government, free, direct), private IRPs empanelled by the GST network (ClearTax, Cygnet, EY, IRIS), and GSPs (for example Masters India and ClearTax). No per-invoice price is public; request quotes. Vendor guides report multi-factor authentication mandatory for e-invoice and e-way bill users from 1 April 2026 (verify). Choice criteria and the shortlist process: [products-and-services-needed.md](../01-architecture/products-and-services-needed.md) (D-060).
