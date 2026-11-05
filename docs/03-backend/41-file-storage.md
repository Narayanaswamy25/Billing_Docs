# File Storage and Media Handling

| Topic | Rule |
| --- | --- |
| Storage | S3-compatible object storage; files never in the database or through the API process |
| Access | Short-lived signed links for upload and download; served from a separate address |
| Allowed types | PDF, images, spreadsheets (attachments); Excel/CSV (imports); PDF (generated documents). Size limit per plan (value not set) |
| Checks | Size and type limits, content inspected (not just the name), virus scan before release |
| Encryption | Per-company file key; checksum (SHA-256) stored with the file; ID proof files are field-protected |
| Naming | `{company}/{module}/{recordId}/{fileId}-{checksum-prefix}` (opaque; no customer names in keys) |
| Metadata | Who added and when, shown on the Files tab; stored in the `files` collection |
| Generated PDFs | Stored once per document version; regenerated only when the version changes |
| Exports | Written to object storage; the person is notified; the link expires |
| Backups | Separate account and credentials, versioned, copy in another region |
| Retention | Per record type; deleted with the company after the notice period and key destruction |
| Spreadsheet safety | Exports neutralise cells starting `=`, `+`, `-`, `@` |
| Logo and print assets | Per company, size-limited, used by print layouts |
| Offline | The Counter keeps only what POS needs; unsent bills stay on the device until synced |
