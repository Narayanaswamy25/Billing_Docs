# Shared Database Tenancy: overview

**Decision D-039 (owner, 2026-10-31):** all companies share the same database and collections. Each company's documents are marked with a `companyId`, and the system is built so that **no query, job, cache, file, export, log or person can ever reach another company's data**.

This replaces the earlier "one database per company" design (D-003). The earlier design gave a hard wall between companies for free. A shared database does not, so the wall is built in software and checked at many levels. This folder explains how, how data is processed, and how we prove it holds.

## What changes and what stays
| Topic | Before (database per company) | Now (shared database) |
| --- | --- | --- |
| Boundary between companies | The database itself | `companyId` on every document, enforced by one mandatory data access layer and checked again at several other levels |
| Collection and file count | 65 collections × companies; hit MongoDB's file limit (F-14) | 66 collections in total per cluster; the file-count limit stops mattering |
| Cost and onboarding | A new database per company | A company is rows, not a database; provisioning takes seconds |
| Schema migration | One job per company | One migration for all, run in tenant batches with canary companies first |
| Restore one company | Restore one database | Restore a snapshot to a temporary cluster, extract that company, import (see 04); plus nightly per-company encrypted exports |
| Delete a company | Drop its database | A verified **purge** by `companyId` across all collections, files, caches and keys (see 04) |
| Noisy neighbour | Contained by the database and cell | Contained by quotas, rate limits, workload separation, and moving heavy companies to their own cell (see 01) |
| Main risk | A bug stays inside one database | **One data-access bug could expose many companies**, so isolation is defended in depth and tested continuously |
| Unchanged | Identity model (D-025), entitlements, permissions, roles, five checks, field rules, audit, number blocks, Flutter apps, Express backend, Atlas |

## What "private and secure" means here
1. **No cross-company reads or writes**, by construction (layer 1 to 3) and by test (05).
2. **No shared business data between companies**: no common customer, item or price catalogue contributed by companies; only platform-owned reference data (tax codes, state codes) is shared and it contains no company data.
3. **Sensitive data stays unreadable** without that company's keys even if the database is copied (03).
4. **Every access is attributable**: who, which company, which record, why (audit), and staff have no standing access to company data (D-023).
5. **Data is processed only for the company's purposes** and only in the places listed in the processing map (02).
6. **Deleted means deleted**: a documented purge with a certificate (04).

## Documents
| # | Document | Answers |
| --- | --- | --- |
| 01 | [Isolation architecture](01-isolation-architecture.md) | The ten layers that keep companies apart, the data access layer, scaling |
| 02 | [Data processing and lifecycle](02-data-processing-lifecycle.md) | How data is collected, processed, stored, shared, analysed, retained and deleted; who can touch it; sub-processors |
| 03 | [Encryption and keys in a shared database](03-encryption-and-keys.md) | Per-company keys, what is and is not encrypted, crypto-shredding limits, key handling |
| 04 | [Company lifecycle and operations](04-tenant-lifecycle-and-operations.md) | Provisioning, migration, backup and restore of one company, purge, moving a company, break-glass, exposure playbook |
| 05 | [Isolation tests and assurance](05-isolation-tests-and-assurance.md) | Test suite, canaries, static checks, runtime monitors, release gate |

## Decisions (see the decision log)
D-039 shared database · D-040 mandatory data access layer · D-041 per-company keys and honest limits · D-042 processing principles · D-043 dedicated cell option · D-044 no shared business data · D-045 cross-company reports done per company · D-046 single-company restore method · D-047 purge on exit.

## One honest warning
A shared database is cheaper and simpler to run, and it is also the design where a single coding mistake matters most. This is why the next four documents are strict. The rule for developers is in CLAUDE.md: **never touch the database driver directly; never take a company ID from the request; never write a query that could return more than one company's data.**
