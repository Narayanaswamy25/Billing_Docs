# Isolation Tests and Assurance

The shared database is only acceptable if isolation is **proved continuously**. A release does not ship unless every item in section 6 passes.

## 1. Unit tests for the access layer (TDAL)
One test per operator and per rule in [01-isolation-architecture.md](01-isolation-architecture.md): `companyId` injected on every read and write; a caller filter mentioning `companyId` rejected; a top-level `$or` cannot escape; `companyId` cannot be changed or upserted; `deleteMany` without a condition refused; `$lookup`, `$graphLookup`, `$unionWith`, `$merge`, `$out`, `$where`, `$function` handled as specified; bulk writes; transactions; result validator throws on a foreign document; limits applied.

## 2. Generated isolation tests per repository method
A registry lists every repository method. For each, a generated test seeds **two companies** with similar data (same names, same invoice numbers, same barcodes), calls the method as company A with inputs designed to hit B (B's IDs, B's numbers, wildcard filters), and asserts that **zero** B documents are returned or changed. A new repository method without registration fails the build.

## 3. Property and fuzz tests
Random filters, sorts, projections, cursors and aggregation stages (from the allowed builders and from deliberately hostile objects such as `{"$ne": null}`, `{"$where": …}`, nested `$or`) run as company A against a database seeded with companies A to E; the property is "every result and every write touches only A". Runs on every pull request touching the TDAL or a repository, and nightly with a larger budget.

## 4. Canary markers
Every field in company B's seed carries a unique marker (for example `CANARY-B-7f3a…`). While running **any** test as company A (API calls, exports, reports, search, notifications, emails, logs, error bodies, Redis dumps, queue payloads, object-storage listings, generated PDFs), the harness scans all outputs and storage for B's markers. Any hit fails the test. The same probe runs against production with two synthetic canary companies every minute (alert on any hit).

## 5. Static and structural checks (CI, every pull request)
| Check | Rule |
| --- | --- |
| Driver import | Only `packages/data-access` may import the MongoDB driver; any other import fails the build |
| Raw collection | No `.collection(`, `db.` or `MongoClient` outside the TDAL |
| Operators | No `$where`, `$function`, `$accumulator`, `$out`, `$merge`, `$unionWith` in repositories |
| Company input | No read of `companyId`, `tenantId`, `x-company` from `req.body`, `req.query`, `req.params` or headers |
| Routes | Every `defineRoute` has a permission and a schema; response schemas exclude `companyId`-bearing foreign fields |
| Indexes | Every index on a company collection starts with `companyId` (checked from the migration files) |
| Collections | Every company collection has the `$jsonSchema` validator requiring `companyId` |
| Cache keys | Helper-only construction with the `t:{companyId}:` prefix |
| Jobs | Every job handler calls `withTenant(payload.companyId, …)` |
| Logs | Logger wrapper rejects objects that contain known sensitive field names |
| Dependencies | Scans for vulnerable packages and unexpected install scripts |

## 6. Release gate (all must pass)
- [ ] TDAL unit and fuzz tests (sections 1 and 3)
- [ ] Generated per-repository isolation tests (section 2), coverage 100% of registered methods
- [ ] Canary scan clean across API, exports, logs, Redis, queues and storage (section 4)
- [ ] Static checks (section 5)
- [ ] Permission matrix and access tests T-01 to T-25
- [ ] Encryption tests (03, section 8)
- [ ] Migration run twice on the canary data set; isolation suite re-run after migration
- [ ] Load test with several companies running at the same time confirming fairness limits hold
- [ ] No open High or Critical security finding

## 7. Runtime assurance (production)
| Monitor | Alert |
| --- | --- |
| Result-validator violations | Any: page and auto-quarantine the route |
| TDAL refusals (attempted foreign access) | Spike or any from internal code paths |
| Canary probe hit | Any: page |
| Per-company 404 rate and ID-scan behaviour | Pattern alert |
| Key service: unwrap with mismatched company | Any: page |
| Purge and migration runner actions outside a ticket | Any |
| Break-glass sessions | Every one reviewed |
| Heaviest company share of cell resources | Threshold alert |

## 8. Human assurance
- **Code ownership:** TDAL, key service, tenant resolver and migration runner have named security owners and two-reviewer rules.
- **Threat model** updated on every architectural change; **tenant-isolation design review** before adding any feature that reads more than one record family (exports, reports, search, imports).
- **Outside penetration test** before launch and yearly, with explicit scenarios: ID tampering across companies (IDOR), mass assignment of `companyId`, NoSQL operator injection in filters and sorts, aggregation injection through report parameters, cursor and export-link tampering, webhook replay and cross-company secrets, timing and error side channels, cache poisoning, queue payload tampering, file path traversal, signed-URL reuse.
- **Red-team exercise** once a year with a seeded isolation bug to test detection and the exposure playbook.
- **Evidence pack** for customers and auditors: architecture summary, test results, pen-test summary, canary history, incident log (no data), sub-processor list.

## 9. Metrics reported monthly
Isolation violations (target 0) · time to detect a seeded bug · repository-method coverage (100%) · mean time to quarantine · canary uptime · number of break-glass sessions · open security findings by age.

## 10. Scenarios that must be tested by name
1. Company A's admin opens B's invoice ID in a URL → 404.
2. A user changes `companyId` in a request body → ignored or rejected, never used.
3. Report parameters carry a `$lookup` or `$where` string → rejected.
4. A cursor from company A is used by company B → rejected.
5. An export link for A is opened while signed in as B → refused.
6. A queue job for A is replayed with B's ID → context check fails.
7. Two companies create the same invoice number and barcode → both succeed independently; unique indexes do not collide.
8. A support grant for A is used to read B → denied and alarmed.
9. A company with the same phone number as a person in another company → different blind indexes; no cross-company link.
10. A deleted company's ID is reused in a request → refused; counts zero after purge.
