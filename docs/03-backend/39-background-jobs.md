# Background Jobs, Queues and Scheduled Tasks

## How events leave the database
1. A module writes its event to `outbox` in the same transaction as the change.
2. A relay watches the outbox (one change stream per cell, filtered to the outbox collection) and stores a resume token so a restart loses nothing.
3. The relay puts a job on a Redis queue and marks the row sent.
4. Delivery is at least once; every handler checks an event ID before acting.

## Queues
Three priorities: **critical** (e-invoice, payment confirmations), **default** (messages, webhooks), **bulk** (reports, imports, exports). A job carries company ID and record IDs, never documents; the worker rebuilds the tenant context with `withTenant(companyId, …)` and re-checks that the company is active. Per-company running-job limit. Failed jobs retry with growing delay and jitter; after the last retry they go to a dead-letter queue that platform staff can see and replay. A scheduler adds daily and monthly jobs per company, skipping companies whose module is off.

## Jobs
| Job | Trigger | Queue |
| --- | --- | --- |
| E-invoice generate / cancel; e-way bill generate, update, close | `invoice.posted`, transport entry | critical |
| Payment link status, gateway callbacks | Callback | critical |
| Send message, payment reminder, offer campaigns | Event or daily | default |
| Webhook delivery | Event | default |
| Recurring invoices | Daily | default |
| Low-stock and expiry alerts | Daily | default |
| Pending e-invoice ageing alert (new) | Daily | default |
| Bank feed sync, GSTR-2B download and matching | Daily or on demand | bulk |
| Depreciation run, year-end closing | On demand or monthly | bulk |
| Scheduled and custom reports, summary tables | Schedule or on demand | bulk |
| Data import, data export, accounting export, company export | On demand | bulk |
| Usage meters, renewals, failed-payment retries | Hourly and daily (platform) | default |
| Schema migration per company | Release | bulk |
| Demand statistics (ADD, σ, seasonal) | Nightly | bulk |
| Restock scan and suggestions | Hourly or nightly | default |
| Expiry and dead-stock scan | Daily | default |
| Customer dunning ladder | Daily | default |
| Supplier dues, MSME and 180-day watch | Daily | default |
| Compliance calendar (GST, TDS, licence, scale verification) | Daily and weekly | default |
| Lead-time learning, supplier scorecard | On goods receipt | default |
| Auto-draft purchase orders (if enabled) | Nightly | default |
| Reminder digest delivery | At set times | default |
| Revision compaction, recycle-bin archive, file version expiry, retention enforcement | Nightly / daily / weekly | bulk |
| Company backup export and manifest check | Nightly | bulk |
| Backup verification restore test | Monthly per cell | bulk |
| Sensitive-change digest | Daily (immediate for Critical) | default |
| Payroll build | — | deferred (D-007) |

## Heavy work techniques
PDFs: small pool of headless browser instances, cached HTML templates, concurrency limit, result stored once per document version. Excel/CSV: stream from a cursor to object storage. Import: stream, validate in batches, bulk write in small transactions, produce an error file. Dashboards: daily incremental summaries merged into summary collections. Stock revaluation and year-end closing: batches by item, resumable from a cursor. Large exports over a set row count run in the background; the person is notified and the link expires.

## Locks
Distributed lock only where unavoidable (migrations, scheduler leader). Elsewhere use unique indexes and conditional updates.
