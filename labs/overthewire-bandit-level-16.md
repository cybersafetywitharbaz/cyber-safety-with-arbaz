# OverTheWire Bandit — Level 16

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / port scanning (nmap) + SSL/TLS
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit16@bandit.labs.overthewire.org -p 2220` (password from Level 15)

## Objective

Find the password for bandit17: submit the current password to a port on localhost in the range 31000–32000. First find which ports listen, then which of them speak SSL/TLS — only one server gives the next credentials; the others just echo back whatever is sent.

## Solution

1. Log in as `bandit16` (password from Level 15).
2. Scan the port range with nmap:
   ```
   nmap -p 31000-32000 localhost
   ```
   Five open ports: `31046`, `31518`, `31691`, `31790`, `31960`.
3. Test each one with an SSL client, sending the current password:
   ```
   for p in 31046 31518 31691 31790 31960; do
     echo == $p ==
     echo kS0Hf0u5HiXFwKMKFqXvPdOTNGGa0X8V | timeout 6 openssl s_client -connect localhost:$p -quiet 2>/dev/null
   done
   ```
   Results: ports 31046, 31691, 31960 answered nothing over SSL; port **31518 echoed the password straight back** (a plain echo server); port **31790** answered:
   ```
   Correct!
   -----BEGIN OPENSSH PRIVATE KEY-----
   b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
   ... (38 lines)
   -----END OPENSSH PRIVATE KEY-----
   ```
4. The "credentials" for bandit17 are a **private SSH key, not a password**. Save it (strip the terminal's `\r` characters, `chmod 600`) and log in with it:
   ```
   ssh -i /tmp/bandit17_key bandit17@bandit.labs.overthewire.org -p 2220
   ```
   Output: `whoami` → `bandit17`.

## Result

Logged in as bandit17 with the private key from port 31790 — Level 16 solved. (No password for bandit17 exists to find; the key is the credential.)

## Practical Explanation

- **Port scanning first, guessing never:** `nmap -p 31000-32000 localhost` lists exactly which ports listen. Without it you would be guessing 1,000 ports.
- **Not every open port speaks the same protocol.** The echo server on 31518 returns whatever you send — it *looks* like an answer, but it only parrots your own password back. The SSL server on 31790 is the only one that validates the password and returns new credentials.
- **Test systematically:** one loop over all five ports, same input each time, and compare the outputs — the odd one out is the target.

## Lessons Learned

- `nmap -p <range> <host>` is the standard way to find listening services.
- An echo reply is not a success — check whether the response contains *new* information.
- Credentials are not always passwords: sometimes the prize is an SSH key.

## Mitigation

- Do not leave unauthenticated echo or test services listening — they leak information about what is running and can be abused for reflection attacks.
- Listening services should be firewalled to only the hosts that need them, and credential-issuing services must authenticate the requester (as the SSL service here did).
