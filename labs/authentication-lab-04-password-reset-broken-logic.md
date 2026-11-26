# PortSwigger Lab — Password Reset Broken Logic (Authentication, Other Mechanisms)

**Lab:** Password reset broken logic
**Category:** Authentication vulnerabilities — Other mechanisms
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic
**Solved:** 26 November 2026 — verified "Congratulations, you solved the lab!" banner on the instance and Solved checkmark on the lab page.

## Vulnerability Type
Broken password-reset logic — the reset endpoint trusts the client-supplied username and does not validate the reset token, so the reset form can be replayed for a different user.

## Objective
Log in as the victim `carlos` by resetting his password without knowing his reset token.

## Solution
1. Launched a fresh instance from the lab page.
2. Requested a password reset for `wiener` through the normal forgot-password form. This produced a valid reset page with a `temp-forgot-password-token` for wiener.
3. Instead of completing wiener's reset, replayed the reset form through the exploit server with the `temp-forgot-password-token` left EMPTY and `username=carlos`, setting the new password to `Carlos123!`.
4. The server accepted the empty token and reset carlos's password. Logged in as `carlos:Carlos123!` — My account showed "Your username is: carlos", "Your email is: carlos@carlos-montoya.net". Lab solved.

## Practical Explanation
The reset flow has two flaws that combine: the token is not validated (an empty token is accepted as if it were valid), and the username comes from the form, not from the token. The token is supposed to bind the reset to one specific account, but because the server never checks it, an attacker can take a reset page generated for their own account and point it at any other user. Requesting a reset for yourself is all the "proof" the server needs.

## Lessons Learned
- A reset token must be validated on the server and must be tied to exactly one account — the username must come from the token lookup, never from a form field.
- An empty or missing token must be rejected, not treated as valid.
- Test password-reset flows by changing the username parameter and by removing the token entirely; either succeeding is a critical flaw.

## Mitigation
- Validate the reset token server-side (single-use, time-limited, random) and look up the account from the token, ignoring any username in the request.
- Reject empty, missing, or expired tokens.
- Log password-reset attempts and rate-limit them per account.

## Progress after this lab
- Authentication: 5/14 solved
- PortSwigger total: 65/272
