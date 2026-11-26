# JWT Lab 3 — JWT authentication bypass via weak signing key — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** JSON Web Tokens (JWT)
**Lab:** JWT authentication bypass via weak signing key
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:17 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

JWT authentication bypass — the server signs HS256 tokens with a weak, guessable secret, so the secret can be brute-forced offline from a single captured token and then used to forge any token.

## Solution

1. Logged in as `wiener:peter` and got the session JWT (header `{"kid":"055cb657-...","alg":"HS256"}`, payload `{"iss":"portswigger","sub":"wiener"}`).
2. Brute-forced the secret **offline** (no server requests): for each candidate word, computed `HMAC-SHA256(candidate, header.payload)` and compared it with the token's signature.
   - First tried 29 common secrets (`secret`, `password`, `123456`, `key`, `mysecret`, `jwt`, `supersecret`, …) — none matched.
   - Then used the SecLists `10k-most-common.txt` wordlist (10,001 words) — **match at line 3622: the secret is `secret1`**.
3. Forged a token with that secret: header `{"alg":"HS256","kid":"055cb657-0a75-41eb-af3b-912484e2be62"}`, payload `sub` changed to `administrator`, signed HS256 with `secret1`.
4. Sent it as the `session` cookie: `GET /my-account` showed the administrator account.
5. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** HS256 uses one shared secret for signing and verifying. If that secret is a common word, anyone who captures one valid token can recover the secret offline (each guess is one HMAC computation — no server interaction, no lockout) and then sign any token they like.
- **Why offline:** the signature is just `HMAC(secret, header.payload)`, so the correct secret is the one whose HMAC equals the signature in the captured token.
- **The key detail:** the secret was not the obvious `secret` — it was `secret1`, the 3,623rd most common password. A short wordlist is not enough; the 10k list found it in seconds.

## Lessons Learned

- HMAC signing secrets must be long, random, and high-entropy (32+ random bytes) — never a dictionary word.
- A weak secret turns a signed token into an unsigned one: anyone can mint administrator tokens.
- Offline brute-force is silent — the server never sees the guessing, so rate-limiting logins does not help.
