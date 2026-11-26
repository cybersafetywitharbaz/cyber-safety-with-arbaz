# PortSwigger Lab — CSRF Vulnerability With No Defenses

**Lab:** CSRF vulnerability with no defenses
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/csrf/lab-no-defenses
**Instance:** https://0a6500fd04d04880807d036a00c1009f.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site Request Forgery — the email-change endpoint accepts a plain POST with no CSRF token, no Referer check, and no SameSite protection, so any website the victim visits can submit the form in the victim's browser with the victim's session cookie.

## Objective
Change the email address of the victim (carlos) using a CSRF attack hosted on the lab's exploit server.

## Solution
1. Logged in as `wiener:peter` and confirmed the email-change endpoint works with a plain POST to `/my-account/change-email` (no token needed).
2. Stored this exploit HTML on the lab's exploit server:
   `<form method="POST" action="https://<instance>/my-account/change-email"><input type="hidden" name="email" value="pwned@web-security-academy.net"></form><script>document.forms[0].submit();</script>`
3. Clicked "Deliver exploit to victim". The victim bot visited the exploit page, the form auto-submitted in the victim's browser, and the victim's email changed. The home page banner confirmed the lab solved.

## Practical Explanation
A state-changing request (changing an email address) must never rely on the browser automatically attaching the session cookie as proof of intent. Without an unpredictable CSRF token tied to the session, any site can forge the request. The fix is a per-session CSRF token on every state-changing form, validated server-side.

## Lessons Learned
- If a POST works with no token at all, the endpoint is CSRF-vulnerable by definition.
- The exploit server + victim bot pattern is how every CSRF lab in this topic is solved: store the exploit, deliver it, and the victim's own browser does the work.
- Always verify with the "Congratulations, you solved the lab!" banner, not just with the request succeeding.
