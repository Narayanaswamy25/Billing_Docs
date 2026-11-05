# Authentication and Authorization

> One-page summary. The full design is in [access/00-README.md](access/00-README.md): identity and sign-in, entitlements, permission catalog, role seeds, user lifecycle, platform security, API and tests. Where this page and that folder differ, the folder wins.

Rule: **each level can only hand down what it was given.** Super admin decides modules and features per company; company admin decides roles and people inside them; the screen shows only what survives both. A company never has more than the product offers, a role never more than the company, a person never more than their role plus the admin's personal changes.

## The three levels, in plain words
1. **Super admin (the software provider).** Runs the service. Has full control over every company's *service*: which companies exist, their plan, and each module and feature switched on or off. Platform staff work under the super admin with their own roles (owner, support, finance, read-only). Reading a company's business records is **not** part of this by default; it needs that company's time-limited support grant (D-023).
2. **Company admin.** Receives what the plan and the super admin allow, and decides how the company uses it: branches, roles, and who gets which access.
3. **Company users.** Employee, salesperson, stock employee, billing officer, manager, accountant, HR officer and so on. Each has their **own email login** and sees only what their role, plus any personal change, allows.

Each level can only pass on what it was given: **what the company can use = plan and super admin switches; what a role can use = the company's modules narrowed by the admin; what a person can use = their role, plus or minus personal changes.**

| Level | Controls | Cannot |
| --- | --- | --- |
| Super admin (Console) | Companies, plans, entitlements, modules and features per company, company admin invitation, platform staff and log | See company invoices, stock or staff records unless the company grants time-limited support access |
| Company admin | Branches; roles and their permissions inside switched-on modules; users (email, role, branches, personal changes); field visibility; turning any permission of a role or a single person on or off at any time | Use, see or grant a switched-off module or feature; change the plan |
| Manager (default) | Add and manage employee accounts in the roles the admin lets managers create; day-to-day sales, returns, stock and approvals (see below) | Create or edit roles; create users above or equal to their own level; grant anything they do not hold |
| Every other user | Own language, theme, quick PIN | Change own role or permissions |

## Plan, payment and entitlements
The service a company gets follows what it pays for.
| Step | Rule |
| --- | --- |
| Plan | A plan sets default modules, features and limits (users, branches, invoices per month, storage). Add-ons (billing only, e-billing, HR, restaurant pack) come through the company's agreement |
| Entitlements | Plan defaults plus add-ons, then any per-company switch the super admin sets on a module or a single feature. The result is the company's **entitlements** |
| Off means hidden | A module or feature that is off is absent from the rail, the role editor, dashboard, reports, search, notifications and API (`MODULE_OFF`). It is never shown greyed out, and the admin cannot tick anything for it. Data stays stored; switching it on again restores the roles' earlier settings |
| Payment state | Trial → Active → Past due (banner) → Grace → Suspended: read-only first, then blocked, using the "This page is not available" wording plus a contact line. The Counter keeps billing offline for a set time so a shop is never stuck at the till |
| Plan change | Moving to a smaller plan first lists what will switch off and what stays readable; nothing is deleted |
| Audit | Every switch and plan change is written to the platform log with before and after, and reaches company users at their next session refresh |

## Role levels and who may create whom
| Level | Default roles | May create user accounts in | Notes |
| --- | --- | --- | --- |
| 1 | Company admin | Any role, any branch | Last admin cannot be removed or demoted |
| 2 | Manager | Billing officer, Salesperson, Stock employee, in the manager's own branches | Cannot create Managers, Accountants, HR officers or Admins (sensitive books and staff data); the admin can widen or narrow this list per role (D-022) |
| 3 | Accountant, HR officer, Billing officer | Nobody | |
| 4 | Salesperson, Stock employee | Nobody | |
Rules: a creator can only give roles **below** their own level (except admin), only inside their own branches, and never a verb or scope they do not hold. The new person starts as `Invited`, receives an email link, sets a password, and signs in under their own email. A manager sees and can change (branches, disable) only users in their Team. The key permissions are `people.users.view`, `people.users.add` and `people.users.change`, with scope Team for managers.

