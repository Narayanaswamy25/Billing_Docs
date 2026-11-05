# Access Threats and Tests

## Threats specific to this design
| Threat | How | Control | Test |
| --- | --- | --- | --- |
| Privilege escalation through "create user" | A manager invites an Accountant or Admin, or a role with more verbs than they hold | Assignable-role list, level check, verb and scope subset check, server-side | T-01, T-02 |
| Self-promotion | A person edits their own role, scope or limits | `SELF_CHANGE_FORBIDDEN`; change endpoints reject the caller's own ID | T-03 |
| Invitation hijack | Link forwarded, guessed or reused | 256-bit random, hashed at rest, single use, 72 h, bound to the invited login; resend invalidates the old | T-04 |
| Account enumeration | Different messages or timings for unknown vs known login | One message and equalised timing; forgot-password always 202 | T-05 |
| Credential stuffing, brute force | Many guesses | Lockout, per-IP and per-identity limits, breached-password check, alerts | T-06 |
| Token confusion | Platform token used on a company API or the reverse; token for company A used on company B | Separate keys and audiences; company ID inside the token must match the resolved tenant | T-07 |
| Stale permissions | Removed person keeps working | Session revocation, permission-version check each request, 24 h offline grant with conflict flagging | T-08 |
| IDOR | Opening another user's or branch's record by ID | Scope filter in every repository query; the two-company and two-branch matrix | T-09 |
| Hidden field leak | Value present in search, export, notification or error text | Field rules applied in the serializer and in every query projection | T-10 |
| Entitlement bypass | Calling a route of a switched-off module | `moduleGate` before `permissionGate`; jobs and search also check | T-11 |
| Seat or limit bypass | Parallel invites exceed the plan | Seat check inside the same transaction as the invite | T-12 |
| Stolen Counter | Device taken with a valid session | PIN, local failure counter, device revoke, 24 h grant expiry, number block void | T-13 |
| Shared PC | Next person sees the last person's data | Per-user encrypted drafts, cleared at sign-out unless kept | T-14 |
| Admin takeover via recovery | Social engineering support | Two verifications, second approver, waiting period, notices to all admins, cancel option | T-15 |
| Support abuse | Staff browse company data | Grant required, read-only, logged both sides, ends at once on request | T-16 |
| Refresh token theft | Stolen refresh token | Rotation and reuse detection revokes the family | T-17 |
| SMS or email pumping | Abusing forgot-password | Rate limits per login and per address; no SMS in release 1 | T-18 |
| Clock or offline replay | Replayed offline grant or bill | Grant signed with device ID and expiry; bills idempotent by offline ID | T-19 |

## Test cases (all must pass before release)
| ID | Case | Expected |
| --- | --- | --- |
| T-01 | Manager tries to invite an Accountant | 403 `ROLE_NOT_ASSIGNABLE`; no user created |
| T-02 | Manager (default list) invites a Salesperson with a branch they do not hold | 403 or validation failure |
| T-03 | Any user calls change-role on their own ID | 403 `SELF_CHANGE_FORBIDDEN` |
| T-04 | Use an invitation twice; use it after 72 hours; use a resent old link | Second use and expired and old links fail with `INVITE_EXPIRED` |
| T-05 | Sign in with an unknown login vs a wrong password | Identical status, body and similar response time |
| T-06 | Six wrong passwords in a row | Locked; admin unlock works; alert raised |
| T-07 | Company A token on company B route; platform token on company API | 401 or 403 every time |
| T-08 | Disable a user with a live session; call any API | Rejected on the next request; refresh fails |
| T-09 | Branch-A manager reads a branch-B invoice by ID | 404 |
| T-10 | Role with cost hidden: search, list, detail, export, notification, error body | Cost absent everywhere |
| T-11 | Switch HR off; call HR routes; check rail, role editor, search, reports, jobs | All absent or `MODULE_OFF`; data intact; switching on restores role settings |
| T-12 | Two managers invite at the last seat in parallel | One succeeds, one `SEAT_LIMIT` |
| T-13 | Revoke a Counter; unused block numbers | Sessions end; numbers listed for Void |
| T-14 | Sign out and sign in as another user on the same PC | No drafts from the previous user |
| T-15 | Recovery case without second approver or before the waiting period | No reset link issued |
| T-16 | Support session reads a record outside the grant | Denied and logged |
| T-17 | Reuse an old refresh token | `AUTH_REUSE_DETECTED`; all sessions in the family revoked |
| T-18 | 20 forgot-password calls | Rate limited |
| T-19 | Replay an offline bill and an expired grant | Bill not duplicated; grant refused with `GRANT_EXPIRED` |
| T-20 | Salesperson views stock items | Item, quantity, selling price only; no cost, margin or add button |
| T-21 | Manager disables a Salesperson in Team; then a Salesperson in another team | First succeeds; second 404 or 403 |
| T-22 | Plan downgrade removes Accounts | Accountant role keeps dormant settings; rail and editor show no Accounts |
| T-23 | Past-due company reaches read-only | Add and post refused; print and export work; Counter keeps billing offline for the set time |
| T-24 | Salesperson sold offline after being disabled | Bill accepted on sync and flagged Authorization conflict |
| T-25 | Admin tries to remove the last admin | 409 `LAST_ADMIN` |
These join the permission matrix and the isolation suite described in [tenancy/05](../tenancy/05-isolation-tests-and-assurance.md) in [26-testing-strategy.md](../../08-process/26-testing-strategy.md).
