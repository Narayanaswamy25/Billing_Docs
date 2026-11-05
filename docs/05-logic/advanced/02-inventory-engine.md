# Inventory Engine

Replaces the thin stock rules in 18-business-logic. The ledger idea (append-only `stock_ledger`, rebuildable `stock_balances`) stays.

## 1. Items
| Field group | Fields |
| --- | --- |
| Identity | Name, SKU, barcodes (many), brand, category, status (Active, Seasonal, Discontinued) |
| Units | Base (stock) unit, sale unit, purchase unit, conversions (1 carton = 12 boxes = 144 pieces), decimals per unit, optional weight per unit |
| Kind | Stock item, non-stock (service), kit or bundle, bulk-with-packs (parent and children), weighed item (catch weight) |
| Tracking | None, batch (with expiry and manufacture date), serial number, lot (farm) |
| Packaged commodity | MRP, net content and unit, "MRP-governed" flag (see 04) |
| Tax | HSN or SAC, GST rate (dated), category: taxable, exempt, nil-rated, non-GST; reverse-charge flag |
| Cost and price | Last cost, valuation method override, default selling price, price lists |
| Reorder | Reorder point, safety stock, max stock, preferred supplier, lead time, MOQ, pack size, ABC class (auto), seasonal index (see 03) |
| Compliance | FSSAI category, drug schedule, shelf life, storage condition, licence required flag |

## 2. Locations and stock states
Locations belong to a branch and have a **type**. Quantity is held per item, location and batch or lot.
| Location type | Sellable? | Purpose |
| --- | --- | --- |
| Sellable | Yes | Normal shelf or godown |
| Damaged | No | Broken or defective, awaiting decision |
| Quarantine | No | Returned or suspect goods awaiting inspection; default for returned food and medicine |
| In transit | No | Between locations or branches (Sent → In transit → Received) |
| Supplier return | No | Waiting to go back to a supplier |
| With customer | No | Delivered on approval or consignment, still ours |
| Farm storage | Yes (lots) | Godown, cold storage, field heap |
Derived quantities shown everywhere:
- **On hand** = physical quantity in all locations of the branch.
- **Available** = On hand in Sellable locations − Reserved − expired batches.
- **Reserved** = held for confirmed orders (see 5).
- **Incoming** = open purchase orders not yet received, with expected dates.
- **Days of cover** = Available ÷ average daily demand (see 03).

## 3. Movements (the ledger)
Every change is a signed row: item, location, batch, quantity in base unit, unit cost, reason, source document, user, device, business date. Types:
Opening · Purchase receipt · Sale issue · Sale return receipt · Purchase return issue · Transfer out · Transfer in · Adjustment (+/−, with reason) · Count variance · Conversion (repack) in/out · Wastage or spoilage · Internal use · Sample or free · Lot harvest · Lot shrinkage. A mistake is a reversing row, never an edit. Each posted movement carries the **cost** it left or entered at, so a later return can use the original cost.

## 4. Valuation and landed cost
- Weighted average (default) or FIFO, chosen per company (changing needs a support request). Batch and lot items may use batch cost.
- **Landed cost:** freight, loading, insurance, duty and other costs on a purchase bill can be allocated to receipt lines by value, quantity or weight, so cost and margin are true. Until the bill arrives the receipt uses the PO price and is corrected by a cost adjustment movement.
- **Returns at original cost:** a sale return restocks at the cost the sale line left at (looked up from the ledger), not the current average. A purchase return leaves at the cost it came in.
- **Below-cost guard:** selling under cost needs the role's limit or an approval (see role limits).

## 5. Picking, reservation and availability
| Rule | Behaviour |
| --- | --- |
| Batch selection | **FEFO** (first expiry, first out) for items with expiry; **FIFO** otherwise; a manual batch choice needs a reason and is logged |
| Expired stock | A batch past expiry becomes non-sellable automatically and appears in the write-off list |
| Reservation | A confirmed sales order reserves available stock (all, or partial with backorder). Reservations expire when the order does, or when cancelled. Counter held bills reserve nothing by default (setting: reserve for N minutes) |
| Oversell guard | At posting, stock is decremented with a conditional update, so two counters cannot sell the last unit; behaviour follows the negative-stock rule (Block, Warn, Allow) |
| Backorder | Unfilled order quantity stays open; when stock arrives the system proposes allocation in order of promised date, then order date |
| Serial items | Every serial scanned at sale; a serial cannot be sold twice or sold after being returned to a supplier |
| Bundles | Selling a kit deducts its components; stock of the kit = the smallest component stock ÷ quantity per kit |

## 6. Conversions and repacking
Break bulk into packs (a 50 kg bag into ten 5 kg bags) or combine items. A **Conversion document** lists inputs and outputs; cost of inputs is spread over outputs by weight or value; shrinkage is a separate line with reason. Needs `stock.conversions` (see catalog additions).

## 7. Counts, shrinkage and wastage
- **Cycle counts by ABC:** class A items counted monthly, B quarterly, C yearly (settings); the system suggests the next list. Blind count option (counter does not see expected quantity).
- A count records the system quantity at start; movements during the count are applied on top, so the shop never has to stop selling.
- Variances above the role limit go to Approve before stock changes.
- **Wastage and spoilage** (expiry, breakage, theft, moisture loss, evaporation) are documents with reason, photos optional, and an approval threshold; they feed a shrinkage report and the restock maths (lost units are not counted as demand).

## 8. Transfers between locations and branches
Sent → In transit → Received. Stock leaves the sender at Sent and enters the receiver only at Received; differences at receipt create a shortage or damage record. If the two branches have **different GST registrations** (for example in different states), the movement is a taxable supply and needs a tax invoice instead of a plain transfer; the document type is chosen from the branches' registrations (verify with the advisor).

## 9. Traceability and recall
Every sale line stores item, batch or serial and customer (when known). Given a batch the system lists **who bought it, where it sits now, and what remains**; the recall action blocks the batch for sale, creates a task list for staff and a message to B2B customers. This supports food and pharmacy rules (see 07).

## 10. Lots for farm and trader profiles
A **lot** is a batch with agricultural fields: crop, variety, grade, plot or source farmer, harvest or arrival date, bag count, gross, tare, net weight, moisture percentage, storage location. Shrinkage (drying, handling) is recorded against the lot. Cost accumulates from inputs and costs allocated to the lot (see 06).

## 11. Controls and reports
Stock summary, stock ledger, valuation, ageing by batch, expiry list, slow-moving and dead stock, shrinkage, negative stock, reservation list, backorders, reorder list, fast and slow movers, ABC. Role limits (`maxStockAdjustment`) apply to every adjustment, wastage and conversion.
