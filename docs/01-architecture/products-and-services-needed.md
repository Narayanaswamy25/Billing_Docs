# Products and Services Needed

Every outside product, service and tool the project needs, what we recommend, what else could be used, when it is needed, what drives its cost, and what is still undecided. Research date: 2026-10-31. **Prices are not stated** where no public price was found; the cost model in section 8 is blank until quotes arrive. Items marked *to verify* were not confirmed by the research.

Status key: **Decided** (owner), **Recommend** (my proposal, needs a yes), **Open** (needs a choice), **Later** (not release 1).

## 1. Release 1 shopping list (the short version)
| # | Need | Recommendation | Status |
| --- | --- | --- | --- |
| 1 | Database | MongoDB Atlas | Decided (D-012) |
| 2 | Cloud and region | AWS, Mumbai (ap-south-1) primary, Hyderabad (ap-south-2) as in-country second region | Recommend (D-057) |
| 3 | App hosting | Managed containers (AWS ECS on Fargate), two or more copies behind a load balancer | Recommend (D-059) |
| 4 | Redis | Managed Redis (AWS ElastiCache) for queues, caches, rate limits | Recommend |
| 5 | Files and backups | Amazon S3 with versioning and object lock; separate backup account | Recommend |
| 6 | Keys and secrets | AWS KMS (root key), a key service we write, AWS Secrets Manager | Recommend |
| 7 | Email | Amazon SES with SPF, DKIM and DMARC | Recommend |
| 8 | Monitoring and errors | OpenTelemetry with Grafana Cloud (or Datadog); Sentry with data scrubbing | Recommend |
| 9 | Source control, CI, scanning | GitHub, GitHub Actions, Dependabot, CodeQL, secret scanning, container scanning | Recommend |
| 10 | Infrastructure as code | Terraform | Recommend |
| 11 | E-invoice and e-way bill | A GSP with a sandbox (shortlist three, get quotes) | Route decided, vendor Open (D-013, D-060) |
| 12 | WhatsApp messages | WhatsApp Business API through a provider (BSP); utility messages only in release 1 | Recommend (D-058) |
| 13 | SMS | A DLT-registered SMS provider; register early (takes weeks) | Recommend (D-058) |
| 14 | UPI at the counter | Show the company's own UPI QR and confirm manually; gateway later | Recommend (D-055) |
| 15 | Local database on devices | Drift (SQLite) with encryption; spike against the community Isar fork | Recommend (D-056) |
| 16 | Printing and scanning | ESC/POS thermal printers over USB, Bluetooth or LAN; scanner in keyboard mode | Recommend, needs device lab |
| 17 | Developer accounts | Google Play, Apple Developer, Windows code-signing certificate | Needed before pilots |
| 18 | Professionals | Tax adviser or CA, lawyer, penetration-test firm | Needed before launch |
| 19 | Domain, DNS, TLS, status page | AWS Route 53 or similar, ACM certificates, a hosted status page | Recommend |
| 20 | Support tooling | A helpdesk and a knowledge base | Later (before first customers) |

