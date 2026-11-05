# Platform and Support Security

The super admin and platform staff hold the most powerful accounts in the system, but by design they run the **service**, not the companies' books.

## 1. Platform roles (D-023, proposed boundary)
| Platform role | Can | Cannot |
| --- | --- | --- |
| Owner (super admin) | Everything on the platform: companies, plans, entitlements, modules and features, platform staff, platform log, key policy requests | Read company records without a support grant; use a company's role; edit audit logs |
| Support | See company status, plan, usage, health; request support access; run an approved **read-only** support session | Change plans or entitlements; see records without a grant |
| Finance | Plans, subscriptions, invoices to companies, credits, extend trial | Support sessions; module changes beyond plan switching; staff management |
| Read-only | View Console lists and the platform log | Any change |
Console permissions use their own keys (`console.companies.view`, `console.plans.change`, `console.entitlements.change`, …) in the platform realm and are never part of a company role editor.

## 2. Sign-in and session rules for platform staff
- Separate address and token audience from company sign-in; a platform token is rejected by the company API and the reverse.
- Hardware-backed second step on every sign-in; optional IP allow-list; session idle 15 minutes, absolute 8 hours.
- No shared accounts; accounts created only by an owner and removed the day someone leaves.
- Production access for engineers is just-in-time, approved, time-limited and logged (see 23-security-policy).

## 3. Two-person approval (maker-checker)
These Console actions need a second platform owner to approve before they run: retire or delete a company, destroy a company key, grant or change a platform role, switch off audit logging features (not allowed at all), bulk entitlement changes across many companies, and any recovery of a company admin. The request, approver and result are logged. A single-owner startup can start with a second named person (for example an external adviser) to meet this.

## 4. Support access (the only way to see company data)
1. The company admin (or the provider support person asking, then the admin approving) creates a **support grant**: reason, scope (which modules and branches), duration (default 24 hours, maximum 7 days), read-only.
2. The platform issues a short-lived token tied to one support person; the company sees "Viewing as support" with the end time and the name.
3. Every record opened and every export is logged in both the company audit log and the platform log; the company admin gets an email when it starts, and can end it at once.
4. Field rules still apply: payroll, bank and ID fields stay masked unless the grant says otherwise.
5. Write access is not available in release 1.

## 5. Company admin recovery
The admin loses their password and two-step device, or leaves, or the only admin is locked out.
1. The company contacts support through the registered channel; support opens a **recovery case**.
2. **Verification** (any two): callback on the company's registered phone, email to the registered address, proof of business (GSTIN and registration documents), a signed request from the owner.
3. A second platform owner approves (maker-checker). A **waiting period** (default 24 hours) starts; all current admins and the registered email and mobile are notified, and any of them can cancel the request.
4. After the wait, the platform issues a one-time reset link for **that admin's identity**, forcing a new password and a new two-step enrolment, and revokes all that admin's sessions and devices.
5. Everything is logged; a summary is emailed to every admin. The super admin never sets or sees the password.
If there is no admin left and none can be verified, the company stays read-only until verification succeeds.

## 6. What the super admin sees about a company
Name, plan, entitlements, payment state, usage meters (users, invoices, storage), health, version, module switches and their history, support sessions, audit of Console actions. **Never** invoices, stock, parties, employees or ledgers.

## 7. Platform realm data
`platform_staff`, `platform_roles`, `platform_audit`, `support_grants`, `recovery_cases`, `agreements`, `company_entitlements`, `usage_meters`, `subscriptions`, `plans`, `modules`, key vault metadata. Held in the platform cluster with its own database users and stricter auditing.
