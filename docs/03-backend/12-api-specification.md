# API Specification

The endpoint-by-endpoint reference is **generated** (OpenAPI from the code's schemas) once code exists; this file fixes the conventions and the first endpoints. Do not hand-maintain a second copy.

## Two APIs
`/api/v1` (company realm) and `/platform/v1` (platform realm) on a separate host, each with its own token audience and keys. The company comes from the signed token, never from a caller-typed field.

## Conventions
| Topic | Rule |
| --- | --- |
| Resources | Plural nouns (`/sales/invoices`); state changes are sub-resource actions (`POST …/post`, `…/cancel`) |
| Idempotency | Posting endpoints require `Idempotency-Key`; a repeated key returns the first result |
| Concurrency | Updates send the version in `If-Match`; mismatch → 409 with the current record |
| Lists | Cursor (keyset) paging, maximum page size, named filters, fixed sort fields; totals optional (expensive) |
| Slow work | 202 + a job resource to poll; notification when done |
| Files | Upload and download via short-lived signed links, not through the API process |
| Rate limits | Per user, company and address, limit headers on every reply; stricter for sign-in, exports, API keys |
| Versioning | Additive changes need no new version; breaking change → `/v2` with a deprecation period; CI compares the OpenAPI file with the last release |
| Live updates | Server-sent events for notifications, approvals, kitchen screens (via Redis fan-out) |
| Errors | Standard shape and codes in [20-error-handling.md](../05-logic/20-error-handling.md) |
| Permissions | Every route is registered with `defineRoute({ permission: "<module>.<screen>.<verb>", ... })`; a route without a permission does not compile and fails a registry test |

## First endpoints
| Method and path | Purpose | Permission | Notes |
| --- | --- | --- | --- |
| `POST /sales/invoices` | Create draft | `sales.invoices.add` | Idempotency-Key |
| `GET /sales/invoices` | List | `sales.invoices.view` | Cursor paging, scope filter |
| `GET /sales/invoices/{id}` | Detail | `sales.invoices.view` | Field rules applied |
| `PATCH /sales/invoices/{id}` | Edit draft | `sales.invoices.change` | If-Match |
| `POST /sales/invoices/{id}/post` | Confirm and post | `sales.invoices.post` | One transaction; idempotent |
| `POST /sales/invoices/{id}/cancel` | Cancel (reversal) | `sales.invoices.cancel` | Reason required |
| `POST /sales/invoices/{id}/convert` | Order → invoice, quote → order | `sales.invoices.convert` | Partial quantities |
| `POST /pos/sync` | Send offline bills in batch | `pos.bills.add` | Per-bill result: synced or conflict |
| `POST /pos/number-blocks` | Request the next block of 50 numbers | `pos.bills.add` | Server refuses until the previous block is fully synced; block size is a company setting |
| `GET /people/assignable-roles` | Roles the caller may create | `people.users.add` | Admin: all; Manager: the allowed list |
| `POST /people/users` | Invite a user (email, role, branches) | `people.users.add` | Role must be below the caller's level; branches within the caller's; invitation emailed |
| `PATCH /people/users/{id}` | Change branches, disable | `people.users.change` | Manager: Team scope only |
| `GET /me/permissions` | Effective permissions + modules + field rules | any signed-in | Builds the rail and buttons |
| `POST /auth/sign-in` `/refresh` `/pin` | Sign in, rotate refresh, quick PIN | — | Rate limited |
| `POST /platform/v1/companies` | Create company | platform | Platform realm only |
| `PUT /platform/v1/companies/{id}/modules` | Module and feature switches | platform | Written to platform audit |

## Inventory, restocking, returns, reminders
| Method and path | Purpose | Permission |
| --- | --- | --- |
| `GET /stock/reorder` | Restock list with reasons | `stock.reorder.view` |
| `POST /stock/reorder/{itemId}/snooze` · `/ignore` · `/mark-ordered` | Snooze, ignore with reason, link to a PO | `stock.reorder.export` or `stock.reorder.view` for flagging |
| `POST /purchases/orders/from-suggestions` | Create PO drafts grouped by supplier | `purchases.orders.add` |
| `POST /stock/reservations` · `DELETE /stock/reservations/{id}` | Reserve and release | `stock.reservations.add` |
| `POST /stock/conversions` · `POST /stock/wastage` | Repack, wastage (with approval flow) | `stock.conversions.add` · `stock.wastage.add` |
| `GET /stock/batches/{id}/trace` | Where did this batch go | `stock.items.view` |
| `POST /sales/returns` · `POST /sales/returns/{id}/inspect` · `/approve` · `/restock` | Return flow | `sales.returns.add` · `.approve` |
| `POST /purchases/returns` · `POST /purchases/claims` | Supplier returns and claims | `purchases.returns.add` · `purchases.claims.add` |
| `GET /settings/reminders` · `PUT /settings/reminders/{ruleId}` | Reminder rules | `settings.reminders.*` |
| `GET /reminders` · `POST /reminders/{id}/snooze` · `/done` | Own reminders | signed-in |
| `GET /settings/compliance` · `PUT /settings/compliance/{ruleId}` | Legal rule settings (dated) | `settings.compliance.*` |
| `POST /farm/weighment` · `POST /farm/lots` · `POST /farm/account-sales` | Weighment, lots, account sale | `farm.*` |

## Auth, people, roles and entitlements
Full list, shapes and error codes: [access/07-auth-and-access-api.md](../06-security/access/07-auth-and-access-api.md).

## Public API and webhooks
Customers' systems use API keys (hashed, scoped, optional IP binding) on the company API. Outgoing webhooks carry a timestamp and an HMAC signature; the receiver rejects old timestamps; each delivery has an ID; companies can replay from the log. Event types follow module events (`invoice.posted`, `payment.received`, …).

## Documentation
OpenAPI generated from the schemas with examples; a sandbox company for partners.
