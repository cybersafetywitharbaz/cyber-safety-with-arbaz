# OverTheWire Bandit — Level 26

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / restricted shell escape + setuid binary
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit26@bandit.labs.overthewire.org -p 2220` (SSH key from Level 25)

## Objective

With a shell as bandit26 (escaped from the `showtext` pager in Level 25), grab the password for bandit27.

## Solution

1. Log in as `bandit26` with the SSH key from Level 25. The shell is `showtext` → `more ~/text.txt`.
2. Escape to a real shell (same technique as Level 25, synchronised with `expect`): tiny pty so `more` pages, press `v` to open `vi`, wait for the `readonly` status line, then:
   ```
   :set shell=/bin/bash
   :shell
   ```
   The prompt `bandit26@bandit:~$` confirms a bash shell as bandit26 (`id` → `uid=11026(bandit26)`).
   (Without `:set shell=/bin/bash`, `:shell` would re-run the login shell `showtext` and land back in the pager.)
3. List the home directory:
   ```
   ls -la
   ```
   It contains `bandit27-do` (`-rwsr-x---`, owned by bandit27) — a setuid binary like the one in Level 19. Reading `/etc/bandit_pass/bandit27` directly fails (`Permission denied`), so use the binary:
   ```
   ./bandit27-do cat /etc/bandit_pass/bandit27
   ```
   Output: `STJLJBRRphMxKB392CT4iOr5CbzPU9ER`

## Result

Password for bandit27: `STJLJBRRphMxKB392CT4iOr5CbzPU9ER` — verified by logging in as bandit27 (`whoami` → `bandit27`).

## Practical Explanation

- **Two escapes chained:** first the pager → editor → shell escape (Level 25), then the setuid binary to run a command as the *next* user. Each step only gives the privileges of one more user — the whole game is a chain of small escalations.
- **`:set shell=/bin/bash` matters:** `vi`'s `:shell` runs `$SHELL`, and the login shell here is `showtext`. Pointing it at bash first is what makes the escape land in a usable shell.
- **Automation lesson (learned the hard way twice):** keystrokes sent before `vi` is ready are interpreted as `vi` commands — the first two attempts typed the commands into the file buffer instead of executing them. Waiting for the `readonly` indicator before sending `:` fixed it.

## Lessons Learned

- A shell obtained by escaping a pager is still limited by the user's own permissions — the setuid binary is what crosses to the next user.
- When scripting interactive tools, wait for a recognisable screen state instead of guessing sleep durations.

## Mitigation

- Same as Level 25 (no pagers as login shells) plus Level 19 (audit setuid binaries; allowlist their commands).
- Defence in depth: any single one of these controls (no pager shell, no setuid helper) would have stopped this chain.