## 2. Core platform
| Need | Recommended | Alternatives | Why | Needed by | Cost driver | Notes and risks |
| --- | --- | --- | --- | --- | --- | --- |
| **Database** | **MongoDB Atlas** (D-012) | Self-managed MongoDB | Transactions, managed backup, encryption, private networking, no database administrator | Step 1 | Cluster tier, storage, backup retention, data transfer | Atlas lists **AWS Mumbai and Hyderabad**; Azure and Google India regions were **not confirmed**; features by tier (continuous backup, sharding, auditing, Queryable Encryption, customer-managed keys) **to verify** per tier before sizing |
| **Cloud and region** | **AWS Mumbai + Hyderabad** | Azure or Google Cloud (India availability for Atlas to verify) | Both Indian AWS regions are listed for Atlas, so a cross-region copy can stay in India; mature KMS, S3, SES, ECS | Step 0 | Compute, data transfer, KMS requests | D-014 stays Recommend until the owner agrees |
| **App hosting** | AWS ECS on Fargate (API, worker, key service) | Kubernetes (EKS), another PaaS | Fewer parts to run for a small team; autoscaling; no node management | Step 1 | vCPU and memory hours | Move to Kubernetes only if needed |
| **Load balancer, WAF, DDoS** | AWS ALB, AWS WAF, Shield Standard, CloudFront for static files | Cloudflare | Edge protection in one account | Step 1 | Requests, rules | Rate limits also in the app |
| **Redis** | Managed Redis (ElastiCache) | Upstash, self-hosted | Needed by queues (BullMQ), caches, rate limits | Step 1 | Node size | BullMQ needs Redis configured not to evict data; configure and test |
| **Object storage** | Amazon S3 with versioning, object lock, separate backup account and region copy | Other S3-compatible | Files, PDFs, exports, company backups | Step 1 | Storage, requests | Per-company keys on top (tenancy/03) |
| **Key management** | AWS KMS (HSM-backed) as root key; our key service | CloudHSM, Vault | One component calls KMS; per-company keys | Step 1 | Key count, requests | Customer-managed Atlas keys optional (tier) |
| **Secrets** | AWS Secrets Manager or SSM | Vault | Rotation, access control | Step 1 | Secret count | |
| **Email** | Amazon SES | Postmark, SendGrid | SPF, DKIM, DMARC; invitations, resets | Step 1 | Messages | Warm up sending reputation before launch |
| **Domain, DNS, certificates** | Route 53, ACM, registrar lock | Cloudflare DNS | Takeover protection | Step 1 | Minimal | Separate hosts for company and platform APIs |

## 3. Quality, delivery and observability
| Need | Recommended | Alternatives | Notes |
| --- | --- | --- | --- |
| Source control and CI | GitHub, GitHub Actions | GitLab | Branch protection, required reviews, code owners |
| Security scanning | Dependabot, CodeQL, secret scanning, Trivy (containers), SBOM generation | Snyk | Runs on every pull request (see 56) |
| Infrastructure as code | Terraform | Pulumi | Drift detection |
| Logs, metrics, traces | OpenTelemetry with Grafana Cloud | Datadog, self-hosted Prometheus and Loki | Scrub data; company ID only |
| Error tracking | Sentry (server and Flutter) | Rollbar | Configure scrubbing; no names, phones, GSTINs, amounts |
| Uptime and status page | Hosted status page plus synthetic checks | Self-hosted | Sign-in and a sandbox post every few minutes |
| Load testing | k6 | Gatling | Mixed-company load (54) |
| Test devices | A device lab: low-end Android, tablet, Windows PC with printer and scanner, Mac | — | Budget the hardware |
| Design | Figma for tokens and wireframes | Penpot | Open item in 08 |

