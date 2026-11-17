# HTTP Request Evidence

## Target

- Product: Modified Shop
- Version: 3.3.0 rev 16825
- Module: Newsletter
- Endpoint: `admin/module_newsletter.php`
- Parameter: `newsletter_body`

## Captured Requests

HTTP traffic was captured using Firefox Developer Tools in the authorized RedScan Academy laboratory environment.

| File | Description |
|---|---|
| `01-post-request.txt` | HTTP POST request headers captured during newsletter submission. The session cookie has been redacted. |
| `02-post-payload.txt` | Multipart form body containing the submitted newsletter parameters, including the XSS payload. |
| `SHA256SUMS.txt` | SHA-256 checksums of the captured request evidence files. |

## Request Details

- Method: `POST`
- Request URI: `/admin/module_newsletter.php?action=save`
- Content-Type: `multipart/form-data`
- Newsletter ID: `1`
- Newsletter title: `Teste Stored XSS - RedScan Academy`

## Relevant Parameter

The captured request contains the following value in `newsletter_body`:

```html
<img src=x onerror=alert(document.domain)>
```

## Technical Significance

The captured HTTP POST confirms that the application receives the crafted HTML content through the `newsletter_body` parameter when an existing newsletter is saved.

This evidence complements the source-code analysis, which demonstrates the subsequent database persistence, retrieval, and unsafe HTML rendering of the submitted content.

## Evidence Handling

- The session cookie was redacted from the public request headers.
- The original multipart payload was preserved in the evidence file.
- SHA-256 checksums were generated for the request headers and payload files.
- Hash verification was successfully performed after transferring the files from Kali Linux to Windows.

## Scope

The captured request documents the submission of the benign proof-of-concept payload.

It does not independently demonstrate script execution or cross-user impact. Those behaviors are documented separately in the PoC and cross-user evidence sections.
```

