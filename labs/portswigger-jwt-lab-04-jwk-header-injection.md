# JWT Lab 4 — JWT authentication bypass via jwk header injection — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** JSON Web Tokens (JWT)
**Lab:** JWT authentication bypass via jwk header injection
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:20 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

JWT authentication bypass — the server trusts a public key embedded in the token's own `jwk` header parameter, so an attacker can sign a token with their own private key and supply the matching public key in the header.

## Solution

1. Logged in as `wiener:peter` and got the session JWT (RS256).
2. Generated a fresh RSA-2048 key pair (attacker's own key, never the server's).
3. Built the forged token: header `{"alg":"RS256","kid":"attacker-key","jwk":{"kty":"RSA","kid":"attacker-key","n":"<base64url modulus>","e":"AQAB"}}`, payload `sub` changed to `administrator`, signed RS256 with the attacker's private key.
4. Sent it as the `session` cookie: `GET /my-account` showed the administrator account.
5. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** instead of using a fixed trusted key, the server takes the verification key from the token itself (`jwk` header). The attacker supplies both the key and a signature made with it, so verification trivially passes.
- **Why the `kid` matches:** the embedded JWK carries its own key ID, and the header `kid` points at it — everything the server needs to "verify" comes from the attacker.
- **The key detail:** a verification key must come from the server's own trusted store, never from inside the token being verified — that is the same as letting the user write their own ID card and stamp it themselves.

## Lessons Learned

- Never accept key material (`jwk`, `jku`, `x5u`, `x5c`) from inside an untrusted token.
- Pin the verification key server-side and verify the signature against that key only.
