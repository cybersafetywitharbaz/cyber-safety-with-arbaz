# OverTheWire Bandit — Level 17

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / diff
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit17@bandit.labs.overthewire.org -p 2220` (SSH key from Level 16)

## Objective

Find the password for bandit18 by comparing the two files `passwords.old` and `passwords.new` in the home directory.

## Solution

1. Log in as `bandit17` with the SSH key from Level 16.
2. The home directory contains `passwords.old` and `passwords.new` (both 3,300 bytes, owned by bandit18). Compare them:
   ```
   diff passwords.old passwords.new
   ```
   Output:
   ```
   42c42
   < 1WJHmb0k7TH4piWp9Pd0lsruErLQZrR9
   ---
   > OQxXZjELndr90zuhOTDYBEomI0SZITXI
   ```
   Only line 42 differs. The old file has `1WJHmb0k7TH4piWp9Pd0lsruErLQZrR9`; the new file has `OQxXZjELndr90zuhOTDYBEomI0SZITXI`.
3. The password in the **new** file is the current one.

## Result

Password for bandit18: `OQxXZjELndr90zuhOTDYBEomI0SZITXI` — verified by logging in as bandit18 (`whoami` → `bandit18`).

## Practical Explanation

- **`diff` shows exactly what changed between two files.** `42c42` means "line 42 changed"; `<` marks lines from the first file, `>` marks lines from the second. When two password lists differ in one line, that line is the changed (new) password.
- This mirrors real work: comparing a config backup with the live config, or two versions of a file, to find what someone changed.

## Lessons Learned

- `diff file1 file2` is the fastest way to spot a change — never eyeball two long files side by side.
- Pay attention to direction: `<` = old file, `>` = new file. The new password is on the `>` side.

## Mitigation

- Passwords should never sit in plaintext files at all — old or new. Use a password manager or a secrets store, and rotate any password that was ever written to disk.
- Keep backups encrypted; an old copy of a password file defeats a password change.
