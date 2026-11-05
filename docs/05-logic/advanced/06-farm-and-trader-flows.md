# Farm and Trader Flows

For farmers, farmer groups and mandi traders or commission agents (arhtiya). Built on the shared engines; the extras are weighment, lots, daily rates, deductions and account sale. Tax treatment varies by crop, processing, registration and state, so every tax and levy value is a dated setting confirmed by the advisor (07).

## 1. Who uses it and how
| Person | Profile | Needs |
| --- | --- | --- |
| Farmer or group | Farm | Record harvest, store, sell, get paid, track input costs and profit per crop; simple screens |
| Mandi trader | Trader | Receive produce from many farmers, weigh, grade, sell to buyers, pay farmers |
| Commission agent | Trader (agent) | Sell on behalf of farmers, deduct commission and fees, give an account sale, handle advances |
| Helper | Stock employee plus weighment | Weigh and enter; no prices |
Farm mode ships a **Simple mode**: large buttons, few fields, quantity in the farmer's unit.

## 2. Weighment slip (kata slip)
Document with: slip number, date and time, party (farmer or buyer), vehicle, commodity, variety, grade, **gross, tare, net weight**, bag count, **moisture %** and quality notes, deductions for moisture or foreign matter by a configured formula, rate, amount. Weighing scale input where supported. The slip is numbered at post (same numbering rules, 16-character limit) and feeds the lot or the sale.

## 3. Lots (harvest and arrival)
Lot fields: crop, variety, grade, source plot or farmer, harvest or arrival date, storage location, bags, net quantity, moisture, cost to date. Lot movements: harvest, receipt, transfer to storage, shrinkage (drying, handling), sale, sample, wastage. FEFO is replaced by **oldest lot first** or a chosen lot (price differs by grade). Lot ageing reminders and quality flags.

## 4. Daily rate board
Commodity, grade and unit rates entered each day (and revised with time stamps). A sale picks the rate in force at the weighment time unless negotiated. The rate card can be exported or shared by message to buyers or farmers.

## 5. Farm sales
| Case | Document | Tax |
| --- | --- | --- |
| Farmer sells raw produce to trader or mandi | Sale receipt or **bill of supply** (company wording), weighment slip | Per sources found, unprocessed agricultural produce is generally exempt and farmers selling their own produce need not register; the exemption breaks once produce is processed or branded (rice was treated as outside the exemption in one ruling). Item tax category decides; advisor confirms |
| Farmer sells processed or packed goods | Tax invoice if registered, else per law | Taxable categories apply |
| Direct sale to consumers | Cash memo | As above |
| Payment | Cash, UPI, cheque, bank | — |

## 6. Trader purchase from farmers
**Procurement slip** (weighment + rate) becomes a **payment voucher** in place of a supplier invoice when the farmer is unregistered. Deductions for moisture, hamali (loading), commission or advances recovery are shown line by line. The payment is recorded against the farmer's ledger; advances given earlier are recovered automatically in the next settlement. Whether reverse-charge or any other tax treatment applies is a **setting per commodity and state, decided by the advisor**; the software does not assume it.

## 7. Commission agent: account sale
When the agent sells on behalf of farmers:
1. Farmer's produce is received (weighment slip) into lots owned by the farmer, held by the agent.
2. Buyer invoices or bills are made from those lots; stock leaves the lot; the sale belongs to the farmer's account.
3. **Account sale (patti)** to the farmer lists: lots, quantities, rates, gross sale value, **less** commission (percentage or per quintal, dated setting), market fee or levy (state-specific, dated), hamali or labour, transport, weighing charges, advances recovered; **net payable**.
4. Payment to the farmer by cash, UPI or bank; the agent's commission income is recorded; the commission's tax treatment follows the commodity (reported as exempt for raw produce through the regulated market and taxable otherwise; advisor to confirm).
5. Buyer collections are tracked with dues, ladder reminders and farmer-wise exposure ("farmer paid before buyer paid" limit setting).
Licence numbers (trader, commission agent, market) print on documents as the state requires.

## 8. Input purchases and costs
Seed, fertiliser, pesticide, fuel, labour, rent, transport and equipment are recorded as purchases or expenses and **allocated to crop, plot and season**. Crop-wise cost, revenue and profit per quintal and per acre appear in Reports. Input purchases from licensed dealers may carry batch, lot and validity dates that must be kept (seeds, pesticides; later pack).

## 9. Controls
Stock limit alerts for commodities where the government sets stock limits (setting per commodity and period; verify), unrealistic weight or moisture flags, duplicate slip detection, price deviation from the rate board needing approval, advance ceilings per farmer.

## 10. Permissions and data (additions)
Permission keys `farm.lots`, `farm.weighment`, `farm.account_sales`, `settings.compliance` (see catalog). Data: lots live in `batches_serials`; weighment slips in `stock_documents`; account sales in `sales_documents` with type; rate board in `lookups`; no new collections (the 65 budget holds).
