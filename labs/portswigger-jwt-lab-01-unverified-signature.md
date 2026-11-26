# JWT Lab 1 — JWT authentication bypass via unverified signature — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** JSON Web Tokens (JWT)
**Lab:** JWT authentication bypass via unverified signature
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~04:06 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

JWT authentication bypass — the server decodes the session JWT and trusts the `sub` claim without verifying the signature at all, so anyone can edit the payload and become any user.

## Solution

1. Logged in as `wiener:peter` and got the session JWT (header `{"kid":"c0e324da-...","alg":"RS256"}`, payload `{"iss":"portswigger","exp":1795737788,"sub":"wiener"}`).
2. Changed only the payload — `sub` from `wiener` to `administrator` — and kept the original signature byte-for-byte:
   ```
   header:  {"kid":"c0e324da-f681-4624-8bab-03deda3d7bd1","alg":"RS256"}
   payload: {"iss":"portswigger","exp":1795737788,"sub":"administrator"}
   signature: (original, unchanged)
   ```
3. Sent the forged token as the `session` cookie: `GET /my-account` showed the administrator account and the Admin panel link; `GET /admin` listed the users.
4. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the server reads the JWT payload and uses `sub` as the logged-in user, but never checks the signature. Changing the payload therefore changes identity, and the stale signature is simply ignored.
- **Why keep the signature:** nothing in the token needed to change except `sub`; keeping the original signature proves the server never validates it.
- **The key detail:** a JWT is only as trustworthy as its signature check — a token the server doesn't verify is just a client-supplied claim.

## Lessons Learned

- Never trust a JWT (or any token) without verifying its signature with the correct key and algorithm.
- The `sub` claim is an identity assertion, not proof — proof comes from the signature.
