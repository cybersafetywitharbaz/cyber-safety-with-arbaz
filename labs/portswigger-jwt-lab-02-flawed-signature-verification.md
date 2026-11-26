# JWT Lab 2 — JWT authentication bypass via flawed signature verification — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** JSON Web Tokens (JWT)
**Lab:** JWT authentication bypass via flawed signature verification
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~04:10 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

JWT authentication bypass — the server accepts tokens with the `none` algorithm, i.e. unsigned tokens, so the signature check can be removed entirely.

## Solution

1. Logged in as `wiener:peter` and got the session JWT (RS256).
2. Built an unsigned token: header `{"alg":"none"}`, payload with `sub` changed from `wiener` to `administrator`, and an **empty** signature (token ends with the trailing dot):
   ```
   eyJhbGciOiJub25lIn0.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc5NTczNzgwMCwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.
   ```
3. Sent it as the `session` cookie: `GET /my-account` showed the administrator account.
4. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the server's verification library accepts the `none` algorithm, so a token with no signature at all passes verification. The attacker only has to choose the identity in the payload.
- **Why an empty signature:** with `alg: none` there is nothing to sign — the third JWT segment is empty and the server accepts it.
- **The key detail:** `none` must never be an accepted algorithm on a server that relies on signatures; some libraries accept it unless explicitly disabled.

## Lessons Learned

- Always whitelist the exact algorithms the server accepts (e.g. only `RS256`); never let the token's own `alg` header choose the verification method.
- An unsigned token is not a token — if the signature can be empty, there is no authentication.
