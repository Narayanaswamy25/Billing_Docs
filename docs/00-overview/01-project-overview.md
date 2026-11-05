# Project Overview

## What it is
A billing and business-management product for Indian businesses, sold as a service. One product serves retail, restaurant, services and wholesale companies; the platform owner (super admin) switches modules and features on or off per company. All companies share one secured database cluster, but each company's data is isolated, encrypted with its own keys, and never visible to another company or to the provider's staff.

## Who it is for
| Person | Uses | Starts on |
| --- | --- | --- |
| Super admin (platform owner) | Console | Company list |
| Company admin | Workspace | Dashboard with setup checklist |
| Manager | Workspace | Dashboard, approvals waiting |
| Accountant | Workspace | Dashboard, items to post |
| Billing officer | Workspace | New invoice |
| HR officer | Workspace | HR |
| Salesperson | Counter | New bill |
| Stock employee | Workspace | Stock entry form |

## Problem it solves
Small and mid-size Indian businesses need GST-correct billing, stock and accounts that a counter employee can use in minutes, that keeps working when the connection drops, and where an owner decides exactly who sees which screen and field.

## Goals
- First bill in under 3 minutes from signing up.
- GST-correct invoices: CGST/SGST/IGST split, e-invoice, e-way bill, TDS/TCS, return data ready to file.
- Counters keep billing offline and sync later without duplicate or unexplained invoice numbers.
- One rule for access: each level can only hand down what it was given (super admin → company admin → role → person).
- A stolen database or backup is unreadable without keys held elsewhere; a breach of one company never reaches another.

## Non-goals (release 1)
Payroll (PF, ESI, payslips), submitting returns to the GST portal, manufacturing/BOM, service jobs and AMC, online-store sync, loyalty points, multi-currency, offline Workspace. See [32-assumptions-scope.md](32-assumptions-scope.md).

## Tech summary
Flutter clients (D-001) · Node.js + Express + TypeScript backend (D-021) · MongoDB Atlas, shared database with strict per-company isolation (D-012, D-039) · Redis + BullMQ queues · S3-compatible storage · GSP adapter for e-invoice. Details: [04-tech-stack.md](../01-architecture/04-tech-stack.md).

## How to run it
Not yet built. Local setup will live in `07-operations/50-local-setup.md` once the repository exists. Build order: [28-roadmap.md](../08-process/28-roadmap.md).

## Where things are
Start at [INDEX.md](../INDEX.md). Problems found in the original specs: [REVIEW-FINDINGS.md](../REVIEW-FINDINGS.md).
