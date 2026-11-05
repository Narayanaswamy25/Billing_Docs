# Auth and Access API

Conventions from [12-api-specification.md](../../03-backend/12-api-specification.md) apply (error shape, rate limits, idempotency, versioning). Company realm under `/api/v1`; platform realm under `/platform/v1`. All bodies are JSON. Unknown input fields are rejected. Shapes are sketches to be fixed in the OpenAPI file.

## Sign-in (company realm)
| Method and path | Body | Reply |
| --- | --- | --- |
| `POST /auth/sign-in` | `{ login, password, deviceId? }` | `{ status: "ok", tokens, me }` or `{ status: "mfa_required", mfaToken }` or `{ status: "select_company", selectToken, companies: [{id, name}] }` |
| `POST /auth/mfa/verify` | `{ mfaToken, code }` or `{ mfaToken, recoveryCode }` | as sign-in |
| `POST /auth/select-company` | `{ selectToken, companyId }` | as sign-in |
| `POST /auth/switch-company` | `{ companyId }` (needs a recent full sign-in) | new tokens for the other company |
| `POST /auth/refresh` | `{ refreshToken }` | rotated tokens; reuse of an old token returns `AUTH_REUSE_DETECTED` and revokes the session family |
| `POST /auth/sign-out` | `{ all?: boolean }` | 204 |
| `POST /auth/step-up` | `{ password }` or `{ code }` | `{ stepUpToken, expiresInSeconds: 300 }` |
| `GET /auth/sessions` / `DELETE /auth/sessions/{id}` | | List and revoke own sessions |
| `POST /auth/password/forgot` | `{ login }` | Always 202 |
| `POST /auth/password/reset` | `{ token, newPassword }` | 204; revokes all sessions |
| `POST /auth/password/change` | `{ currentPassword, newPassword }` | 204 |
| `POST /auth/mfa/enrol` / `POST /auth/mfa/confirm` | `{}` / `{ code }` | secret and QR / recovery codes |
| `POST /auth/invitations/accept` | `{ token, name, password? }` | Tokens; no password needed for an existing identity who is signed in |
| `POST /auth/pin/set` | `{ deviceId, pin }` | 204; PIN verifier stored on the device |
| `POST /auth/offline-grant` | `{ deviceId }` | Signed grant (see below) |

## Devices (Counter)
| Method and path | Purpose | Permission |
| --- | --- | --- |
| `POST /devices/pair` | New device asks to pair; returns a short code | signed-in person |
| `POST /pos/terminals/{id}/approve` | Approve a pending device, name it, choose its branch | `pos.terminals.add` |
| `DELETE /pos/terminals/{id}` | Revoke a device | `pos.terminals.remove` |
| `POST /pos/number-blocks` | Next block of 50 numbers | `pos.bills.add` and an approved terminal |

## What the app asks after sign-in: `GET /me/permissions`
```json
{
  "versions": { "permissions": 42, "entitlements": 17 },
  "me": { "userId": "…", "name": "…", "role": "Manager", "branches": ["BLR"], "reportsTo": "…" },
  "company": { "id": "…", "paymentState": "active" },
  "modules": [{ "id": "sales", "features": ["invoice", "customers"] }],
  "permissions": ["sales.invoices.view", "sales.invoices.add", "stock.items.view"],
  "scopes": { "sales.invoices": "branch", "pos.bills": "own" },
  "fields": { "stock.items.cost_price": "hide", "sales.customers.phone": "show" },
  "limits": { "maxDiscountPercent": 25, "backdateDays": 2 },
  "assignableRoles": ["Billing officer", "Salesperson", "Stock employee"],
  "offlineGrant": { "token": "…", "expiresAt": "…" }
}
```
The app builds the rail, tabs and buttons from this. The server never trusts it.

