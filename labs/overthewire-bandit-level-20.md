# OverTheWire Bandit — Level 20

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / setuid binary + netcat listener
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit20@bandit.labs.overthewire.org -p 2220` (password from Level 19)

## Objective

Use the setuid binary `suconnect` to get the password for bandit21: it connects to a port you choose on localhost, and if it receives the correct password for the current level, it sends back the next password.

## Solution

1. Log in as `bandit20` (password from Level 19).
2. The home directory contains `suconnect` (`-rwsr-x---`, owned by bandit21). With no arguments it prints:
   ```
   Usage: ./suconnect <portnumber>
   This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
   ```
3. Start a netcat listener in the background that serves the current password, then run `suconnect` against it — all in one SSH session:
   ```
   (echo 4pIjcunZ0fK2vmp3IwfG8Vf7VhxD6pOA | nc -l -p 12345 &)
   sleep 1
   ./suconnect 12345
   ```
   Output:
   ```
   Read: 4pIjcunZ0fK2vmp3IwfG8Vf7VhxD6pOA
   Password matches, sending next password
   bW9kBv5WC3P4yoDyf12LSdGuNz5ka6hY
   ```

## Result

Password for bandit21: `bW9kBv5WC3P4yoDyf12LSdGuNz5ka6hY` — verified by logging in as bandit21 (`whoami` → `bandit21`).

## Practical Explanation

- **`suconnect` is a setuid client:** it runs as bandit21, connects to localhost on the port you give, and compares whatever it receives with bandit20's password. It only reveals the next password to someone who already knows the current one — a simple challenge/response check.
- **The listener side is netcat:** `echo <password> | nc -l -p 12345` makes netcat listen on port 12345 and send the password to whoever connects. The `&` backgrounds it; `sleep 1` makes sure it is listening before `suconnect` connects.
- The two halves meet on localhost: your listener speaks, the setuid binary listens and verifies.

## Lessons Learned

- A network service that "checks a password and returns a secret" can be satisfied by running your own server on the port it connects to.
- Background a listener with `&` and give it a moment (`sleep 1`) before the client connects, or the connection is refused.

## Mitigation

- Challenge/response checks must run over authenticated, encrypted channels; a localhost-only check is only as safe as local access control.
- Setuid binaries that connect to arbitrary local ports can be tricked into talking to an attacker-controlled server — validate the peer, not just the password.
