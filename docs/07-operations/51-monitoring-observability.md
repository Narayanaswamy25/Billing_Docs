# Monitoring and Observability

Every request carries a company ID and a trace ID; the trace ID travels inside queue jobs so a slow e-invoice can be followed from the tap to the portal.

## Metrics
| Group | Metrics | Alert |
| --- | --- | --- |
| API | Rate, errors, duration per route; split read vs post latency | p95 read > 300 ms, post > 800 ms |
| Node | Event-loop delay, heap vs limit, container memory | p99 loop delay > 100 ms; heap growth |
| MongoDB | Transaction abort and retry rate, connections per cell and per copy, index size vs memory, per-company share of cell resources, replication lag, cache use, disk growth | Abort > 1%; connections > 70%; lag; one company above its share of the cell |
| Queues | Depth and age per queue, job duration, failures, dead-letter count | Critical queue age; dead letters > 0 |
| Cache | Hit rate, invalidation delays | Low hit rate |
| Business | Invoices posted per minute, failed e-invoices, pending e-invoice age, offline bills pending/conflict, number gaps | Failed e-invoices; ageing before reporting window |
| Client | Crash-free sessions, sync status, offline chip counts | Crash spike (no personal data in reports) |
| Security | Failed sign-ins, denied requests, large exports, support sessions, key-service use | See [49-audit-logging.md](../06-security/49-audit-logging.md) |
| Outside | GSP and gateway error rate | Provider outage must not look like ours |

## Dashboards
Per-company health page for platform staff: cell, version, usage, failed jobs, portal errors. Cell dashboard: the measures above. Release dashboard: error rate and latency by version.

## Health checks
Start-up, liveness, readiness (see [25-deployment-devops.md](25-deployment-devops.md)); a sick cell marks only its own companies unavailable. Public status page; synthetic checks sign in and post a test invoice in a sandbox company every few minutes.

## Logs
JSON lines, structured, asynchronous, shipped off host within seconds, kept at least 180 days rolling in an India region; no personal data or secrets; access to logs is logged.

## Tracing
OpenTelemetry spans for each use case, database call and adapter call.

## On-call
Named roles for triage; page for cross-company errors, key-service failures, failed backups; ticket for everything else. Thresholds above are starting values to tune after the first load test.
