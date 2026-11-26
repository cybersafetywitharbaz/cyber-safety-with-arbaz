# PortSwigger Lab — CSRF Where Token Is Duplicated in Cookie

**Lab:** CSRF where token is duplicated in cookie
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-duplicated-in-cookie
**Instance:** https://0a1900ad045067b980025876003a003e.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site Request Forgery with "double submit" validation — the server only checks that the `csrf` body parameter equals the `csrf` cookie. The attacker can invent any value, set it as the victim's cookie, and submit the same value.

## Objective
Change the email address of the victim (carlos) by planting a chosen token as the victim's cookie and submitting the same token in the form.

## Solution
1. Confirmed the double-submit logic as `wiener:peter`: a POST with `csrf=fake` and a `csrf=fake` cookie changes the email.
2. The lab's search function reflects input into a `Set-Cookie` response header (CRLF injection), which can set a cookie in the victim's browser. Stored this exploit HTML on the lab's exploit server:
   `<img src="https://<instance>/?search=test%0d%0aSet-Cookie:%20csrf=fake%3b%20SameSite=None%3b%20Secure" onerror="document.forms[0].submit();"/>` plus a form with `email=pwned@web-security-academy.net` and `csrf=fake`.
3. Clicked "Deliver exploit to victim". The victim's browser loaded the image URL, the CRLF-injected header set the victim's `csrf` cookie to `fake`, then the form submitted `csrf=fake` — parameter matched cookie and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
Double-submit validation assumes the attacker cannot write the victim's cookie — but a CRLF injection in the search function lets the attacker do exactly that. One detail mattered: the injected cookie needs `SameSite=None; Secure` — modern browsers reject a `SameSite=None` cookie without the `Secure` flag, so the first attempt (without `Secure`) silently failed and the victim's cookie was never set.

## Lessons Learned
- Double-submit cookies only work if the cookie cannot be set by the attacker; any header-injection flaw breaks the whole scheme.
- When a cookie-setting step silently fails, check the cookie flags — `SameSite=None` without `Secure` is rejected by modern browsers.
- The fix is server-side session binding for the token, plus stripping CR/LF from reflected input.
