# Data Flow Diagrams

Text diagrams; replace with drawn ones when a diagramming tool is chosen.

## 1. Online request (any screen)
```
Flutter app ──HTTPS + bearer token──► Edge ──► API
  API: authenticate → tenant context (company from token) → module on? → permission verb? → validate input
     → use case → repository (scope filter) ──► shared collections in the cell, via the data access layer (companyId injected)
  ◄── view model (hidden fields removed, masked fields masked) ◄──
```

## 2. Posting an invoice
```
Counter/Workspace ─POST …/post + Idempotency-Key─► API
  one transaction in the shared collections (every row stamped with `companyId`):
    sales_documents (invoice)  stock_ledger + stock_balances  journal_entries
    number_series (atomic increment)  outbox (invoice.posted)
  commit ──► reply
Outbox relay (change stream) ─► Redis queue (critical) ─► Worker ─► GSP adapter ─► e-invoice portal
                                                          └► messaging, reporting summaries
Worker writes IRN back to the invoice (status Generated) or retries (Pending/Failed).
```

## 3. Offline Counter sync
```
Device: bill + offline ID + number from its block ─► local DB ─► print
online:  POST /pos/sync (ordered batch, offline ID = idempotency key)
         API posts each bill in its own transaction; short stock ⇒ accepted + Conflict flag
         ◄ per-bill result (synced | conflict)
device refreshes items, prices, tax codes, permissions; requests a new number block at 20 left
```

## 4. Permission list
```
Role/override change ─► permission version++ ─► cache miss on next request ─► recompute ─► cache
Client: GET /me/permissions on start, resume, and after any "not available" reply ─► rail, tabs, buttons
```

## 5. Module switch
```
Super admin ─► platform DB (company_modules) + platform_audit ─► Redis broadcast ─► API cache refresh
next request: routes refuse / menus hide / jobs skip; switch-on may start a setup job for that company
```

## 6. Encrypted field write and read
```
value ─► key service (company data key from KMS) ─► AES-256-GCM(random IV, AAD=company+collection+field+record)
      ─► envelope {keyId, iv, ciphertext, tag} (+ blind index HMAC) ─► shared collection (stamped with `companyId`)
read: envelope ─► key lookup ─► tag check ─► decrypt ─► serializer drops/masks per field rule
```

## 7. Import
```
file ─► object storage ─► worker streams rows ─► validate in batches ─► bulk write (small transactions)
     ─► summary (added/updated/skipped) + error file ─► notification; undo allowed 24 h if unused
```

## 8. Reports and exports
```
request ─► scope + field rules on the server ─► small: query/summary collection ─► reply
                                              large: bulk queue ─► worker streams to object storage
                                                    ─► notification with expiring link (logged)
```
