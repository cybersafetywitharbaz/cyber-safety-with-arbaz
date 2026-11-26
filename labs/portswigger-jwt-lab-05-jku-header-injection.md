# JWT Lab 5 — JWT authentication bypass via jku header injection — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** JSON Web Tokens (JWT)
**Lab:** JWT authentication bypass via jku header injection
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:26 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

JWT authentication bypass — the server trusts a JSON Web Key Set (JWKS) fetched from a URL given in the token's own `jku` header parameter, so an attacker can host their own JWKS containing their own public key and sign a token with the matching private key.

## Solution

1. Logged in as `wiener:peter` and got the session JWT (RS256, `kid` `8bd09459-1ce5-40c3-a53f-a1a58efdb77f`).
2. Generated a fresh RSA-2048 key pair (attacker's own key, never the server's).
3. Built a JWKS document with the public key:
   ```json
   {"keys": [{"kty": "RSA", "kid": "attacker-key", "n": "<base64url modulus>", "e": "AQAB"}]}
   ```
4. Hosted the JWKS on the lab's own exploit server — this was the key step: the lab server **never** fetches an arbitrary `jku` URL (two earlier attempts with webhook.site and paste.rs showed 0 fetches in the request logs). The JWKS had to live on `https://exploit-0a08007e03cd11d381abc9e50194001a.exploit-server.net` (URL taken from the lab page's "Go to exploit server" link in a browser session), stored via its "Craft a response" form (`POST /` with `responseFile=/jwks.json`, `responseHead` `Content-Type: application/json`, `responseBody` = the JWKS, `formAction=STORE`). Verified with `GET https://exploit-0a08007e03cd11d381abc9e50194001a.exploit-server.net/jwks.json` → HTTP 200, `application/json`, containing `attacker-key`.
5. Built the forged token: header `{"alg":"RS256","kid":"attacker-key","jku":"https://exploit-0a08007e03cd11d381abc9e50194001a.exploit-server.net/jwks.json"}`, payload `sub` changed to `administrator`, signed RS256 with the attacker's private key.
6. Sent it as the `session` cookie: `GET /my-account` showed the administrator account.
7. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!" (verified again with an independent `curl` of the home page).

## Practical Explanation

- **Why it works:** instead of using a fixed trusted key, the server fetches the verification key set from a URL the token itself names (`jku`). The attacker points that URL at their own JWKS, so the server "verifies" the signature with the attacker's own key.
- **Why the exploit server mattered:** the lab's server only fetches `jku` URLs from the lab's own trusted exploit-server domain — arbitrary external hosts (webhook.site, paste.rs) are never contacted. Hosting the JWKS on the exploit server is what made the fetch happen.
- **Why the `kid` matches:** the JWKS key carries `kid` `attacker-key` and the token header `kid` points at it, so the server picks exactly the attacker's key from the set.
- **The key detail:** a verification key set must come from a server-side trusted location, never from a URL inside the token being verified — that is the same as letting the user name their own referee and write the reference themselves.

## Lessons Learned

- Never accept key material (`jku`, `x5u`, `jwk`, `x5c`) from inside an untrusted token; fetch keys only from a fixed, server-side trusted URL.
- If a `jku`/`x5u` URL must be used, allowlist the trusted host(s) — an allowlist that only permits the lab's own exploit-server domain is exactly what made the arbitrary-host attempts fail here.
- When a server silently ignores an external URL, check where it *does* fetch from (access logs / the lab's own exploit server) instead of assuming the payload was wrong.
