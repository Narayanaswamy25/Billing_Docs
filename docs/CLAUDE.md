# CLAUDE.md: rules for the LLM working on this project

Read this file first. Then read [INDEX.md](INDEX.md) and only the docs the task needs.

## What this is
Multi-tenant billing and business software for Indian businesses (GST). Three apps (Console, Workspace, Counter) over one Express (Node.js) backend with shared MongoDB collections isolated by company (D-039). Docs are the source of truth; code follows them.

## Before any task
| Task | Read first |
| --- | --- |
| Any screen or widget | 02-frontend/08-ui-ux-design-system.md, 35-sitemap-routes.md, 37-accessibility.md |
| Permissions, roles, visibility, sign-in, plans | 06-security/22-authentication-authorization.md, then 06-security/access/00-README.md |
| Posting, numbering, tax, stock | 05-logic/18-business-logic.md, 21-state-machines.md |
| Inventory, restocking, reminders, returns, pricing, farm or trader flows, Indian Acts | 05-logic/advanced/00-README.md, then the numbered file for the topic |
| New API endpoint | 03-backend/12-api-specification.md, 05-logic/20-error-handling.md |
| New collection or field | 04-data/14-data-model.md, 16-validation-rules.md |
| Anything touching secrets, PII, keys | 06-security/23-security-policy.md |
| Change tracking, undo, restore, backups, audit of who changed what | 05-logic/history-and-backup/00-README.md, then 01 and 02 |
| Any data access, new collection or query, jobs, caches, files, exports (company isolation) | 06-security/tenancy/00-README.md, then 01 and 05 |
| Deciding between options | 01-architecture/29-decision-log.md and REVIEW-FINDINGS.md |

## Hard rules
1. **Server is the authority.** The app only decides what to draw. Every request runs the five checks (module on, role verb, no personal deny, record in scope, field visible).
2. **Permission keys:** `<module>.<screen>.<verb>`, lowercase, the 15 verbs only. Never invent `see`, `edit`, `full`, `own` (retired).
3. **A missing module or forbidden screen is absent, never greyed out.** Blocked pages say "This page is not available. Contact your admin." in every case.
4. **Money is Decimal128 through the money service.** Never a JavaScript number or Dart double for amounts.
5. **Posted documents are never edited.** Fix by credit note, debit note or reversing entry.
6. **Invoice numbers are given at post, never at draft save, max 16 characters.** Count separators.
7. **Every company-owned document has an immutable `companyId`; all data access goes through the tenant data access layer (`packages/data-access`). Never import the MongoDB driver anywhere else, never take a company ID from request input, never write a query that could return more than one company's data.** Read 06-security/tenancy/00-README.md before touching data access.
8. **Tax rates, thresholds, windows and rules are dated settings, never constants.**
9. **No secrets, customer names, phones, GSTINs or amounts in logs or crash reports.**
10. **One module owns its collections.** Other modules call its public service or listen to its events.
11. **Never edit production data by hand.** Repairs are reviewed scripts.
12. **Every change to company data writes a change event and, for editable classes, a revision, in the same transaction. History is never deleted or edited. Posted documents and ledgers are reversed with linked documents, never edited or reverted.** Read 05-logic/history-and-backup/00-README.md before touching writes.
13. Do not resolve an open decision (status Open or Proposed in the decision log) silently. Ask, or follow the proposed default and say so.

## Style
- Backend: TypeScript strict, Express, layers `http → application → domain ← data`. Domain is pure. Functions under 40 lines, files under 300. No `any`.
- Client: Flutter, Bloc, `go_router`; widgets read from the shared design tokens, no hardcoded colours or spacing. Strings go through localization, short first.
- Names are business words (Invoice, GoodsReceipt, PostInvoice). Comments say why.
- Tests are named as sentences and live beside the code.

## Keeping docs honest
- Change the doc in the same change as the code. Add a decision-log row for any large choice.
- If two docs disagree, stop and record it in REVIEW-FINDINGS.md instead of picking quietly.
