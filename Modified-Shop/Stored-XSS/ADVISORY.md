# Security Advisory — Stored XSS in Modified Shop Newsletter Module

**CVE:** Not assigned  
**Status:** Public advisory — vendor notification pending  
**Date:** 2026-11-13  
**Last Updated:** 2026-11-17  
**Advisory Version:** 1.4  

---

## 1. Executive Summary

A Stored Cross-Site Scripting (Stored XSS) vulnerability was identified in the Newsletter Module of Modified Shop 3.3.0 rev 16825.

The vulnerability affects the administrative endpoint:

`/admin/module_newsletter.php`

The vulnerable parameter is:

`newsletter_body`

User-controlled content submitted through this parameter is stored in the database and subsequently rendered in the administrative interface without appropriate contextual output encoding or HTML sanitization.

A proof of concept using the following benign payload demonstrated JavaScript execution:

```html
<img src=x onerror=alert(document.domain)>
```

The vulnerability was confirmed through dynamic testing and static source-code analysis.

Testing demonstrated that:

- The submitted payload is accepted and stored.
- JavaScript executes after the newsletter is saved.
- JavaScript executes again when the stored newsletter is accessed through a subsequent preview request.
- The stored payload executes in two separate authenticated administrative sessions.
- A second account could trigger the stored payload without submitting it again.

Cross-user execution was confirmed using two separate accounts with access to the Newsletter Module.

A separate authenticated same-origin HTTP request test was also performed manually through the browser Developer Tools Console while authenticated as the second account. The request returned HTTP 200 OK without redirection.

**Important distinction:** this HTTP request was initiated manually by the researcher through the Developer Tools Console. It was not initiated by the stored XSS payload and does not demonstrate an unauthorized administrative action.

No session theft, cookie extraction, session hijacking, privilege escalation, unauthorized administrative modification, data exfiltration, or account compromise was demonstrated during testing.

The vulnerability is classified as CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting').

CVSS scoring and severity assessment remain pending.

---

## 2. Affected Product

| Field | Value |
|---|---|
| Product | Modified Shop |
| Confirmed Version | 3.3.0 rev 16825 |
| Release Tag | v3.3.0 |
| Commit | `c531afbf7279f6e60ca7cda09f2c9d7ca5305c46` |
| Affected Component | Newsletter Module |
| Affected File | `admin/module_newsletter.php` |
| Vulnerable Parameter | `newsletter_body` |
| Vulnerability Type | Stored Cross-Site Scripting |
| CVE | Not assigned |

The product version metadata identifies 2026-01-29 as the source-declared release date.

Only Modified Shop 3.3.0 rev 16825 was tested and confirmed vulnerable.

Other versions have not been tested. Their vulnerability status is therefore unknown.

---

## 3. Vulnerability Description

The Newsletter Module accepts user-controlled content through the `newsletter_body` POST parameter.

The submitted value is processed using:

```php
$newsletter_body = xtc_db_prepare_input($_POST['newsletter_body']);
```

The value is subsequently stored in the database.

When an existing newsletter is retrieved, its stored content is inserted directly into the generated HTML response.

The relevant source-to-sink flow is:

1. User-controlled input.
2. Input preparation.
3. Database persistence.
4. Database retrieval.
5. HTML rendering without appropriate output encoding or sanitization.

### 3.1 Input Source

File:

`admin/module_newsletter.php`

Line 39:

```php
$newsletter_body = xtc_db_prepare_input($_POST['newsletter_body']);
```

The application receives the `newsletter_body` parameter from the submitted HTTP request.

The use of `xtc_db_prepare_input()` does not constitute contextual HTML output encoding.

Input preparation and output encoding are different security operations.

### 3.2 Database Persistence

The newsletter content is included in the database data array:

```php
'body' => $newsletter_body
```

The application subsequently performs the database operation using:

```php
xtc_db_perform(TABLE_MODULE_NEWSLETTER, $sql_data_array);
```

Relevant source-code lines: 55–68.

This allows the submitted content to persist beyond the original HTTP request.

### 3.3 Database Retrieval

The application retrieves the newsletter using:

```php
$newsletters_query = xtc_db_query("SELECT *
                                     FROM ".TABLE_MODULE_NEWSLETTER."
                                    WHERE newsletter_id = '".(int)$_GET['ID']."'");
$newsletters_data = xtc_db_fetch_array($newsletters_query);
```

Relevant source-code lines: 172–175.

The previously stored content is retrieved and made available for rendering.

### 3.4 HTML Output Sink

The retrieved content is inserted directly into an HTML response:

```php
echo '<table style="border: 1px solid #a3a3a3; width:100%"><tr><td>'.$newsletters_data['body'].'</td></tr></table>';
```

Relevant source-code line: 375.

The output sink does not apply contextual HTML encoding or an HTML sanitization process before rendering the stored content.

Consequently, executable HTML event-handler attributes can be interpreted by the browser.

---

## 4. Security Impact

The confirmed impact is persistent JavaScript execution in the browser context of authenticated users who can access the affected Newsletter Module.

### 4.1 Persistent Execution

The submitted content remains stored in the database.

JavaScript execution was reproduced:

- After submitting and saving the newsletter.
- During a subsequent request to access the stored newsletter preview.

The second execution did not require submitting the payload again.

### 4.2 Cross-User Execution

Cross-user validation was performed using two separate accounts.

**Account A — Payload Submission**

Account A was an existing administrative account used to submit and store the proof-of-concept payload.

**Account B — Cross-User Validation**

Account B was originally created as a standard customer account. Using Account A, administrative permissions were subsequently granted to Account B, including permission to access the Newsletter Module.

Account B accessed the previously stored newsletter through a separate authenticated session, triggering JavaScript execution without submitting the payload again.

A separate HTTP request test was manually performed through the browser Developer Tools Console while authenticated as Account B.

The following JavaScript was executed manually:

```javascript
fetch('/admin/module_newsletter.php?ID=1', {
  credentials: 'same-origin'
})
.then(response => response.text())
.then(html => console.log('Status: conteúdo recebido', html.length))
.catch(error => console.error(error));
```

Observed results:

| Field | Result |
|---|---|
| Requested Endpoint | `/admin/module_newsletter.php?ID=1` |
| HTTP Status | 200 OK |
| Final URL | `http://192.168.56.144:8080/admin/module_newsletter.php?ID=1` |
| Redirected | No |
| Response Body Length | 30,313 characters |

This test documents the behavior of a manually initiated same-origin request in the authenticated browser session.

It was not triggered by the stored XSS payload.

The HTTP status and response length alone do not establish the contents or sensitivity of the returned response. No unauthorized administrative action was demonstrated through this test.

### 4.4 Session Cookie Attributes

The following cookie attributes were observed during testing:

| Attribute | Observed Value |
|---|---|
| HttpOnly | Present |
| Secure | Not present |
| SameSite | Lax |

The presence of HttpOnly provides protection against direct JavaScript access to the session cookie.

However, HttpOnly does not prevent JavaScript executing in the application's origin from potentially making same-origin requests using the browser's existing authenticated session.

The absence of the Secure attribute means the cookie is not explicitly restricted to HTTPS transmission.

These observations are security-relevant configuration findings. They do not establish that session theft or session hijacking occurred.

### 4.5 Actions Not Demonstrated

The following activities were not performed or demonstrated:

- Session cookie extraction.
- Session hijacking.
- Unauthorized administrative actions.
- Privilege escalation.
- Unauthorized account modification.
- Data exfiltration.
- Account compromise.
- Execution against users without access to the affected administrative module.

The impact assessment must remain limited to the demonstrated behavior.

---

## 5. Vulnerability Classification

| Classification | Value |
|---|---|
| CWE | CWE-79 |
| OWASP Top 10 | A03:2021 – Injection |
| Vulnerability Type | Stored Cross-Site Scripting |
| Attack Vector | Web application |
| Attacker Privileges | Administrative access with permission to modify newsletter content |
| Victim Access | Authenticated access to the affected newsletter content |
| User Interaction | Victim must access the affected content |
| CVSS | Pending assessment |
| Severity | Pending assessment |

---

## 6. Proof of Concept

### 6.1 Payload

The following benign payload was used to validate JavaScript execution:

```html
<img src=x onerror=alert(document.domain)>
```

The payload displays the current document domain when the browser processes the injected HTML.

No external communication, credential collection, session extraction, or destructive operation is performed by this payload.

### 6.2 Reproduction Steps

1. Authenticate to the Modified Shop administrative interface using an account with permission to access the Newsletter Module.
2. Navigate to the Newsletter Module.
3. Open or create a newsletter.
4. Submit the proof-of-concept payload through the `newsletter_body` parameter.
5. Save the newsletter.
6. Observe JavaScript execution in the browser.
7. Access the stored newsletter through a subsequent preview request.
8. Observe that JavaScript executes again without resubmitting the payload.
9. Authenticate using a separate account with permission to access the same module.
10. Access the previously stored newsletter.
11. Observe JavaScript execution in the second account's browser session.