## 4. India-specific services
| Need | Options found | Recommendation | Notes |
| --- | --- | --- | --- |
| **E-invoice and e-way bill** | **NIC IRP** (government, free, direct); **private IRPs** empanelled by the GST network: ClearTax, Cygnet, EY, IRIS; **GSPs** such as Masters India and ClearTax that sit between you and an IRP | Use a **GSP with a sandbox** for release 1; keep the adapter swappable (EInvoicePort); consider direct NIC later | **No public per-invoice price was found**; request quotes. Vendor guides report that **multi-factor authentication became mandatory for e-invoice and e-way bill users from 1 April 2026** (*verify*); sandbox at einv-apisandbox.nic.in per vendor guides |
| GST data (GSTR-2B download, matching) | Same GSPs | Include in the GSP quote | Filing itself is out of release 1 |
| GSTIN verification | Portal lookup through the GSP | Optional Verify button | |
| **WhatsApp** | WhatsApp Business API through a provider (BSP); Meta charges per delivered template message | Start with **utility** messages (invoice share, reminders); marketing later | Reported 2026 rates (secondary sources, **verify with Meta**): utility about ₹0.115, marketing about ₹0.86 per message; providers add **10 to 30%** or a few paise per message; plus **18% GST**. Examples of providers to evaluate (not verified): Gupshup, Interakt, WATI, AiSensy, Twilio |
| **SMS** | DLT-registered SMS providers | Register the company on a telecom operator's **DLT portal early**: reported **2 to 4 weeks** for the first set of headers and templates | Transactional SMS reported about ₹0.15 to ₹0.25 per message (*verify*); examples to evaluate: MSG91, Kaleyra, Gupshup |
| **Payments (UPI, cards)** | Gateways: Razorpay, Cashfree, PayU, PhonePe PG, Paytm | **Release 1: no gateway needed at the counter** (see D-055). Add a gateway for payment links and automatic UPI confirmation later | Fees are unsettled: gateways quote about **2%** platform fees (Razorpay shows zero MDR plus about 2%; Cashfree about 1.95%); several reports say a **network UPI merchant fee (0.4% on UPI above ₹2,000) starts 15 October 2026**, while another says it is still a proposal. **Treat UPI cost as unknown until confirmed** |
| Subscription billing for **our** customers | Gateway subscriptions or invoicing with e-invoice | Decide with the pricing model | Our own invoices to companies must be GST invoices |
| Bank feeds | Account Aggregator or bank APIs; CSV statement import | Release 1: CSV import only | Later: Account Aggregator |
| Penetration test | A reputable security firm; check for CERT-In empanelled auditors (*verify*) | Book before launch | Annual |
| Tax adviser, CA | Retained individual or firm | Needed now | Confirms every Verify row |
| Lawyer | Technology and privacy counsel | Needed before launch | Terms, privacy notice, processing agreement |
| Cyber insurance | Insurer | Consider before launch | |
| Company registrations | GST registration for the provider, Udyam if eligible, trademark | Before first invoice | |

## 5. Client and device stack
| Need | Recommended | Alternatives | Notes |
| --- | --- | --- | --- |
| Framework | Flutter (Android, Windows, macOS, web) | — | Decided (D-001) |
| State, routing | `flutter_bloc`, `go_router` | Riverpod | |
| **Local database** | **Drift (SQLite) with SQLCipher-style encryption** (to verify the packaging for each platform) | Isar community fork; Hive | The **original Isar is reported to be no longer maintained** (a community fork exists, claims active maintenance; verify); Drift is active but largely a one-maintainer project; spike both on the Counter, decide at the prototype (D-056) |
| Secure storage | `flutter_secure_storage` | Platform APIs | Tokens, database key |
| HTTP, models | `dio` plus generated client from OpenAPI | `http` | |
| Printing | A `PrinterPort` abstraction with ESC/POS drivers per platform; PDF printing for A4 and A5 | Native plugins | **Not researched; needs a device spike** (Windows, Android, Bluetooth, LAN) |
| Scanning | Barcode scanners in **keyboard mode** (USB or Bluetooth) need no package; camera scanning for phones | `mobile_scanner` or similar | Keep focus rules in the Counter |
| Crash reporting | Sentry Flutter with scrubbing | Firebase Crashlytics | No personal data |
| Distribution | Google Play, Microsoft Store or signed installer, Mac notarisation | Direct APK | Accounts and certificates needed before pilots |
| Updates | Store updates plus minimum-version enforcement (see API versioning) | In-app updater | |

## 6. Hardware (what the shop needs)
| Item | Requirement | Notes |
| --- | --- | --- |
| Thermal receipt printer | 58 mm and 80 mm, ESC/POS compatible, USB, Bluetooth or LAN, auto-cutter, cash-drawer port | Publish a **tested-printer list** after the device lab; do not promise all brands |
| Cash drawer | Opens through the printer (RJ11) | Manager-only manual open |
| Barcode scanner | USB or Bluetooth keyboard mode | 1D and 2D |
| Label printer | Optional (barcode labels) | Later |
| Weighing scale | Serial or USB; **verified and stamped** for trade use (legal-metrology rules, to verify) | Reminder before verification expires |
| Device | Low-end Android phone, tablet, Windows PC or Mac | Performance targets in 31 |
| Network | Stable Wi-Fi; mobile data as a fallback | Counter works offline for 24 hours and 50 bills |
| Power backup | UPS for the counter PC and printer | Customer advice |

