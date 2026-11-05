# Business Profiles

## 1. What a profile is
At company creation (and any time later in Settings, by the admin within entitlements) the company picks:
| Choice | Options | Drives |
| --- | --- | --- |
| **Business kind** (one or more) | Store, Wholesale or distributor, Trader or commission agent, Farm or producer, Restaurant (later), Services (later), Pharmacy (later pack) | Feature switches, documents, reminders, screens |
| **Customer mix** | Mostly B2C, mostly B2B, both | Default price mode, invoice format, credit rules, e-invoice prompts |
| **Tax status** | Regular, Composition, Unregistered, Exempt-only (for example raw farm produce), Mixed | Which document type is issued and whether tax lines are drawn |
A profile is **a preset of entitlements, settings and templates**, not a code branch. The super admin's entitlements still cap it, and the admin can change any default afterwards.

## 2. The profiles
| Profile | Who | Typical customers | Documents | Stock model | Special features | Phase |
| --- | --- | --- | --- | --- | --- | --- |
| **Store (B2C retail)** | Kirana, apparel, electronics, general store | Walk-in public, some regulars | Retail invoice or cash memo; consolidated day summary for tiny bills (verify); returns slip; credit note | Barcode items, MRP, optional batch and expiry, counter scale | Scan-first Counter, MRP guard, quick return, held bills, FEFO for food | **Release 1** |
| **Wholesale or distributor (B2B)** | Wholesaler, distributor, supplier to shops | Registered businesses, some unregistered | Quote, order, delivery challan, tax invoice with e-invoice, credit and debit notes, e-way bill | Multi-unit (carton, box, piece), reservations, backorders, price lists by customer group | Credit limit and ageing, dunning, schemes and slabs, route and delivery, e-invoice, MSME and TDS awareness | **Release 1** |
| **Trader or commission agent** | Mandi trader, arhtiya, broker | Farmers (suppliers), shops and processors (buyers) | Weighment slip, procurement voucher, sale invoice or bill of supply, **account sale** to the farmer | Lots by commodity, grade and farmer; weight-based; short holding times | Kata and moisture, commission, market fee and other deductions, advances, farmer ledger | Release 1.5 (D-032) |
| **Farm or producer** | Farmer, farmer group, small producer | Traders, mandi, local buyers, direct consumers | Sale receipt or bill of supply (no tax when exempt), weighment slip, purchase receipts for inputs | Harvest lots by crop and plot, storage locations, shrinkage | Simple mode, crop-wise profit, input costs | Release 1.5 (D-032) |
| Pharmacy pack | Chemist | Public, hospitals | Retail invoice with batch and expiry, controlled-drug register | Batch and expiry mandatory, returns to quarantine | Prescription register, licence tracking | Later |
| Restaurant | Eatery | Dine-in, takeaway | KOT, bill | Recipe consumption | Tables, kitchen screens | Later |
| Services | Repair, salon, clinic | Public, businesses | Invoice, quote | None or parts only | Jobs, appointments | Later |
Mixed businesses (a store that also sells wholesale) combine profiles; the stricter rule applies per document.

## 3. What changes by customer type (B2C vs B2B)
| Topic | B2C | B2B |
| --- | --- | --- |
| Customer record | Walk-in default; optional name and mobile (consent for messages) | Registered party: GSTIN, address, credit terms, price list, contacts, MSME flag |
| Price entry | Tax-inclusive, MRP-capped for packaged goods | Usually tax-exclusive, from price list or contract |
| Document | Retail or simple invoice; bill of supply if exempt or composition | Full tax invoice with HSN, place of supply, e-invoice when in scope, e-way bill when goods move |
| Payment | Cash, UPI, card at the counter; no credit by default | Credit with terms (for example net 30); part payments; cheques; advance |
| Returns | Quick, at the counter, original mode or store credit | Return request, inspection, credit note, tax adjustment |
| Reminders | Few: restock, expiry | Many: dues, ageing, supplier dues, compliance dates |
| Control | Shift cash | Credit limit, approvals, TDS awareness, ITC timing |

## 4. What a profile switches on
| Switch | Store | Wholesale | Trader | Farm |
| --- | --- | --- | --- | --- |
| Counter app | Yes | Optional | Yes (weighment) | Yes |
| Units and conversions (`uom_conversions`) | Light | Heavy | Weight | Weight |
| Batches, expiry (`batches_serials`) | Per item | Per item | No | Lots |
| Reservations, backorders (`reservations`) | No | Yes | No | No |
| Price lists and schemes | Light | Yes | Daily rate | Daily rate |
| Delivery challan, e-way bill | No | Yes | Sometimes | Rare |
| E-invoice prompts | No | Yes | If B2B | No |
| Restock reminders (`reorder_planning`) | Yes | Yes | No | Lot ageing instead |
| Dues and dunning | Light | Yes | Yes | Yes (buyers) |
| Commission and account sale (`account_sale`) | No | No | Yes | No |
| Farm lots and weighment (`farm_lots`, `weighment`) | No | No | Yes | Yes |
| Compliance packs | MRP, FSSAI if food | MSME, TDS, e-invoice | Market fee, licences | Exempt bill of supply |

## 5. Setup wizard (extends "Who works here?")
1. Business kind and customer mix (plain-word questions: "Do you sell to the public, to shops, or both?", "Do you grow what you sell?").
2. Tax status ("Are you registered for GST?"); unregistered or exempt farmers skip tax screens.
3. Unit style (pieces, kg, quintal, bag, carton) and decimal places.
4. Reminder defaults (accept the proposed ones; change later).
5. Licences to record (FSSAI, drug, trader, commission agent, shop) with expiry dates.
6. Sample data matching the profile; target: first bill or first weighment slip in under three minutes.

## 6. Scope decision (D-032, proposed)
Release 1 builds Store and Wholesale fully on the shared engines. Trader and Farm reuse those engines and add weighment, lots and account sale in release 1.5. Pharmacy, Restaurant and Services come as packs later. This keeps "for all businesses" true in the design without delaying the first release.