## What the default roles do, in plain words
| Role | Can do | Cannot |
| --- | --- | --- |
| Salesperson | Billing on the Counter (scan or search items, pick or quick-add a customer, take payment); **see inventory** only as item, quantity and selling price | Add or change stock, see cost or profit, open any other module |
| Stock employee | See stock and add stock entries; Request correction | See prices, other modules; change saved entries |
| Manager | Create invoices; **take returns** (return or credit note, restock or damaged); **update inventory** (adjustments, transfers, counts, approve stock corrections); approve discounts and other requests; **add employee accounts** (see above); see reports for their branches | Change roles, settings or plan; see payroll |
| Billing officer | Create and edit invoices, quotes, orders, receipts; read items | See cost and profit; open Purchases or Accounts |
| Accountant | Books, journals, bank reconciliation, GST and tax views; read sales and purchases | Change stock; see customer contacts unmasked |
| HR officer | Attendance, leave, staff records (payroll later) | See sales, stock or accounts |
| Company admin | Everything inside the company's entitlements, plus roles, users and settings | Anything the entitlements do not include |
The admin can change any of this at any time: tick or untick a verb, scope, field or branch for a **role** (everyone in it) or for **one person** (a personal change, shown with a badge). Changes are saved with a review, logged, and reach people at their next refresh; a form already open finishes first.

## Vocabulary (the only one)
**Verbs (15):** `view`, `add`, `change`, `remove`, `post`, `approve`, `cancel`, `print`, `reprint`, `send`, `export`, `convert`, `view_cost`, `import`, `revert`.
Dependencies: every verb except `view` needs `view`; `post` needs `add`; `revert` also needs `change`; clearing `view` clears the rest.

| Verb | Allows | Screen change |
| --- | --- | --- |
| view | Open module, list, records | Icon, rows, tabs |
| add | Create a record or draft | Add button |
| change | Edit a record not posted | Edit button |
| remove | Delete a draft | More menu |
| post | Make a document final | Primary button reads Post |
| approve | Decide a pending request | Waiting for you card |
| cancel | Void a posted document inside the allowed window | More menu |
| print / reprint | First copy / duplicate copy (marked Duplicate, logged) | Print, Reprint buttons |
| send | WhatsApp, email, PDF | Send button |
| export | Download lists and reports | More menu |
| convert | Quote→order, order→invoice | Convert button |
| view_cost | Purchase cost and margin | Cost fields, margin column |
| import | Upload files, bulk edit | Import, Edit selected |
| revert | Restore an earlier version of an editable record or undo a change set (not for posted documents) | Restore this version, Undo this change |

**Scopes (4):** Own (records they created), Team (people who report to them), Branch (their branches), All. A user gets the intersection of the role's scope and their own branches.

**Presets:** No access · Just view (view) · Can work (view, add, change, post, print, send, convert) · Full control (all 15) · Custom (anything else). `approve` is never in Can work. Approvals stay off until the admin sets a rule.

**Key format:** `<module>.<screen>.<verb>`, e.g. `stock.items.view`, `stock.entry.add`, `sales.invoices.post`. Client helper: `session.can('stock.items.view')`.

