# Security Advisory — Stored XSS in Modified Shop Newsletter Module

**CVE:** Not assigned  
**Status:** Public advisory — vendor notification pending  
**Date:** 2026-11-13  
**Last Updated:** 2026-11-15  
**Advisory Version:** 1.3

---

## 1. Executive Summary

A Stored Cross-Site Scripting (Stored XSS) vulnerability was identified in the Newsletter Module of Modified Shop 3.3.0 (rev 16825).

The vulnerability affects the `newsletter_body` parameter in the administrative endpoint `admin/module_newsletter.php`. User-supplied content is stored in the database and subsequently rendered in the administrative interface without proper output encoding.

An authenticated user with permission to access the Newsletter Module can submit crafted HTML content containing JavaScript. The stored payload executes in the browser when the affected newsletter content is rendered.

The vulnerability was successfully reproduced in an authorized laboratory environment. JavaScript execution was confirmed both immediately after saving the content and during a subsequent request to the newsletter preview, demonstrating persistent storage and execution.

Cross-user execution was subsequently confirmed using two separate administrative accounts operating in independent authenticated sessions.

Account A submitted and stored the malicious newsletter content. Account B, a separate administrator with equivalent permissions, accessed the same stored newsletter and triggered JavaScript execution without submitting the payload again.

Additional testing confirmed that JavaScript executing in the context of Account B could perform a same-origin HTTP request to the administrative Newsletter Module using the browser's existing authenticated session.

The session cookie was configured with the `HttpOnly` attribute, preventing direct JavaScript access to its value through `document.cookie`. However, this protection does not prevent JavaScript executing within the application's origin from making authenticated same-origin requests.

No session theft, data exfiltration, privilege escalation, unauthorized administrative modifications, or account compromise was demonstrated.

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

Three relevant behaviors were examined:

- **Immediate execution:** The payload executes after the newsletter content is saved.
- **Persistent execution:** The payload executes again when the stored newsletter content is rendered in a subsequent preview request.
- **Cross-user execution:** The payload executes in the browser of a separate administrator who accesses the previously stored newsletter, without submitting the payload again.

Cross-user execution was confirmed using two separate administrative accounts operating in independent authenticated sessions.

Additional testing confirmed that JavaScript executing in the context of the second administrator could perform a same-origin HTTP request to the Newsletter Module using the browser's existing authenticated session.

The request returned HTTP 200 OK without redirection.

The session cookie attributes observed in the second administrator's browser were:

- `HttpOnly`: true
- `Secure`: false
- `SameSite`: Lax

The `HttpOnly` attribute prevents direct JavaScript access to the session cookie through `document.cookie`. However, it does not prevent JavaScript executing in the application's origin from making authenticated same-origin requests using the browser's existing session.

Potential security consequences, depending on the execution context, browser security controls, and privileges of the affected user, include:

- Manipulation of administrative interface content.
- Execution of unauthorized actions available to the affected user.
- Access to information exposed to JavaScript within the affected page.
- Abuse of administrative functionality available to the affected user.

These consequences represent potential risks and were not independently demonstrated during testing.

**Testing limitations:**

Session theft, session hijacking, sensitive data extraction, unauthorized administrative modifications, privilege escalation, and account compromise were not demonstrated.

The authenticated same-origin request was manually initiated through the browser Developer Tools Console. It was not demonstrated as an administrative action directly triggered by the stored XSS payload.

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

Testing was initially performed using the default administrative account. Cross-user validation was subsequently performed using a second administrative account with equivalent permissions.

---

## 6. Exploitation Scenario

An authenticated user with permission to access the Newsletter Module submits crafted HTML content through the `newsletter_body` parameter.

The application stores the submitted content in the database without applying appropriate HTML output encoding or sanitization.

When the stored newsletter content is rendered, the browser interprets the injected markup and executes the embedded JavaScript.

The vulnerability was confirmed through immediate execution after saving and persistent execution during a subsequent preview request.

Cross-user exploitation was subsequently validated:

1. Administrator A submitted and stored the malicious newsletter content.
2. Administrator B, operating in a separate authenticated session, accessed the same stored newsletter.
3. The JavaScript payload executed in Administrator B's browser without a new payload submission.

This confirms that exploitation is not limited to the account that originally created the malicious content.

Additional testing demonstrated that JavaScript executing in the context of Administrator B could make a same-origin request to the administrative Newsletter Module using the browser's existing authenticated session.

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

### Cross-User Validation

Cross-user execution was tested using two separate administrative accounts:

- **Account A:** Original administrative account used to submit and store the XSS payload.
- **Account B:** Separate administrative account (`test`), initially created as a customer account and subsequently granted administrative permissions equivalent to Account A.

Both accounts operated in independent authenticated sessions.

The same previously stored newsletter was accessed by both accounts.

The JavaScript payload executed in both sessions.

No additional payload submission was performed during cross-user validation.

