# Workflows and User Flows

## 1. Super admin: add a company
1. Add company: name, business type (retail, restaurant, services, wholesale); the preset switches on a starting module set.
2. Review the Modules tab: flip modules and single features (Dashboard, People, Settings are always on).
3. Pick the plan; send the invitation to the company admin.
4. Save: platform DB updated, platform log written, change reaches users at their next session refresh.
Provisioning (idempotent, keyed by company ID, takes seconds): registry row `Provisioning` → company keys → seed documents stamped with `companyId` (roles, chart of accounts, number series, India tax settings, default rules, reminders, compliance rules, profile presets) → admin invitation → entitlements snapshot → `Ready` (until then the company cannot sign in) → platform audit. No database is created.

## 2. Company admin: first week
Accept invitation → setup checklist (3–5 steps, each skippable, sample data on request): company setup (details, GSTIN, financial-year start, branches, numbering) → import (parties, items, opening stock, opening balances; Excel/CSV templates) → opening balances (debit = credit) → go-live date. Pick **Who works here?** (Just me / Me and counter staff / A full team), adjust role templates (only switched-on modules appear), add users (email, role, branches; form says "Will see: Sales, Stock. Starts on Sales."), send invitations.

## 3. Add a user
Admin, or a Manager with `people.users.add` (only for roles below their own and inside their own branches; the form lists only those roles), enters email, role, branches → invitation stored with a hashed token and emailed → membership `Invited` (an existing identity just accepts; no second password) → user opens link, sets password, email verified, directory `Active` → first sign-in asks the second step if required → lands on the role's start screen with two or three first-time tips.

## 4. Add or edit any record (same in every module)
Tap Add in the panel header (absent without `add`) → the body becomes an empty form; panel shows a temporary row "New invoice (draft)" → fill; required fields marked; checked when leaving each field; totals update → action bar: Cancel left, one primary action right; **no Save draft button**, the form autosaves → Done: "Invoice saved", record in read mode, row selected at top of the panel → Cancel keeps the draft in Drafts. Server refusal: summary of fields at top, nothing lost. Edit: Edit button turns fields into inputs (quantity and notes edit in place); posted documents stay locked. Remove shows "Invoice removed. Undo" for ten seconds instead of asking.

## 5. Invoice posting with e-invoice
Draft → Post (needs `post`; approval rule may hold it) → one transaction (see 18-business-logic) → committed → worker generates e-invoice via the GSP adapter (critical queue). Portal slow or down: invoice is saved, chip Pending, retries with growing delay; Failed shows the reason and Retry; Dashboard alert counts failures and tells the invoice owner and admin. Company may choose to block the final print until the IRN arrives.

## 6. Counter: sell offline and sync
Bill created on the device with an offline ID and the next number from its block of 50 (the server issues the next 50 only after the previous 50 are synced and finished) → saved locally and printed → on reconnect, bills sent in order, offline ID is the idempotency key → server posts each bill in its own transaction; short stock does not reject the bill but flags a Conflict for the manager → the chip shows pending and conflicting counts. The device keeps a limited copy of items, prices, tax codes and the user's permissions, refreshed when online; a bill keeps the price used at sale. Past 24 h without sync the Counter locks (D-008). Shared PC: drafts are per user, encrypted, cleared at sign-out.

## 7. Stock employee: count a shelf
Sign in → Add stock entry form already open, scanner focused → scan or search item, location, quantity (numbers only, unit beside, warns on unusually large), reason → **Save and add next** (keeps location and reason) → mistake later: Request correction → manager sees it under Waiting for you → Approve posts reversing + new entry.

## 8. Approval
Rule fires → record shows "Waiting for approval from <name>" → approver sees row under Waiting for you with reason, amount, requester → Approve/Reject (reject needs a reason, record returns to draft) → requester notified; decision on the History tab; unanswered requests escalate. At the till: manager enters quick PIN.

## 9. Switch a module on or off
Super admin flips the switch → platform DB + audit → Redis message refreshes API caches (60 s cache limit as backstop) → next request sees it: routes refuse, menus hide, jobs skip the company → switching on a module that needs setup starts a setup job (indexes, defaults) → switching off keeps data, read-only export stays for a grace period → switching back restores earlier role settings ("Accounts is back on. Roles regained earlier settings. Review them.").

## 10. Change a role
Admin picks role → ticks presets/Advanced → Fields and Branches tabs → Preview as this role → Save review ("4 changes affecting 4 people", before/after) → saved and logged → people see it at next refresh; open forms finish first.

## 11. Import
Upload → map → check → fix → run → summary → undo within 24 h. Needs `import`.

## 12. Data export and leaving
Company export job writes each collection to object storage; retire = mark retired → export → wait retention → destroy company key → delete database → log.

## 13. Payroll run (deferred)
Specified in the architecture doc (build per employee, review, approve, one summary journal, publish payslips). Not in release 1.
