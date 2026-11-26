# PortSwigger Lab — CSRF Where Token Validation Depends on Token Being Present

**Lab:** CSRF where token validation depends on token being present
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-validation-depends-on-token-being-present
**Instance:** https://0afd00a803f6d94881baca520057009e.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page. (First instance died with a 504; solved on a fresh instance launched the same evening.)

## Vulnerability Type
Cross-Site Request Forgery with broken token validation — the server only checks the CSRF token when the `csrf` parameter is present in the request. Omit the parameter entirely and no check runs.

## Objective
Change the email address of the victim (carlos) by bypassing the CSRF token check.

## Solution
1. Logged in as `wiener:peter` and confirmed the email change works via POST with no `csrf` parameter at all.
2. Stored the plain Lab 1 style exploit HTML on the lab's exploit server (email field only, no csrf field):
   `<form method="POST" action="https://<instance>/my-account/change-email"><input type="hidden" name="email" value="pwned@web-security-academy.net"></form><script>document.forms[0].submit();</script>`
3. Clicked "Deliver exploit to victim". The victim's browser submitted the form without any token, the server accepted it, and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
A validation that only runs "if the token is present" is backwards: the absence of a token must be a failure, not a free pass. The server must require the token on every state-changing request and reject the request when it is missing, empty, or wrong.

## Lessons Learned
- "Token checked" is not the same as "token required" — always test the request with the token parameter removed completely.
- The simplest bypass is often just deleting a field; try omission before trying to forge a value.
- Server-side validation must treat a missing security parameter as invalid, never as "nothing to check".
