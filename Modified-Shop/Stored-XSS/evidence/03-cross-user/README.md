# Cross-User Validation

## Objective

Demonstrate Stored XSS execution across two different administrative accounts.

## Environment

- Product: Modified Shop
- Version: 3.3.0 rev 16825
- Target: 192.168.56.144
- Component: Newsletter Module
- Endpoint: admin/module_newsletter.php

## Test Scenario

The malicious newsletter content was previously stored using Account A.

The same stored newsletter was subsequently accessed using two different administrative accounts in separate browser sessions.

JavaScript execution was observed in both sessions without submitting the payload again.

## Evidence

| File | Description |
|---|---|
| cross-user-account-test-researcher.png | Original screenshot showing execution in the Test Researcher account. |
| cross-user-account-admin.png | Original screenshot showing execution in the Carlos Tuma account. |

## Observed Result

The JavaScript payload executed in both administrative contexts and displayed the lab IP address:

192.168.56.144

This confirms cross-user Stored XSS execution in the tested environment.

## Scope and Limitations

- Both accounts had administrative privileges.
- Independent browser sessions were used, according to the test procedure.
- No session theft was demonstrated.
- No cookie extraction or reuse was performed.
- No unauthorized administrative action was demonstrated.

## Integrity

Original screenshots are preserved without annotations or modifications.
