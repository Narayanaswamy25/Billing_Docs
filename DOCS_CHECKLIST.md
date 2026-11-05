# Billing Software: Documentation Checklist

Master list of the documents for the billing software project. Written files are in [docs/INDEX.md](docs/INDEX.md). Status: `[ ]` not started, `[~]` draft (needs owner input), `[x]` done. Items marked *(if relevant)* can be skipped when they do not apply.

## Core documents (create these first)

- [x] 1. **Project Overview (README / PROJECT.md)**: What the software is, who it's for, the problem it solves, goals, non-goals, tech stack summary, and how to run it.
- [x] 2. **Product Requirements (PRD)**: Features, user roles, user stories, acceptance criteria, and priorities (must-have vs. later).
- [x] 3. **Architecture Overview**: Components, how they communicate, deployment layout, and a diagram.
- [x] 4. **Tech Stack and Dependencies**: Every language, framework, and package with its version and the reason it was chosen.
- [x] 5. **Coding Rules for the LLM (CLAUDE.md or AGENTS.md)**: Naming conventions, style rules, do's and don'ts, and which docs to read before each kind of task.

## Frontend documents

- [x] 6. **Frontend Architecture**: Framework, state management, routing, and how components are organized.
- [~] 7. **Folder and File Structure (frontend)**: Directory tree, purpose of each folder and file, naming rules.
- [~] 8. **UI/UX and Design System**: Colors, fonts, spacing, reusable components, page layouts, wireframes.
- [~] 9. **Frontend Packages**: Libraries used, why, and any restrictions.

## Backend documents

- [x] 10. **Backend Architecture**: Pattern used (layered, MVC, microservices), request flow, module boundaries.
- [x] 11. **Folder and File Structure (backend)**: Directory tree and the purpose of each part.
- [x] 12. **API Specification**: Every endpoint with method, URL, request/response format, status codes, auth requirements (OpenAPI/Swagger ideal).
- [~] 13. **Backend Packages**: Libraries, versions, reasons.

## Data documents

- [~] 14. **Database Schema / Data Model**: Tables or collections, fields, types, relationships, ER diagram.
- [x] 15. **Indexing and Query Strategy**: Indexed fields, why, and expected query patterns.
- [x] 16. **Data Validation Rules**: Per field and input: allowed format, limits, required/optional, error messages.
- [~] 17. **Data Flow Diagrams**: How data moves from user input through the API to the database and back.

## Logic and behavior documents

- [x] 18. **Business Logic and Function Specifications**: For each key function: purpose, inputs, outputs, steps, edge cases, errors.
- [x] 19. **Workflows and User Flows**: Step-by-step flows such as sign-up, checkout, or report generation.
- [x] 20. **Error Handling and Logging**: Error codes, messages, retry rules, what gets logged.
- [x] 21. **State Machines** *(if relevant)*: Statuses an entity can have and the allowed transitions (e.g. invoice: draft → sent → paid).

## Security and operations

- [x] 22. **Authentication and Authorization**: Login method, token handling, roles, permissions matrix.
- [x] 23. **Security Policy**: Input sanitization, secrets management, rate limiting, data privacy.
- [x] 24. **Environment and Configuration**: Environment variables, config files, dev/staging/production differences.
- [x] 25. **Deployment and DevOps**: Hosting, CI/CD pipeline, build steps, rollback plan.

## Quality and process

- [x] 26. **Testing Strategy**: Test types, tools, coverage goals, sample test cases.
- [x] 27. **Third-Party Integrations**: External APIs or services, their limits, how failures are handled.
- [x] 28. **Roadmap and Task Breakdown**: Milestones, ordered tasks, and current status, so the LLM knows what to build next.
- [x] 29. **Decision Log (ADR)**: Why key choices were made, so they don't get undone.
- [x] 30. **Glossary**: Definitions of project-specific terms.

## Requirements and planning

- [x] 31. **Non-Functional Requirements**: Performance, scalability, uptime, browser/device support, response time targets.
- [x] 32. **Assumptions, Constraints, and Out-of-Scope**: Budget, deadlines, team size, technical limits, and what the software must not do.
- [x] 33. **Success Metrics (KPIs)**: How you will judge whether the project works.
- [x] 34. **Risk Register**: Known risks, impact, and mitigation.

## Frontend extras