### 6.3 Expected Result

User-controlled content should not execute as JavaScript when rendered in the administrative interface.

### 6.4 Actual Result

The submitted content was stored and subsequently rendered as executable HTML.

JavaScript execution was confirmed in two separate authenticated administrative sessions.

---

## 7. Source Code Verification

Static source-code analysis confirmed the complete source-to-sink flow:

| Stage | Location | Behavior |
|---|---|---|
| Input | Line 39 | Receives `newsletter_body` |
| Persistence | Lines 55–68 | Stores submitted content |
| Retrieval | Lines 172–175 | Retrieves stored newsletter |
| Output | Line 375 | Renders stored content without appropriate contextual encoding or sanitization |

### 7.1 Source Integrity

The official source file and the laboratory source file were compared.

The comparison identified a difference in the revision-control header. The executable code was otherwise identical.

SHA-256 hashes:

**Official source file:**

```text
fc5d364e85e83d76972db4a35f7820dbc8e390ce8c0861c3b9802df124f482ba
```

**Laboratory source file:**

```text
65956b2366a917653933b2b3b306b87405819b04d3ffcf91240220b1b936bdfa
```

The source-code excerpts and their corresponding SHA-256 manifest are available in the evidence repository.

---

## 8. Cross-User Validation

Cross-user validation was performed on 2026-11-15.

The objective was to determine whether the stored payload could execute in a separate authenticated session.

### 8.1 Test Configuration

| Account | Description |
|---|---|
| Account A | Existing administrative account used to submit and store the payload. |
| Account B | Originally a standard customer account. Administrative permissions, including access to the Newsletter Module, were granted using Account A. |

### 8.2 Procedure

1. Account A submitted the benign payload.
2. The application stored the content.
3. Account B authenticated through a separate browser session.
4. Account B accessed the same stored newsletter.
5. The browser executed the stored JavaScript without a new payload submission.

### 8.3 Result

Cross-user execution was confirmed.

The result demonstrates that the vulnerability is not restricted to the original submitting session.

The test does not establish that every administrative role, permission configuration, or application user is affected.

Screenshots documenting the test are available in:

`evidence/03-cross-user/`

---

## 9. HTTP Request Evidence

HTTP request evidence was captured on 2026-11-17 using Firefox Developer Tools.

The captured request corresponds to the submission of the newsletter content through:

`POST /admin/module_newsletter.php?action=save`

The request includes the `newsletter_body` parameter containing the proof-of-concept payload.

The evidence package contains:

| File | Description |
|---|---|
| `01-post-request.txt` | Captured HTTP POST request headers |
| `02-post-payload.txt` | Captured multipart request body |
| `SHA256SUMS.txt` | SHA-256 integrity manifest |

The session cookie was redacted from the published request evidence.

The captured request documents how the payload was transmitted to the application.

It does not independently establish database persistence or JavaScript execution; those behaviors were validated separately.

Evidence directory:

`evidence/04-http-requests/`

---

## 10. Evidence Repository

The supporting evidence is organized into the following directories:

```text
Stored-XSS/
├── ADVISORY.md
└── evidence/
    ├── README.md
    ├── 01-source-code/
    ├── 02-poc/
    ├── 03-cross-user/
    ├── 04-http-requests/
    └── 05-validation/
```

### 10.1 Evidence Categories

