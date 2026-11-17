# Evidence — Stored XSS

Technical evidence supporting the Stored Cross-Site Scripting vulnerability identified in Modified Shop 3.3.0 rev 16825.

## Target

- Product: Modified Shop
- Version: 3.3.0 rev 16825
- Component: Newsletter Module
- Endpoint: `admin/module_newsletter.php`
- Vulnerable Parameter: `newsletter_body`
- Vulnerability: Stored Cross-Site Scripting (CWE-79)

## Evidence Structure

| Directory | Description |
|---|---|
| `01-source-code/` | Source-code excerpts, source-to-sink analysis, and SHA-256 integrity information. |
| `02-poc/` | Proof-of-concept payload submission and JavaScript execution screenshots. |
| `03-cross-user/` | Evidence of Stored XSS execution across two separate authenticated accounts. |
| `04-http-requests/` | Captured HTTP POST request, multipart payload, and SHA-256 checksums. |
| `05-validation/` | Consolidated validation results, testing scope, and limitations. |

## Evidence Methodology

The evidence was collected through authorized testing in the RedScan Academy laboratory environment.

The investigation included:

- Static source-code analysis.
- Identification of the vulnerable input-to-output flow.
- Dynamic proof-of-concept validation.
- Verification of persistent JavaScript execution.
- Cross-user execution testing.
- HTTP request capture.
- SHA-256 integrity verification.

## Scope and Limitations

The evidence documents only behaviors reproduced in the tested laboratory environment.

No session theft, cookie extraction, session hijacking, privilege escalation, unauthorized administrative actions, data exfiltration, or account compromise was demonstrated.

Each evidence directory contains its own README documenting the corresponding test methodology, results, and limitations.

## Integrity

SHA-256 manifests are provided for the source-code excerpts and captured HTTP request evidence.

Original screenshots are preserved in their respective evidence directories.

The integrity of the six source-code and HTTP request evidence files was independently verified on Windows after transfer from Kali Linux.
