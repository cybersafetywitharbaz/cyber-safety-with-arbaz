# OverTheWire Bandit — Level 3

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / hidden files
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit3@bandit.labs.overthewire.org -p 2220` (password from Level 2)

## Objective

Find the password for bandit4 in a hidden file inside the `inhere/` directory.

## Solution

1. Log in as `bandit3` (password `7ZZ2LFrykP2zEyvBl4m3clcL7tGYJPME`).
2. List the `inhere/` directory including hidden files:
   ```
   ls -la inhere/
   ```
   Output shows one file: `...Hiding-From-You` (33 bytes).
3. Read it:
   ```
   cat inhere/...Hiding-From-You
   ```
   Output: `xzTXq1rDJQVVAzdv5cHq1TQytTWufAMq`

## Result

Password for bandit4: `xzTXq1rDJQVVAzdv5cHq1TQytTWufAMq` — verified by logging in as bandit4 (`whoami` → `bandit4`).

## Practical Explanation

- **Hidden files start with a dot.** A plain `ls` does not show them; `ls -la` (the `-a` flag = "all") reveals them. In real systems, configuration files (`.bashrc`, `.ssh/`) and malware both hide this way — always check with `-a`.
- **The filename itself is a hint.** `...Hiding-From-You` (three leading dots) is easy to miss even in a listing; copying the exact name matters. Tab-completion helps, but in a non-interactive command the exact name must be typed.
- Note: the first connection attempt failed with "Connection timed out during banner exchange" (the egress-proxy tunnel is occasionally flaky); refreshing the proxy token and retrying once fixed it.

## Lessons Learned

- Always use `ls -la` first — plain `ls` hides dotfiles.
- Hidden does not mean protected: dotfiles are only hidden from casual listing, not from anyone who looks.
- When a tunnel/connection fails once with a banner-exchange timeout, retry once after refreshing the token before assuming the target is down.

## Mitigation

- Do not rely on "hidden" filenames for security — permissions (`chmod 600`, ownership) are the real control.
- Audit dotfiles in home directories regularly; attackers hide persistence mechanisms there.
