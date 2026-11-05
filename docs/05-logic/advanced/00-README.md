# Advanced Business Logic: overview

This folder upgrades the logic in [18-business-logic.md](../18-business-logic.md) for a product that must serve **a farmer, a retail store, a wholesaler, a trader or commission agent, and later a pharmacy, restaurant or service business** from the same code. The earlier logic assumed one kind of shop. These documents make the product adapt to the business instead.

## The idea in one paragraph
A company chooses a **business profile** at setup. The profile decides which documents it uses (tax invoice, bill of supply, cash memo, delivery challan, weighment slip, account sale), which inventory features are on (batches and expiry, units and weight, reservations, lots), which reminders run, and which legal checks apply. Under the profiles sits one **inventory engine**, one **pricing engine**, one **returns engine** and one **reminder engine**, so a rule fixed once is fixed for every kind of business.

## Documents
| # | Document | Answers |
| --- | --- | --- |
| 01 | [Business profiles](01-business-profiles.md) | B2B vs B2C vs farmer vs store: what changes, what is shared, release phasing |
| 02 | [Inventory engine](02-inventory-engine.md) | Stock states, units, batches and expiry, picking, reservation, landed cost, repack, counts, wastage |
| 03 | [Restocking and reminders](03-restocking-and-reminders.md) | Reorder maths, restock list, purchase suggestions, the reminder engine and every reminder type |
| 04 | [Selling and pricing](04-selling-and-pricing.md) | Price resolution, MRP, discounts, B2C vs B2B documents, credit and collection |
| 05 | [Returns and reverse flows](05-returns-and-reverse-flows.md) | Sale returns, purchase returns, claims, exchange, restock cost, fraud controls |
| 06 | [Farm and trader flows](06-farm-and-trader-flows.md) | Harvest lots, weighment, mandi sales, commission agent account sale, farm-level profit |
| 07 | [Acts and compliance map](07-compliance-acts-map.md) | Which Indian laws touch the product, what the software does about each, and what is verified |
| 08 | [End-to-end workflows](08-end-to-end-workflows.md) | Step-by-step flows per business type |

## Principles
1. **Profile drives behaviour; the core never forks.** No `if (isFarmer)` in business code; profiles switch features, documents and rules that already exist.
2. **Every rule is a setting with an effective date.** Thresholds, rates, windows and limits from any law are data (D-017), checked by the company's advisor.
3. **Warn first, block when the law is clear, always log.** A company can tighten a rule; it cannot hide a legal block the platform sets.
4. **Explainable suggestions.** Every reorder or reminder shows *why* ("sold 12 a day over 28 days, lead time 5 days, reorder at 78, you have 64").
5. **Stock is a ledger.** Quantities are never overwritten; every change is a movement with a reason, a cost and a source document.
6. **Money and goods stay linked.** A return links to its sale, batch and cost; a refund links to its credit note; a payment links to its invoice.

## Decisions and findings
New decisions D-032 to D-036 in [29-decision-log.md](../../01-architecture/29-decision-log.md); new findings F-35 to F-41 (F-40 withdrawn) in [REVIEW-FINDINGS.md](../../REVIEW-FINDINGS.md). Nothing here is legal or tax advice; every legal statement is marked *sources found* or *verify with the advisor* in document 07.
