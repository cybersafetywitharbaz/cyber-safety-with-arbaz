# OverTheWire Bandit — Level 21

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / cron jobs
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit21@bandit.labs.overthewire.org -p 2220` (password from Level 20)

## Objective

Find the password for bandit22: a program runs automatically at regular intervals from cron. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

## Solution

1. Log in as `bandit21` (password from Level 20).
2. List the cron configuration:
   ```
   ls -la /etc/cron.d/
   ```
   Relevant entry: `cronjob_bandit22`:
   ```
   * * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
   ```
   (Runs every minute as user bandit22.)
3. Read the script it runs:
   ```
   cat /usr/bin/cronjob_bandit22.sh
   ```
   Output:
   ```bash
   #!/bin/bash
   chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
   cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
   ```
   Every minute, bandit22's own password is copied to a world-readable file in `/tmp`.
4. Read that file:
   ```
   cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
   ```
   Output: `RYVux2rHEm9tiXHmLFzuR7Vhx6AZQMEz`

## Result

Password for bandit22: `RYVux2rHEm9tiXHmLFzuR7Vhx6AZQMEz` — verified by logging in as bandit22 (`whoami` → `bandit22`).

## Practical Explanation

- **Cron runs commands on a schedule as a specific user.** `/etc/cron.d/` holds system cron entries in the form `minute hour day month weekday user command`. Here bandit22 runs a script every minute — and that script writes a secret to a world-readable file.
- **The vulnerability is the output, not cron itself:** the script copies a root-only password file to `/tmp` and `chmod 644`s it, so any user on the system can read it. The schedule (every minute) just means you never have to wait long.

## Lessons Learned

- `/etc/cron.d/` and `crontab -l` are the first places to look for scheduled jobs — and for the secrets those jobs touch.
- Read the script a cron job runs, not just the schedule: the script shows *where* the result goes.

## Mitigation

- Cron jobs must never write secrets to world-readable files; use `chmod 600` and a private directory.
- Apply least privilege: a job that only needs to read a password file should not also publish it.
