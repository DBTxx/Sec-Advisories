# Source Code Evidence

## Target

- Product: Modified Shop
- Version: 3.3.0 rev 16825
- File: admin/module_newsletter.php
- Official source file SHA-256: fc5d364e85e83d76972db4a35f7820dbc8e390ce8c0861c3b9802df124f482ba

## Evidence Chain

| File | Lines | Description |
|---|---:|---|
| 01-input-source.txt | 35–45 | Receives newsletter_body from POST. |
| 02-database-persistence.txt | 55–70 | Stores newsletter_body in the database. |
| 03-database-retrieval.txt | 172–180 | Retrieves the stored newsletter content. |
| 04-html-output-sink.txt | 365–380 | Renders stored content directly into HTML. |

## Integrity

SHA-256 hashes are recorded in SHA256SUMS.txt.

To verify:

sha256sum -c SHA256SUMS.txt

## Technical Assessment

The source-to-sink flow demonstrates that user-controlled newsletter content is received, persisted, retrieved and rendered in an HTML context without output encoding at the identified sink.

These code excerpts support the technical analysis of the Stored XSS vulnerability.
