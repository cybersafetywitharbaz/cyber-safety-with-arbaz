# PortSwigger Lab — CSRF Where Referer Validation Depends on Header Being Present

**Lab:** CSRF where Referer validation depends on header being present
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER (Academy badge on the lab page; the reference notes had listed it as Apprentice)
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-depends-on-header-being-present
**Instance:** https://0afc00ea03aa34418278987b003300b6.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site Request Forgery with Referer validation that only runs when the Referer header is present — suppress the header entirely and no check runs.

## Objective
Change the email address of the victim (carlos) by submitting the email-change form with no Referer header.

## Solution
1. Logged in as `wiener:peter` and confirmed the email change works via POST with no Referer header at all.
2. Stored this exploit HTML on the lab's exploit server — a meta tag suppresses the Referer, then the plain POST form:
   `<meta name="referrer" content="no-referrer">` plus `<form method="POST" action="https://<instance>/my-account/change-email"><input type="hidden" name="email" value="pwned@web-security-academy.net"></form><script>document.forms[0].submit();</script>`
3. Clicked "Deliver exploit to victim". The victim's browser submitted the form with no Referer header, the server skipped its check, and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
Referer-based CSRF defence is fragile by design: the header is optional, browsers and extensions can suppress it, and the server cannot distinguish "no header because the request is legitimate" from "no header because the attacker removed it". A missing header must be treated as suspicious (or the defence must not rely on Referer at all) — and a real CSRF token is the correct control.

## Lessons Learned
- Same pattern as the token labs: a check that only runs "when present" is bypassed by removing the thing it checks.
- `<meta name="referrer" content="no-referrer">` is the standard way to strip the header from a CSRF exploit.
- Referer validation is at best a second layer; it must never be the only CSRF defence.
