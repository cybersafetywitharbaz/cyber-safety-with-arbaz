# OverTheWire Bandit — Level 1

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / special filenames
**Difficulty:** Beginner
**Solved:** 25 November 2026
**Connection:** `ssh bandit1@bandit.labs.overthewire.org -p 2220` (password from Level 0)

## Objective

Find the password for bandit2 in a file called `-` (a single dash) in the home directory.

## Solution

1. Log in as `bandit1` (password `6y2kwnwK6grgvwvpvLaa2T1cpFEKOhNR`).
2. List the home directory:
   ```
   ls
   ```
   The only file is `-`.
3. `cat -` does NOT work — `-` means "read from standard input" to most commands, so `cat` waits for keyboard input instead of reading the file.
4. Read it with an explicit path instead:
   ```
   cat ./-
   ```
   Output: `PK8fYLZg2hnHSz83plBL1iEPKdD3QToB`

## Result

Password for bandit2: `PK8fYLZg2hnHSz83plBL1iEPKdD3QToB` — verified by logging in as bandit2.

## Practical Explanation

In Linux, a leading dash makes a filename look like a command-line option. `cat -` is interpreted as "read from stdin", not "read the file named dash". Giving the path `./-` removes the ambiguity: `./` says "this file, in this directory", so `cat` treats it as a filename. The same trick works for any file starting with `-` (or use `cat < -`, or `cat -- -`).

## Lessons Learned

- Filenames starting with `-` are a classic Linux gotcha — always use `./` or `--` to disambiguate.
- When a command seems to hang, check whether it is waiting on stdin.
- `ls -la` is the habit to build: it shows hidden files too, so nothing is missed.

## Mitigation

- Avoid creating files whose names start with `-` in shared directories.
- In scripts, always use `--` before filenames (`cat -- "$file"`) so unexpected filenames cannot be parsed as options.
