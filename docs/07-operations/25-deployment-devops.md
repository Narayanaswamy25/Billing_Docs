# Deployment and DevOps

## Topology
| Part | Setup |
| --- | --- |
| API | At least three stateless copies behind a load balancer across two or three zones |
| Workers | Scaled by queue depth; separate from the API |
| Platform database | Own replica set, smaller, tightly guarded, backed up more often |
| Cells (shared by many companies) | Each an Atlas cluster, a replica set of three or more nodes in different zones (transactions need a multi-node replica set); sharded later if needed |
| Redis | Highly available (managed or replicas with failover) |
| Files and backups | Object storage with versioning, copy in another region, separate account for backups |
| Key service | Two or more copies, reachable only from API and workers over mutual TLS, only caller of the KMS |
| Environments | Development, CI test, staging mirroring production, production, sandbox cell for partner and GSP testing |

**MongoDB Atlas is the database host (D-012).** Each cell is an Atlas cluster (replica set, at least three nodes across zones) and the platform database is its own cluster, in a separate Atlas project from company cells. Atlas provides storage encryption, Cloud Backup with point-in-time restore, private endpoints, monitoring and alerts, and rolling upgrades, so no database administrator or host-level disk encryption is needed. Choices still open (D-014): which cloud (AWS, Azure or Google Cloud) and which India region; confirm that region offers the cluster tier needed and check the tier's limits on collections, connections and storage before sizing a cell. Cost grows with the number of clusters, which links to the pooled-trial question (D-003).

Recommended hosting and tools (AWS Mumbai with Hyderabad, ECS on Fargate, managed Redis, S3, KMS, SES, GitHub Actions, Terraform, Sentry, Grafana): [products-and-services-needed.md](../01-architecture/products-and-services-needed.md), D-057 and D-059.

## Release pipeline
1. CI runs lint, types, unit tests, API tests on a real replica set, the two-company isolation suite, contract checks and security scans (see [26-testing-strategy.md](../08-process/26-testing-strategy.md)).
2. Image goes to staging; migrations run on test companies.
3. Production: internal companies first, then the canary cell, then the rest, rolling updates with health checks.
4. Schema changes are **expand then contract**: new code reads both shapes, a migration moves data, a later release drops the old shape; the code supports the current and previous schema version, so rollback is a redeploy.
5. Index builds are rolling builds outside business hours, one collection at a time (a build can wait behind transactions on the collection); every index starts with `companyId`.
6. Feature flags per company and per cell are separate from the super admin's module switches and are removed once stable.
7. Migration runner: company batches, canary companies first, pause and resume; each migration runs twice on a copy to prove it is repeatable (tenancy/04).

## Setting up a company (provisioning)
Seconds, no database to create: registry row, company keys, seed documents stamped with `companyId`, first admin invitation, entitlements snapshot, status `Ready`. Idempotent and reversible (tenancy/04).

## Reliability in the process
Start-up check (config valid, registry reachable), liveness (process answers, event loop not stuck), readiness (platform DB, Redis, the cells it serves). A sick cell marks only its own companies unavailable. Graceful shutdown: fail readiness, finish requests, finish current job, close MongoDB and Redis; container grace period exceeds the slowest request. Config typed and validated at start; the process refuses to start if one is missing; no defaults for secrets.

## Rollback
Redeploy the previous image (schema is expand-then-contract). Data fixes only through reviewed, logged repair scripts; nobody edits production data by hand.

## Not yet decided
Cloud provider and region, Kubernetes vs managed containers, CI product, domain names, status page tool.
