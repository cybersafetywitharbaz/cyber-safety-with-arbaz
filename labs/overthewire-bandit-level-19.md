# OverTheWire Bandit — Level 19

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / setuid binaries
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit19@bandit.labs.overthewire.org -p 2220` (password from Level 18)

## Objective

Use the setuid binary `bandit20-do` in the home directory to run a command as user bandit20 and read the password for the next level.

## Solution

1. Log in as `bandit19` (password from Level 18).
2. The home directory contains `bandit20-do` (`-rwsr-x---`, owned by bandit20). Running it with no arguments prints its usage:
   ```
   ./bandit20-do
   ```
   Output:
   ```
   Run a command as another user.
     Example: ./bandit20-do whoami
   ```
3. The `s` in the permissions (`rws`) means the binary is **setuid**: it runs with the privileges of its *owner* (bandit20), not the caller. Use it to read bandit20's password file:
   ```
   ./bandit20-do cat /etc/bandit_pass/bandit20
   ```
   Output: `4pIjcunZ0fK2vmp3IwfG8Vf7VhxD6pOA`

## Result

Password for bandit20: `4pIjcunZ0fK2vmp3IwfG8Vf7VhxD6pOA` — verified by logging in as bandit20 (`whoami` → `bandit20`).

## Practical Explanation

- **Setuid (`s` in the owner-execute position)** makes a program run as its owner, whatever user starts it. Here `bandit20-do` is owned by bandit20, so any command it runs executes as bandit20 — including `cat` on a file only bandit20 can read.
- This is a legitimate Unix feature (e.g. `passwd` is setuid root so users can change their own password), but a setuid binary that runs an *arbitrary* command is effectively a full privilege escalation to its owner.

## Lessons Learned

- `ls -la` tells you everything: `rws` = setuid, and the owner column tells you *whose* privileges the program runs with.
- A "run a command as another user" helper is only safe if it restricts which commands are allowed — this one does not.

## Mitigation

- Audit setuid binaries (`find / -perm -4000`) and remove the setuid bit where it is not needed.
- Setuid helpers must allowlist commands and never pass user input straight to a shell.
