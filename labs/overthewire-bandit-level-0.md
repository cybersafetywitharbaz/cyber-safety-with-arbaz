# OverTheWire Bandit — Level 0

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / command line
**Difficulty:** Beginner
**Solved:** 25 November 2026
**Connection:** `ssh bandit0@bandit.labs.overthewire.org -p 2220` (password: `bandit0`)

## Objective

Log in to the game server and find the password for the next level (bandit1) hidden in a file in the home directory.

## Solution

1. Log in as `bandit0` (password `bandit0`).
2. List the home directory:
   ```
   ls
   ```
   The only file is `readme`.
3. Read it:
   ```
   cat readme
   ```
4. The file says: "The password you are looking for is: `6y2kwnwK6grgvwvpvLaa2T1cpFEKOhNR`"

## Result

Password for bandit1: `6y2kwnwK6grgvwvpvLaa2T1cpFEKOhNR` — verified by logging in as bandit1.

## Practical Explanation

Every Bandit level stores the next level's password in a file somewhere on the server. Level 0 teaches the two most basic Linux skills: listing files (`ls`) and reading a file (`cat`). Nothing is hidden — the password is in plain sight in `readme`.

## Lessons Learned

- `ls` shows what is in the current directory; `cat <file>` prints a file's contents.
- In CTFs and real investigations, the first step is always reconnaissance: look at what files exist before guessing.
- Passwords and secrets are often left in readable files — a habit that is dangerous in real systems.

## Mitigation

- Never store passwords in plaintext files.
- Restrict file permissions (`chmod 600`) so only the owner can read sensitive files.
