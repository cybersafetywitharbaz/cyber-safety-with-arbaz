# PortSwigger Lab — CSRF Where Token Is Tied to Non-Session Cookie

**Lab:** CSRF where token is tied to non-session cookie
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-tied-to-non-session-cookie
**Instance:** https://0a2c008e04bba1d9809b035100da0069.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site Request Forgery with a CSRF token tied to a `csrfKey` cookie instead of the session — the cookie is not a session cookie, so the attacker can set it in the victim's browser and then use the matching token.

## Objective
Change the email address of the victim (carlos) by planting the attacker's token cookie in the victim's browser and submitting the matching token.

## Solution
1. Logged in as `wiener:peter` and read wiener's `csrfKey` cookie and `csrf` token from the account page. (The token was not spent on a self-test.)
2. The lab's search function reflects input into a `Set-Cookie` response header (CRLF injection), which can set a cookie in the victim's browser. Stored this exploit HTML on the lab's exploit server:
   `<img src="https://<instance>/?search=test%0d%0aSet-Cookie:%20csrfKey=<KEY>%3b%20SameSite=None" onerror="document.forms[0].submit()">` plus a form with `email=pwned@web-security-academy.net` and `csrf=<TOKEN>`.
3. Clicked "Deliver exploit to victim". The victim's browser loaded the image URL, the CRLF-injected header set the victim's `csrfKey` cookie to wiener's key, then the form submitted with the matching token and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
Tying the token to a cookie instead of the session only helps if the attacker cannot control that cookie. Here the search endpoint reflects a CRLF into a `Set-Cookie` header, so the attacker can plant any cookie value. Two flaws combine: the CRLF injection that sets cookies, and a token design that trusts a client-controlled cookie instead of server-side session state.

## Lessons Learned
- A token tied to a cookie is only as strong as the attacker's inability to set that cookie — CRLF injection breaks it.
- Always look for a second flaw that lets you plant the value the first flaw trusts.
- The real fix is binding the token to the server-side session, and stripping CR/LF from reflected input so headers cannot be injected.
