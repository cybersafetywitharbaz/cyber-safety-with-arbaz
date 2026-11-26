# OverTheWire Bandit — Level 2

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / filenames with spaces and dashes
**Difficulty:** Beginner
**Solved:** 25 November 2026
**Connection:** `ssh bandit2@bandit.labs.overthewire.org -p 2220` (password from Level 1)

## Objective

Find the password for bandit3 in a file whose name contains spaces.

## Solution

1. Log in as `bandit2` (password `PK8fYLZg2hnHSz83plBL1iEPKdD3QToB`).
2. List the home directory:
   ```
   ls -la
   ```
   The file is actually named `--spaces in this filename--` (with leading and trailing double dashes), not just `spaces in this filename`.
3. Read it — the name needs quoting (spaces) and `--` (leading dashes):
   ```
   cat -- "--spaces in this filename--"
   ```
   Output: `7ZZ2LFrykP2zEyvBl4m3clcL7tGYJPME`

## Result

Password for bandit3: `7ZZ2LFrykP2zEyvBl4m3clcL7tGYJPME` — verified by logging in as bandit3 (`whoami` → `bandit3`).

## Practical Explanation

Two Linux rules combine here:

- **Spaces split arguments.** Without quotes, `cat spaces in this filename` is five separate arguments (five files that do not exist). Double quotes keep the whole name as one argument.
- **Leading dashes look like options.** The filename starts with `--`, so `cat` would parse it as flags. The `--` separator tells `cat` "everything after this is a filename, not an option".

Always run `ls -la` first and copy the exact filename — guessing the name (as I first did with `spaces in this filename`) fails.

## Lessons Learned

- Quote filenames with spaces; use `--` for filenames starting with dashes.
- Never guess a filename — list the directory and read the exact name.
- Tab-completion in a real terminal escapes these characters automatically; learning the manual form builds the underlying understanding.

## Mitigation

- Avoid spaces and leading dashes in filenames on shared systems.
- In scripts, always quote variables (`"$file"`) and use `--` before filenames so unusual names cannot break the command or be parsed as options.
