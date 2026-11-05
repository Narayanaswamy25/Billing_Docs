# Docs Index

Start with [CLAUDE.md](CLAUDE.md), then [REVIEW-FINDINGS.md](REVIEW-FINDINGS.md) and [DRAWBACKS-REVIEW.md](DRAWBACKS-REVIEW.md) (risks, costs and weak spots of the current design) (what was wrong in the source specs and what was decided). Master checklist: [../DOCS_CHECKLIST.md](../DOCS_CHECKLIST.md).

Status: **Done** (written from the source specs) · **Draft** (written, but facts or wording from the owner are still needed) · **N/A**.

## 00 Overview
| # | Document | Read when | Status |
| --- | --- | --- | --- |
| 1 | [Project overview](00-overview/01-project-overview.md) | First | Done |
| 2 | [PRD](00-overview/02-prd.md) | Scoping any feature | Done |
| 31 | [Non-functional requirements](00-overview/31-non-functional-requirements.md) | Performance, uptime, devices | Done |
| 32 | [Assumptions, constraints, out of scope](00-overview/32-assumptions-scope.md) | Before adding scope | Done |
| 33 | [Success metrics](00-overview/33-success-metrics.md) | Judging a release | Done |
| 34 | [Risk register](00-overview/34-risk-register.md) | Planning | Done |

## 01 Architecture
| # | Document | Read when | Status |
| --- | --- | --- | --- |
| 3 | [Architecture overview](01-architecture/03-architecture-overview.md) | First technical read | Done |
| 4 | [Tech stack](01-architecture/04-tech-stack.md) | Choosing a library | Done (pin versions) |
| 4a | [Products and services needed](01-architecture/products-and-services-needed.md) | Choosing vendors, budgeting | Draft: D-055 to D-060, no prices yet |
| 10 | [Backend architecture](01-architecture/10-backend-architecture.md) | Writing backend code | Done |
| 29 | [Decision log](01-architecture/29-decision-log.md) | Before changing a decision | Done |
| 54 | [Scalability plan](01-architecture/54-scalability-plan.md) | Shared cells, sharding, fairness | Done |

## 02 Frontend
| # | Document | Status |
| --- | --- | --- |
| 6 | [Frontend architecture](02-frontend/06-frontend-architecture.md) | Done (Flutter, D-001) |
| 7 | [Folder and file structure](02-frontend/07-folder-structure.md) | Draft (layout proposed) |
| 8 | [UI/UX and design system](02-frontend/08-ui-ux-design-system.md) | Draft: token values and wireframes needed |
| 9 | [Frontend packages](02-frontend/09-frontend-packages.md) | Draft (unpinned) |
| 35 | [Sitemap, pages, routes](02-frontend/35-sitemap-routes.md) | Done |
| 36 | [Component inventory](02-frontend/36-component-inventory.md) | Draft (props proposed) |
| 37 | [Accessibility](02-frontend/37-accessibility.md) | Done (WCAG 2.2 AA proposed) |
| 38 | [Internationalization](02-frontend/38-internationalization.md) | Done |

## 03 Backend
| # | Document | Status |
| --- | --- | --- |
| 11 | [Folder and file structure](03-backend/11-folder-structure.md) | Done |
| 12 | [API specification](03-backend/12-api-specification.md) | Conventions + first endpoints; rest generated from code |
| 13 | [Backend packages](03-backend/13-backend-packages.md) | Draft (unpinned) |
| 27 | [Third-party integrations](03-backend/27-third-party-integrations.md) | Done; GSP not chosen |
| 39 | [Background jobs](03-backend/39-background-jobs.md) | Done |
| 40 | [Caching strategy](03-backend/40-caching-strategy.md) | Done |
| 41 | [File storage and media](03-backend/41-file-storage.md) | Draft (size limits not set) |
| 42 | [Search design](03-backend/42-search-design.md) | Done |
| 43 | [Notifications and templates](03-backend/43-notifications-templates.md) | Draft: wording needed |
| 45 | [Payments and billing](03-backend/45-payments-billing.md) | Draft: prices and periods needed |
| 46 | [API versioning and changelog](03-backend/46-api-versioning-changelog.md) | Done |

## 04 Data
| # | Document | Status |
| --- | --- | --- |
| 14 | [Data model](04-data/14-data-model.md) | Draft: field dictionary needs the missing source document |
| 15 | [Indexing and query strategy](04-data/15-indexing-query-strategy.md) | Done |
| 16 | [Validation rules](04-data/16-validation-rules.md) | Done for shared fields; per-form rules as forms are built |
| 17 | [Data flow diagrams](04-data/17-data-flow-diagrams.md) | Draft (text diagrams) |
| 44 | [Migration, seeding, fixtures](04-data/44-migration-seeding-fixtures.md) | Draft: role tick seeds, chart of accounts, tax settings need input |

