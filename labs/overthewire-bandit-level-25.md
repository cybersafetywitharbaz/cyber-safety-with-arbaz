# OverTheWire Bandit — Level 25

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / restricted shell escape (more → vi)
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit25@bandit.labs.overthewire.org -p 2220` (password from Level 24)

## Objective

Log in as bandit26. Its shell is not bash but `/usr/bin/showtext`, which only runs `more ~/text.txt` — escape from the pager to a shell and read bandit26's password.

## Solution

1. Log in as `bandit25` (password from Level 24).
2. The home directory contains `bandit26.sshkey` (the private key for bandit26) plus `.banner`, `.flag` and `.pin` (the `.pin` file confirms the pincode from Level 24: `1141`).
3. Copy the key to the local machine (strip the terminal's `\r` characters, `chmod 600`), verify it, and log in as bandit26:
   ```
   ssh -i /tmp/bandit26_key bandit26@bandit.labs.overthewire.org -p 2220
   ```
   The shell is `/usr/bin/showtext`:
   ```sh
   #!/bin/sh
   export TERM=linux
   exec more ~/text.txt
   exit 0
   ```
4. Escape the pager: shrink the terminal (6 rows) so `more` pages and shows `--More--`, then press **`v`** — `more` opens the file in `vi`. In vi, open the password file:
   ```
   :e /etc/bandit_pass/bandit26
   ```
   Vi shows `" /etc/bandit_pass/bandit26" [readonly] 1L, 33B` and the password on line 1: `jHdv2ELQhT22BkprMNDjybZDAkw1zeBJ`
   (This was done with an `expect` script: tiny pty via `stty_init "rows 6 cols 100"`, wait for `--More--`, send `v`, wait for `readonly`, then `:e ...`.)
5. Verified: a password login as bandit26 with that password is **accepted** (no "Permission denied"; the session then sits in `more`, as expected for this restricted shell).

## Result

Password for bandit26: `jHdv2ELQhT22BkprMNDjybZDAkw1zeBJ` — login as bandit26 verified (authentication accepted; shell is the restricted `showtext` pager).

## Practical Explanation

- **A pager is a shell escape hatch.** `more` (and `less`) can open an editor with `v`, and the editor can open *any* file the user can read — including `/etc/bandit_pass/bandit26`. The restricted shell only controls what runs at login; it cannot contain a program that shells out.
- **Timing matters when scripting this:** keystrokes sent before `vi` finishes starting get interpreted as `vi` commands (`:` lost → `s` enters insert mode and the command text is typed into the buffer). The working script waits for the `readonly` status line before sending the first `:` command, and sets the editor's shell explicitly (`:set shell=/bin/bash`) before `:shell`.
- **`/etc/bandit_pass/bandit26` is bandit26's own password** (same convention as Level 14) — reading it proves the escape worked.

## Lessons Learned

- `more`/`less` + `v` = editor = shell. Never give an untrusted user a pager as their login shell.
- When automating interactive programs, synchronise on screen output (`expect` patterns) instead of fixed sleeps.

## Mitigation

- Do not use pagers or editors as login shells; use a properly restricted shell (e.g. `rbash` with a fixed command set) or no shell at all (`/usr/sbin/nologin`).
- If a program must display a file, use `cat` or a viewer with no shell/editor escape, and set `LESSSECURE=1` for `less`.
