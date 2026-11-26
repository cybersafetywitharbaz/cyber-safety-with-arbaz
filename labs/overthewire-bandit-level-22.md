# OverTheWire Bandit — Level 22

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / cron jobs + hashing
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit22@bandit.labs.overthewire.org -p 2220` (password from Level 21)

## Objective

Find the password for bandit23: a cron job runs every minute as bandit23 and copies the password to a file in `/tmp` — but the filename is not fixed this time.

## Solution

1. Log in as `bandit22` (password from Level 21).
2. Read the cron script:
   ```
   cat /usr/bin/cronjob_bandit23.sh
   ```
   Output:
   ```bash
   #!/bin/bash
   myname=$(whoami)
   mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
   echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"
   cat /etc/bandit_pass/$myname > /tmp/$mytarget
   ```
   The target filename is the **MD5 hash of the string `I am user bandit23`** (with `echo`'s trailing newline).
3. Compute that hash — the same command the script uses:
   ```
   echo I am user bandit23 | md5sum
   ```
   Output: `8ca319486bfbbc3663ea0fbe81326349`
4. Read the file the cron job writes every minute:
   ```
   cat /tmp/8ca319486bfbbc3663ea0fbe81326349
   ```
   Output: `gKXDTAXnIz3OBxiPjRZ2uqutUlPZrBsw`

## Result

Password for bandit23: `gKXDTAXnIz3OBxiPjRZ2uqutUlPZrBsw` — verified by logging in as bandit23 (`whoami` → `bandit23`).

## Practical Explanation

- **The filename is derived, not random:** the script builds it from `whoami` + `md5sum`. Running the same two commands yourself reproduces the exact filename — no guessing, no scanning `/tmp`.
- **MD5 here is just a filename generator,** not security. `echo <text> | md5sum` prints the hash of the text (including the newline `echo` adds — the same newline the script adds, so the hash matches).
- Same flaw as Level 21: a secret is copied to a world-readable `/tmp` file; only the filename changed.

## Lessons Learned

- When a script builds a path from a variable, run the same computation yourself to find the path.
- `md5sum` of a known string is reproducible — a hash is not a hiding place when the input is predictable.

## Mitigation

- Never publish secrets to shared directories like `/tmp`; if a job must stage a file, use a private directory (`mktemp -d`, mode 700).
- Predictable filenames in `/tmp` are also vulnerable to symlink attacks — use `mktemp` for temporary files.