## People, roles and access (company realm)
| Method and path | Purpose | Permission |
| --- | --- | --- |
| `GET /people/users` · `GET /people/users/{id}` | List and read (Team scope for managers) | `people.users.view` |
| `GET /people/assignable-roles` | Roles the caller may create | `people.users.add` |
| `POST /people/users` | Invite `{ name, login, roleId, branches, reportsTo }` | `people.users.add` |
| `POST /people/users/{id}/resend-invitation` | New link | `people.users.add` |
| `PATCH /people/users/{id}` | Branches, reportsTo, disable, reactivate | `people.users.change` |
| `PUT /people/users/{id}/role` | Change role | `people.roles.change` (admin) |
| `PUT /people/users/{id}/personal-changes` | Add or remove a personal change | `people.roles.change`, step-up |
| `POST /people/users/{id}/reassign` | Move held bills, drafts, approvals, direct reports | `people.users.change` |
| `POST /people/users/{id}/credentials/reset-link` | Email a reset link | `people.users.change`, step-up, lower level |
| `POST /people/users/{id}/credentials/temp-password` | Issue a temporary password (mobile-only) | same |
| `POST /people/users/{id}/mfa/reset` | Reset two-step | admin, step-up |
| `POST /people/users/{id}/unlock` | Unlock | `people.users.change` |
| `GET /people/users/{id}/effective-access?key=…` | "Why can this person see this?" | `people.users.view` |
| `GET /people/roles` · `POST` · `PATCH /people/roles/{id}` | Role list and edit (`permissions`, `scopes`, `fields`, `limits`, `assignableRoles`) | `people.roles.*` |
| `POST /people/roles/{id}/preview` | Rail, start screen and one sample screen as that role, read-only | `people.roles.view` |
| `POST /people/roles/{id}/reset-to-template` | Show the difference, then reset | `people.roles.change` |
| `GET /people/roles/compare?a=&b=` | Side-by-side | `people.roles.view` |
| `GET /people/access-log` | Access changes log | `people.activity.view` |
| `POST /support/grants` | Company admin grants support access | admin, step-up |
| `GET /support/sessions` · `DELETE /support/grants/{id}` | See and end support access | admin |

## Platform realm
| Method and path | Purpose | Permission |
| --- | --- | --- |
| `POST /platform/v1/auth/sign-in` (+ WebAuthn steps) | Staff sign-in | — |
| `GET /platform/v1/companies/{id}/entitlements` | Resolved set, sources, versions | `console.companies.view` |
| `PUT /platform/v1/companies/{id}/modules` | Switch modules and features (returns the cascade list before confirming) | `console.entitlements.change` |
| `PUT /platform/v1/companies/{id}/agreement` | Plan, add-ons, overrides, dates | `console.plans.change` |
| `POST /platform/v1/support/requests` | Ask a company for access | `console.support.add` |
| `POST /platform/v1/recovery-cases` and `/approve` | Admin recovery with maker-checker | owner |
| `POST /platform/v1/approvals/{id}/approve` | Second-person approval | owner (not the requester) |

## Error codes added
| Code | HTTP | Meaning |
| --- | --- | --- |
| `AUTH_INVALID` | 401 | Wrong login or password (same for unknown login) |
| `AUTH_MFA_REQUIRED` | 401 | Second step needed |
| `AUTH_LOCKED` | 423 | Too many attempts; try later |
| `AUTH_TOKEN_EXPIRED` | 401 | Access token expired; refresh |
| `AUTH_REUSE_DETECTED` | 401 | Old refresh token used; session family revoked |
| `STEP_UP_REQUIRED` | 401 | Sensitive action needs a fresh password or code |
| `ROLE_NOT_ASSIGNABLE` | 403 | Creator may not give that role |
| `SEAT_LIMIT` | 409 | No seat left on the plan |
| `LAST_ADMIN` | 409 | Cannot remove or demote the last admin |
| `SELF_CHANGE_FORBIDDEN` | 403 | A person cannot change their own role or access |
| `INVITE_EXPIRED` | 410 | Invitation link expired or used |
| `DEVICE_NOT_APPROVED` | 403 | Counter not registered |
| `GRANT_EXPIRED` | 401 | Offline grant older than 24 hours |
| `APPROVAL_REQUIRED` | 202 | Action waits for a second platform owner |
