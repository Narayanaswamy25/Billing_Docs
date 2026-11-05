# Caching Strategy

A cached value never decides money: posting re-reads prices, rules and stock inside the transaction (or checks the version it used).

| What | Where | Key | Invalidated by | Lifetime |
| --- | --- | --- | --- | --- |
| Company record (status, cell, modules) | Process memory + Redis | company | Change message on Redis | Max 60 s backstop |
| Effective permissions | Process memory + Redis | user + permission version | Version rises when a role, override or user changes | Until version changes |
| Business rules and settings | Process memory | company + settings version | Version rises on any change | Until version changes |
| Masters for lookups and Counter (items, prices, taxes) | Redis; copy on the Counter device | company + masters version | Version rises on write | Device refreshes when online |
| Report summaries | MongoDB summary collections | company + period | Rebuild job | Per schedule |
| Static assets, masters over HTTP | ETag and CDN | URL with version | New version in URL | Long |
| Rate limit buckets | Redis | user / company / address | Time | Per rule |
| Idempotency results | MongoDB unique index | key | Expiry (set time) | Set time |

## Rules
- Version numbers, not time, drive invalidation wherever possible; time limits are only a backstop.
- Module switch changes broadcast on Redis; a lost message is covered by the 60 s limit.
- On the Counter, a permission list older than 24 h locks new bills (D-008).
- Cache keys include company ID so nothing can cross companies; tests assert this.
- If the cache is down, fall back to the database (slower, correct) rather than allow or deny blindly; if the permission check itself cannot run, fail closed.
- HTTP replies carrying personal or financial data are marked not to be stored by browsers or proxies.
- Counter search runs on a local index, not a server call per keystroke.
