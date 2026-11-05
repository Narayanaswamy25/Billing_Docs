# Identity and Sign-in

Numbers marked *proposed* are starting values the owner or company can change; they are settings, not constants.

## 1. Identity model (D-025, proposed)
The original design stored each user inside one company database. That breaks a case your UI spec already promises: an accountant who serves several companies and switches between them. With per-company credentials that person has a separate password and two-step enrolment in every company. Fix: separate **who you are** from **where you work**.

| Concept | Lives in | Holds |
| --- | --- | --- |
| **Identity** | Platform database, `identities` | Login (email and/or mobile), password hash (Argon2id), two-step secret (encrypted), recovery codes (hashed), status (Active, Locked, Disabled), verified flags, last sign-in, failed-attempt counter. **No business data.** |
| **Membership** | Platform database, `memberships` (replaces `login_directory`) | identityId, companyId, companyUserId, status (Invited, Active, Disabled), invited by, dates |
| **Company user** | `users` collection (stamped with `companyId`) | Name, role, branches, reportsTo (team), personal changes, limits, seat status, preferences, quick-PIN verifier per device, display data. No password. |
| **Platform staff** | Platform database, `platform_staff` | Separate realm and credential store, hardware-backed second step (see 06) |

Why this is safe: the platform database already holds the registry and is the most guarded database; it now also holds credential hashes, protected by Argon2id plus a pepper kept in the key vault. A bug in business-data access is contained by the tenant data access layer (tenancy/01), not by the credential store. Trade-off to accept: a platform-database breach exposes hashes for all companies, which is why the pepper, a stricter cell and breach alerts matter.

One email can have one identity and many memberships. Disabling a person in one company disables only that membership. A company's own policy (for example "two-step required") is enforced per membership at sign-in to that company.

## 2. Login IDs and who needs what (D-026, proposed)
Many shop staff have no email address. Requiring one would stall onboarding.

| Role group | Login ID | Notes |
| --- | --- | --- |
| Company admin, Manager, Accountant, HR officer | Email (required) | Receives invitation, resets, alerts |
| Billing officer, Salesperson, Stock employee | Email **or** mobile number | Mobile-only staff get an admin- or manager-issued temporary password (shown once, must change at first sign-in); no SMS needed |
| Everyone | Optional quick PIN per device | Fast unlock on a Counter or Workspace device already signed in |
SMS one-time passwords are **not** required for release 1: in India, SMS needs registered sender IDs and templates and costs money per message. Email or admin-issued credentials cover it. Mobile OTP can be added later behind the same sign-in interface.

## 3. Passwords
- Minimum 10 characters, up to 128, any characters (including Hindi and Tamil), no forced composition rules, no scheduled expiry (*proposed*, current NIST guidance favours length and breach checks over rotation).
- Reject passwords found in a breached-password list and the user's own name, email or company name.
- Stored only as Argon2id (≥19 MiB, 2 iterations, parallelism 1, unique salt, pepper from the vault); rehashed at sign-in when settings improve.
- A company may raise the length or require two-step for chosen roles; it cannot lower the platform minimum.
- Change password: needs the current password, revokes all other sessions, notifies the person by email.

## 4. Two-step sign-in
| Who | Rule |
| --- | --- |
| Platform staff | Always, hardware-backed (security key or device passkey) |
| Company admin | Required (authenticator app, TOTP) |
| Accountant, Manager | Strongly recommended; the company can require it |
| Others | Optional; the company can require it per role |
Ten one-time recovery codes at enrolment, shown once, stored hashed. Resetting someone's two-step is an admin action needing a fresh step-up and an email to the person; resetting an **admin's** two-step uses the recovery process in 06.

## 5. Sign-in flow (company realm)
1. Person enters login and password (and picks language if first time).
2. Server looks up the identity. A wrong login, wrong password and unknown login all return the same message ("Email or password is wrong") in the same time, so accounts cannot be guessed.
3. Password checked; failed attempts counted (see lockout).
4. If the identity has several active memberships, show the **company picker**; pick one. If one, continue.
5. Check the membership: Active, company not suspended or retired, seat valid.
6. If this membership requires two-step, ask for the code.
7. Issue tokens for that company only (company ID is inside the token), register the session, return `GET /me/permissions` data.
8. Land on the role's start screen.
Switching company later: no password again if the last full sign-in was under 12 hours ago (*proposed*) and the target company's two-step is satisfied; a new token is minted for the target company and the old company's token is dropped.

