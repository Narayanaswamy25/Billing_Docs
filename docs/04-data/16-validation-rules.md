# Data Validation Rules

One definition per field, used by the API (parsed with a schema that names every field; unknown fields rejected) and mirrored on the client (generated from OpenAPI). Each field is checked when the person leaves it; the server's answer is final. Message style: plain, names the next step, under about twelve words.

Formats below are standard Indian formats; confirm any that the tax advisor changes. Limits marked *proposed* are mine and need owner sign-off.

| Field | Rule | Required | Message |
| --- | --- | --- | --- |
| Email | RFC 5322 syntax, max 254 chars, stored normalised (trim, lower-case) | By form | "Enter a valid email." |
| Mobile (India) | 10 digits starting 6–9; country code optional, stored normalised | By form | "Enter a 10-digit mobile number." |
| GSTIN | 15 chars: `^[0-9]{2}[A-Z]{5}[0-9]{4}[A-Z][1-9A-Z]Z[0-9A-Z]$`; first two digits = a valid state code; optional Verify against the portal | Required for registered parties | "Check this GSTIN." |
| PAN | `^[A-Z]{5}[0-9]{4}[A-Z]$` | Optional | "Check this PAN." |
| Aadhaar | Never stored whole; capture last four or a token | Optional | — |
| Invoice number | Max 16 characters; letters, digits, `/` and `-` only; unique within series | Set by the system at post | Format editor: "Pattern is too long." (count separators) |
| Series format | Contains document code, branch, financial year, running number; length computed at 16 max with the widest possible number | Yes | as above |
| Quantity | Numbers only; non-negative; decimal places per unit (e.g. 0 for pieces, 3 for kg); warn on unusually large value | Yes | "Enter a quantity." / warning, never block, on large values |
| Unit conversion | Positive factor to the base unit | If used | "Enter a conversion above zero." |
| Price, discount, amounts | Decimal128, scale per currency (2 for INR), non-negative; discount ≤ line value; negative only where a document type allows | Yes | "Enter an amount." |
| GST rate | One of the dated rates in settings; HSN/SAC required where the company's turnover band requires it (settings) | Per rule | "Choose a tax rate." |
| Place of supply | Valid state/UT code; defaults from the customer's state; warn if it differs from billing address | Yes on invoices | "Choose a place of supply." |
| Dates | Day first on screen (dd-mm-yyyy); stored UTC + business date; not before the go-live date without Approve; not in a locked period | Yes | "This date is before go-live." / "This period is locked." |
| Quick PIN | Exactly 6 digits; five wrong tries lock it | Optional | "PIN must be 6 digits." |
| Password | Minimum length and breached-password check; company settings set the policy | Yes | "Choose a stronger password." |
| Names, addresses | Trimmed, length limits (*proposed* 200 / 500), Unicode (Hindi, Tamil) | By form | — |
| Attachments | Allowed: PDF, images, spreadsheets; size limit per plan; content type checked and virus-scanned | — | "This file type is not allowed." |
| Import file | Max rows and size shown up front; bad rows go to an error file with a Reason column | — | "12 rows need fixing." |
| Credit note | Reason required (return, price change, error); total ≤ the invoice | Yes | "Add a reason." |
| Journal | Debit total = credit total | Yes | "Debits and credits must match." |
| Opening balances | Debits = credits before posting the opening | Yes | same |
| Reject reason | Required text | Yes | "Add a reason." |

## Behaviour rules
- A field hidden from a role is never required for that role; the server fills a default.
- Server refusal: a summary at the top of the form lists the fields; nothing typed is lost.
- Duplicate check: saving a party with an existing GSTIN or phone shows the match.
- Numbers display with Indian grouping (12,34,567.50) by default; company switch for international.