This confirms that the stored payload can execute in the browser of a separate administrator who accesses the affected newsletter.

### Authenticated Same-Origin Request Validation

While authenticated as Account B, a same-origin HTTP request was manually executed through the browser Developer Tools Console:

```javascript
fetch('/admin/module_newsletter.php?ID=1', {
  credentials: 'same-origin'
})
.then(response => response.text())
.then(html => console.log('Status: conteúdo recebido', html.length))
.catch(error => console.error(error));
```

Observed response:

- **Requested endpoint:** `/admin/module_newsletter.php?ID=1`
- **HTTP status:** 200 OK
- **Final URL:** `http://192.168.56.144:8080/admin/module_newsletter.php?ID=1`
- **Redirected:** false
- **Response body length:** 30,313 characters

The request was performed in the authenticated browser session of Account B and successfully returned a response from the administrative endpoint.

This demonstrates that same-origin JavaScript can make authenticated requests using the browser's existing session.

The request was manually initiated through Developer Tools. It was not directly triggered by the stored XSS payload.

### Session Cookie Attributes

The session cookie observed in Account B's browser had the following attributes:

| Attribute | Observed value |
|---|---|
| HttpOnly | true |
| Secure | false |
| SameSite | Lax |

The `HttpOnly` attribute prevents direct access to the cookie value through JavaScript APIs such as `document.cookie`.

No session cookie theft or session hijacking was demonstrated.

---

## 8. Proof of Concept (PoC)

Testing was performed in an authorized laboratory environment using administrative accounts with permission to access the Newsletter Module.

**Payload:**

```html
<img src=x onerror=alert(document.domain)>
```

The payload uses a JavaScript alert to demonstrate execution. It does not perform data exfiltration or modify application records.

### Observed Behavior

Three relevant observations were documented:

1. **Immediate execution:** After submitting the crafted newsletter content and saving the record, the JavaScript payload executed in the browser.
2. **Persistent execution:** During a subsequent request to the newsletter preview, the same payload executed again without submitting a new POST request.
3. **Cross-user execution:** A second administrator, operating in an independent authenticated session, accessed the same stored newsletter and triggered JavaScript execution without submitting the payload.

The JavaScript alert displayed the laboratory application host:

`192.168.56.144`

These observations confirm that the application persistently stores the injected content and subsequently renders it in a way that allows JavaScript execution in the browser, including in the session of a separate administrator.

### Authenticated Request Observation

A same-origin request to the administrative Newsletter Module was manually executed while authenticated as the second administrator.

The request returned HTTP 200 OK without redirection.

This confirms that the browser could retrieve the administrative endpoint using its existing authenticated session.

The request was manually initiated through Developer Tools and should not be interpreted as proof that the stored payload itself performed an administrative operation.

### Validation Scope

The following behaviors were confirmed:

- Immediate JavaScript execution after saving.
- Persistent JavaScript execution during a subsequent preview request.
- Cross-user JavaScript execution in a separate administrative session.
- Successful same-origin authenticated request to the administrative Newsletter Module.

The following behaviors were not demonstrated:

- Session cookie theft.
- Session hijacking.
- Sensitive data exfiltration.
- Unauthorized administrative modifications.
- Privilege escalation.
- Account compromise.

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

### Cross-User Reproduction

10. Authenticate using a second administrative account in an independent browser session.
11. Ensure that the second account has permission to access the Newsletter Module.
12. Access the same previously stored newsletter through its preview functionality.
13. Observe that the JavaScript payload executes in the second administrator's browser without a new payload submission.

### Authenticated Request Observation

14. While authenticated as the second administrator, open the browser Developer Tools Console.
15. Execute a same-origin fetch request to the Newsletter Module endpoint.
16. Observe the HTTP response status and final URL.

**Expected behavior:**

User-supplied HTML should be safely sanitized or encoded according to the application's intended functionality.

**Actual behavior:**

The stored markup is interpreted by the browser, resulting in JavaScript execution, including when accessed by a separate administrator.

**Testing scope:** Cross-user execution was confirmed. Authenticated same-origin request behavior was also observed through a manually initiated browser request. Session theft and unauthorized administrative actions were not demonstrated.

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
| 2026-11-15 | Cross-user Stored XSS execution confirmed using two separate administrative accounts |
| 2026-11-15 | Authenticated same-origin request behavior examined |
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
| 1.3     | 2026-11-15 | Added cross-user Stored XSS validation, documented authenticated same-origin request behavior and session cookie attributes, and updated testing scope and impact limitations |

---

## 18. Legal Notice

This advisory is published for educational and defensive security purposes.

Testing was conducted in an authorized and controlled laboratory environment.

The information provided must not be used to access, test, or compromise systems without explicit authorization.

---

## 19. Contact

- **Researcher:** Diego Teixeira
- **Research environment:** RedScan Academy
- **GitHub:** https://github.com/DBTxx/Sec-Advisories
```

