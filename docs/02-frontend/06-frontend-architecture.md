# Frontend Architecture

Flutter (D-001, accepted).

## Apps
| App | Used by | Layout | Special |
| --- | --- | --- | --- |
| Console | Super admin, platform staff | Three-area (rail 6 icons) | Own address and sign-in, strict security |
| Workspace | Admin, manager, accountant, billing officer, HR officer, stock employee | Three-area | Online only (release 1) |
| Counter | Salesperson | Own scan-first screen + drawer | Offline, thermal printing, scanner |
Later: restaurant screens, employee self-service (Me, inside Workspace), customer portal.

## Packages
| Package | Contains |
| --- | --- |
| `app_shell` | `ThreePaneShell`, `IconRail`, `ModuleDefinition` (manifest), `PermissionGate`, `PaneWidths` |
| `ui_kit` | `PanelScaffold`, `RecordScaffold`, form, cards, table, matrix editor, status chips, design tokens |
| `company_workspace` | Workspace feature screens; `AccessEditor` with People |
| `console` | Console screens |
| `counter` | Counter screens |
| `api_client` | Generated from OpenAPI; token refresh; idempotency header |
| Feature packages | one per module (sales, stock, …), each registers a `ModuleDefinition` |

## Pane widths (one tested class)
```dart
factory PaneWidths.forWidth(double width) {
  final rail = (width * 0.05).clamp(64.0, 80.0);
  final panel = (width * 0.25).clamp(240.0, 400.0);
  return PaneWidths(rail: rail, panel: panel, body: width - rail - panel);
}
```
| Width | Rail | Panel | Body |
| --- | --- | --- | --- |
| 1024 | 64 | 256 | 704 |
| 1280 | 64 | 320 | 896 |
| 1440 | 72 | 360 | 1008 |
| 1920 | 80 | 400 | 1440 |
Medium (600–1023): rail 64, panel ≥240 visible in landscape, slide-over when portrait or body < 420 px. Narrow (<600): bottom bar (first four icons + More), full-screen list, then record page; round Add button; form action bar above the bottom bar. Desktop windows minimum 800×600. Panel divider draggable 240–400 px, remembered per person.

## Manifest
One declaration per module generates the rail, access-editor rows, dashboard cards and role tables, so they cannot drift:
```dart
const stock = ModuleDefinition(
  id: 'stock', icon: Icons.inventory_2_outlined, label: 'Stock',
  screens: [
    ScreenDefinition(id: 'items',  view: 'stock.items.view',  add: 'stock.entry.add'),
    ScreenDefinition(id: 'counts', view: 'stock.counts.view', add: 'stock.counts.add'),
  ],
  dashboardCards: [CardDefinition(id: 'stock-alerts', view: 'stock.alerts.view')],
);
```
An icon shows when the session has `view` on any of its screens; a view inside appears only with `view` on that screen. Keys use `<module>.<screen>.<verb>` everywhere (the UI spec's `stock.see` example is retired, F-02).

## Routing
`go_router`; every selection has an address (`/sales/invoices/INV-1042?tab=payments`) that reopens the same panel row and tab; after sign-in the app returns to the opened link; routes redirect to `/not-available` when `session.can(...)` is false. The server still checks every request.

## State
`SessionCubit` holds the permission list (reloaded on start, resume, and after any "not available" reply; offline limit 24 h). Feature Blocs per screen (list, record, form). Remembered per person per module: sort, filters, last tab, column widths. Lists page from the server; selection survives refresh; rows never move while someone reads.

## Forms and drafts
Autosave drafts to an encrypted local DB per user (D-009); leaving by any route keeps the draft; Drafts switch in the Filter menu with a count; stale drafts (> 1 day) are re-checked for prices, tax and stock before posting; save conflicts show Keep mine / Take theirs per field; soft lock after 15 minutes of inactivity releases.

## Counter specifics
Scan box always focused; large grid of recent/favourite items; one tap per payment method; big plain total; no dialog on success ("Bill saved", cart clears); tap targets ≥48 px; Pay at the bottom; offline chip with waiting count; drawer: Held bills, Customers, Returns, Shift, Sync status (each only if allowed). Wide/landscape: items left, cart + Pay right. Portrait/phone: items full screen, cart as a bottom sheet, full-width Pay.

## Testing hooks
Golden screenshots per role per width (599, 600, 1023, 1024, 1280, 1440, 1920) in light and dark, generated from the manifest, so an added or dropped icon fails a test.
