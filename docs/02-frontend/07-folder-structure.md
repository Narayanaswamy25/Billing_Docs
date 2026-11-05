# Frontend Folder and File Structure

Flutter (D-001, accepted); folder layout is proposed. A Dart workspace/monorepo; each package has its own `pubspec.yaml`, `lib/`, `test/`. Package names are snake_case.

```
apps/
  console/            Super admin app (entry point, routes, DI)
  workspace/          Company back-office app
  counter/            Salesperson Counter app (own layout)
packages/
  app_shell/          ThreePaneShell, IconRail, ModuleDefinition, PermissionGate, PaneWidths, NarrowShell, SlideOverPanel
  ui_kit/             Design tokens, PanelScaffold, RecordScaffold, FormScaffold, cards, tables, matrix editor, status chips, empty/error states
  api_client/         Generated from OpenAPI; auth, refresh, idempotency headers
  session/            SessionCubit, permission model (verbs, scopes, field rules), offline permission cache
  local_store/        Encrypted local DB (drafts, offline bills, masters copy)
  l10n/               ARB files (en, hi, ta), formatters (Indian grouping, dates, amount in words)
  printing/           ESC/POS thermal, PDF, preview
  company_workspace/  AccessEditor, Dashboard, shared workspace screens
  features/
    sales/ purchases/ stock/ accounts/ hr/ people/ reports/ settings/ import/
    console_companies/ console_plans/ console_features/ console_support/ console_log/ console_staff/
```

## Inside a feature package
```
sales/
  lib/
    sales.dart                 public barrel, exports the ModuleDefinition only
    src/
      module.dart              const ModuleDefinition (icon, screens, verbs, cards)
      data/                    repositories, DTO mappers
      bloc/                    list, record, form Blocs/Cubits (sealed state, Equatable)
      view/                    panel and body widgets (Page + View split)
      l10n/                    feature strings
  test/                        mirrors lib/src; golden tests under test/goldens/
```

## Naming
Files `snake_case.dart`; one public widget or class per file; Blocs `InvoiceListBloc`, events and states sealed; routes constants in `routes.dart`; permission keys as constants generated from the manifest (never typed inline).

## Rules
- Apps depend on packages; feature packages depend on `app_shell`, `ui_kit`, `session`, `api_client`; never on each other (talk through routes or events).
- No hardcoded colours, text styles or spacing: read tokens from `ui_kit`.
- No user-facing literals: localisation only.
