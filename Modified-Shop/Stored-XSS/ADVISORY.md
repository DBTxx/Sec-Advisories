```markdown
# Security Advisory — Stored XSS in Modified Shop Newsletter Module

**CVE:** Not assigned  
**Status:** Public advisory — vendor notification pending  
**Date:** 2026-11-13  
**Last Updated:** 2026-11-14

---

## 1. Executive Summary

A Stored Cross-Site Scripting (Stored XSS) vulnerability was identified in the Newsletter Module of Modified Shop 3.3.0 (rev 16825).

The vulnerability affects the `newsletter_body` parameter in the administrative endpoint `admin/module_newsletter.php`. User-supplied content is stored in the database and subsequently rendered in the administrative interface without proper output encoding.

An authenticated user with permission to access the Newsletter Module can submit crafted HTML content containing JavaScript. The stored payload executes in the browser when the affected newsletter content is rendered.

The vulnerability was successfully reproduced in an authorized laboratory environment. JavaScript execution was confirmed both immediately after saving the content and during a subsequent request to the newsletter preview, demonstrating persistent storage and execution.

Both scenarios were reproduced using the same administrative account. Execution in a separate administrative session has not been verified.

No session theft, data exfiltration, privilege escalation, unauthorized administrative actions, or account compromise was attempted or demonstrated.

---

## 2. Affected Product and Versions

- **Vendor:** Modified Shop
- **Vendor Repository:** https://github.com/modified-shop/modified-shop
- **Product:** Modified Shop
- **Version:** 3.3.0 rev 16825
- **Official release tag:** `v3.3.0`
- **Official commit:** `c531afbf7279f6e60ca7cda09f2c9d7ca5305c46`
- **Component:** Newsletter Module
- **Affected file:** `admin/module_newsletter.php`
- **Affected endpoint:** `/admin/module_newsletter.php`
- **Vulnerable parameter:** `newsletter_body`
- **Source-declared release date (version metadata):** 2026-01-29

**Affected versions:** Confirmed in Modified Shop 3.3.0 rev 16825. Other versions have not been tested.

---

## 3. Vulnerability Description

The vulnerability exists in the Newsletter Module administrative endpoint:

`admin/module_newsletter.php`

The application accepts user-controlled content through the POST parameter `newsletter_body`.

The input is processed using `xtc_db_prepare_input()` (line 39), then included in the database record through `xtc_db_perform()` (lines 61–67).

When an existing newsletter is retrieved, its stored content is loaded from the database (lines 173–175).

The vulnerable output occurs at line 375, where the stored newsletter body is concatenated directly into an HTML response without contextual output encoding:

```php
echo '<table style="border: 1px solid #a3a3a3; width:100%"><tr><td>'.$newsletters_data['body'].'</td></tr></table>';
```

No explicit HTML output encoding or HTML sanitization is applied to the stored content at this output sink.

As a result, an authenticated user with permission to access the Newsletter Module can store HTML markup containing JavaScript, which is subsequently interpreted by the browser when the newsletter preview is rendered.

The issue was dynamically confirmed using a benign JavaScript payload in an authorized laboratory environment.

The use of `xtc_db_prepare_input()` does not provide HTML output encoding. Database input preparation and output encoding serve different security purposes. Stored content must be safely handled according to its output context.

---

## 4. Impact

Successful exploitation results in JavaScript execution in the security context of the Modified Shop administrative interface.

Two execution scenarios were confirmed:

- **Immediate execution:** The payload executes after the newsletter content is saved.
- **Persistent execution:** The payload executes again when the stored newsletter content is rendered in a subsequent preview request.

Potential security consequences, depending on the execution context, browser security controls, and privileges of the affected user, include:

- Manipulation of administrative interface content.
- Execution of unauthorized actions available to the affected user.
- Access to information exposed to JavaScript within the affected page.
- Abuse of administrative functionality available to the affected user.

These consequences represent potential risks and were not independently demonstrated during testing.

**Testing limitations:**

Session theft, sensitive data extraction, unauthorized administrative actions, privilege escalation, and account compromise were not attempted or demonstrated.

The impact on other administrative users remains unconfirmed because cross-user execution was not tested.

The practical impact depends on the permissions required to access the Newsletter Module and whether other administrative users can access the affected stored content.

---

## 5. Classification

- **Vulnerability type:** Stored Cross-Site Scripting (Stored XSS)
- **CWE:** CWE-79 — Improper Neutralization of Input During Web Page Generation
- **OWASP category:** A03:2021 — Injection
- **Authentication required:** Yes
- **Privilege requirement:** Administrative access with permission to access the Newsletter Module
- **CVSS:** Pending assessment
- **Severity:** Not yet assigned

Source code analysis confirmed that access to administrative pages is controlled by `xtc_check_permission()` in `admin/includes/functions/general.php`.

The function requires an administrative session and explicit permission for the requested page, except for specific predefined exceptions.

The Newsletter Module is subject to this permission check through `admin/includes/application_top.php`.

Testing was performed using the default administrative account.

---

## 6. Exploitation Scenario

An authenticated user with permission to access the Newsletter Module submits crafted HTML content through the `newsletter_body` parameter.

The application stores the submitted content in the database without applying appropriate HTML output encoding or sanitization.

When the stored newsletter content is rendered, the browser interprets the injected markup and executes the embedded JavaScript.

The vulnerability was confirmed through immediate execution after saving and persistent execution during a subsequent preview request.

Both scenarios were reproduced using the same authenticated administrative account. Execution in a separate administrative session has not been independently verified.

---

## 7. Technical Evidence

### Source

File: `admin/module_newsletter.php`

Line 39:

```php
$newsletter_body = xtc_db_prepare_input($_POST['newsletter_body']);
```

### Database persistence

Lines 61–67:

```php
'body' => $newsletter_body
```

The data is persisted through:

```php
xtc_db_perform(TABLE_MODULE_NEWSLETTER, $sql_data_array);
```

### Database retrieval

Lines 173–175:

```php
$newsletters_query = xtc_db_query("SELECT *
                                     FROM ".TABLE_MODULE_NEWSLETTER."
                                    WHERE newsletter_id = '".(int)$_GET['ID']."'");
$newsletters_data = xtc_db_fetch_array($newsletters_query);
```

### Vulnerable output

Line 375:

```php
echo '<table style="border: 1px solid #a3a3a3; width:100%"><tr><td>'.$newsletters_data['body'].'</td></tr></table>';
```

### Root cause

The application stores user-controlled newsletter content and later inserts it directly into an HTML response without context-appropriate output encoding or sanitization.

### Source code verification

The vulnerable file from the laboratory environment was compared against the corresponding file in the official Modified Shop v3.3.0 source repository.

The comparison identified only one difference: the revision-control header. The executable code was otherwise identical.

This establishes that the vulnerable implementation observed in the laboratory matches the official v3.3.0 source file.

---

## 8. Proof of Concept (PoC)

Testing was performed in an authorized laboratory environment using the default administrative account with permission to access the Newsletter Module.

**Payload:**

```html
<img src=x onerror=alert(document.domain)>
```

The payload uses a JavaScript alert to demonstrate execution. It does not perform data exfiltration or modify application records.

### Observed Behavior

Two execution scenarios were confirmed:

1. **Immediate execution:** After submitting the crafted newsletter content and saving the record, the JavaScript payload executed in the browser.
2. **Persistent execution:** During a subsequent request to the newsletter preview, the same payload executed again without submitting a new POST request.

The JavaScript alert displayed the laboratory application host:

`192.168.56.144`

These observations confirm that the application persistently stores the injected content and subsequently renders it in a way that allows JavaScript execution in the browser.

### Validation Scope

Both execution scenarios were reproduced using the same authenticated administrative account.

Execution in a separate administrative session was not tested. Cross-user exploitation therefore remains unconfirmed.

---

## 9. Reproduction Steps

1. Access the Modified Shop administrative panel in an authorized test environment.
2. Authenticate using an administrative account with permission to access the Newsletter Module.
3. Open `admin/module_newsletter.php`.
4. Locate the newsletter body field.
5. Submit the following payload:

   ```html
   <img src=x onerror=alert(document.domain)>
   ```

6. Save the newsletter.
7. Observe the JavaScript alert.
8. Access the saved newsletter preview again using its corresponding ID (e.g., `?ID=1`).
9. Observe that the JavaScript alert executes again, confirming persistent storage and execution during a subsequent request without submitting a new payload.

**Expected behavior:**

User-supplied HTML should be safely sanitized or encoded according to the application's intended functionality.

**Actual behavior:**

The stored markup is interpreted by the browser, resulting in JavaScript execution.

**Testing scope:** Both execution scenarios were reproduced using the same authenticated account. Cross-user execution was not tested.

---

## 10. Mitigation

Until a definitive fix is available:

- Restrict access to the Newsletter Module to trusted users who require it.
- Avoid submitting untrusted or unsanitized HTML through the Newsletter Module.
- Review existing newsletter records for potentially malicious markup.
- Consider implementing a restrictive Content Security Policy (CSP) as an additional security layer.

These measures reduce exposure but do not replace the required code correction.

---

## 11. Recommended Fix

The application should apply server-side HTML sanitization using a well-maintained HTML sanitization library with a strict allowlist if newsletter formatting requires HTML.

The sanitization policy should:

- Allow only HTML elements required for legitimate newsletter formatting.
- Remove executable elements, including `<script>`.
- Remove event-handler attributes such as `onerror`, `onload`, and `onclick`.
- Restrict URL attributes to explicitly permitted schemes.
- Reject dangerous elements and attributes, including unsafe SVG and embedded content.
- Ensure that sanitized content is rendered safely in its intended HTML context.

If HTML formatting is not required, the application should use context-appropriate output encoding, such as `htmlspecialchars()` for HTML text contexts.

The application must not rely on database input preparation as an XSS prevention mechanism.

Existing stored newsletter records should also be reviewed and sanitized before being rendered.

The definitive fix must prevent untrusted JavaScript execution while preserving the module's intended newsletter formatting functionality.

---

## 12. Detection and Indicators

Potential indicators include:

- Newsletter records containing unexpected HTML elements.
- Event handlers such as `onerror`, `onload`, and other executable attributes.
- Suspicious JavaScript-related content in stored newsletter bodies.
- Unexpected modifications to newsletter records.

A database review should be performed to identify previously stored content that may contain malicious markup.

Example search terms:

```text
<script
<iframe
<svg
onerror=
onload=
onclick=
onmouseover=
javascript:
```

These indicators are not exhaustive and should be treated as investigation leads rather than definitive proof of exploitation.

### Suggested Monitoring

Monitor administrative POST requests to `admin/module_newsletter.php`, particularly submissions containing unexpected HTML elements or event-handler attributes in the `newsletter_body` parameter.

Correlate suspicious submissions with subsequent newsletter preview requests to identify possible stored XSS exploitation attempts.

Monitoring should be performed without logging sensitive newsletter content unnecessarily.

---

## 13. Disclosure Timeline

| Date       | Event |
|------------|-------|
| 2026-11-13 | Initial vulnerability identification |
| 2026-11-13 | Static source code analysis |
| 2026-11-13 | Dynamic reproduction in an authorized laboratory |
| 2026-11-13 | Verification against official source code |
| 2026-11-14 | Advisory published on GitHub |
| 2026-11-14 | Vulnerability submitted to VulDB |
| Pending | Private vulnerability disclosure to the vendor |
| Pending | Vendor acknowledgment |
| Pending | Vendor technical assessment |
| Pending | Security fix or mitigation availability |

---

## 14. Vendor Communication

- **Intended channel:** Private communication through official Modified Shop maintainer contact channels.
- **Status:** Pending.

The vendor should be provided with the technical description, affected source code location, reproduction procedure, and recommended remediation.

No vendor confirmation or patch availability is currently claimed.

---

## 15. Credits

- **Researcher:** Diego Teixeira
- **Research type:** Independent security research
- **Disclosure:** Responsible vulnerability disclosure

---

## 16. References

- CWE-79 — https://cwe.mitre.org/data/definitions/79.html
- OWASP Cross Site Scripting Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP Top 10: A03:2021 Injection — https://owasp.org/Top10/A03_2021-Injection/
- Modified Shop Official Repository — https://github.com/modified-shop/modified-shop

---

## 17. Revision History

| Version | Date       | Change |
|---------|------------|--------|
| 1.0     | 2026-11-13 | Initial consolidated advisory |
| 1.1     | 2026-11-13 | Technical review, privilege requirements clarification, impact qualification, and disclosure scope corrections |
| 1.2     | 2026-11-14 | Clarified cross-user testing limitations, refined impact statements, reduced repetition, and improved remediation guidance |

---

## 18. Legal Notice

This advisory is published for educational and defensive security purposes.

Testing was conducted in an authorized and controlled laboratory environment.

The information provided must not be used to access, test, or compromise systems without explicit authorization.

---

## 19. Contact

- **Researcher:** Diego Teixeira
- **Research environment:** RedScan Academy laboratory
- **GitHub:** https://github.com/DBTxx/Sec-Advisories
```
