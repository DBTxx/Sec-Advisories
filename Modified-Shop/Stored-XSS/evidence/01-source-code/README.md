# Source Code Evidence

## Target

- Product: Modified Shop
- Version: 3.3.0 rev 16825
- Release Tag: v3.3.0
- Commit: `c531afbf7279f6e60ca7cda09f2c9d7ca5305c46`
- Affected File: `admin/module_newsletter.php`
- Vulnerable Parameter: `newsletter_body`

## Source File Integrity

The official source file and the laboratory source file were independently hashed using SHA-256.

**Official source file SHA-256:**

```text
fc5d364e85e83d76972db4a35f7820dbc8e390ce8c0861c3b9802df124f482ba
```

**Laboratory source file SHA-256:**

```text
65956b2366a917653933b2b3b306b87405819b04d3ffcf91240220b1b936bdfa
```

The source comparison identified a difference in the revision-control header. The executable code was otherwise identical.

## Evidence Chain

| File | Source Lines | Description |
|---|---|---|
| `01-input-source.txt` | 35–45 | Receives `newsletter_body` from the POST request. |
| `02-database-persistence.txt` | 55–70 | Includes the submitted content in the database persistence operation. |
| `03-database-retrieval.txt` | 172–180 | Retrieves the stored newsletter content. |
| `04-html-output-sink.txt` | 365–380 | Renders the stored content directly into the HTML response. |

These files contain selected source-code excerpts documenting the relevant input-to-output flow.

They are not complete copies of the original PHP source file.

## Evidence Integrity

SHA-256 hashes for the four source-code excerpts are recorded in:

`SHA256SUMS.txt`

The manifest covers the four evidence files and does not include its own checksum.

### Linux Verification

Run from this directory:

```bash
sha256sum -c SHA256SUMS.txt
```

### Windows PowerShell Verification

Run from this directory:

```powershell
Get-FileHash .\01-input-source.txt -Algorithm SHA256
Get-FileHash .\02-database-persistence.txt -Algorithm SHA256
Get-FileHash .\03-database-retrieval.txt -Algorithm SHA256
Get-FileHash .\04-html-output-sink.txt -Algorithm SHA256
```

Compare the calculated values against `SHA256SUMS.txt`.

All four evidence files were independently verified on Windows after transfer from Kali Linux.

## Technical Assessment

The source-to-sink analysis documents the following sequence:

1. User-controlled content is received through the `newsletter_body` POST parameter.
2. The submitted content is included in the database persistence operation.
3. The stored content is retrieved from the database.
4. The retrieved content is rendered directly into an HTML response without appropriate contextual output encoding or HTML sanitization at the identified sink.

The use of input preparation does not replace contextual output encoding.

These source-code excerpts support the technical assessment of the Stored Cross-Site Scripting vulnerability.
```

