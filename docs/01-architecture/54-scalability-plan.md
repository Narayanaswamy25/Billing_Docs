# Scalability Plan

Companies share collections inside a cell (D-039). Capacity is now a question of **data volume, throughput and the heaviest companies**, not file count.

## What scales how
| Pressure | Response |
| --- | --- |
| More requests | More stateless API copies behind the load balancer (≥3, across zones) |
| Heavy jobs | More workers by queue depth; bulk work on lower-priority queues |
| More data or writes | Bigger Atlas tier, then **sharding** on `{ companyId: 1, _id: 1 }` (verify the tier supports sharded clusters) |
| One heavy company | Quotas and rate limits, then **zone sharding** or a **dedicated cell** (D-043) |
| Reports | Secondary or analytics nodes; per-company summary collections built by workers |
| Growing history | Archive closed years to encrypted object storage after retention |
| Many small companies | They cost almost nothing: no database or collection per company |

## Fairness controls (always on)
Per-company rate limits and concurrent-job caps; `maxTimeMS` on every query; export and import quotas; separate queues per priority; metrics for the heaviest companies per cell; alerts when one company uses more than a set share of a cell's CPU, memory, connections or storage.

## Sizing a cell
1. Define a reference company (users, invoices per day, lines per invoice, stock movements per day, attachments) and a **large** and a **small** profile.
2. Load a test cell with a realistic **mix** (many small, a few large) at 2× expected peak, with several companies hammering at once.
3. Read: p95 latency per route (read, post), transaction abort rate, index memory vs RAM, replication lag, connections per copy, queue depth, per-company resource share.
4. Set the cell's **company cap** and **trigger for sharding or a new cell** from the result, with headroom, and store it in `cells`.
5. Repeat the isolation suite under load (05) to confirm fairness limits hold.

## Connection arithmetic
`API copies × cells × pool size + workers` stays under about 70% of a cell's connection limit. Start near 20 per cell per copy; set a wait-queue timeout so a full pool fails fast.

## Sharding notes (only when needed)
- Start unsharded; shard when measured size or load requires.
- Ranged `{ companyId, _id }` keeps a company's data together (fast targeted queries, easy export and purge) but can create hot ranges for a very active company; hashed `companyId` spreads writes but scatters a company's data. Decide with a load test.
- Zone sharding can pin a tier of companies to dedicated shards (for example a "large" zone) so heavy companies do not share hardware; zone moves trigger chunk migration, so plan them.
- Every query and index keeps the `companyId` prefix so queries stay targeted.

## If traffic grows 10×
1. More API and worker copies; check pool arithmetic.
2. Move reports to analytics nodes; rely on summary collections.
3. Shard the largest collections; add zones for large companies.
4. Move the largest companies to dedicated cells (04 of the tenancy folder).
5. Split a module into a service only if its boundary and events are already clean.

## Atlas (D-012)
A cell is an Atlas cluster. Tiers set memory, connections and storage; check the tier's sharding and backup features before relying on them. More clusters cost more; most companies share one.
