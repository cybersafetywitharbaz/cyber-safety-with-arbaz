# OverTheWire Bandit — Level 4

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / file types
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit4@bandit.labs.overthewire.org -p 2220` (password from Level 3)

## Objective

Find the password for bandit5 in the only human-readable file inside the `inhere/` directory.

## Solution

1. Log in as `bandit4` (password `xzTXq1rDJQVVAzdv5cHq1TQytTWufAMq`).
2. List the directory — it contains ten files named `-file00` to `-file09`:
   ```
   ls -la inhere/
   ```
3. Check what type each file is (extensions lie; contents do not):
   ```
   file inhere/*
   ```
   Output: `-file00` to `-file06`, `-file08` are `data`; `-file02` and `-file09` are `Non-ISO extended-ASCII text`; only `-file07` is `ASCII text`.
4. Read the one human-readable file (the leading dash needs `--`):
   ```
   cat -- inhere/-file07
   ```
   Output: `6C7h9GD8M6ai5nr7wo1RonrzFjj9yIrG`

## Result

Password for bandit5: `6C7h9GD8M6ai5nr7wo1RonrzFjj9yIrG` — verified by logging in as bandit5 (`whoami` → `bandit5`).

## Practical Explanation

- **`file` reads magic bytes, not extensions.** A file named `report.txt` can be a binary and a file with no extension can be plain text. When hunting for readable content among many files, `file` is the fastest filter.
- **Filenames starting with `-` look like options.** `cat inhere/-file07` would make `cat` try to parse `-file07` as flags; `--` (or a `./` prefix) forces it to be treated as a filename — the same lesson as Level 2.
- The other files were deliberately noisy: raw `data` blobs and non-ISO text with escape sequences. Only one file was plain ASCII.

## Lessons Learned

- Use `file` to classify unknown files before opening them — never `cat` everything blindly (binary output can corrupt a terminal).
- Combine `ls` + `file` as a standard first pass when exploring an unknown directory.

## Mitigation

- Secrets should never sit in world-readable files; use strict permissions and a secrets manager.
- On shared systems, audit directories for stray readable files containing credentials.
