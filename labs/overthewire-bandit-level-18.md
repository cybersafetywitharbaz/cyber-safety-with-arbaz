# OverTheWire Bandit — Level 18

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / shell startup files
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit18@bandit.labs.overthewire.org -p 2220` (password from Level 17)

## Objective

Find the password for bandit19 in the file `readme` in the home directory — but someone has modified `.bashrc` so that an interactive login logs you out immediately.

## Solution

1. Log in as `bandit18` (password from Level 17).
2. The home directory contains `readme` (33 bytes, owned by bandit19) and a modified `.bashrc` (3,874 bytes, owned by bandit19) that logs interactive shells out.
3. The trick: **pass the command as an argument to SSH** instead of opening an interactive shell. A non-interactive shell does not run the modified `.bashrc` logout trap:
   ```
   ssh bandit18@bandit.labs.overthewire.org -p 2220 'cat readme'
   ```
   (In practice the tunnel helper runs every command this way already, so the login worked normally.)
   Output: `KpsOfPkcP7i1FlIExk2QEjyt6dw8dxZI`

## Result

Password for bandit19: `KpsOfPkcP7i1FlIExk2QEjyt6dw8dxZI` — verified by logging in as bandit19 (`whoami` → `bandit19`).

## Practical Explanation

- **`.bashrc` runs for interactive shells.** Whoever controls it controls what happens at login — here it was weaponised to kick the user out. But `ssh host 'command'` starts a *non-interactive* shell, which skips `.bashrc`, so the trap never fires.
- This is the same principle behind persistence tricks in real attacks (and the reason defenders audit shell startup files): anything in `.bashrc`, `.bash_profile` or `.profile` runs with the user's privileges at login.

## Lessons Learned

- If an interactive login kicks you out, try `ssh host 'command'` — a non-interactive command often still works.
- Always inspect unfamiliar dotfiles (`.bashrc`, `.profile`) in a home directory before trusting them.

## Mitigation

- Keep shell startup files owned by the user and not writable by others; audit them for unexpected commands.
- On shared systems, monitor for modified dotfiles — they are a classic persistence mechanism.
