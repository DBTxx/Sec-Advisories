```markdown
# Stored XSS in Modified Shop 3.3.0

Security research documenting a Stored Cross-Site Scripting (XSS) vulnerability identified in the Newsletter Module of Modified Shop.

## Vulnerability Overview

| Field | Details |
|---|---|
| Product | Modified Shop |
| Version | 3.3.0 rev 16825 |
| Component | Newsletter Module |
| Affected file | `admin/module_newsletter.php` |
| Parameter | `newsletter_body` |
| Vulnerability | Stored Cross-Site Scripting (XSS) |
| CWE | CWE-79 |
| CVE | Not assigned |
| CVSS | Pending assessment |

## Description

The Newsletter Module stores user-supplied HTML content and subsequently renders it without adequate output sanitization.

A crafted payload containing an event-handler attribute can execute JavaScript in the browser when the newsletter content is rendered.

The vulnerability was reproduced in an authorized RedScan Academy laboratory environment.

## Proof of Concept

The following harmless payload was used to demonstrate JavaScript execution:

```html
<img src=x onerror=alert(document.domain)>
```

Two scenarios were confirmed:

- Immediate execution after saving the newsletter.
- Persistent execution during a subsequent newsletter preview request.

Both scenarios were reproduced using the same authenticated default administrative account.

Cross-user execution was not independently tested.

## Repository Structure

- `SECURITY-ADVISORY.md` — Full security advisory.
- `poc/stored-xss.md` — Proof of Concept and reproduction details.
- `analysis/source-to-sink.md` — Source code analysis.
- `evidence/README.md` — Evidence documentation.

## Testing Scope

All testing was conducted in an authorized and controlled laboratory environment.

No data exfiltration, session theft, privilege escalation, or unauthorized administrative actions were demonstrated.

## Disclosure Status

Responsible disclosure is pending.

No vendor confirmation, security patch, or CVE assignment is currently claimed.

## Researcher

**Diego Teixeira**

GitHub: [DBTxx](https://github.com/DBTxx)

---

This repository is intended for educational and defensive security research.
