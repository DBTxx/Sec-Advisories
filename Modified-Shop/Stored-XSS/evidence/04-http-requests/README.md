# HTTP Request Evidence

## Target

- Product: Modified Shop
- Version: 3.3.0 rev 16825
- Module: Newsletter
- Endpoint: `/admin/module_newsletter.php`
- Parameter: `newsletter_body`
- Laboratory Target: `192.168.56.144`

## Capture Method

HTTP traffic was captured using Firefox Developer Tools in the authorized RedScan Academy laboratory environment.

The captured request corresponds to editing and saving an existing newsletter (ID 1), not creating a new newsletter.

## Captured Evidence

| File | Description |
|---|---|
| `01-post-request.txt` | HTTP POST request headers captured during newsletter submission. The session cookie was redacted. |
| `02-post-payload.txt` | Multipart form body containing the submitted newsletter parameters, including the proof-of-concept payload. |
| `SHA256SUMS.txt` | SHA-256 checksums of the captured request evidence files. |

## Request Details

- Method: `POST`
- Request URI: `/admin/module_newsletter.php?action=save`
- Content-Type: `multipart/form-data`
- Newsletter ID: `1`
- Newsletter title: `Teste Stored XSS - RedScan Academy`

## Relevant Parameter

The captured multipart request contains the following value in `newsletter_body`:

```html
<img src=x onerror=alert(document.domain)>
```

This payload was used exclusively to demonstrate JavaScript execution in the laboratory environment.

## Technical Significance

The captured HTTP POST documents the transmission of the crafted HTML content through the `newsletter_body` parameter when an existing newsletter is saved.

This evidence establishes that the application received the submitted parameter as part of the captured request.

It complements the static source-code analysis, which documents the subsequent database persistence, retrieval, and unsafe HTML rendering of the submitted content.

The HTTP capture alone does not establish database persistence or JavaScript execution. Those behaviors were validated separately and are documented in the corresponding evidence sections.

## Evidence Handling and Integrity

- The session cookie was redacted from the published request headers.
- The original multipart payload was preserved in the evidence file.
- SHA-256 checksums were generated for both captured evidence files.
- Hash verification was successfully performed after transferring the files from Kali Linux to Windows.
- The published checksum manifest covers the two evidence files, not the manifest itself.

## Scope and Limitations

The captured request documents the submission of the benign proof-of-concept payload.

It does not independently demonstrate:

- Database persistence.
- JavaScript execution.
- Cross-user execution.
- Session theft or hijacking.
- Unauthorized administrative actions.

These behaviors are addressed separately in the source-code, PoC, cross-user, and validation evidence.

## Capture Date

2026-11-17
```

