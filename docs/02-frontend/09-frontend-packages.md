# Frontend Packages

Concrete Dart/Flutter packages are **proposed** and unpinned: choose and pin versions when the workspace is created. License check each one (use the license-compliance step in CI).

| Need | Proposed package | Reason | Restriction |
| --- | --- | --- | --- |
| State | `flutter_bloc`, `bloc`, `equatable`, `bloc_test` | UI spec uses Bloc/Cubit; sealed states | No other state library |
| Routing | `go_router` (typed routes) | Deep links, redirects for permissions | Redirects only decide drawing; server enforces |
| Local DB | `drift` with encryption (proposed, D-056); community `isar` fork as the alternative | Offline bills, drafts, masters copy | The original Isar is reported unmaintained; decide after a Counter spike |
| Secure storage | `flutter_secure_storage` | Tokens, DB key | Never store tokens in plain preferences |
| HTTP | `dio` or `http` + generated client | Retry, interceptors | Idempotency key on posting calls |
| Models | `json_serializable` / `freezed` or OpenAPI generator output | Typed DTOs | Generated from the OpenAPI file, not hand-copied |
| i18n | `flutter_localizations`, `intl`, ARB | EN/HI/TA | No hardcoded strings |
| Printing | ESC/POS plugin per platform; `pdf` + `printing` for A4/A5 | Thermal and tax invoices | Bill saved before printing |
| Scanner | Keyboard-wedge (focus a field) first; camera scan plugin for phones | Scanner acts as a keyboard | — |
| Connectivity | `connectivity_plus` | Sync dot, offline chip | Not a source of truth; a failed call is |
| Testing | `flutter_test`, `mocktail`, `bloc_test`, golden tests | Per-role, per-width goldens | — |
| Lint | `very_good_analysis` | Consistent rules | — |

## Restrictions
No package that sends data to a third party by default (analytics, crash) unless it can be configured to strip personal data; crash and usage reporting must carry no customer names, phones, GSTINs or amounts. Prefer maintained packages with permissive licences; record any exception in the decision log.
