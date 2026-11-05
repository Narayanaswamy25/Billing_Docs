# Access Design: overview

This folder is the full design for **who can sign in, what a company gets, and what each person can see and do**. [22-authentication-authorization.md](../22-authentication-authorization.md) is the one-page summary; when they differ, this folder is more detailed and the decision log says which is newer.

## The model in one picture
```
PLATFORM (software provider)
  Super admin + platform staff ──► sets ENTITLEMENTS per company
                                   (plan + add-ons + per-module/feature switches + payment state)
                                          │  can only be narrowed below, never widened
COMPANY                                   ▼
  Company admin ───────────────► builds ROLES inside the entitlements
                                   (verbs × screens × scope × fields × limits)
                                          │
  Manager (default) ───────────► adds employee accounts in allowed roles
                                          ▼
PEOPLE                            each person = one IDENTITY (own login)
  Salesperson, Stock employee,      + a MEMBERSHIP in a company (role, branches, team, personal changes)
  Billing officer, Accountant, HR officer, Manager …
```
**What a person can do = entitlements ∩ role ∩ personal changes ∩ branch scope ∩ field rules.** The server computes it on every request; the app only draws what the server says.

## Documents
| # | Document | Answers |
| --- | --- | --- |
| 01 | [Identity and sign-in](01-identity-and-sign-in.md) | One login for many companies, email or mobile login, passwords, two-step, sessions, devices, PIN, offline sign-in, recovery |
| 02 | [Entitlements and plans](02-entitlements-and-plans.md) | What a company gets for what it pays, switch resolution, dependencies, limits, payment states |
| 03 | [Permission catalog](03-permission-catalog.md) | Every screen and which of the 15 verbs apply; scopes; field rules; limits |
| 04 | [Role seeds and limits](04-role-seeds-and-limits.md) | Exact starting permissions for each role, discount and refund caps |
| 05 | [User lifecycle](05-user-lifecycle.md) | Invite, accept, change role, team, leave, transfer work, disable, remove |
| 06 | [Platform and support security](06-platform-and-support.md) | Super admin and staff roles, two-person approval, support access, admin recovery |
| 07 | [Auth and access API](07-auth-and-access-api.md) | Endpoints, shapes, error codes |
| 08 | [Threats and tests](08-threats-and-tests.md) | What can go wrong and the tests that prove it cannot |

## Decisions this design depends on
D-011 (15 verbs after D-048), D-021 (Express), D-022 (managers add employees), D-023 (super admin reads company data only by grant), D-024 (entitlements), and the new D-025 to D-031 in [29-decision-log.md](../../01-architecture/29-decision-log.md).
