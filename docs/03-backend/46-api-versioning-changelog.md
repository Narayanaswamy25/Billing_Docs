# API Versioning and Changelog

## Policy
- Two APIs, versioned separately: `/api/v1` (company) and `/platform/v1` (platform).
- **Additive changes** (new optional field, new endpoint, new enum value that clients may ignore) need no new version.
- **Breaking changes** (removed or renamed field, changed meaning, new required input, stricter validation of existing input) ship as `/v2` with a deprecation period.
- Clients must ignore unknown fields; the server rejects unknown *input* fields.
- Event types carry a version; consumers ignore unknown fields; a breaking event change is a new version with the old kept working for a period.
- CI compares the generated OpenAPI file with the last release and fails on a hidden breaking change.
- Error codes are stable; messages are not part of the contract.
- App update required: the API returns an "update required" signal for client versions below the minimum supported; Counters offline keep working until they sync.

## Deprecation
Announce in the changelog and by response header; keep the old version at least [period: owner to set, proposed two releases or 90 days]; usage per version tracked so removal is safe.

## Changelog format
```
## v1.0.0 (unreleased)
### Added
### Changed
### Deprecated
### Removed
### Fixed
### Security
```
No API has shipped yet; nothing to list.
