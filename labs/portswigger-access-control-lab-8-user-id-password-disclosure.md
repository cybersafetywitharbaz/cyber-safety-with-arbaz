# PortSwigger Lab — User ID Controlled by Request Parameter with Password Disclosure (Access Control)

**Lab:** User ID controlled by request parameter with password disclosure
**Category:** Access control
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure
**Instance:** https://0ae600cd0368119d80d5447d00530065.web-security-academy.net/
**Solved:** 26 November 2026 — user `carlos` deleted from the admin panel; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Insecure direct object reference (IDOR) — the account page takes a user ID from the request parameter and shows that user's data, including the administrator's password.

## Objective
Log in as the administrator and delete the user `carlos`.

## Solution
1. Logged in as `wiener:peter`.
2. Opened `/my-account?id=administrator` — the page showed the administrator's account, including the administrator's password in a password input's value.
3. Logged in as `administrator` with that password.
4. Opened `/admin` and submitted the delete action for `carlos`. The lab solved; the home page banner confirmed it.

## Practical Explanation
The account page trusts the `id` parameter to decide whose account to show, without checking that the logged-in user is allowed to see it. Changing `id=wiener` to `id=administrator` returns another user's private data — here, the admin's password in plain text. Once the password is known, the attacker simply logs in as the admin. The fix: the server must load the account from the session's own user ID and ignore the parameter, or verify the caller owns (or may administer) the requested ID.

## Lessons Learned
- Any `?id=` parameter is a test target: try other users' IDs and watch whether the app enforces ownership.
- Sensitive data must never be rendered into another user's page — a password in a form field value is readable by anyone who can reach the page.
- IDOR and password disclosure chain together: one flaw leaks the credential, the other account is then fully compromised.
