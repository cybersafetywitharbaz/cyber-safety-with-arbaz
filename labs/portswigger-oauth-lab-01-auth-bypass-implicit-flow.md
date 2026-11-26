# OAuth Lab 1 — Authentication bypass via OAuth implicit flow — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OAuth 2.0 authentication
**Lab:** Authentication bypass via OAuth implicit flow
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~04:36 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

OAuth authentication bypass (improper implementation of the implicit grant type) — the client application receives the user's email and access token from the OAuth service via JavaScript and then logs the user in by POSTing that data to its own `/authenticate` endpoint, without checking that the access token actually belongs to the email in the request.

## Solution

1. Went to `/my-account` — the page meta-refreshes to the OAuth provider:
   ```
   GET https://oauth-<id>.oauth-server.net/auth?client_id=<id>&redirect_uri=https://<lab>/oauth-callback&response_type=token&nonce=<n>&scope=openid%20profile%20email
   ```
2. The provider redirected to `/interaction/<uid>` (sign-in page). Logged in with the social account `wiener:peter`, then confirmed consent on the second interaction page (`POST /interaction/<uid>/confirm`).
3. The provider redirected to the lab's callback with the access token in the URL fragment:
   ```
   https://<lab>/oauth-callback#access_token=<token>
   ```
   (The fragment is never sent to the server — it is read by the callback page's JavaScript.)
4. The callback page's JavaScript does `GET https://<oauth>/me` with `Authorization: Bearer <token>` (returns `{"sub":"wiener","email":"wiener@hotdog.com",...}`) and then `POST /authenticate` with `{"email": ..., "username": ..., "token": ...}`.
5. Replayed that POST with the email swapped to the victim's, keeping wiener's own valid token:
   ```json
   POST /authenticate
   {"email": "carlos@carlos-montoya.net", "username": "carlos", "token": "<wiener's access token>"}
   ```
6. The server logged the session in as carlos (no error, no token/email binding check). Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** in the implicit flow the access token travels through the browser, so the client must send it to the server to create a session. The server here trusts the email/username in that POST without verifying that the token was issued for that user — the token is only checked for existence/validity, not for whose it is.
- **Why wiener's own token is enough:** the token is genuinely valid (issued for wiener), so every server-side check on the token passes; the only unchecked part is the email field, which the attacker controls.
- **The key detail:** the fragment (`#access_token=...`) is the giveaway that the implicit flow is in use — anything the browser can read from the URL can be replayed, and the server must bind the token to the identity it returns from `/me`, never to a client-supplied email.

## Lessons Learned

- Never trust identity data sent by the client alongside a token — always fetch the identity from the OAuth provider yourself using the token, server-side.
- The implicit flow exposes tokens in the browser; prefer the authorization code flow with PKCE, and validate that the token's `sub`/email matches the account being logged in.
- Any `POST /authenticate`-style endpoint that takes both a token and an email/username is a prime target for exactly this swap.
