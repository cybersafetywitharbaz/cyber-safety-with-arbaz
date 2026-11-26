# OverTheWire Bandit — Level 27

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / git (clone a remote repository)
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit27@bandit.labs.overthewire.org -p 2220` (password from Level 26)

## Objective

Clone the git repository at `ssh://bandit27-git@localhost:2220/home/bandit27-git/repo` from the local machine (not the game server) and find the password for the next level. The password for user `bandit27-git` is the same as for `bandit27`.

## Solution

1. Log in as `bandit27` (password from Level 26). The home directory is empty — the repository is the whole level.
2. Clone from the local machine (the VM) through the egress-proxy tunnel. Git's SSH needs the proxy, so use a wrapper script as `GIT_SSH_COMMAND`:
   ```sh
   # /tmp/gitssh.sh
   #!/bin/sh
   exec ssh -o StrictHostKeyChecking=no -o ConnectTimeout=20 -o ProxyCommand='/tmp/ssh_via_proxy.sh %h %p' "$@"
   ```
   and drive the password prompt with `expect` (the password is bandit27's):
   ```
   GIT_SSH_COMMAND=/tmp/gitssh.sh git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo /tmp/b27repo
   ```
   (Two gotchas: `GIT_ASKPASS` was ignored because the session has a TTY, and an inline `-o ProxyCommand=... %h %p` value gets split by the shell — the wrapper script avoids both.)
3. Read the repository's README (plain file viewing):
   ```
   cat README
   ```
   Output: `The password to the next level is: y8Yd2ssKcpHpud7UvOSOxwamRMzIGIeQ`

## Result

Password for bandit28: `y8Yd2ssKcpHpud7UvOSOxwamRMzIGIeQ` — verified by logging in as bandit28 (`whoami` → `bandit28`).

## Practical Explanation

- **Git over SSH is just SSH plus the git protocol.** The clone authenticates as `bandit27-git` with bandit27's password; the repository is then an ordinary local copy. `localhost` in the level's URL means "the game server" — from the local machine that is `bandit.labs.overthewire.org:2220`.
- **Secrets in repositories:** the password sits in a tracked file in the repo. Anyone who can clone the repo gets it — exactly why credentials must never be committed to git.
- **Tooling notes:** `GIT_SSH_COMMAND` must be a single executable (a wrapper script) when the SSH options contain spaces; `GIT_ASKPASS` only applies when git has no terminal to prompt on.

## Lessons Learned

- Clone first, look at the files: the answer is sometimes in the current checkout, not hidden in history.
- When a password prompt does not accept scripted input, drive the tool with `expect` instead of fighting it.

## Mitigation

- Never commit passwords, keys or tokens to git — use environment variables or a secrets manager, and rotate anything that was ever committed.
- Restrict repository access to the users who need it, and audit clone access.
