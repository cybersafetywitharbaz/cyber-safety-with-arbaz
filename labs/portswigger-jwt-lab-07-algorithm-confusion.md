# JWT Lab 7 — JWT authentication bypass via algorithm confusion — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** JSON Web Tokens (JWT)
**Lab:** JWT authentication bypass via algorithm confusion
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:27 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

JWT algorithm confusion (key confusion) — the server verifies RS256 tokens but will also accept HS256, and for HS256 it uses the RSA **public** key as the HMAC secret. Since the public key is published, anyone can sign HS256 tokens with it.

## Solution

1. Logged in as `wiener:peter` and got the session JWT (RS256).
2. Fetched the published public key: `GET /jwks.json` returned the JWK Set with one RSA key (`kid` `574056ae-2c79-4dca-a089-c2114ece2ad8`, `e` `AQAB`, plus the modulus `n`). Note: `/.well-known/jwks.json` returned 404 on this instance — the key lives at `/jwks.json`.
3. Converted the JWK (`n`, `e`) to a SubjectPublicKeyInfo PEM.
4. Forged a token: header `{"alg":"HS256","kid":"574056ae-2c79-4dca-a089-c2114ece2ad8"}`, payload `sub` changed to `administrator`, signed HS256 with the **PEM bytes as the HMAC secret** (signed manually with HMAC-SHA256, because PyJWT refuses to use an RSA PEM as an HMAC secret).
5. Sent it as the `session` cookie: `GET /my-account` showed the administrator account.
6. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** RS256 (asymmetric) and HS256 (symmetric) both take a "key", but the server passes the same RSA public key to the verifier for either algorithm. For HS256 that public key becomes the HMAC secret — and the public key is public, so the attacker knows the "secret".
- **Why the PEM bytes matter:** the server uses the public key in PEM form as the HMAC key, so the forged token must be signed with exactly those PEM bytes, not the raw modulus or another encoding.
- **The key detail:** the algorithm must be pinned server-side. Letting the token's `alg` header switch the verification from asymmetric to symmetric collapses the whole trust model.

## Lessons Learned

- Pin the expected algorithm on the server (accept only `RS256`) — never let the token choose its own verification algorithm.
- A public key is not a secret: it must never be usable as an HMAC secret.
