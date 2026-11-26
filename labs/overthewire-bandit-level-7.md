# OverTheWire Bandit — Level 7

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / grep
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit7@bandit.labs.overthewire.org -p 2220` (password from Level 6)

## Objective

Find the password for bandit8 in the file `data.txt` — it is on the line next to the word "millionth".

## Solution

1. Log in as `bandit7` (password `Bmnnvf82KzQlfxgAI2d1zYbr1u9pr3E3`).
2. The home directory contains `data.txt` (4,184,396 bytes) — far too large to read by eye.
3. Search for the keyword instead of reading the file:
   ```
   grep millionth data.txt
   ```
   Output: `millionth	VR1ljMayciFxbnUokuQmJFw6QC9VKtub`

## Result

Password for bandit8: `VR1ljMayciFxbnUokuQmJFw6QC9VKtub` — verified by logging in as bandit8 (`whoami` → `bandit8`).

## Practical Explanation

- **`grep` searches content, not filenames.** When a secret is buried in a large file, `grep <keyword> <file>` prints only the matching lines. Reading a 4 MB file with `cat` would flood the screen.
- **The keyword is the anchor.** The task gave the word "millionth" — every other line is noise (number words like "first", "second", …). One keyword search replaced thousands of lines of manual reading.
- Variants worth knowing: `grep -i` (case-insensitive), `grep -n` (show line numbers), `grep -r` (search recursively through a directory).

## Lessons Learned

- Never `cat` a huge file to find one line — `grep` for a known word first.
- In log analysis this is the daily workflow: a 4 MB log is normal, and `grep` is how you find the one interesting event.

## Mitigation

- Secrets must not live in large data files on disk; rotate any credential found this way.
- Restrict file permissions (`chmod 600`) and audit who can read files containing sensitive strings.