## 05 Logic
| # | Document | Status |
| --- | --- | --- |
| 18 | [Business logic](05-logic/18-business-logic.md) | Done |
| 19 | [Workflows](05-logic/19-workflows.md) | Done |
| 20 | [Error handling and logging](05-logic/20-error-handling.md) | Done |
| 21 | [State machines](05-logic/21-state-machines.md) | Done |
| 18a | [Advanced logic: overview](05-logic/advanced/00-README.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 18b | [Business profiles](05-logic/advanced/01-business-profiles.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 18c | [Inventory engine](05-logic/advanced/02-inventory-engine.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 18d | [Restocking and reminders](05-logic/advanced/03-restocking-and-reminders.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 18e | [Selling and pricing](05-logic/advanced/04-selling-and-pricing.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 18f | [Returns and reverse flows](05-logic/advanced/05-returns-and-reverse-flows.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 18g | [Farm and trader flows](05-logic/advanced/06-farm-and-trader-flows.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 18h | [Acts and compliance map](05-logic/advanced/07-compliance-acts-map.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 18i | [End-to-end workflows](05-logic/advanced/08-end-to-end-workflows.md) | Draft: proposals D-032 to D-036, legal rows marked Verify |
| 21a | [History and backup: overview](05-logic/history-and-backup/00-README.md) | Draft: proposals D-048 to D-054 |
| 21b | [Change tracking](05-logic/history-and-backup/01-change-tracking.md) | Draft: proposals D-048 to D-054 |
| 21c | [Revert and restore](05-logic/history-and-backup/02-revert-and-restore.md) | Draft: proposals D-048 to D-054 |
| 21d | [Backups and restore points](05-logic/history-and-backup/03-backup-and-restore-points.md) | Draft: proposals D-048 to D-054 |
| 21e | [History API, data and tests](05-logic/history-and-backup/04-history-api-data-tests.md) | Draft: proposals D-048 to D-054 |

## 06 Security
| # | Document | Status |
| --- | --- | --- |
| 22 | [Authentication and authorization](06-security/22-authentication-authorization.md) | Done |
| 23 | [Security policy](06-security/23-security-policy.md) | Done |
| 47 | [Privacy and compliance](06-security/47-privacy-compliance.md) | Done; lawyer review needed |
| 48 | [Terms and privacy: outline](06-security/48-terms-privacy-drafts.md) | Draft outline only; a lawyer writes the text |
| 49 | [Audit and activity logging](06-security/49-audit-logging.md) | Done |
| 22a | [Access design: overview](06-security/access/00-README.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22b | [Identity and sign-in](06-security/access/01-identity-and-sign-in.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22c | [Entitlements and plans](06-security/access/02-entitlements-and-plans.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22d | [Permission catalog](06-security/access/03-permission-catalog.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22e | [Role seeds and limits](06-security/access/04-role-seeds-and-limits.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22f | [User lifecycle](06-security/access/05-user-lifecycle.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22g | [Platform and support security](06-security/access/06-platform-and-support.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22h | [Auth and access API](06-security/access/07-auth-and-access-api.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22i | [Access threats and tests](06-security/access/08-threats-and-tests.md) | Draft: proposals D-025 to D-031 need owner sign-off |
| 22j | [Tenancy: overview](06-security/tenancy/00-README.md) | Shared-database isolation (D-039 accepted; D-040 to D-047) |
| 22k | [Isolation architecture](06-security/tenancy/01-isolation-architecture.md) | Shared-database isolation (D-039 accepted; D-040 to D-047) |
| 22l | [Data processing and lifecycle](06-security/tenancy/02-data-processing-lifecycle.md) | Shared-database isolation (D-039 accepted; D-040 to D-047) |
| 22m | [Encryption and keys](06-security/tenancy/03-encryption-and-keys.md) | Shared-database isolation (D-039 accepted; D-040 to D-047) |
| 22n | [Company lifecycle and operations](06-security/tenancy/04-tenant-lifecycle-and-operations.md) | Shared-database isolation (D-039 accepted; D-040 to D-047) |
| 22o | [Isolation tests and assurance](06-security/tenancy/05-isolation-tests-and-assurance.md) | Shared-database isolation (D-039 accepted; D-040 to D-047) |

## 07 Operations
| # | Document | Status |
| --- | --- | --- |
| 24 | [Environment and configuration](07-operations/24-environment-configuration.md) | Done (variable names proposed) |
| 25 | [Deployment and DevOps](07-operations/25-deployment-devops.md) | Done; hosting undecided |
| 50 | [Local development setup](07-operations/50-local-development-setup.md) | Draft: needs the repository |
| 51 | [Monitoring and observability](07-operations/51-monitoring-observability.md) | Done |
| 52 | [Backup and disaster recovery](07-operations/52-backup-disaster-recovery.md) | Done |
| 53 | [Runbook and troubleshooting](07-operations/53-runbook-troubleshooting.md) | Draft: owners and contacts needed |

## 08 Process
| # | Document | Status |
| --- | --- | --- |
| 26 | [Testing strategy](08-process/26-testing-strategy.md) | Done |
| 28 | [Roadmap](08-process/28-roadmap.md) | Done (no dates) |
| 55 | [Git workflow](08-process/55-git-workflow.md) | Draft (proposed) |
| 56 | [Definition of done](08-process/56-definition-of-done.md) | Done |
| 57 | [Known issues and technical debt](08-process/57-known-issues-technical-debt.md) | Done |
| 58 | [Release notes / changelog](08-process/58-release-notes-changelog.md) | Template; nothing shipped |
| 61 | [Backend status report](08-process/61-backend-status-report.md) | Audit 2026-11-05: done, not done, bugs, code vs docs |
| 62 | [Frontend status report](08-process/62-frontend-status-report.md) | Audit 2026-11-05: done, not done, bugs, code vs docs |

## Root
| # | Document | Status |
| --- | --- | --- |
| 5 | [CLAUDE.md](CLAUDE.md) | Done |
| 30 | [GLOSSARY.md](GLOSSARY.md) | Done |
| 59–60 | AI prompt library and AI safety | N/A: no AI features in the specs |
