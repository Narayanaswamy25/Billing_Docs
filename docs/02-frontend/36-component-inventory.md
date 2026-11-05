# Component Inventory

Derived from the UI spec. Props are proposed signatures; finalise when wireframes exist. Every component reads tokens, localises its strings, and meets the 48 px target and accessibility rules.

| Component (package) | Props (key) | States | Notes |
| --- | --- | --- | --- |
| `ThreePaneShell` (app_shell) | manifest, session, selected | wide, medium, narrow | Uses `PaneWidths`; slide-over when body < 420 px |
| `IconRail` (app_shell) | modules, selected, badges | active, badge, overflow | Absent icons not drawn; caption always |
| `NarrowShell` (app_shell) | modules, selected | bottom bar (4 + More) | Single-module role has no bottom bar |
| `PermissionGate` (app_shell) | key, child | allowed / draws nothing | No greyed-out fallback |
| `PanelScaffold` (ui_kit) | companyPicker?, branchPicker?, header, search, filters, list, footer | loading, empty (can add / cannot), no match, error | At most five stacked things |
| `PanelHeader` | title menu, addLabel?, onAdd | Add absent without `add` | One Add button |
| `FilterMenu` | up to 3 filters, drafts switch, count | chips shown under search | |
| `ListRow` | first line, second line, status chip, selected | selected, multi-select | 56 px |
| `RecordScaffold` (ui_kit) | header, tabs, content, summary | read, edit, locked, waiting for approval | Two columns ≥680 px, summary card ≥1000 px |
| `RecordHeader` | name, chip, primary action?, moreMenu | | Primary = next step this role can take |
| `FormScaffold` (ui_kit) | sections, actionBar, errors | autosaving, saving, conflict | Cancel left, one primary right; no Save draft |
| `Field` family | label, value, mask, hidden | read, edit, masked (+Reveal), error | Hidden = not drawn |
| `StatusChip` | state (8 named) | done, waiting, attention, cancelled | Icon or word as well as colour |
| `DashboardCard` | id, title, count/items, onOpen | loading, error (independent), hidden when empty | |
| `ReportTable` | filters, columns, rows, export | loading, empty, large (background export) | Server paging |
| `MatrixEditor` (company_workspace) | modules, verbs, presets, scope | view, edit, advanced | Presets, dependencies, preview, save review |
| `ModuleSwitchRow` (console) | module, features, on | on, off | Writes platform log |
| `StatesBanner` | kind (offline, access changed, locked by, support, maintenance) | | Shared wording from 20-error-handling |
| `UndoBar` | message, onUndo | 10 s | |
| `ImportWizard` | steps: upload, map, check, fix, run | error file, undo | Needs `import` |
| `NotificationCentre` | items, filters | | Filtered to what the person may see |
| `QuickJump` | query | | Never lists hidden items |
| `CounterScreen` (counter) | scanBox, grid, cart, pay | offline chip, held bills | Wide and portrait layouts |
| `PaymentSheet` | modes, split lines, total | disabled until equal or Credit | |
| `PrintPreview` | document, paper, printer | printer not reached | |
| `SyncStatusDot` | state | synced, pending, failed | Opens sync details |
| `ApprovalPrompt` | rule, amount | PIN entry on Counter | |
| `HistoryTimeline` | events, revisions, permissions | loading, empty, filtered | Shows who, role, device, summary; Compare, Restore, Undo |
| `CompareDialog` | revision A, revision B | fields changed | Respects field rules |
| `RevertDialog` | preview, effects, fields, reason, stepUp | conflict, blocked, success + undo | |
| `ChangesToReviewCard` | counts by type | hidden when empty | Opens the Activity feed |
| `RecycleBinList` | deleted records, restore | empty, conflict | |
| `RestoreAsOfWizard` | types, date, preview, apply | blocked list | Admin, step-up |
| `BackupsPanel` | status, list, download, restore requests | failed, unverified | |

Add rows as components are built; link each to its widget tests and goldens.
