# OverTheWire Bandit — Level 14

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / network services (netcat)
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit14@bandit.labs.overthewire.org -p 2220` (SSH key from Level 13)

## Objective

Retrieve the password for bandit15 by submitting the current level's password to a service on localhost port 30000.

## Solution

1. Log in as `bandit14` with the SSH key from Level 13.
2. The password file `/etc/bandit_pass/bandit14` holds **bandit14's own password** (each `bandit_pass` file holds that user's own password — it is NOT the next level's password):
   ```
   cat /etc/bandit_pass/bandit14
   ```
   Output: `aaWecNkG4FhxJQxz07uiwzVP6bJiYS65`
   (Verified: this password logs in as bandit14 — and it does NOT log in as bandit15.)
3. The level goal (overthewire.org/wargames/bandit/bandit15.html) says: submit the password of the current level to **port 30000 on localhost**. Send it with netcat:
   ```
   echo aaWecNkG4FhxJQxz07uiwzVP6bJiYS65 | nc localhost 30000
   ```
   Output:
   ```
   Correct!
   pbLYuZtTg4MgaqfJx8jbA9gKKGqM68A7
   ```

## Result

Password for bandit15: `pbLYuZtTg4MgaqfJx8jbA9gKKGqM68A7` — verified by logging in as bandit15 (`whoami` → `bandit15`).

## Practical Explanation

- **The trap in this level:** `/etc/bandit_pass/bandit14` looks like the answer, but it is the *current* user's password. The level is solved by *using* that password as a credential against a local service, not by logging in with it as the next user.
- **`nc localhost 30000` (netcat)** opens a raw TCP connection and sends stdin to it. The daemon reads one line, checks it against the expected password, and replies. This is the simplest form of a network client — no HTTP, no protocol, just text over TCP.
- **Always read the level goal page** (overthewire.org/wargames/bandit/) before assuming what a file means — the file name alone misled the first attempt here.

## Lessons Learned

- A password file named after the current user contains the current user's password — verify what a credential is *for* before using it.
- `echo <text> | nc <host> <port>` is the quick way to talk to a plain TCP service.
- When a login fails, re-read the task instead of guessing: the first attempt (logging in as bandit15 with bandit14's password) failed three times for exactly this reason.

## Mitigation

- Services that accept a password over a raw TCP connection should use encryption (TLS) and rate-limiting — a plaintext password check on localhost is only acceptable because the connection never leaves the machine.
- Do not store passwords in world-readable locations or reuse one credential for two purposes.
