# PortSwigger Lab — 2FA Simple Bypass (Authentication, Multi-factor)

**Lab:** 2FA Simple Bypass
**Category:** Authentication vulnerabilities — Multi-factor authentication
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass
**Instance:** https://0a8300d404719771810c8e59004100f5.web-security-academy.net/
**Solved:** 25 November 2026 — verified "Congratulations, you solved the lab!" banner on the instance and Solved checkmark on the lab page.

## Vulnerability Type
Broken multi-factor authentication — the 2FA verification step is not enforced server-side. The application trusts the client to complete the second factor and lets an authenticated user reach the account page directly.

## Objective
Log in as the victim `carlos` without knowing his 2FA code.

## Solution
1. Launched a fresh instance from the lab page.
2. Logged in as `wiener:peter`. Opened the Email client, read the 2FA code (`1097`), entered it at `/login2`, and reached `/my-account?id=wiener`. This confirmed how the normal 2FA flow works.
3. Logged out.
4. Logged in as the victim `carlos:montoya`. At the `/login2` verification-code prompt, did NOT enter any code — instead navigated directly to `/my-account` (then `/my-account?id=carlos`).
5. The page loaded as Carlos: "Your username is: carlos", "Your email is: carlos@carlos-montoya.net". Lab solved.

## Practical Explanation
The login step creates a session for carlos after the password check, and the 2FA step is only a client-side redirect — the server never re-checks whether `/login2` was actually completed before serving `/my-account`. So an attacker who knows (or guesses) the victim's password can skip the second factor entirely by requesting the account URL directly. The 2FA code only matters if you follow the intended flow.

## Lessons Learned
- MFA is only real MFA if the server enforces it on every protected request — a redirect to a verification page is not enforcement.
- Always test whether a "step 2" page can be skipped by direct navigation after step 1.
- Session state after password login must be marked "partially authenticated" until the second factor completes; full account access must require the fully-authenticated flag.

## Mitigation
- Enforce 2FA server-side: mark the session as partially authenticated after the password step and require the verified second factor before serving any authenticated page or API.
- Never rely on client-side navigation flow to protect a step.
- Log and rate-limit direct access attempts to post-login pages from partially-authenticated sessions.

## Progress after this lab
- Authentication: 4/14 solved
- PortSwigger total: 63/272
