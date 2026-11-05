# Environment and Configuration

Settings are typed, validated against a schema at start, and the process refuses to start if one is missing. No defaults for secrets. A settings document is generated from the schema. Values come from environment variables and the key vault.

## Environments
| Environment | Purpose | Data | Keys |
| --- | --- | --- | --- |
| Development | Local work | Masked or synthetic only | Own keys |
| CI test | Pipeline tests on a real replica set | Fixtures | Test keys |
| Staging | Mirrors production; migrations rehearsed | Masked or synthetic | Own keys |
| Production | Customers | Real | Production KMS |
| Sandbox cell | Partner and GSP testing | Synthetic | Own keys |
Separate cloud accounts or projects for production, staging and development. Production data never goes to development unless masked.

## Variables (names proposed; values never in the repository)
| Variable | Used by | Secret? |
| --- | --- | --- |
| `NODE_ENV`, `APP_MODE` (`api` or `worker`) | All | No |
| `PLATFORM_DB_URI` | API, worker | Yes (via vault) |
| `CELL_REGISTRY_REFRESH_MS` | API, worker | No |
| `REDIS_URL` | API, worker | Yes |
| `OBJECT_STORAGE_ENDPOINT`, `OBJECT_STORAGE_BUCKET` | API, worker | No (credentials via IAM role) |
| `KEY_SERVICE_URL`, `KEY_SERVICE_CLIENT_CERT_REF` | API, worker | Ref only |
| `JWT_COMPANY_KEYSET_REF`, `JWT_PLATFORM_KEYSET_REF` | API | Ref only (separate keys and audiences) |
| `PASSWORD_PEPPER_REF` | API | Ref only |
| `ACCESS_TOKEN_TTL`, `REFRESH_TOKEN_TTL` | API | No (also company settings) |
| `OTEL_EXPORTER_ENDPOINT`, `LOG_LEVEL` | All | No |
| `GSP_BASE_URL`, `GSP_MODE` (`sandbox`/`production`) | Worker | No (credentials per company in vault) |
| `FEATURE_FLAGS_SOURCE` | API, worker | No |
| `OFFLINE_PERMISSION_MAX_HOURS` (default 24) | Client config | No |
| `NUMBER_BLOCK_SIZE` (default 50; also a company setting) | API | No |
| `MIN_SUPPORTED_APP_VERSION` | API | No |

## Company-level settings (database, not env)
Token lifetimes, lockout limits and password rules; two-step requirement; business rules (valuation, negative stock, tax mode, discounts, posted-document policy, credit rule, posting mode); numbering formats; tax and statutory values with effective dates; approval rules; print layouts; offline limits; IP and device limits.

## Differences by environment
| Aspect | Dev | Staging | Production |
| --- | --- | --- | --- |
| GSP | Fake | Sandbox | Production |
| Payment gateway | Fake | Test mode | Live |
| Keys | Local | Staging KMS | Production KMS/HSM |
| Logging | Verbose | Normal | Normal, shipped off-host |
| Auth second step | Optional | Required for admins | Required for platform staff; company setting for others |
| Backups | None | Daily | Continuous with point-in-time |

Secrets are loaded at start, rotated on a schedule, scanned for in repositories and images.
