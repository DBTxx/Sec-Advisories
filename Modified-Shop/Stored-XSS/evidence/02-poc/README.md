# Proof of Concept Evidence

## Target

- Product: Modified Shop
- Version: 3.3.0 rev 16825
- Module: Newsletter
- Endpoint: admin/module_newsletter.php
- Parameter: newsletter_body

## Test Payload

```html
<img src=x onerror=alert(document.domain)>
```

## Evidence

| File | Description |
|---|---|
| 01-payload-submission.png | Shows the crafted HTML payload submitted through the Newsletter Module. |
| 02-javascript-execution.png | Shows JavaScript execution in the browser context, displaying the target domain. |

## Reproduction Summary

1. An authenticated administrator submits the crafted payload through the `newsletter_body` parameter.
2. The application stores the submitted content in the database.
3. The stored newsletter is accessed through a subsequent HTTP request.
4. The browser executes the injected JavaScript without requiring the payload to be submitted again.

## Result

Stored JavaScript execution was successfully reproduced in the Modified Shop 3.3.0 rev 16825 laboratory environment.

The evidence demonstrates persistent client-side script execution in the administrative interface.

## Scope

The proof of concept uses a benign JavaScript alert to demonstrate execution.

No session theft, cookie extraction, data exfiltration or unauthorized administrative actions were performed.
```

