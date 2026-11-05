# Architecture Overview

## Shape
A modular monolith (Node.js + Express) started in two modes, API and worker, plus three Flutter clients. All companies share MongoDB collections inside a cell (an Atlas cluster); every document carries its `companyId` and all access goes through one tenant data access layer; a separate platform database is the registry (D-039).

```
Flutter apps (Console · Workspace · Counter)
        │ HTTPS, bearer token
   Edge (TLS, firewall, rate limit, DDoS)
        │
   API (Express)  ──► Tenant context ──► Data access layer ──► shared collections in a cell
        │                                   (Cell A, B, C… clusters; many companies per cell)
   outbox ► Redis queues ► Workers ──► GSP, payment gateway, WhatsApp/SMS/email,
        │                              online store, bank feed
   Platform DB (registry, keys, catalog, platform audit)
   Object storage (files, PDFs, backups) · Key service ► KMS · Monitoring
```

## Components
| Component | Job |
| --- | --- |
| API | Ordered middleware (token, tenant, module on, permission, schema validation), route handlers → one use case each |
| Worker | Same code, queue consumers, schedulers, outbox relay |
| Tenant context and data access layer | Company comes only from the signed token; the data access layer is the only code that touches the database, adds `companyId` to every operation and validates results; refuses suspended, retired or quarantined companies |
| Platform database | companies, cells, tenants (company, cell, status, key references), company_modules, identities, memberships, plans, agreements, company_entitlements, platform_staff, platform_audit, support_grants, subscriptions, usage_meters |
| Key service | Only caller of the KMS; unwraps company data keys; mutual TLS |
| Redis | Permission and company caches, rate limits, BullMQ queues, change broadcasts |
| Object storage | Attachments, PDFs, import files, backups, exports |
| Observability | OpenTelemetry traces, Prometheus metrics, JSON logs with company ID and trace ID |

## Two sign-in realms
Platform realm (super admin, staff; separate address, keys and audience, second step always required) and company realm (admin and users). A platform token is useless on a company API and the reverse.

## Request path
Edge → middleware (request ID, trace, headers) → middleware (authenticate, tenant, module on, permission) → idempotency, timeout and metrics middleware → schema validation (rejects unknown fields) → route handler → one use case (unit of work, repositories, pure domain, outbox) → response middleware (store idempotent result, record duration) → error handler (standard error shape).

## Key flows
Invoice posting and e-invoice, offline POS sync, module switching, adding a user: see [19-workflows.md](../05-logic/19-workflows.md). Payroll run is deferred (D-007).

## Backend modules
Company modules: kernel, identity, masters, tax, sales, pos, purchases, inventory, accounting, promotions, crm, hr, india, compliance, messaging, integrations, packs, workflow, reporting, privacy. Platform modules: tenancy, catalog, platform-access, platform-billing, support, platform-audit. Rules: public interface and events only across modules; dependencies point inward; effects go out as outbox events; each module has a manifest. Detail: [10-backend-architecture.md](10-backend-architecture.md).

## UI module → backend module map
| UI icon | Backend modules behind it |
| --- | --- |
| Dashboard | cards from every module via manifests |
| Sales | sales, pos, promotions, crm, packs |
| Purchases | purchases |
| Stock | inventory |
| Accounts | accounting, india (tax views), compliance (e-invoice filing view) |
| HR | hr |
| People | identity, workflow (Activity), kernel audit |
| Reports | reporting |
| Settings | kernel settings, tax setup, numbering, print layouts, integrations |
| Console | tenancy, catalog, platform-access, platform-billing, support, platform-audit |

The e-invoice send button lives on the invoice in Sales, so a Billing officer needs no Accounts icon for it.

## Deployment unit
The unit of capacity is the cell. Add stateless API and worker copies for load; add cells for companies. See [54-scalability-plan.md](54-scalability-plan.md) and [25-deployment-devops.md](../07-operations/25-deployment-devops.md).
