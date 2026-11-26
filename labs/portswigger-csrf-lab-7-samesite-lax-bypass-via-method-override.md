# PortSwigger Lab — SameSite Lax Bypass via Method Override

**Lab:** SameSite Lax bypass via method override
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-lax-bypass-via-method-override
**Instance:** https://0a4d00c404c4302280b203e8005f00be.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site Request Forgery bypassing SameSite=Lax — the session cookie has no explicit SameSite attribute, so the browser defaults to Lax, which still sends cookies on cross-site top-level GET navigations. The endpoint also accepts a `_method=POST` parameter to override the request method.

## Objective
Change the email address of the victim (carlos) with a cross-site top-level GET navigation that the server treats as a POST.

## Solution
1. Logged in as `wiener:peter` and confirmed the email change works via `GET /my-account/change-email?email=<addr>&_method=POST`.
2. Stored this exploit HTML on the lab's exploit server — a top-level navigation, not a form POST:
   `<script>document.location = "https://<instance>/my-account/change-email?email=pwned@web-security-academy.net&_method=POST";</script>`
3. Clicked "Deliver exploit to victim". The victim's browser navigated top-level to the URL, the Lax cookie was sent (top-level GET), the server treated it as a POST because of `_method=POST`, and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
SameSite=Lax is not a CSRF defence on its own: it still allows cookies on top-level GET navigations, and if the server lets a GET perform a state change (here via a method-override parameter), the protection evaporates. State changes must require POST with a CSRF token, and the session cookie should be `SameSite=Strict` or `Lax` combined with real token validation — never rely on the cookie policy alone.

## Lessons Learned
- Lax still sends cookies on top-level GETs — any endpoint that changes state on GET is CSRF-able under Lax.
- Method-override parameters (`_method`) turn a safe-looking GET into a state-changing POST; audit for them.
- A CSRF token on the state-changing request is the defence; SameSite is only a second layer.
