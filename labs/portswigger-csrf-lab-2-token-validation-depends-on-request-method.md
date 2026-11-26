# PortSwigger Lab — CSRF Where Token Validation Depends on Request Method

**Lab:** CSRF where token validation depends on request method
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-validation-depends-on-request-method
**Instance:** https://0a1e000304ceb9a4807b71f000e90039.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page. (First instance died with a 504; solved on a fresh instance launched the same evening.)

## Vulnerability Type
Cross-Site Request Forgery with broken token validation — the server checks the CSRF token on POST requests but not on GET requests, so the email change can be triggered with a GET.

## Objective
Change the email address of the victim (carlos) by bypassing the CSRF token check.

## Solution
1. Logged in as `wiener:peter` and confirmed the email change works via GET (no token is validated on GET requests).
2. Stored this exploit HTML on the lab's exploit server — a form with no `method` attribute, which defaults to GET:
   `<form action="https://<instance>/my-account/change-email"><input type="hidden" name="email" value="pwned@web-security-academy.net"></form><script>document.forms[0].submit();</script>`
3. Clicked "Deliver exploit to victim". The victim's browser submitted the GET request with the victim's session cookie and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
Validating a security token only for one HTTP method is the same as not validating it: the attacker just uses the unchecked method. The token check must run on every state-changing request, whatever the method — and ideally the endpoint should only accept the method it was designed for (state changes belong on POST, not GET).

## Lessons Learned
- Always test the same action with a different method; a defence that only covers POST is easy to walk around.
- A form without a `method` attribute submits as GET — that is all the bypass needs here.
- "Token validated" means validated on every path that performs the action, not just the obvious one.
