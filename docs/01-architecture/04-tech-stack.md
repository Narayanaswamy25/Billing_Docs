# Tech Stack and Dependencies

Chosen by the owner on 2026-10-31: **Flutter** clients (D-001), **Node.js + Express + MongoDB Atlas** backend (D-021, D-012). Versions are not pinned yet: pin each in the lock files from what is installable. "Verify" marks claims not yet confirmed (see [REVIEW-FINDINGS.md](../REVIEW-FINDINGS.md) F-19).

## Backend
| Layer | Choice | Reason | Note |
| --- | --- | --- | --- |
| Runtime | Node.js 26 | Scheduled to enter Active LTS on 28 Oct 2026, end of life Apr 2029; run only Active or Maintenance LTS | **Verify** the promotion on nodejs.org; pin the major in the image |
| Language | TypeScript, strict | Large codebase over years | |
| Framework | Express (current stable major) | Owner's choice; small and well known | Middleware order is part of the design; we write the module loader, dependency wiring and `defineRoute` helper ourselves (see [10-backend-architecture.md](10-backend-architecture.md)) |
| API | REST + JSON, OpenAPI generated from the same schemas that validate input, signed webhooks | Easy to test and give to partners | |
| Database | **MongoDB Atlas** replica-set clusters | Multi-document transactions for posting; managed backups, encryption, private networking | Use the MongoDB version Atlas offers for the chosen tier; MongoDB 9.0 availability to confirm |
| DB access | Official Node driver, typed repositories, `$jsonSchema` validators | A mandatory tenant-scoped access layer must own every query; ODMs hide queries | No Prisma/ODM (D-002) |
| Cache, queues, locks | Redis (managed), BullMQ | Permission caches, limits, job queues | |
| Files | S3-compatible storage, signed links | Keeps files out of the database | |
| Auth | Argon2id, short-lived access tokens, rotating refresh tokens with reuse detection, TOTP | Reviewable standards | |
| PDF / Excel | HTML templates → PDF via headless browser pool; streaming Excel | Layouts are ordinary HTML | |
| Crypto | Node `crypto` on OpenSSL or libsodium; AES-256-GCM | No home-made crypto | |
| Keys | Cloud KMS (same cloud as Atlas) behind a key service | One component calls the KMS | |
| Observability | OpenTelemetry, Prometheus, structured logs, error tracker | Trace from tap to portal | |
| Delivery | Docker, managed containers or Kubernetes, infrastructure as code, CI | Repeatable environments | Cloud and region open (D-014) |
| Testing | Unit, API tests on a real replica set, two-company isolation suite, load tests | Posting and tenancy bugs only show on a real replica set | |

## Clients (Flutter)
| Layer | Choice | Reason |
| --- | --- | --- |
| Framework | Flutter (Android, Windows, macOS, web) | One code base for Console, Workspace, Counter; hardware reach |
| State | Bloc / Cubit (`SessionCubit` for permissions) | UI spec examples |
| Routing | `go_router`, redirect to `/not-available` without permission | UI spec |
| Offline store | **Drift (SQLite) with encryption** proposed (D-056); the original Isar is reported unmaintained (a community fork exists), so spike both | Offline bills, drafts, masters copy |
| Models | Dart classes generated from the OpenAPI file | One source of truth for fields |
| Packages | `app_shell`, `ui_kit`, `company_workspace`, per-module feature packages | UI spec |
| Tokens | One shared design-token package (colours, type, spacing, corners, light/dark) | UI spec |
| Printing | Native/printer plugins for ESC/POS thermal, PDF for A4/A5 | No separate print agent |
| Token storage | Secure storage (Keychain/Keystore/DPAPI) | Not browser cookies; Flutter web builds use HttpOnly cookies + CSRF |

The original architecture's React section (React + Vite, TanStack Query, Zod shared with the browser, service worker + IndexedDB, local print agent) is **superseded**.

All outside products, vendors, options and cost drivers: [products-and-services-needed.md](products-and-services-needed.md).

## Why each choice stays reviewable
Swap a tool freely but keep the reason in mind and record the change in the decision log.
