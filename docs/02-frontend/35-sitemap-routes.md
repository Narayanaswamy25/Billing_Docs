# Sitemap, Page Inventory and Routes

Routes use `/<module>/<screen>[/<id>][?tab=<tab>]`. Each screen's access key is `<module>.<screen>.view` (and the other verbs). Pages the role cannot view are absent from the rail and return "This page is not available. Contact your admin." when typed.

## Workspace
| Module (icon) | Screen / route | Panel lists | Body | Add |
| --- | --- | --- | --- | --- |
| Dashboard | `/dashboard` | Waiting for you, Alerts | Cards, setup checklist (admin) | — |
| Sales | `/sales/quotes` `/orders` `/invoices` `/receipts` `/returns` | Documents | Invoice: customer, lines, GST, payments, e-invoice status, print and send; tabs Details, Lines, Payments, Linked, Files, Notes, History | Add invoice / quote / receipt / return |
| Sales | `/sales/customers` | Name, outstanding, status | Details, Invoices, Payments, Statement, Files, History | Add customer |
| Purchases | `/purchases/orders` `/receipts` `/bills` `/suppliers` | Documents / suppliers | Order with lines, receipts and bill match; supplier tabs | Add purchase order |
| Stock | `/stock/items` `/locations` `/counts` `/transfers` `/price-lists` | Items with quantity | Quantity by location, batches and expiry, movement history | Add stock entry |
| Accounts | `/accounts/journal` `/ledger` `/bank` `/tax` | Entries | Ledger lines, bank reconciliation, tax registers and GSTR data | Add journal |
| HR | `/hr/attendance` `/leave` `/staff` | Employees | Attendance calendar, leave balance | Add leave or correction |
| People | `/people/users` `/roles` `/activity` | Users, roles (roles only for admin) | User: role, branches, personal changes, Activity tab; role opens the access editor (admin). A Manager sees only Users in their Team and Activity | Add user (admin: any role; Manager: assignable roles only), Add role (admin) |
| Reports | `/reports/<group>/<report>` | Reports by module | Filters, table or chart, Export | — (Save filters as favourite) |
| Settings | `/settings/branches` `/rules` `/numbering` `/tax` `/print` `/integrations` `/approvals` | Setting groups | Setting form with an effective date | Add branch, add rule version |
| Settings (history and backup) | `/settings/backups` `/settings/recycle-bin` | Backups and restore points, deleted records | Status, list, download, restore as of a date | Backup now, request restore |
| Account menu | `/me` `/me/access` | — | Own attendance and leave; plain-words My access page | — |
| Import | `/import/<type>` (needs `import`) | — | Upload, map, check, fix, run | — |

## Console
`/console/companies` (with tabs Overview, Modules, Admin, Usage, Support access; and Offboarding view), `/console/plans` (Plans, Subscriptions, Usage), `/console/features`, `/console/support-access`, `/console/platform-log`, `/console/platform-staff`.

## Counter
`/counter/new-bill` (home), drawer: `/counter/held`, `/counter/customers`, `/counter/returns`, `/counter/shift`, `/counter/sync`.

## System pages
`/sign-in`, `/not-available`, session-expired, update-required, maintenance, first-sign-in tips.

## Access by role (module level)
See the seed table in [22-authentication-authorization.md](../06-security/22-authentication-authorization.md). Components per page are listed in the module's feature package once built; add a column here when the component inventory (checklist item 36) exists.
