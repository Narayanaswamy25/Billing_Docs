# Local Development Setup Guide

**Draft:** no repository exists yet, so commands are placeholders to confirm when the code is created. Goal from the architecture doc: a new developer reaches a green local test run on the first day.

## Prerequisites
| Tool | Why | Version |
| --- | --- | --- |
| Node.js (Active or Maintenance LTS; 26 once confirmed) | Backend | Pin in `.nvmrc` |
| Package manager (pnpm or npm) | Workspace | Pin in lock file |
| Docker (or Podman) | MongoDB replica set, Redis, object storage emulator | Current |
| Flutter SDK | Clients | Pin with the project's SDK constraint |
| Git | Source | — |

## Steps (proposed)
1. Clone the repository; install the pinned Node and Flutter versions.
2. `install` dependencies for the backend workspace and the Flutter workspace.
3. Start local services with one compose file: **a MongoDB replica set with at least one node configured as a replica set** (transactions require it, a plain standalone server is not enough), Redis, an S3-compatible emulator, a fake GSP.
4. Copy the example environment file; fill only non-secret local values. Local keys come from a dev key service stub, never production keys.
5. Run database provisioning for two sample companies (A and B) with fixture data.
6. Start API and worker; start the client against the local API.
7. Run the test commands: domain tests, API tests, the two-company isolation suite, client tests and goldens.

## Common errors
| Symptom | Likely cause | Fix |
| --- | --- | --- |
| "Transaction numbers are only allowed on a replica set member" | MongoDB started standalone | Start the compose replica set and initiate it |
| Posting tests hang or abort | Collections created without majority write concern before the transaction | Create collections through the provisioning script |
| Signed in as company A but see 403 on every route | Module switches missing in the platform DB seed | Re-run provisioning for the sample company |
| Golden tests differ | Different font or Flutter version | Use the pinned SDK and bundled fonts |
| Counter cannot print | No printer plugin on the dev machine | Use the PDF preview or the fake printer |

Fill this out with real commands after the repository exists; add it to the Definition of Done for the foundations step.
