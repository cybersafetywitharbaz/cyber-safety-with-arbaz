# Reflected XSS protected by CSP, with CSP bypass

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Practitioner
**Solved:** 15 November 2026

## Objective

Bypass the Content Security Policy by injecting a new script-src rule through the CSP report-uri token parameter.

## Method

1. Put the payload in the search term: `<script>alert(1)</script>`.
2. Put the CSP injection in the `token` parameter (not the search term): `;script-src-elem 'unsafe-inline'` — the token value is reflected into the CSP report-uri, so the semicolon adds a new rule allowing inline scripts.

## Result

The inline script runs despite the CSP. Lab solved.

## Why it works

The CSP header is built from the `token` parameter, which is reflected without proper handling. A semicolon ends the current directive and starts a new `script-src-elem 'unsafe-inline'` rule — so the attacker's own inline script becomes allowed. (The first attempt put the CSP injection in the search term — that was wrong; the token is what goes into the report-uri.)

## Lessons learned

- A CSP built from user input is not a defence — it is another injection point.
- The payload went in one parameter and the policy bypass in another; both had to be found.

## Mitigation

Never build a CSP header from user input. Use a fixed, strict CSP and fix the underlying XSS.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
