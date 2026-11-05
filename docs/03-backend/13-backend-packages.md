# Backend Packages

Unpinned; pin exact versions in the lock file. "Verify" items are in [REVIEW-FINDINGS.md](../REVIEW-FINDINGS.md) F-19.

| Need | Package or tool | Reason | Restriction |
| --- | --- | --- | --- |
| Framework | `express` (current stable major), `helmet`, an origin allow-list for CORS, Redis-backed rate limiting | Owner's choice (D-021); minimal core | Middleware order fixed in 10-backend-architecture; no framework-provided DI, so a composition root wires dependencies |
| Database | `mongodb` (official driver) | Raw control, per-cell client, transactions | No ODM; no Prisma (D-002) |
| Validation | Zod (or the schema library chosen once) plus `$jsonSchema` generation | One definition of each field | Reject unknown fields; schema is the OpenAPI source |
| OpenAPI | A schema-to-OpenAPI generator (e.g. zod-to-openapi) fed from the same schemas that validate input | Docs and Flutter client generation | Build fails on hidden breaking change |
| Queues | `bullmq`, Redis client | Priorities, retries, dead letters | Per-company concurrency limits |
| Auth | `argon2`, JWT library with key rotation, TOTP library | Reviewed primitives | No home-made crypto |
| Crypto | Node `crypto` or `libsodium` | AES-256-GCM, HMAC | Fresh random IV every encryption |
| Observability | OpenTelemetry SDK, Prometheus client, structured logger | Traces and metrics with company ID | No personal data in logs |
| PDF | headless browser pool | HTML templates → PDF | Concurrency limited, cached templates |
| Excel | streaming writer | Large exports without memory blowup | Neutralise cells starting `= + - @` |
| Files | S3-compatible SDK | Signed links | Virus scan before release |
| Money | internal `money` package over Decimal128 | No float amounts | Only way to do arithmetic |
| Testing | Jest or Vitest, containerised MongoDB replica set, supertest-style API tests | Real transactions | No in-memory Mongo for posting tests |
| Quality | ESLint + TypeScript rules, Prettier, module-boundary rule, dependency/secret/container scanners | Gates in section 21 of the architecture | High finding blocks merge |

Dependency policy: weekly automatic update PRs; patch and minor merge on green; major updates are planned work; security fixes first; waiting period before adopting brand-new releases; software bill of materials and signed images on every build.
