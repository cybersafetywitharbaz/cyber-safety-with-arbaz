# PortSwigger Lab — Offline Password Cracking (Authentication, Other Mechanisms)

**Lab:** Offline password cracking
**Category:** Authentication vulnerabilities — Other mechanisms
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking
**Solved:** 26 November 2026 — verified "Congratulations, you solved the lab!" banner on the instance and Solved checkmark on the lab page.

## Vulnerability Type
Weak "stay logged in" cookie — the cookie contains the username and an unsalted MD5 hash of the password, so stealing it lets an attacker recover the password offline.

## Objective
Log in as the victim `carlos` and delete his account, by recovering his password from his stay-logged-in cookie.

## Solution
1. Launched a fresh instance from the lab page.
2. Used the blog's stored XSS to steal carlos's `stay-logged-in` cookie: a comment containing a script that exfiltrates `document.cookie` was posted, and when carlos (the victim) viewed the post, his cookie was sent to the exploit server log.
3. Decoded the cookie: it was base64 of `carlos:26323c16d5f4dabff3bb136f2460a943` — username plus an MD5 hash of the password.
4. Cracked the MD5 hash offline (hash lookup / dictionary attack): the hash decodes to the password `onceuponatime`.
5. Logged in as `carlos:onceuponatime` and deleted his account from My account. Lab solved.

## Practical Explanation
The "stay logged in" feature is meant to remember the user, but it stores the password's MD5 hash in the cookie itself. MD5 is fast and unsalted, so a common password falls to a dictionary attack in seconds — no server interaction is needed once the cookie is stolen. The stored XSS is only the delivery mechanism; the real flaw is that the cookie hands the attacker the password hash. Combined, they turn a blog comment into a full account takeover.

## Lessons Learned
- Never store a password hash (or anything derived from the password) in a client-side cookie — a remember-me token must be a random, server-side-stored value.
- MD5 is not password storage: fast, unsalted hashes are cracked offline with wordlists.
- Stored XSS anywhere on the site is critical when session cookies are in reach — even a blog comment can steal them.

## Mitigation
- Remember-me tokens must be random, single-purpose values stored server-side and revocable; never encode the username or a password hash in the cookie.
- Store passwords with a slow, salted hash (bcrypt, Argon2) and never expose hashes to the client.
- Set cookies HttpOnly and Secure, and fix stored XSS by encoding output.

## Progress after this lab
- Authentication: 6/14 solved
- PortSwigger total: 66/272
