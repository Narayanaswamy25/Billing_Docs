# Acts and Compliance Map

**Not legal or tax advice.** This maps Indian laws to what the software does. Search findings come from tax-advice sites, news reports and government replies, not from the statutes themselves. Each row says what was found (**Sources found**) or what still needs the company's advisor or lawyer (**Verify**). **All thresholds, rates, windows and wording are dated company settings** (D-017), pre-filled with the values below and confirmed before launch. Section numbers can change: the Income-tax Act, 2025 is reported to take over from the 1961 Act from 1 April 2026 (**not verified here**), so settings store the rule by subject and the current section text, not by hard-coded section number.

Date of research: 2026-10-31.

## A. GST (CGST, SGST, IGST Acts and Rules)
| Rule | What it asks | What the software does | Applies to | Status |
| --- | --- | --- | --- | --- |
| Tax invoice (Rule 46) | Consecutive serial number in one or multiple series, up to 16 characters (letters, digits, `-`, `/`), unique per financial year; required particulars | Numbering rules in 18; format editor counts characters; multiple series allowed (blocks of 50) | Registered sellers | Sources found |
| HSN on invoices | 4 digits up to ₹5 crore turnover on B2B, 6 digits above; B2C may omit the digit count up to ₹5 crore (Notification 78/2020 as summarised) | Item master requires HSN; digit rule is a setting by turnover | Registered | Sources found |
| Small B2C invoices (≤ ₹200) | A tax invoice may be waived for very small supplies to unregistered buyers with a consolidated invoice at day end (conditions not confirmed) | Optional consolidation setting | Store | **Verify** |
| Bill of supply | Composition sellers and exempt supplies issue a bill of supply, not a tax invoice | Document type by tax status | Composition, exempt, farmers | **Verify** |
| Credit notes (Section 34) | Issue on or before 30 November following the financial year, or the annual return date if earlier; sources disagree whether this is issuing or reporting; one source says the Finance Act 2026 amended related provisions | Credit-note screen shows the last date; reminder before it | All registered | Sources found, **Verify** the 2026 change |
| Input-credit payment rule (180 days) | Input credit reverses if the supplier is not paid within 180 days of the invoice | Bill ageing and reminders at day 150 and 170 | Buyers | **Verify** (not confirmed by search) |
| E-invoice | Mandatory above ₹5 crore turnover; report within 30 days of invoice date for ₹10 crore and above; cancel within 24 hours of IRN | See 27-third-party-integrations | B2B above threshold | Sources found, **Verify** thresholds |
| E-way bill | Goods movement above ₹50,000 (default); validity by distance | Prompt, generation via GSP, expiry reminder | Wholesale, traders | **Verify** |
| Delivery challan | Goods moved without invoice (approval, job work, transfer) | Document type and conversion | Wholesale | **Verify** |
| Place of supply, CGST/SGST vs IGST | By supply place | Tax engine | All | Standard |
| Goods on approval | Invoice limit after removal (reported as six months) | Reminder before the limit | Wholesale | **Verify** |
| Records | GST records kept for a set number of months after the annual return (reported as 72) | Retention setting | All | **Verify** |

## B. Income-tax (cash, source deduction, payments)
| Rule | What it asks | What the software does | Applies to | Status |
| --- | --- | --- | --- | --- |
| **TCS on sale of goods** | Seller TCS (0.1% above ₹50 lakh) was switched off from 1 April 2025; buyers deduct under the purchase-TDS rule instead | **TCS on goods defaults to off**; setting kept for other TCS categories | Sellers | Sources found |
| **TDS on purchase of goods (194Q)** | Buyer above the turnover limit deducts 0.1% on purchases above ₹50 lakh from a seller in a year (buyer turnover threshold not in the snippet) | Buyer-side flag per supplier, running purchase total, TDS line on the bill when due | B2B buyers | Sources found, **Verify** thresholds |
| TDS deposit and return dates | Monthly deposit and quarterly returns | Calendar reminders | Those who deduct | **Verify** |

## C. MSME payment protection (MSMED Act, 2006 and the Income-tax disallowance)
| Rule | What it asks | What the software does | Status |
| --- | --- | --- | --- |
| Payment within agreed days, **maximum 45 days** (15 days if there is no written agreement) for micro and small suppliers | Due date = acceptance + terms, capped as stated | Udyam number and size on the supplier; due date computed; reminders at due −10, −3, due day | Sources found |
| Interest on delay | Compound interest at three times the RBI bank rate; not deductible | Estimate shown on the overdue list (rate is a setting) | Sources found (do not rely on any quoted rate) |
| Deduction for unpaid MSME dues at year end | The expense is deductible only if paid within the time limit (in force from the 2023-24 year; applies to micro and small, one source says not medium) | Year-end list of unpaid MSME dues beyond the limit | Sources found |
| Half-yearly return of overdue MSME payments by companies | Reported by companies | Report of dues over 45 days | **Verify** |
Seller side: a company that is itself an MSME records this on customers to claim interest on late payment (setting).

## D. Weights, measures, labelling and consumers
| Law | What it asks | What the software does | Status |
| --- | --- | --- | --- |
| Legal Metrology Act 2009, Packaged Commodities Rules 2011 | Packaged goods cannot be sold above the printed MRP; one packaged item cannot carry different MRPs; penalties apply (one source gives ₹2,000 for selling above MRP) | MRP stored per batch; MRP guard blocks; report of items sold at MRP | Sources found |
| Legal Metrology: weighing scales | Scales for trade must be verified and stamped periodically | Scale register with next verification date and reminder | **Verify** (period) |
| Consumer Protection Act 2019 and e-commerce rules | Unfair trade practices, overcharging, return and refund information, grievance | Return policy printed on documents; complaint log; refund timelines in the returns engine | **Verify** |
| Sale of Goods Act 1930 | Goods must match description and be of satisfactory quality | Records of terms, return window, replacement flow | **Verify** |