**01-source-code/**

Contains source-code excerpts documenting the input, database persistence, retrieval, and vulnerable output sink.

**02-poc/**

Contains screenshots documenting payload submission and JavaScript execution.

**03-cross-user/**

Contains screenshots documenting execution in two separate authenticated accounts.

**04-http-requests/**

Contains the captured HTTP POST request, multipart payload, and SHA-256 manifest.

**05-validation/**

Contains the validation record, confirmed results, testing methodology, and limitations.

The evidence repository is intended to support independent technical review of the reported vulnerability.

---

## 11. Remediation Recommendations

The vulnerable output operation must be corrected according to the intended content format.

### 11.1 If Newsletter Content Is Intended to Be Plain Text

Apply contextual HTML output encoding before inserting the content into the HTML response.

For PHP, an appropriate encoding function is:

```php
htmlspecialchars(
    $newsletters_data['body'],
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

### 11.2 If Newsletter Content Is Intended to Contain HTML

If the application requires HTML formatting in newsletter content, applying `htmlspecialchars()` to the entire body may break legitimate functionality.

In this case, implement server-side HTML sanitization using a maintained allowlist-based sanitizer.

The sanitizer should:

- Allow only explicitly required HTML elements.
- Restrict permitted attributes.
- Remove event-handler attributes such as `onerror` and `onclick`.
- Reject dangerous URL schemes.
- Prevent script execution through injected markup.
- Apply sanitization before the content is rendered.

#### 11.3 Additional Security Recommendations

- Review all administrative output locations that render newsletter content to identify similar injection vulnerabilities.
- Apply context-appropriate output encoding consistently across the application.
- Review and enforce least-privilege access controls for the Newsletter Module, separating content creation, modification, and viewing permissions where appropriate.
- Consider implementing Content Security Policy (CSP) as an additional defense-in-depth measure.
- Review session cookie configuration and enable the Secure attribute when the application is served exclusively over HTTPS.
- Implement regression tests covering newsletter creation, modification, preview, and subsequent retrieval to ensure that stored content cannot execute unauthorized JavaScript.

Content Security Policy should not be considered a replacement for proper contextual output encoding or HTML sanitization.

---

## 12. Detection and Defensive Considerations

Potential indicators of exploitation include:

- Unexpected HTML markup in newsletter content.
- Event-handler attributes such as `onerror`, `onload`, or `onclick`.
- Suspicious content submitted through the `newsletter_body` parameter.
- Unexpected JavaScript execution when administrators access stored newsletters.

Defensive review should include:

- Inspection of existing newsletter records.
- Review of administrative access logs.
- Examination of newsletter creation and modification events.
- Validation of stored HTML content.
- Review of other application components that render user-controlled content.

The presence of suspicious HTML in a newsletter should be investigated in context. HTML content alone is not sufficient to establish exploitation.

---

## 13. Disclosure Timeline

| Date | Event |
|---|---|
| 2026-11-13 | Initial vulnerability identification |
| 2026-11-13 | Static source-code analysis and source-to-sink identification |
| 2026-11-13 | Dynamic reproduction of Stored XSS |
| 2026-11-14 | Public advisory published |
| 2026-11-14 | Vulnerability submission sent to VulDB |
| 2026-11-15 | Cross-user execution confirmed using two separate authenticated accounts |
| 2026-11-15 | Same-origin request behavior examined through manual browser testing |
| 2026-11-17 | HTTP POST request and multipart payload evidence captured and documented |
| 2026-11-17 | Evidence repository updated with HTTP request and validation documentation |
| Pending | Vendor notification |
| Pending | Vendor acknowledgment and technical assessment |
| Pending | Patch availability and verification |

The VulDB submission does not imply acceptance, publication, or CVE assignment.

Vendor notification and response remain pending.

---

## 14. Vendor Communication

**Vendor notification status:** Pending.

At the time of this advisory revision, vendor notification has not been confirmed.

No vendor acknowledgment, patch confirmation, or official remediation statement is claimed.

The vulnerability remains publicly documented while vendor communication is pending.

Any future vendor response or remediation status will be reflected in a subsequent advisory revision.

---

## 15. Credits

**Researcher:** Diego Teixeira  
**Research Type:** Independent security research  
**Disclosure Status:** Public advisory; vendor notification pending  

The vulnerability was identified and validated through authorized testing in a controlled RedScan Academy laboratory environment.

---

## 16. References

- CWE-79 — Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')  
  https://cwe.mitre.org/data/definitions/79.html

- OWASP Cross Site Scripting Prevention Cheat Sheet  
  https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html

- OWASP Top 10: A03:2021 – Injection  
  https://owasp.org/Top10/A03_2021-Injection/

- PHP Manual — htmlspecialchars()  
  https://www.php.net/manual/en/function.htmlspecialchars.php

---

## 17. Revision History

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-11-13 | Initial vulnerability documentation and proof of concept |
| 1.1 | 2026-11-14 | Additional technical analysis and public advisory |
| 1.2 | 2026-11-15 | Cross-user validation and impact clarification |
| 1.3 | 2026-11-15 | Additional testing details and same-origin request observations |
| 1.4 | 2026-11-17 | HTTP request evidence, validation documentation, source integrity details, and clarification of testing limitations |

---

## 18. Legal Notice

This advisory documents a security vulnerability identified through authorized research in a controlled laboratory environment.

Testing was limited to the research environment and the accounts used for validation.

The proof-of-concept payload was designed to demonstrate JavaScript execution and did not perform credential theft, session extraction, data exfiltration, destructive operations, or unauthorized system modifications.

The information is provided for security research, defensive analysis, and remediation purposes.

The researcher makes no claim that systems outside the tested environment are affected without independent verification.
```

