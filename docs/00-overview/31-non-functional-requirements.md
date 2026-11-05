# Non-Functional Requirements

All values are starting targets to agree with first customers and prove by test, not promises.

| Area | Target | Proved by |
| --- | --- | --- |
| Availability | Company API 99.9% (about 43 min downtime per 30-day month) | Uptime monitoring |
| Read latency | p95 under 300 ms | Load test per release candidate |
| Post latency | p95 under 800 ms | Load test; quick test on PRs touching posting |
| Data loss after failure | Up to 5 minutes (point-in-time recovery) | Restore drill |
| Restore a failed cell | Within 1 hour | Annual full-cell drill |
| Revert or self-undo of a record | Under 2 seconds | Performance test |
| Restore as of a date (masters) | Preview in under 30 s for 100,000 records; apply in minutes | Load test |
| One company restored by the provider | Within the plan's hours (proposed 4 to 8) | Quarterly drill |
| Counter add item | Under 100 ms | Device test |
| Panel list opens | Under 1 s on a mid-range phone | Device test |
| Search results | Under 300 ms | Device test |
| Counter first scan after opening | About 2 s on a mid-range counter device | Device test |
| Local bill save | Well under 1 s | Device test |
| List of 10,000 rows | Scrolls without delay | Widget/perf test |
| Transaction abort rate | Under 1% | Metric and alert |
| Event-loop delay p99 | Under 100 ms | Metric and alert |
| DB connections | Under 70% of a cell's limit | Metric and alert |
| First bill time | Under 3 minutes | User test, 5 staff |
| Vulnerability fix | Critical 2 days, High 7, Medium 30, Low next release | Register |

## Device and browser support
- Counter devices tested: low-end Android phone, tablet in portrait, Windows PC with 80 mm thermal printer and scanner, Mac.
- Desktop window minimum 800×600. Layout steps: wide ≥1024, medium 600–1023, narrow <600 (starting values; test on real devices).
- Tap targets 48 px or larger; text scales to the largest system size.

## Scale
Lakhs of items and years of invoices per company: server-side paging and search only; never load everything. Companies share cells; a cell's company cap and sharding trigger come from load tests (54-scalability-plan); heavy companies can move to a dedicated cell (D-043).

## Languages
English, Hindi, Tamil at launch. Strings written short first. Document language and screen language are separate settings.
