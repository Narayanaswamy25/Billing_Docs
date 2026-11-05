# Success Metrics (KPIs)

| Metric | Target | How measured |
| --- | --- | --- |
| Time to first bill | Under 3 minutes | Timed test with five real shop staff; later, first-bill event minus sign-up event |
| Taps for a repeat invoice or stock entry | 5 or fewer | Usability test |
| Choices on any one screen | About 7 or fewer | Design review checklist |
| "Where is X?" support questions | Falling after My access ships | Support tags |
| Permission matrix | 100% pass, no exceptions, every release | CI |
| Cross-company leak tests | 0 successes | CI isolation suite |
| Unexplained invoice-number gaps | 0 | Gaps report |
| Duplicate invoices after offline sync | 0 | Offline test suite and idempotency metric |
| Failed e-invoices older than 24 h | Trending to 0 | Dashboard alert count |
| Post latency p95 | Under 800 ms | Load test and production metric |
| Restore drill | Single company restored from snapshot to a temporary cluster quarterly; full cell yearly | Drill log |

Treat three panels for every module, read-then-Edit, and the one-primary-button form as hypotheses (D-010): judge them by the first three rows before building more.
