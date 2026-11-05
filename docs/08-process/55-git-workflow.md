# Git Workflow and Contribution Guide

Proposed; adjust when the team exists.

## Branches
- `main`: always releasable, protected; merges only through pull requests with a green pipeline and one reviewer from the owning module (code-owners file).
- Short-lived branches merged often: `feat/<module>-<short-name>`, `fix/<module>-<short-name>`, `docs/<topic>`, `chore/<topic>`. Unfinished features hide behind feature flags.
- Releases are tags `vMAJOR.MINOR.PATCH` from `main`.

## Commits
Conventional style: `type(scope): summary` with types `feat, fix, docs, refactor, test, chore, perf, security`; scope is a module key (e.g. `sales`, `stock`, `identity`). Summary under 72 characters, imperative. Body explains why. Reference the decision-log or finding ID when relevant (e.g. `D-005`, `F-14`).

## Pull requests
Template sections: what and why; docs changed (required for any behaviour change); decision-log row if a large choice; permission keys added or changed; migration and index changes; screenshots (client); test evidence. Checklist is the Definition of Done ([56](56-definition-of-done.md)).

## Rules
- No secrets, customer data or production dumps in the repository.
- No force-push to `main`; no merging with a failing pipeline.
- Dependency updates arrive as automatic weekly PRs.
- Security fixes may use a private branch and are disclosed after release.
- Schema changes follow expand-then-contract across releases.
- Reviews check the reviewer questions in the Definition of Done.
