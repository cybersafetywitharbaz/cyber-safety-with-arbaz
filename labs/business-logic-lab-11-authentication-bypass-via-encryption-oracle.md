# Business Logic Lab 11 — Authentication bypass via encryption oracle

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Practitioner
**Solved:** 25 November 2026

## Objective

Log in as the administrator without knowing the password, by abusing the blog comment feature's encryption and decryption oracles to forge a `stay-logged-in` cookie — then delete user `carlos` from the admin panel.

## Method

1. **Log in** with the lab's test account `wiener:peter` and tick **"Stay logged in"** — the server sets an encrypted `stay-logged-in` cookie.
2. **Decrypt oracle:** send that cookie as the `notification` cookie in `GET /post?postId=1`. The page's notification header reflects the decrypted plaintext: `wiener:1795614859890` — so the cookie format is `username:timestamp` (timestamp in milliseconds).
3. **Encrypt oracle:** post a comment (`POST /post/comment`) with an **invalid** email address. The server encrypts `"Invalid email address: " + email` and returns it as the `notification` cookie — so any text can be encrypted on demand. The email used was `xxxxxxxxxadministrator:1795614859890`: the 9 `x` characters pad the 23-character `"Invalid email address: "` prefix to exactly 32 bytes (two 16-byte cipher blocks).
4. **Cut the ciphertext:** base64-decode the oracle's cookie (64 bytes = 4 blocks), **delete the first 32 bytes** (the two prefix blocks), re-encode. The remaining ciphertext decrypts to exactly `administrator:1795614859890`.
5. **Verify the forgery first** with the decrypt oracle — it returned `administrator:1795614859890`, no prefix, exactly as intended.
6. **Log in as administrator:** fresh session, **no `session` cookie**, only the forged `stay-logged-in` cookie — the home page showed the Admin panel link. Opened `/admin` and called `/admin/delete?username=carlos` — the lab showed "Congratulations, you solved the lab!".

The solve was done with a free Python `requests` script (no Burp Pro). Solver script: `ps-lab11-encryption-oracle.py`.

## Result

✅ **Solved.** Verified in the saved dump `lab11-admin-delete-result.html`, which contains "Congratulations, you solved the lab!". The forged cookie's decryption was also verified separately before it was used.

## Why it works

The cookie encryption is **ECB mode**: every 16-byte block is encrypted independently, so blocks can be cut, pasted and reordered and each block still decrypts to the same plaintext. That is exactly what the attack does — it encrypts a longer text through the oracle, then throws away the first two blocks (the known error-message prefix) and keeps the blocks that decrypt to the chosen admin identity.

Two more design flaws make it possible:

- The app **hands out both oracles on the same key**: the comment form encrypts attacker-chosen text (encrypt oracle) and the blog page decrypts any `notification` cookie and reflects the plaintext in an error message (decrypt oracle).
- The `stay-logged-in` cookie is **trusted on its decrypted content alone** — there is no signature (MAC), no server-side session lookup, and no check that the cookie was ever issued to that user. Decrypting to `administrator:<timestamp>` is enough to become the administrator.

## What went wrong first — and the fix (worth remembering)

The first decrypt-oracle probe **looked like a failure**: the check searched the page for the literal text `"Invalid email address: ..."`, but decrypting the `stay-logged-in` cookie returns just `wiener:<timestamp>` — the notification header shows only the decrypted value, with no prefix. The oracle was working the whole time; the test was wrong. The fix: test the oracle with its *own* output first (the encrypt oracle's cookie decrypts to `"Invalid email address: bad-email"`), which proved both oracles worked before any forgery was attempted. Always verify the tool against a known answer before trusting a "not working" result.

## Lessons from this lab (worth remembering)

- **ECB is not safe for cookies or tokens.** Identical plaintext blocks give identical ciphertext, and blocks can be cut and pasted between messages. Never use ECB for anything that carries identity or permissions.
- **Never expose both an encrypt oracle and a decrypt oracle on the same key.** Either one alone is bad; together they let an attacker encrypt any text and read back any ciphertext.
- **An error message that reflects decrypted data is a decrypt oracle** — even if it looks like a harmless "invalid email" message.
- **A cookie's decrypted content is not authentication.** Without a MAC/signature and a server-side check, anyone who can produce valid ciphertext can become any user.
- **Verify the forgery before using it.** One decrypt-oracle call confirmed the forged cookie read `administrator:<timestamp>` — cheaper than debugging a failed admin login.

## Mitigation

- Use **authenticated encryption** (for example AES-GCM) for cookies, or add an **HMAC over the ciphertext** and verify it before decrypting — a forged or tampered cookie must be rejected, not decrypted.
- Never reflect decrypted cookie values in error messages; log decryption failures server-side instead.
- Don't encrypt user-controlled input (like a comment email) with the same key used for session tokens — use separate keys per purpose.
- Validate stay-logged-in tokens **server-side** (look the token up in a session store) instead of trusting whatever the cookie decrypts to.
- Give stay-logged-in tokens a real expiry and bind them to the account (and ideally the device); reject old or reused timestamps.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
