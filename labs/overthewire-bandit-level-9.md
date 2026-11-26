# OverTheWire Bandit — Level 9

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / strings
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit9@bandit.labs.overthewire.org -p 2220` (password from Level 8)

## Objective

Find the password for bandit10 in the file `data.txt` — it is one of the few human-readable strings in the file, preceded by several `=` characters.

## Solution

1. Log in as `bandit9` (password `EjmOSvuAu7sGAHqHVcBDPirRe9T03kxl`).
2. The home directory contains `data.txt` (19,382 bytes) — mostly binary garbage, not readable text.
3. Extract the printable strings, then filter for the `====` marker:
   ```
   strings data.txt | grep "===="
   ```
   Output:
   ```
   ========== the
   ========== password
   ========== is
   ========== B0s2khmbT9u0geKuOoVGW3JZKhndE3BG
   ```

## Result

Password for bandit10: `B0s2khmbT9u0geKuOoVGW3JZKhndE3BG` — verified by logging in as bandit10 (`whoami` → `bandit10`).

## Practical Explanation

- **`strings` pulls readable text out of binary files.** A binary (or a file full of random bytes) still contains printable character sequences; `strings` prints them, one per line, and hides the rest.
- **Then `grep` filters the result.** The task said the password is preceded by several `=` characters, so `grep "===="` left only four lines — the password was the last one.
- This is a core forensics skill: when you get a suspicious binary, firmware dump or memory image, `strings` is usually the first thing you run to see what readable text it contains (URLs, paths, credentials, error messages).

## Lessons Learned

- `cat` on a binary file is useless (and can corrupt the terminal); `strings` is the right tool.
- Chain tools: `strings` extracts, `grep` filters — small tools combined solve the problem.

## Mitigation

- Hiding a password among binary noise does not protect it — `strings` finds it in seconds.
- Never embed credentials in binaries or data files; use a secrets manager and rotate exposed credentials.