## 7. Business and operations products
| Need | Recommendation |
| --- | --- |
| Helpdesk and knowledge base | A hosted helpdesk (Freshdesk, Zoho Desk or similar) before the first customers |
| Product analytics (counts only, no content) | A privacy-friendly tool or our own counters; never company data (D-042) |
| Internal accounting for the provider | An accounting package (Zoho Books or similar) |
| Customer onboarding | Templates, sample data, video guides (later) |
| Documentation hosting | These docs plus a public help site |
| Incident and on-call | A paging tool (Opsgenie, PagerDuty or similar) |
| Status communication | Status page and email list |

## 8. Cost model (to fill in)
Cost is driven per company and per cell. Fill with quotes; do not price the product until it is filled.
| Driver | Unit | Who gives the price |
| --- | --- | --- |
| Atlas cluster | Tier per cell per month | MongoDB Atlas calculator |
| Atlas storage and backup | GB per month | Same |
| Compute (API, workers, key service) | vCPU and GB hours | AWS |
| Redis | Node per month | AWS |
| S3 storage, backups, exports | GB per month, requests | AWS |
| KMS | Keys and requests | AWS |
| Email | Messages | AWS SES |
| WhatsApp | Delivered templates by category + provider markup + GST | Provider and Meta |
| SMS | Messages + DLT setup | Provider |
| GSP | Per invoice or monthly plan | Provider quotes |
| UPI or gateway | Percentage per payment (if used) | Gateway |
| Monitoring and errors | Hosts, events | Vendor |
| Support staff, on-call | People | Owner |
| Professionals | Tax adviser, lawyer, pen test | Quotes |
Per-company cost = (share of cell) + (storage and backups) + (messages) + (GSP invoices) + (support). Revision and backup retention settings change storage cost, so plan tiers should cap them.

## 9. Products we should **not** add (for now)
A separate search engine (Elasticsearch), Kafka or a message bus beyond BullMQ, Kubernetes, microservices beyond the key service, GraphQL, a data warehouse, machine-learning platforms, blockchain, a second database technology. Each adds cost and failure points; the design does not need them for release 1.

## 10. Selection criteria (use when comparing vendors)
- **GSP:** sandbox, e-invoice, e-way bill and GSTR data coverage; API stability and versioning; uptime commitment; support hours; per-invoice and setup pricing; data-processing contract; India hosting; exit and data portability; multi-factor handling.
- **WhatsApp provider:** template approval time, webhooks for delivery, markup, support, number onboarding, GST invoicing, exit.
- **SMS provider:** DLT support, delivery reports, speed, price, fallback routes.
- **Payment gateway:** UPI QR with payment confirmation callbacks, settlement time, fees including UPI changes, refunds, subscriptions, sandbox, support.
- **Hosting and Atlas tier:** India region for the chosen tier, backup and sharding features, support plan, price.
- **Monitoring:** scrubbing controls, cost at our volume, alert quality.

## 11. What the research did **not** confirm
Per-invoice prices of any e-invoice provider · Azure and Google India regions for Atlas · Atlas features per tier · current UPI fee status · ESC/POS printer and Flutter desktop plugins · camera-scanner packages · DLT timelines and message prices beyond secondary sources · CERT-In empanelled auditor list. Each is an action item before a purchase.

## 12. New decisions (see the decision log)
D-055 UPI at the counter without a gateway in release 1 · D-056 Drift for the local database, spike against the Isar fork · D-057 AWS Mumbai with Hyderabad as the second region · D-058 WhatsApp and SMS: utility messages only in release 1, start DLT early · D-059 managed containers instead of Kubernetes · D-060 e-invoice through a GSP with a sandbox, three quotes.