## The five checks (server runs them on every request, in order; any failure ends it)
1. Module (and feature) is switched on for the company.
2. The role grants the verb on that screen.
3. No personal change removes it (a deny beats an allow; a personal allow can add a verb).
4. The record is in scope (scope ∩ the user's branches), applied as a query filter.
5. Each field is visible to the role (hidden fields never leave the server; masked fields leave masked).

Then business limits apply: maximum discount, refund and approval limits, with the approval workflow when crossed. Failing a check yields the same "This page is not available" for module-off and role-lacks, so a screen's existence is never revealed.

## How the code enforces it
- Each route is declared with `defineRoute({ permission: "module.screen.verb" })`; a permission middleware runs the five checks before the handler; a route without a permission fails a registry test.
- Repositories receive a scope filter built from branches and scope, so a query cannot forget the branch.
- Serializers drop fields the user may not see.
- The client asks `GET /me/permissions` and builds the rail, buttons and tabs from it; the server remains the authority.
- Search, counts, badges, quick-jump, reports, exports and notifications are all computed on the server under the same rules.

## Default role seeds (module level; tick-level seeds are in [access/04-role-seeds-and-limits.md](access/04-role-seeds-and-limits.md))
Legend: **V** just view · **A** view+add · **E** view+add+change · **F** full control (admin ticks each of approve, remove, export) · **Own** records created by the person · **–** hidden. Payroll rows are deferred (D-007).

| Screen | Admin | Manager | Accountant | Billing officer | HR officer | Salesperson | Stock employee |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Dashboard | V | V | V | – | V | – | – |
| Sales (quotes, orders, invoices) | F | F | V | E | – | A (scope Own) | – |
| Customers | F | E | V | E | – | A | – |
| Purchases | F | F | V | – | – | – | – |
| Stock | F | F | V | V | – | V (quantities only) | A |
| Accounts | F | V | F | – | – | – | – |
| GST and e-invoice | F | V | F | A | – | – | – |
| HR: attendance, leave | F | V (team) | – | – | F | – | – |
| Self-service (Me) | Own | Own | Own | Own | Own | Own | – |
| People: users | F | A + change (Team scope, assignable roles only) | – | – | – | – | – |
| Activity | V all | V team | – | – | – | – | – |
| Returns (return, credit note, restock) | F | F | V | E | – | – (admin can switch on; then the Counter drawer shows Returns) | – |
| Inventory view (item, quantity, selling price) | F | F | V | V | – | V | A (entries) |
| Reports | F | V | F | V (sales) | V (HR) | – | – |
| Settings | F | – | V | – | – | – | – |

A whole row is hidden for a company lacking that module, whatever the table says. Console screens are for the super admin only.

## Field rules (defaults; admin edits per role in the access editor)
| Field | Shown to | Hidden or masked for |
| --- | --- | --- |
| Cost and supplier price | Admin, Manager, Accountant (and any role with `view_cost`) | Billing officer, Salesperson, Stock employee |
| Selling price | Anyone with Sales or Stock access except Stock employee | Stock employee |
| Profit and margin | Admin, Manager, Accountant | Everyone else |
| Customer phone and email | Admin, Manager, Billing officer, Salesperson | Accountant masked; HR officer and Stock employee hidden |
| Salary and bank details (when payroll exists) | Admin, HR officer | Everyone else |
| PAN, Aadhaar token, bank account numbers | Masked for all; reveal permission shows the full value for about 15 s, logged every time | — |
| User list and role settings | Admin | Everyone else |

Masking: PAN/Aadhaar/bank → last four (`******1234`); phone → last two; email → first letter + domain (`j***@gmail.com`); money, names, addresses → shown or hidden, never masked. A hidden field is never required for that role; the server fills a default.

## Rules that protect the structure
- Admin can give a role only actions inside switched-on modules; the last admin cannot be removed or demoted; no self-promotion; the access change log and masked-reveal logging cannot be switched off.
- Role editor: dependencies enforced, scope per row, Preview as this role (read-only), summary line, save review ("4 changes affecting 4 people"), risk warnings (e.g. export on salary, remove on posted records), Modified-from-template badge, Reset to template showing the difference, Compare roles, Effective access with "Why can this person see this?".
- Role ticks are edited on tablets and computers only; phones can view roles and change a person's role.
- A change applies after the person finishes or leaves the form they are in; if the screen disappears the draft is kept.
- Permission list reloads on start, on resume and after any "not available" reply. A device offline keeps the last list for at most 24 hours (D-008).

## Sign-in and support access (summary)
- One **identity** per person (platform database) with **memberships** in companies; the token is issued for one company; accountants switch company without a second password (D-025). Staff without email may sign in by mobile number with an admin-issued temporary password (D-026). Details: [access/01-identity-and-sign-in.md](access/01-identity-and-sign-in.md).
- Access token 15 minutes, rotating refresh token with reuse detection, quick PIN per device, step-up for sensitive actions, offline grant of 24 hours on registered Counters.
- Support mode is a company-admin grant, read-only, time-limited and logged on both sides; admin recovery needs verification, a second approver and a waiting period. Details: [access/06-platform-and-support.md](access/06-platform-and-support.md).

## API keys and webhooks
Keys stored as SHA-256 hashes, scoped, optionally IP-bound. Outgoing webhooks signed with HMAC and a timestamp, retried with delay, switched off after repeated failures with a notice to the admin.
