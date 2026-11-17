# Cross-User Validation

## Objective

Validate whether stored JavaScript content submitted through the Newsletter Module executes in a separate authenticated administrative session.

## Environment

- Product: Modified Shop
- Version: 3.3.0 rev 16825
- Target: 192.168.56.144
- Component: Newsletter Module
- Endpoint: `/admin/module_newsletter.php`

## Test Scenario

Two separate accounts were used during validation.

**Account A — Test Researcher**

The account used to submit and store the proof-of-concept payload.

**Account B — Carlos Tuma**

A separate account, initially created as a customer account and subsequently granted administrative access, including permission to access the Newsletter Module.

The stored newsletter was accessed through separate authenticated browser sessions.

Account B accessed the previously stored newsletter without submitting the payload again.

## Evidence

| File | Description |
|---|---|
| `cross-user-account-test-researcher.png` | Screenshot showing JavaScript execution in the Test Researcher account. |
| `cross-user-account-admin.png` | Screenshot showing JavaScript execution in the Carlos Tuma account. |

## Observed Result

The JavaScript payload executed in both authenticated browser sessions.

The proof-of-concept displayed the laboratory IP address:

`192.168.56.144`

Execution in Account B occurred without a new payload submission.

This confirms cross-user Stored XSS execution between two separate authenticated accounts with access to the affected Newsletter Module.

## Scope and Limitations

- Both accounts had administrative access to the Newsletter Module during validation.
- Separate authenticated browser sessions were used.
- Equivalent administrative permissions across both accounts were not established.
- No session theft was demonstrated.
- No cookie extraction or reuse was performed.
- No session hijacking was demonstrated.
- No unauthorized administrative action was demonstrated.
- No privilege escalation or account compromise was demonstrated.

The validation was limited to the two tested accounts and the laboratory environment.

## Integrity

Original screenshots are preserved without annotations or modifications.

The screenshots document the observed JavaScript execution in both authenticated sessions.