## 6. Lockout and abuse limits (*proposed*)
| Control | Value |
| --- | --- |
| Failed sign-ins | 5 in 15 minutes locks that identity for 15 minutes, doubling after each repeat up to 24 hours |
| Per address | Rate limit by IP and by identity separately |
| Quick PIN | 6 digits; 5 wrong tries on a device require full sign-in and are logged |
| Admin unlock | Admin can unlock a person (`people.users.change`, step-up); the person is emailed |
| Alerts | Repeated failures for one account or address, and sign-in from a new country or device for admins |
| Breached device | Admin can "Sign out everywhere" for a person; person can do it for themselves |

## 7. Sessions and tokens
| Item | Rule (*proposed*) |
| --- | --- |
| Access token | 15 minutes, signed with the realm's key, carries user ID, company ID, session ID, device ID, **permission version** and **entitlements version**, never the permission list |
| Refresh token | Random, stored only as a hash with its device; **rotates on every use**; reuse of an old one revokes the whole session family |
| Workspace session | Idle timeout 30 minutes (company can set 15–120); absolute 14 days |
| Counter session | Stays signed in on its registered device up to 30 days; locks to the quick PIN after 5 minutes idle; full sign-in needed after 30 days or after PIN failures |
| Token storage | Flutter: secure storage (Keychain, Keystore, DPAPI). Flutter web: HttpOnly, Secure, SameSite cookies plus a CSRF token |
| Permission change | Changing a role, personal change or entitlement raises a version; the next request recomputes; no token reissue needed |
| Revocation | Disable a person, sign out everywhere, password change or refresh reuse revokes sessions at once; the access token dies within 15 minutes at most, and the permission middleware also checks the session is not revoked on every request |
| Session list | Each person sees devices and last use under the account menu and can sign out any; admins can sign out anyone |

## 8. Devices and the Counter (D-027, proposed)
A shared Counter PC and an unknown phone are different risks, so Counters are **registered terminals**.
1. A new device shows a short pairing code and asks the person to sign in; it stays pending.
2. A user with `pos.terminals.add` (admin, or a manager if the admin allows) approves it in the Workspace, naming it and choosing its branch. This creates a `pos_terminals` record and binds a device key pair; the private key never leaves the device.
3. Only an approved terminal can request number blocks (50 numbers) and sync bills.
4. A lost device is revoked in one action: its sessions end and its unused block numbers are listed for Void.
Workspace devices need no approval but appear in the person's session list.

## 9. Offline sign-in on the Counter (D-028, proposed)
Short access tokens cannot be renewed offline, but the Counter must keep billing.
- When online, the server gives the signed-in person an **offline grant** for that device: user, branch, role name, the permission snapshot, limits, field rules, permission and entitlement versions, issued time and an expiry. Expiry is **24 hours after the last successful sync** (ties to D-008); the grant is signed by the server and verified on the device.
- The quick PIN unlocks the grant locally (Argon2id verifier in secure storage, with a local failure counter).
- While offline the device authorises actions from the grant, and records each bill with the grant version it used.
- At sync the server re-checks each bill against the permissions that applied **at the time of sale**. If the person was disabled or lacked the permission before that time, the bill is still accepted (the sale happened and the invoice number was consumed), but flagged **Authorization conflict** for the admin and manager. This keeps the books complete and the exception visible.
- After 24 hours without sync the grant expires: the Counter shows "Sync to continue billing" (the same message as when 50 numbers run out). A person removed from the company can therefore sell for at most 24 hours offline, and every such sale is flagged.

## 10. Recovery
| Case | Flow |
| --- | --- |
| Forgot password (email login) | "Forgot password" always answers "If this email exists, we sent a link." One-time link, 256-bit random, stored hashed, valid 30 minutes; success signs out all sessions and emails a notice |
| Forgot password (mobile-only login) | Admin or manager (for their team) issues a new temporary password; shown once; must change at next sign-in |
| Lost authenticator | Use a recovery code; or admin resets two-step for non-admins |
| Change email | Verify the new address by link; notify the old address; revoke other sessions |
| Admin loses access or leaves | See 06 (verified recovery with a waiting period) |
| Sensitive actions | Need a fresh **step-up** (password or code) valid 5 minutes: change bank details, whole-table export, change roles or overrides, create API keys, turn support mode on, change security settings, reset someone's credentials |

## 11. Email and invitation handling
Invitation and reset emails come from a domain with SPF, DKIM and DMARC. Tokens never appear in logs. Email addresses are normalised (trim, lower-case) before lookup. A person may belong to many companies; an invitation to an existing identity asks them to sign in and accept, instead of creating a second password.
