# OverTheWire Bandit — Level 15

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / SSL/TLS with OpenSSL
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit15@bandit.labs.overthewire.org -p 2220` (password from Level 14)

## Objective

Retrieve the password for bandit16 by submitting the current level's password to an SSL/TLS service on localhost port 30001.

## Solution

1. Log in as `bandit15` (password from Level 14).
2. The level goal (overthewire.org/wargames/bandit/bandit16.html) says: submit the password of the current level to **port 30001 on localhost using SSL/TLS encryption**. Plain `nc` will not work here — the service speaks TLS, so use OpenSSL's client:
   ```
   echo pbLYuZtTg4MgaqfJx8jbA9gKKGqM68A7 | openssl s_client -connect localhost:30001 -quiet 2>/dev/null
   ```
   Output:
   ```
   Correct!
   kS0Hf0u5HiXFwKMKFqXvPdOTNGGa0X8V
   ```
   (Note the hint on the level page: if `s_client` prints `DONE`, `RENEGOTIATING` or `KEYUPDATE`, read the "CONNECTED COMMANDS" section of the manpage — those are interactive `s_client` commands, not errors.)

## Result

Password for bandit16: `kS0Hf0u5HiXFwKMKFqXvPdOTNGGa0X8V` — verified by logging in as bandit16 (`whoami` → `bandit16`).

## Practical Explanation

- **Same idea as Level 14, one layer up:** the service now requires TLS. `openssl s_client -connect host:port` performs the TLS handshake (like a browser does with HTTPS) and then passes stdin through the encrypted tunnel. `-quiet` suppresses the certificate/handshake noise; `2>/dev/null` hides the remaining diagnostics.
- **Why TLS matters:** on Level 14 the password travelled in plaintext (fine on localhost, fatal on a real network). TLS encrypts it, so anyone sniffing the traffic sees only ciphertext.
- **Netcat cannot do TLS** — that is the whole point of this level: match the tool to the protocol.

## Lessons Learned

- `openssl s_client` is the quick TLS test client — use it whenever a service speaks SSL/TLS instead of plaintext.
- A service that looks identical to the previous level (same "submit the password" idea) can differ only in transport security — read the goal page carefully.

## Mitigation

- Never accept credentials over plaintext protocols on a network; require TLS.
- Even on localhost, prefer encrypted channels so a local sniffer or a misconfigured service cannot leak credentials.
