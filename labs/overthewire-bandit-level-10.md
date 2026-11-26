# OverTheWire Bandit — Level 10

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / base64 encoding
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit10@bandit.labs.overthewire.org -p 2220` (password from Level 9)

## Objective

Find the password for bandit11 in the file `data.txt` — it is base64 encoded.

## Solution

1. Log in as `bandit10` (password `B0s2khmbT9u0geKuOoVGW3JZKhndE3BG`).
2. Read the file:
   ```
   cat data.txt
   ```
   Output: `VGhlIHBhc3N3b3JkIGlzIHBZZk9ZNkh3VXNEajVyTDlVdnloVTdNQ212OHZONVJvCg==`
   (The `==` padding at the end is the giveaway that this is base64.)
3. Decode it:
   ```
   base64 -d data.txt
   ```
   Output: `The password is pYfOY6HwUsDj5rL9UvyhU7MCmv8vN5Ro`

## Result

Password for bandit11: `pYfOY6HwUsDj5rL9UvyhU7MCmv8vN5Ro` — verified by logging in as bandit11 (`whoami` → `bandit11`).

## Practical Explanation

- **Base64 is encoding, not encryption.** It converts binary data into printable ASCII (A–Z, a–z, 0–9, `+`, `/`, with `=` padding). Anyone can decode it — `base64 -d` reverses it instantly.
- **How to recognise base64:** a long string of letters/digits/`+`/`/` ending in `=` or `==` padding. The decoded text here literally says "The password is …".
- **Encoding ≠ security.** Base64 is used to transport binary data safely in text (email attachments, JWT tokens, basic-auth headers), never to protect secrets. A password stored as base64 is effectively plaintext.

## Lessons Learned

- Recognise base64 on sight (alphabet + `=` padding) and decode with `base64 -d`.
- Always check the decoded output — here it even contained a readable sentence.

## Mitigation

- Never store passwords base64-encoded and call them "protected" — use proper hashing (bcrypt/Argon2) for stored passwords and encryption for secrets at rest.
- In code review, flag `base64` used on credentials as a finding, not a control.
