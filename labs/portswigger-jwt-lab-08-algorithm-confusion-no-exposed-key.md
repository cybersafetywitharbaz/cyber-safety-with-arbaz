# JWT Lab 8 — JWT authentication bypass via algorithm confusion with no exposed key — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** JSON Web Tokens (JWT)
**Lab:** JWT authentication bypass via algorithm confusion with no exposed key
**Difficulty:** Expert
**Solved:** 27 November 2026 ~04:35 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

JWT algorithm confusion (key confusion) with public-key recovery — the server verifies RS256 tokens, accepts HS256, and uses the RSA public key as the HMAC secret, but the public key is **not** published anywhere (no JWKS endpoint). The public key is recovered mathematically from two server-issued signatures (the `sig2n` / RSA parameter-recovery attack), then used as the HMAC secret.

## Solution

1. Logged in as `wiener:peter` twice (two seconds apart) and captured two RS256 tokens with different `exp` values — two different signed messages under the same key.
2. Recovered the RSA modulus `n` from the two signatures: for each token, `v = s^e − EM` (where `s` is the signature as an integer and `EM` is the PKCS#1 v1.5 padded SHA-256 hash of `header.payload`) is a multiple of `n`, so `gcd(v1, v2)` is `n` times a small factor.
   - First tried `e = 3` — the GCD was 1, so the exponent is not 3.
   - Then tried `e = 65537` (with `gmpy2` for the huge power): the GCD was 2050 bits — `n` (2048 bits) times a small factor. Factoring out small primes gave the 2048-bit candidate `n`.
3. Built the server's public key from the recovered `n` (exponent 65537) as a SubjectPublicKeyInfo PEM.
4. Forged a token: header `{"alg":"HS256","kid":"f500c182-565d-4acd-8877-77b702af55f3"}`, payload `sub` changed to `administrator`, signed HS256 with the **PEM bytes as the HMAC secret** (manual HMAC-SHA256 — PyJWT refuses an RSA PEM as an HMAC secret).
5. Sent it as the `session` cookie: `GET /my-account` showed the administrator account.
6. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

Also tried and ruled out first: both `/.well-known/jwks.json` and `/jwks.json` return 404, `/public-key.pem`, `/key.pem`, `/keys` and similar paths all 404, and the Lab 6 trick (`kid` path traversal to `/dev/null` with an empty key) does not work on this instance.

## Practical Explanation

- **Why it works:** RSA signatures leak the modulus when the exponent is small enough to compute `s^e`: each `s^e − EM` is divisible by `n`, so the GCD of two such values (from two different signed messages) exposes `n`. With `n` and the known exponent, the full public key is reconstructed — and the public key is the HMAC secret for the confusion attack.
- **Why two tokens:** one signature gives `s^e − EM = k·n` with unknown `k`; a second signature gives another multiple of `n`, and their GCD strips away the unknown multipliers, leaving (a small multiple of) `n`.
- **Why `e = 65537`:** the server's key uses the standard exponent 65537, so `s^65537` is a ~134-million-bit number — only feasible with a fast big-integer library (`gmpy2`); plain Python is far too slow.
- **The key detail:** "no exposed key" only means the key is not published — it is still recoverable from signatures the server hands out freely at login.

## Lessons Learned

- A public key is public even when it is not published: RSA signatures themselves leak enough to reconstruct it.
- Algorithm confusion is fixed the same way as always — pin the algorithm server-side and never use a public key as an HMAC secret.
- The `kid` path-traversal trick from Lab 6 does not apply here; each instance's code path differs, so verify the flaw on the instance in front of you instead of assuming.
