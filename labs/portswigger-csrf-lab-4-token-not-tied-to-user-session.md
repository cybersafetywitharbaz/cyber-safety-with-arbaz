# PortSwigger Lab — CSRF Where Token Is Not Tied to User Session

**Lab:** CSRF where token is not tied to user session
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-not-tied-to-user-session
**Instance:** https://0af8004604110a83805e3f60008100a5.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site Request Forgery with a CSRF token that is not bound to the user's session — the server accepts any token from its pool, including one issued to a different user, so the attacker's own token works for the victim.

## Objective
Change the email address of the victim (carlos) using a CSRF token issued to another user.

## Solution
1. Logged in as `wiener:peter` and took a FRESH CSRF token from wiener's own account page. Tokens here are single-use, so the token was fetched last and never spent on a self-test.
2. Stored this exploit HTML on the lab's exploit server, with wiener's token in the form:
   `<form method="POST" action="https://<instance>/my-account/change-email"><input type="hidden" name="email" value="pwned@web-security-academy.net"><input type="hidden" name="csrf" value="<FRESH-TOKEN>"></form><script>document.forms[0].submit();</script>`
3. Clicked "Deliver exploit to victim". The victim's browser submitted the form with wiener's token; the server accepted it because the token is not tied to any session, and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
A CSRF token only proves the request came from someone who could read a token — if the token is not bound to the session that submits it, any user's token will do. The token must be generated per session (or per user) and validated against that same session server-side; a global pool of valid tokens defeats the purpose.

## Lessons Learned
- A token that is valid for everyone is valid for the attacker too — always check whether the token is bound to the session.
- Single-use tokens must be fetched at the last moment; spending one on a test burns it.
- The fix is session binding: the server must compare the submitted token with the one stored for the submitting user's session.
