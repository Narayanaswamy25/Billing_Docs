# Definition of Done and Code Review Checklist

A change is finished only when every box is ticked.

- [ ] Layer rules pass in the build (no cross-module or direct-database imports)
- [ ] Domain rules have unit tests incl. rounding, zero and negative edge cases
- [ ] An API test runs against a real replica set
- [ ] The route is declared with `defineRoute` and its permission (`<module>.<screen>.<verb>`) and a two-company test
- [ ] New indexes ship with a migration and a query-plan test; collection budget respected
- [ ] Logs, metrics and trace spans added; no secret or personal data logged
- [ ] OpenAPI file and error codes updated
- [ ] Migration follows expand-then-contract and has been run twice on a sample company
- [ ] Any new restricted or personal field is classified, encrypted and, if searched, given a blind index
- [ ] Client: strings localised (short first), tokens not hardcoded, 48 px targets, captions, focus order, empty/loading/error states from the shared set
- [ ] Manifest updated; rail, editor rows and cards still generated from it; permission matrix passes
- [ ] Relevant doc changed in the same change; decision-log row added for any large choice
- [ ] Module owner reviewed; pipeline green

## Reviewer questions
1. Could this show a person something their role was not given, in any list, search, count, export or notification?
2. Does posting still run in one short transaction with an idempotency key and an outbox event?
3. Is any amount a floating-point number?
4. Is any tax rate, threshold or window a constant instead of a dated setting?
5. Does the invoice number format stay within 16 characters including separators?
6. Does failure fail closed?
7. Does it contradict another doc? If so, update REVIEW-FINDINGS.md.
