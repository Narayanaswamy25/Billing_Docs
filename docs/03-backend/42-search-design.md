# Search Design

No separate search engine in release 1: MongoDB indexes plus a local index on the Counter. Revisit only if measured search time passes the 300 ms target.

| Where | How | Fields |
| --- | --- | --- |
| Panel search (any module) | Server query with a lower-case search key and indexes | Number, name, and the module's few key fields |
| Counter item search | Local index on the device (IndexedDB equivalent: the local DB), not a server call per keystroke; scanner types into the focused box | Name, barcode, SKU |
| Exact private fields | Blind index (HMAC) | Mobile, email, person PAN |
| Not searchable | Bank accounts, salary amounts, anything hidden from the viewer | — |
| Quick jump (Ctrl/Cmd+K) | Server search over screens and records the person may see | Screen names and record numbers/names; never lists hidden things |

## Rules
- Search, counts and badges are computed on the server under the five checks: never matches hidden fields or other branches.
- Filters: up to three per module plus a Drafts switch; an owner filter appears only for roles that see others' records.
- Sorting uses a fixed list of fields; paging is keyset on an indexed field.
- Results under 300 ms (tested on the lowest supported phone for local search).
- "inventory" is a search keyword for the Stock module (F-10).
- Names are readable at the field level so search works; hiding names needs Queryable Encryption (open question 9).