- [x] 35. **Sitemap, Page Inventory, and Routes**: Every page and URL, components used, who can access it.
- [~] 36. **Component Inventory**: Reusable components with props, states, usage examples.
- [x] 37. **Accessibility Guidelines**: Standards (such as WCAG), keyboard navigation, screen-reader rules.
- [x] 38. **Internationalization and Localization**: Supported languages, translation file layout, date/currency/number formats.

## Backend and data extras

- [x] 39. **Background Jobs, Queues, and Scheduled Tasks**: Cron jobs, workers, retry rules, triggers.
- [x] 40. **Caching Strategy**: What is cached, where, for how long, when invalidated.
- [~] 41. **File Storage and Media Handling**: Upload limits, allowed types, storage location, naming rules.
- [x] 42. **Search Design** *(if relevant)*: Search engine, indexed fields, ranking rules, filters.
- [~] 43. **Notifications and Email/SMS Templates**: Each message, its trigger, content, and channel.
- [~] 44. **Data Migration, Seeding, and Fixtures**: How the database changes over time, initial data, sample test data.
- [~] 45. **Payments and Billing** *(if relevant)*: Pricing plans, payment provider flow, refunds, invoices, failure cases.
- [x] 46. **API Versioning and Changelog**: How breaking changes are handled and what changed per version.

## Security, legal, and compliance

- [x] 47. **Privacy and Compliance**: Data retention, user consent, deletion requests, rules such as GDPR or India's DPDP Act.
- [~] 48. **Terms of Service and Privacy Policy Drafts**: What you promise users about their data.
- [x] 49. **Audit and Activity Logging**: Which user and admin actions are recorded, and for how long.

## Operations and maintenance

- [~] 50. **Local Development Setup Guide**: Step-by-step setup on a new machine, prerequisites, common errors.
- [x] 51. **Monitoring and Observability**: Metrics, alerts, dashboards, health checks.
- [x] 52. **Backup and Disaster Recovery**: Backup schedule, restore steps, recovery time goals.
- [~] 53. **Runbook and Troubleshooting Guide**: Known problems and how to fix them.
- [x] 54. **Scalability Plan**: What changes when users or data grow 10x.

## Team and process

- [~] 55. **Git Workflow and Contribution Guide**: Branch naming, commit message format, pull request rules.
- [x] 56. **Definition of Done and Code Review Checklist**: What must be true before a task counts as finished.
- [x] 57. **Known Issues and Technical Debt**: Shortcuts taken and problems to fix later.
- [x] 58. **Release Notes / Changelog**: What shipped in each version.

## For projects using AI features *(if relevant)*

- [ ] 59. **Prompt Library and AI Behavior Spec**: Each prompt, inputs and outputs, model used, guardrails. (N/A: no AI features specified)
- [ ] 60. **AI Evaluation and Safety Plan**: How AI output quality is tested, wrong answers handled, cost controlled. (N/A: no AI features specified)

---

## Suggested folder layout

```
/docs
  INDEX.md
  00-overview/        (1, 2, 32, 33, 34)
  01-architecture/    (3, 4, 10, 29, 54)
  02-frontend/        (6, 7, 8, 9, 35, 36, 37, 38)
  03-backend/         (11, 12, 13, 39, 40, 41, 42, 43, 45, 46)
  04-data/            (14, 15, 16, 17, 44)
  05-logic/           (18, 19, 20, 21)
  06-security/        (22, 23, 47, 49)
  07-operations/      (24, 25, 50, 51, 52, 53)
  08-process/         (26, 28, 55, 56, 57, 58)
  CLAUDE.md           (5)
  GLOSSARY.md         (30)
```

Not placed by the layout above, so decide a home for them: 27 (Third-Party Integrations), 31 (Non-Functional Requirements), 48 (Terms and Privacy drafts), 59 and 60 (AI docs). Suggested: 27 → `03-backend/`, 31 → `00-overview/`, 48 → `06-security/`, 59/60 → a new `09-ai/`.

## Tips for keeping the docs LLM-friendly

- Keep each document focused on one topic; smaller files are easier to load and follow.
- Keep a short `INDEX.md` in `/docs` listing every document and when to read it.
- Be specific, not vague: "Email must match RFC 5322 and be under 254 characters" beats "validate email."
- Include examples: sample requests/responses, data rows, error messages.
- Keep docs updated as the project changes, or the LLM will build from stale information.
- For a small project, start with 1, 3, 4, 5, 12, 14, 16, and 28.
