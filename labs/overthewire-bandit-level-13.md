# OverTheWire Bandit — Level 13

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / SSH private-key authentication
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit13@bandit.labs.overthewire.org -p 2220` (password from Level 12)

## Objective

Log in as bandit14 using the private SSH key `sshkey.private` in the home directory.

## Solution

1. Log in as `bandit13` (password from Level 12).
2. The home directory contains a `HINT` file and `sshkey.private` (2,602 bytes). The hint says:
   - the level's info page is `overthewire.org/wargames/bandit/bandit14.html`,
   - the current OverTheWire setup **prevents logging in from one level to another via localhost** — so `ssh bandit14@localhost` from inside the game server does not work. The key must be used from your own machine.
3. Copy the key to the local machine and fix it up (terminal output adds carriage returns, which corrupt the key — strip them):
   ```
   cat sshkey.private   # over the SSH session, save output locally
   sed -n '/BEGIN OPENSSH PRIVATE KEY/,/END OPENSSH PRIVATE KEY/p' output | tr -d '\r' > /tmp/bandit14_key
   chmod 600 /tmp/bandit14_key
   ssh-keygen -y -f /tmp/bandit14_key   # prints the public key — proves the key is valid
   ```
4. Log in with the key (SSH goes through the egress-proxy tunnel):
   ```
   ssh -i /tmp/bandit14_key bandit14@bandit.labs.overthewire.org -p 2220
   ```
   Output: `whoami` → `bandit14`.

## Result

Logged in as bandit14 with the private key — Level 13 solved. (There is no password for bandit14 to find here; the key *is* the credential.)

## Practical Explanation

- **SSH key authentication replaces passwords.** A private key on the client proves identity; the matching public key sits in the server's `~/.ssh/authorized_keys`. No password is sent at all.
- **Private keys are sensitive files:** SSH refuses a key that is readable by others (`chmod 600` is mandatory), and a leaked private key is a full account compromise — exactly what this level demonstrates.
- **Terminal output mangles files:** when a key is copied through terminal output, `\r` (carriage return) characters sneak in and `ssh-keygen` fails with `error in libcrypto`. Stripping `\r` fixes it — a common gotcha when moving files through a terminal instead of `scp`.

## Lessons Learned

- `ssh -i <keyfile>` uses a specific private key instead of the default `~/.ssh/id_*`.
- `ssh-keygen -y -f <key>` derives the public key and is a quick validity check for a private key file.
- Always `chmod 600` a private key before use.

## Mitigation

- Protect private keys like passwords: `chmod 600`, never commit them to git, never paste them into chat or tickets, and rotate them if exposed.
- Prefer key authentication with a passphrase over password-only SSH, and disable password authentication where possible.
