# UI/UX and Design System

## Principles
One accent colour; two font weights (regular, semibold); generous space instead of borders; same place, same job on every screen (primary button bottom-right of a form, Cancel bottom-left, Add at the top of the panel); rail icons always show a one-word caption; a panel holds at most five stacked things (branch picker, header, search + filter, list, footer); buttons use two words (three where needed, "Add stock entry"); messages under about twelve words; strings written short first so Hindi and Tamil fit; colour never carries meaning alone.

## Tokens (shared package)
Colours, type scale, spacing, corner values, light and dark from the same tokens. Concrete values are **not yet chosen**: designers set them before any screen is drawn. Status chip colours: done, waiting, needs attention, cancelled.

## The three areas
**Rail (5%, 64–80 px).** One icon per module the person may use, fixed order: Dashboard, Sales, Purchases, Stock, Accounts, HR, People, Reports, Settings; never reorders by use. 24 px glyph in a 48 px target with caption. Active = filled glyph + 3 px accent bar. Badge = dot or count for waiting work (counts only records the person may see). Missing icons are absent, never greyed. Top: company logo or initial, then **+ New** (lists everything the role may add). Bottom: notification bell with count, sync dot (green synced, amber pending, red failed), Help, account menu (name, role, branch, company; My access, Me, language, theme, quick PIN, sign out). No top bar. Overflow: last icons collapse into More above the bell. If a role would have more than six icons, Sales + Purchases + Accounts fold into **Money**, then HR + People into **Team**.

**Panel (25%).** Top to bottom: company picker (multi-company only) → branch picker (only with more than one branch) → header (module title menu + one Add button named for the view: Add invoice, Add quote, Add receipt, Add return) → search with Filter (up to three filters, Drafts switch, chips) → list (56 px rows: bold first line, quiet second line, status chip) → footer (count, sort). Empty, loading and no-match states per [20-error-handling.md](../05-logic/20-error-handling.md).

**Body (70%).** Five templates: **Record** (header, tabs, content, summary card), **Form** (sections + bottom action bar), **Cards** (Dashboard), **Table or report** (filters, table or chart, Export in More), **Matrix editor** (access editor, module switches). Record header: name, status chip, one primary button (next step this role can take; none if none), More menu (Remove, Export only with permission). Content two columns when ≥680 px wide (each ≥320), else one. Summary card at right when body ≥1000 px, else a strip above. Read first; Edit swaps text for inputs; posted records show "Posted. Changes need a credit note."; nothing selected shows the module summary if totals are allowed. The branch name appears in the header when the panel shows All my branches.

## Role starts
| Role | Rail (generated; before folding) | Add | Opens on |
| --- | --- | --- | --- |
| Super admin | Companies, Plans and billing, Features, Support access, Platform log, Platform staff | Add company | Company list |
| Company admin | Dashboard, Money, Stock, Team, Reports, Settings | Add user + each module's Add | Dashboard |
| Manager | Dashboard, Money, Stock, HR (team), People (add employee accounts, Activity), Reports | Add invoice, Add purchase order | Dashboard |
| Accountant | Dashboard, Money, Stock, Reports, Settings | Add journal | Dashboard |
| Billing officer | Sales, Stock, Reports | Add invoice | New invoice |
| HR officer | Dashboard, HR, Reports | Add leave | HR |
| Salesperson | Counter + drawer | Pay | New bill |
| Stock employee | Stock | Add stock entry | Entry form, ready to scan |

## Dashboard
Answers "what needs me now" in three to five items with numbers, not charts. Panel: Waiting for you, Alerts. Body: cards (two columns ≥680 px, else one), ordered by role priority, load and fail independently, hidden when empty, only for modules the company has and the role may see. Admin: setup checklist until done, changes to review, sales today, stock alerts, approvals waiting, payments due, attendance today, latest access changes. Manager: approvals, sales today, stock alerts, goods to receive. Accountant: items to post, payments due/overdue, bank items to reconcile, GST filing due. HR officer: attendance today, leave requests waiting (payroll status card deferred). Billing officer, Salesperson, Stock employee: no Dashboard.

## Plain words
Every role except Accountant: Post invoice → Finish invoice; Counts → Stock check; Journal and Ledger → Money entries and Account history; Credit note → Refund note; Goods received → Items received.

## Behaviour
Shortcuts (single-key shortcuts only when no field has focus): Ctrl/Cmd+K quick jump (never lists hidden things), N add, Enter open row, Ctrl/Cmd+S save, ? shortcut sheet; Esc closes dialog → slide-over → leaves form (draft kept). Right-click row menu on desktop (Open, Print, Send, Duplicate, Copy link) limited to verbs. Multi-select bar shows allowed actions and how many rows were skipped. Files tab (PDF, images, spreadsheets, size limit by plan, drag onto body). Notes tab with at-mentions that never reveal a record. Notification centre with per-person preferences (admin alerts cannot be switched off for the admin). Compact row density option for desktop. Quick-add inside an invoice when the role has Add on Parties. Defaults first (today, current branch, last-used location/reason/customer); fewer than six fields above the fold, rest under More details; warn only on unusual things; Undo instead of confirm (ten seconds; posted documents still ask).

## Screen-by-screen
Panel lists, body content and how-to-add for each icon follow the UI spec tables (Workspace: Dashboard, Sales, Purchases, Stock, Accounts, HR, People, Reports, Settings; Console: Companies, Plans and billing, Features, Support access, Platform log, Platform staff). See [35-sitemap-routes.md](35-sitemap-routes.md).

## Open design work
Token values, icon set, wireframes per screen, print layouts, and the five-staff usability test (D-010).

## History, undo and backups (UI)
Every record has a **History** tab (timeline, Compare, Restore this version, Undo this change when permitted). Edits show a 10-second "Undo" bar, and **Me → My activity** keeps a 5-minute self-undo. **Settings → Recycle bin** restores deleted records. **Settings → Backups and restore** shows backup status, restore points, "Restore as of a date" and download. The admin Dashboard has a **Changes to review** card. Details: [history-and-backup/02](../05-logic/history-and-backup/02-revert-and-restore.md) and [03](../05-logic/history-and-backup/03-backup-and-restore-points.md).
