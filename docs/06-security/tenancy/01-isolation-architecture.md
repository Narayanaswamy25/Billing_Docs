# Isolation Architecture

Ten layers, each assumed able to fail, so the next catches it. Layers 1 to 3 are the core; 4 to 10 stop leaks through every other door.

## Layer 1. Tenant context (who is asking, for which company)
- The **company ID comes only from the signed access token**, verified by the `tenantResolver` middleware after `authenticate`. It is never read from a URL, body, header, query, cursor or file the caller supplies. Two sign-in realms already stop a platform token working on the company API.
- The resolved context (`companyId`, `userId`, `sessionId`, permission and entitlement versions, request ID) is placed in an `AsyncLocalStorage` store at the start of the request and is **read-only** for the rest of it. Code cannot "switch company" mid-request.
- The resolver loads the company's status from the platform registry; Suspended, Retired and **Quarantined** companies are refused before any data call.
- Background jobs and webhooks carry `companyId` in the payload, but a worker must rebuild the context through the same resolver (checking the company still exists and is active) before running any code. Platform jobs that touch many companies run **one company at a time**, each in its own context.
- If no context exists, every data call throws. There is no default company.

## Layer 2. The Tenant Data Access Layer (TDAL), the only door to the database
The TDAL is a small package (`packages/data-access`) that wraps the official driver. **Nothing else may import the driver** (a lint rule and a CI check enforce it, see 05). Each repository gets a `TenantCollection` from the TDAL, never a raw collection.

What the TDAL does on every call:
| Operation | Enforcement |
| --- | --- |
| `find`, `findOne`, `count`, `distinct` | Adds `companyId` to the filter; rejects a caller filter that mentions `companyId` or any operator that could widen it (`$or` at the top level is rewritten as `$and: [{companyId}, {$or…}]`) |
| `insert*` | Stamps `companyId` from the context; rejects a document that already contains a different one |
| `update*`, `replace*`, `findOneAndUpdate` | Adds `companyId` to the filter; **forbids any change to `companyId`** (`$set`, `$unset`, `$rename`, `$setOnInsert` on that field) and any `upsert` that could create a document for another company |
| `delete*` | Adds `companyId`; `deleteMany` with only `companyId` and no other condition is refused (except the purge job in platform context) |
| `bulkWrite` | Applies the same rules to each operation |
| `aggregate` | Prepends `{ $match: { companyId } }` as the first stage; **rejects** `$out`, `$merge`, `$unionWith`, `$function`, `$accumulator`, `$where`; rewrites every `$lookup` so the foreign collection is also matched on the same `companyId` (a `$lookup` without a tenant match cannot be expressed); `$graphLookup` is allowed only with `restrictSearchWithMatch` on `companyId` |
| Change streams | Filtered by `companyId` for company consumers; the outbox relay is a platform component that reads the outbox only and re-enters a context per event |
| Transactions | The session is created inside the context; all calls inside use the same `companyId` |
| Index hints and sorts | Allowed only from the repository's declared list so a caller cannot force a plan that skips the tenant prefix |
| Results | **Result validator:** every returned document's `companyId` must equal the context; a mismatch throws, writes a security event and raises an alert (it should never happen, see 05) |
| Limits | `maxTimeMS` on every call, maximum page size, projection required on lists |
| Operator allow-list | Only operators the repositories declare in their query builders are accepted; free-form filters from the API never reach the TDAL as raw objects |

Sketch:
```ts
// packages/data-access/src/tenant-collection.ts
export class TenantCollection<T extends { companyId: CompanyId }> {
  constructor(private readonly col: Collection<T>, private readonly ctx: TenantContext) {}

  find(filter: SafeFilter<T>, opts: ReadOptions) {
    assertNoCompanyId(filter);                         // caller may not mention it
    const scoped = { $and: [{ companyId: this.ctx.companyId }, filter] };
    return this.col.find(scoped, { ...opts, maxTimeMS: opts.maxTimeMS ?? 2000 })
      .map((doc) => assertSameCompany(doc, this.ctx)); // result validator
  }
  // insertOne, updateOne, deleteOne, aggregate … apply the same rules
}
```
The TDAL is covered by its own exhaustive unit tests and a fuzz test (05) and has the highest code-review bar in the repository: two reviewers, a security owner among them.

## Layer 3. Schema, indexes and keys
- **Every company-owned collection** has `companyId` (ObjectId) as a required, **immutable** field enforced by `$jsonSchema` validation. Platform collections have their own rules.
- **Every index starts with `companyId`**, including unique ones (for example unique `{companyId, type, series, number}`), so uniqueness is per company and a query without the prefix cannot use an index (and fails the query-plan test).
- Company IDs are random ObjectIds. Business IDs (invoice numbers, SKUs) are only unique inside a company. Public identifiers shown in URLs or QR codes are opaque and never encode the company.
- **Shard key (if and when sharded):** `{ companyId: 1, _id: 1 }` keeps one company's data together; see layer 10.
- Collection budget: about 66 collections **in total** on a cluster (file-count limit no longer scales with companies, F-14 is closed by this design).

