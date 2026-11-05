# Backend Architecture

## Layers (ports and adapters)
| Layer | Folder | Contains | May import | Must not |
| --- | --- | --- | --- | --- |
| Entry points | `http/`, `jobs/`, `events/` | Controllers, queue consumers, schedules, event handlers | application, contracts | Make business decisions or touch `data/` |
| Application | `application/` | Use cases (verb phrases: PostInvoice), unit of work, port interfaces | domain, ports, kernel | Use HTTP or MongoDB types |
| Domain | `domain/` | Entities, value objects, pure functions, domain errors, event types | other domain files, money library | Do I/O, read the clock, import anything else |
| Infrastructure | `data/`, adapters | Repositories, provider adapters, cache | the ports they implement, driver libraries | Hold a business rule |
| Kernel | `kernel` package | Request context, audit, numbering, outbox, settings | std libs | Grow into a dump; additions need a decision record |

## Module layout
```
<module>/
  manifest.ts      routes, permissions, menu, events, jobs, migrations
  http/ application/ domain/ data/ events/ jobs/ migrations/ tests/
```
Each module declares itself in its manifest; the platform builds the running app from manifests, and switching a module off removes everything its manifest declares. The same manifest feeds the client rail, access-editor rows, dashboard cards and role tables.

## Rules between modules
1. Public service interface and events only; no reading another module's collections.
2. Dependencies point inward, never in a circle; a build rule fails wrong imports.
3. Side effects go out as events written to the outbox in the same transaction. Only the posting chain (invoice → stock → accounts) calls across modules synchronously, inside one transaction.
4. Industry packs depend on core; core never depends on a pack.

## Company rules as strategies
Each choosable company rule is an interface with one class per option; a rule resolver reads settings once per request.
| Rule | Interface | Options |
| --- | --- | --- |
| Stock valuation | CostingStrategy | WeightedAverage, Fifo, StandardCost |
| Tax mode, rounding | PriceModeCalculator, RoundingPolicy | Inclusive, Exclusive; per line or per total |
| Negative stock | StockAvailabilityPolicy | Block, Warn, Allow |
| Discounts | DiscountResolver | BestForCustomer, PriorityOrder, StackAll, ManualOnly |
| Posted documents | PostedDocumentPolicy | ReversalOnly (default), EditWithinWindow, CancelWithApproval |
| Credit | CreditPolicy | Block, Warn, Ignore |
| Accounting | PostingMode | Automatic, SemiAutomatic, Manual |
Numbering, backdating, below-cost sales, batch picking, reservation, over-receipt, price change and cash difference follow the same pattern.

## Code rules
Business names; branded ID types; function < 40 lines, file < 300, ≤ 7 constructor dependencies, complexity ≤ 10; no `any`; guard clauses; statuses as unions with exhaustive switch; money is a value type over Decimal128; time from an injected `Clock`; domain errors with stable codes; one public entry file per module; comments say why; tests named as sentences.

## Runtime choices
One MongoClient per cell per process (never per request or company); a time limit on every query; projections always; keyset paging; no joins on hot paths; bulk writes; transactions short (aim well under 1 s; default limit about 1 minute); majority write concern for postings; heavy work in workers or worker threads; Argon2 asynchronous; graceful shutdown; typed validated config at start.

## Express structure (D-021)
Express has no built-in modules, dependency injection or guards, so these three things are ours and are tested like any other code.

**1. Middleware order (fixed; a test asserts it).**
`requestContext` (request ID, trace, security headers) → `authenticate` (verify token, session not revoked) → `tenantResolver` (company from token, refuses suspended or retired) → `moduleGate` (module and feature on) → `permissionGate` (verb on `<module>.<screen>`, scope filter attached to the request) → `idempotency` (posting routes) → `timeout` → `validate` (schema names every field; unknown fields rejected) → route handler → response middleware (store idempotent result, record duration) → `errorHandler` (standard error shape, request ID). Failure at any step ends the request and fails closed.

**2. `defineRoute` helper.** Every route is declared as
```ts
defineRoute({
  method: 'post', path: '/sales/invoices/:id/post',
  permission: 'sales.invoices.post',   // required: a route without it does not compile
  input: PostInvoiceInput,            // schema; also feeds OpenAPI
  idempotent: true,
  handler: ({ params, ctx }) => postInvoice.execute({ invoiceId: params.id }, ctx),
});
```
A registry test lists every registered route and fails if any lacks a permission, and the same registry builds the OpenAPI file and the permission list sent to the Flutter manifest.

**3. Composition root.** One file per app (`apps/api/src/compose.ts`, `apps/worker/src/compose.ts`) builds repositories, services and use cases with plain constructor injection; modules export a `register(container)` function from their manifest. A light container library may be used if the wiring grows, but domain code never imports it. Request-scoped values (company handle, user, clock) travel in `RequestContext`, never in globals.

**4. Module loader.** At start the app reads each module's `manifest.ts`, mounts its routes under its prefix only for companies where the module is on (checked per request by `moduleGate`), registers its queue handlers and migrations, and exposes its menu and permission entries to the client.

## Tenant data access layer (D-040)
Repositories never see the driver; they receive a `TenantCollection` from `packages/data-access` that adds `companyId`, forbids changing it, rewrites `$lookup`, rejects dangerous operators and validates every result. The `tenantResolver` middleware builds the context from the token; jobs rebuild it with `withTenant(companyId, fn)`. Full design: [06-security/tenancy/01-isolation-architecture.md](../06-security/tenancy/01-isolation-architecture.md).