## E. Food, medicine and agri inputs
| Law | What it asks | What the software does | Status |
| --- | --- | --- | --- |
| Food Safety and Standards Act, 2006 (FSSAI) | Food businesses print their 14-digit licence or registration number on bills, invoices, cash memos and receipts (effective date reported as 1 Oct 2021 and again as 1 Jan 2022; current status not confirmed) | Licence number field on company and on every document layout; expiry reminder; batch and expiry trace; recall | Sources found, **Verify** current order |
| Drugs and Cosmetics Act 1940 and Rules | Licence; batch and expiry; Schedule H1 supply entered in a separate register with prescriber, patient, medicine, quantity, kept three years; Pregabalin reported as newly added to Schedule H1 in May 2026 | Pharmacy pack: register, batch and expiry mandatory, returns to quarantine | Sources found (secondary), later pack |
| Fertiliser (Control) Order, Seeds Act, Insecticides Act | Dealer licences, lot or batch and validity; fertiliser sales on a government point-of-sale device (not confirmed here) | Licence fields and lot tracking; integration is a later pack | **Verify**, later pack |

## F. Agriculture and markets
| Law | What it asks | What the software does | Status |
| --- | --- | --- | --- |
| GST on agricultural produce | Unprocessed produce generally exempt; farmers selling their own produce need not register; processing can break the exemption; commission agent services for raw produce reported as exempt (Entry 54(g)), taxable for processed goods | Item tax category; document type; commission tax setting per commodity | Sources found (blogs and rulings), **Verify** |
| APMC and state market laws | Licences for traders and commission agents, market fee, account sale to the farmer, state formats | Licence numbers on documents; market fee as dated per-state setting; account sale document | **Verify** per state |
| Essential Commodities Act 1955 | Government may set stock limits on specified goods | Per-commodity stock limit alert (setting) | **Verify** |

## G. Payments, data and records
| Law | What it asks | What the software does | Status |
| --- | --- | --- | --- |
| Negotiable Instruments Act 1881, section 138 (cheque dishonour) | Written demand notice within 30 days of learning of dishonour; drawer has 15 days to pay; complaint within one month after; cheque valid for three months | Bounced cheque starts a checklist with dates (information only) | **Verify with a lawyer** (general knowledge) |
| Payment and Settlement Systems Act 2007, RBI card rules | Use licensed providers; no card number storage | Gateway tokens only | Standard |
| DPDP Act 2023 and Rules 2025 | Consent for personal data; breach reporting from 13 May 2027 | Consent records, customer messages only with consent, privacy docs | Sources found |
| Telecom and WhatsApp rules for commercial messages | Registered senders and templates; opt-in | Approved templates, consent flag, opt-out | **Verify** |
| Records retention (GST, Companies Act and Income-tax) | Multi-year retention | Default retention = the longest applicable (proposed 8 years) as a setting | **Verify** |
| Shops and Establishments Acts (state) | Registration, hours, wages | HR pack (later) | Out of release 1 |

## H. How the checks behave
| Check | Default | Override |
| --- | --- | --- |
| Price above MRP | Block | Admin may unflag the item |
| FSSAI number missing on a food company's layout | Block printing | Add the number |
| MSME supplier overdue | Reminder, interest estimate | None |
| 180-day input-credit risk | Reminder | None |
| Credit-note date passed | Warn and need Approve | Accountant with reason |
| Expired batch | Block sale | None |
| Licence expired (food, drug, trader) | Warn daily, block drug sales | Admin with reason |
Each check is a rule in `settings.compliance` with its text, thresholds, effective dates and the advisor's name and date of confirmation.

## I. Pre-launch confirmation
For each row marked **Verify**, the company's advisor or lawyer confirms or changes the setting and the date is recorded. Nothing marked **Verify** is hard-coded.

## Sources used (secondary; check primary texts)
- TCS and 194Q: [taxguru FAQ on TCS removal](https://taxguru.in/income-tax/tcs-sale-goods-removed-april-1-2025-faqs.html)
- MSME payments and 43B(h): [taxguru explainer](https://taxguru.in/income-tax/payment-msme-45-15-days-section-43b-h-explained.html)
- FSSAI number on invoices: [taxguru](https://taxguru.in/corporate-law/fssai-food-businesses-mention-license-no-invoices-w-e-f-1st-october-2021.html)
- Legal metrology and MRP: [Lok Sabha reply (PDF)](https://fcainfoweb.nic.in/PMS/writereaddata/2025_LS_B_4362.pdf); [Bar and Bench report](https://www.barandbench.com/news/consumer-commission-slaps-7000-fine-restaurant-selling-water-beverages-above-mrp)
- Credit notes: [taxguru on Section 34](https://taxguru.in/goods-and-service-tax/section-34-understanding-credit-notes-gst.html)
- Invoice numbering: [CBIC invoice rules](https://cbic-gst.gov.in/gst-invoice-rules.html)
- Schedule H1 register: [NHSRC PDF](https://www.nhsrcindia.org/sites/default/files/2021-07/Attachment%20A%2C%20Schedule%20H1%20register%20and%20NDPS%20Forms.pdf)
- Agricultural produce under GST: [taxscan](https://www.taxscan.in/?p=145973) and advance-ruling summaries