## Layer 4. Database credentials and network
- No human holds a standing database login to production. Engineers use a **break-glass** request: approved by a second owner, time-limited, to a read-only role, through a bastion, every command logged, and only via a tenant-filtered "support viewer" tool built on the TDAL (04).
- Service accounts are separate per role: API (read/write app collections only), worker, migration (schema only, release windows), backup, monitoring (no data read), analytics (none; see 02). No account can create users, change roles or read other databases.
- Atlas private endpoints, IP access list, TLS enforced, server-side scripting off (D-012, 23-security-policy). Atlas auditing records privileged access on tiers that include it (confirm tier).
- **MongoDB has no built-in row-level security by company.** Features such as views using the `USER_ROLES` variable (MongoDB 7.0+) filter by the *database role* of the connecting user, which would need one database role per company and one connection identity per company; that does not scale and is not used. The access layer above is therefore the real boundary, and the other nine layers exist because of that.

## Layer 5. Caches, queues, files, logs, search
| Component | Rule |
| --- | --- |
| Redis | Every key is prefixed `t:{companyId}:`; a helper builds keys from the context and refuses others; ACLs deny key patterns outside the prefix for company-scoped services; no cross-company keys; rate-limit buckets are per company and per user |
| Queues (BullMQ) | Payload holds `companyId` and record IDs, never documents; the worker rebuilds the context and re-validates company status; per-company concurrency limits; dead-letter entries show the company |
| Object storage | Path `tenants/{companyId}/{module}/{recordId}/…`; bucket policy denies listing; signed URLs are issued for one object and short lifetime; objects encrypted with that company's file key (03) |
| Search | No external search engine in release 1; local Counter index is on the device and per user; Atlas Search, if added later, must index with `companyId` as a filter field and the TDAL must inject it |
| Logs and traces | Include `companyId` (an identifier), never field values; no filters or payloads; sampling on error bodies is redacted; log access is per role and logged |
| Metrics | Labelled by company only where cardinality is bounded (top companies) or by hashed bucket; no customer data in labels |
| Exports and reports | Created inside the context; stored under the company path; signed link; expiry; logged |
| Webhooks and API keys | Per-company secrets; signed payloads; keys carry `companyId`; replay protected |
| Emails and messages | Built from one company's data; sender and template per company; no cross-company mailing lists |

## Layer 6. API surface
- **Cross-company lookups answer 404**, not 403, so another company's IDs cannot be tested for existence.
- No endpoint takes a company ID as input (except platform-realm routes, which are separate and permissioned).
- Cursors, export tokens and upload tokens are signed and bound to company, user and session.
- Mass assignment is blocked: request schemas name every field; `companyId`, `createdBy`, `version` and server-owned fields are never accepted.
- Errors never echo values from other documents; validation errors show only the caller's own input.
- Rate limits and query budgets per company and per user protect neighbours (layer 10).

## Layer 7. Platform operations and the Console
- The Console and platform APIs expose **entitlements, usage meters, health, support grants and logs**, never business records (D-023). There is no "browse company X's invoices" feature.
- Support access uses a grant: a scoped, read-only token that runs the same TDAL for that one company, with every read logged on both sides (06 of the access folder).
- Platform jobs (usage meters, billing, migrations) iterate companies sequentially in context; none runs a query without `companyId` except the purge and the outbox relay, which are reviewed, logged and tested components.
- Group reports for one owner who runs several companies are produced by **running each company's query under that company's context and the owner's membership, then merging in the application**; there is never a single query across companies (D-045).

## Layer 8. Detection
- **Result-validator violations** and any TDAL refusal are security events; a violation pages security and **automatically quarantines the affected route** (feature kill switch) pending review.
- **Canary companies** in production: two synthetic companies with unique marker strings in every field; a probe signs in as one and scans every response (API, exports, notifications, errors) for the other's markers every minute; any hit pages security.
- Behavioural alerts: one user requesting many unrelated record IDs, rapid cross-company token switching, unusual export volumes, spikes in 404s.
- Audit log is per company and append-only; platform audit records every Console action.

## Layer 9. Kill switches and containment
| Switch | Effect |
| --- | --- |
| Quarantine a company | Requests refused with the standard "not available" message; data untouched; admins informed |
| Disable a module or route globally | Feature flag; propagates in seconds |
| Global read-only | Blocks writes while an issue is investigated |
| Revoke a key | Per-company or per-purpose data key becomes unusable (03) |
| Rotate service credentials | Rolling, without downtime |

## Layer 10. Scale and fairness without weakening isolation
- **Start unsharded** on one Atlas cluster per cell. Shard only when measured size or load requires; verify the Atlas tier that supports sharded clusters and zone sharding before relying on it.
- **Shard key** `{ companyId: 1, _id: 1 }` (ranged) keeps a company's data together for fast, targeted queries and easy purge. A hashed company key spreads writes better but scatters a company's data; choose after a load test.
- **Zones:** map tiers of companies to dedicated shards (for example a "large" zone), so a heavy company does not share hardware with small ones. Zone moves trigger chunk migrations, so plan them.
- **Dedicated cell (D-043):** a company can be routed to its own cluster (and keys, region, backup policy) through the registry. The same code runs; only the `cell` mapping changes. This is the answer for a regulated or very large customer who needs physical separation.
- **Fairness:** per-company rate limits and concurrency caps, `maxTimeMS` everywhere, bulk work in workers at lower priority, reports on secondary nodes or analytics nodes, quotas on exports and imports, and a metric for the heaviest companies per cell.
- **Cell registry:** `tenants` (company → cell, status, quarantine flag, schema version, key references) in the platform database.

## What this design cannot do (stated plainly)
- It cannot make a coding mistake in the TDAL harmless; it makes such a mistake unlikely, detectable and containable.
- It does not hide business data fields from a person with full database access; that is why nobody has standing access, and why sensitive fields have their own per-company keys (03).
