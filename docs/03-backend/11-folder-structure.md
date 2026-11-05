# Backend Folder and File Structure

One repository for backend, contracts and (if Flutter, D-001) a generated client folder; a change to a shared schema reaches API and clients in one commit.

```
billing/
  apps/
    api/                Express app: HTTP entry; builds itself from module manifests
    worker/             Same modules in worker mode: queues, schedulers, outbox relay
    key-service/        Only caller of the KMS
  packages/
    data-access/        Tenant data access layer: the only code that imports the database driver
    kernel/             Request context, permissions, audit, numbering, settings, outbox
    security/           Encryption envelopes, hashing, blind indexes, key-service client
    money/              Money value type (Decimal128), tax and rounding helpers
    config/             Typed settings validated at start
    observability/      Logging, metrics, tracing
    contracts/          Schemas, types, error codes, event types; source of the OpenAPI file
    modules/
      identity/ masters/ tax/ sales/ pos/ purchases/ inventory/ accounting/
      promotions/ crm/ hr/ india/ compliance/ messaging/ integrations/
      packs/ (retail, restaurant, services, wholesale)  workflow/ reporting/ privacy/
    platform/
      tenancy/ catalog/ platform-access/ platform-billing/ support/ platform-audit/
    testing/            Replica-set fixtures, two-company helpers, fake GSP
  infra/                Infrastructure code, deployment files, pipelines
  docs/                 These docs (plus runbooks)
  tools/                Module generator, migration runner (company batches), reviewed repair scripts
  security/             Threat models, policies, key procedures, incident runbooks
```

## Inside a module
```
sales/
  manifest.ts   http/   application/   domain/   data/   events/   jobs/   migrations/   tests/
```
Purpose of each: see [10-backend-architecture.md](../01-architecture/10-backend-architecture.md).

## Rules
- One public entry file per module; other modules import only from it.
- Lint blocks: cross-module internal imports, direct database imports outside `packages/data-access`, routes registered without a permission (the `defineRoute` helper requires one), `any`, the plain `Date` constructor outside the clock adapter.
- A module generator creates the standard folders, manifest, lint settings and a first test.
- Each module has an owner in the code-owners file.
- File names `kebab-case.ts`; classes PascalCase; use cases are verb phrases (`PostInvoice`); ID types branded.
