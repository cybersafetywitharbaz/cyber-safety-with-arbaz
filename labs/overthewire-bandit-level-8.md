# OverTheWire Bandit — Level 8

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / sort and uniq
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit8@bandit.labs.overthewire.org -p 2220` (password from Level 7)

## Objective

Find the password for bandit9 in the file `data.txt` — it is the only line that occurs exactly once; every other line repeats many times.

## Solution

1. Log in as `bandit8` (password `VR1ljMayciFxbnUokuQmJFw6QC9VKtub`).
2. The home directory contains `data.txt` (33,033 bytes) full of repeated lines.
3. Sort the file, then show only lines that are unique:
   ```
   sort data.txt | uniq -u
   ```
   Output: `EjmOSvuAu7sGAHqHVcBDPirRe9T03kxl`

## Result

Password for bandit9: `EjmOSvuAu7sGAHqHVcBDPirRe9T03kxl` — verified by logging in as bandit9 (`whoami` → `bandit9`).

## Practical Explanation

- **`sort` groups identical lines together.** `uniq` only detects duplicates on adjacent lines, so sorting first is mandatory — `uniq` without `sort` would miss most duplicates.
- **`uniq -u` prints only lines that occur once.** The default `uniq` prints one copy of each line; `-u` ("unique") drops every line that appears more than once, leaving exactly the password.
- This is the same technique used in real log analysis: `sort access.log | uniq -c | sort -n` finds the rarest (or most common) entries — a single odd line in a sea of repeats is often the interesting one.

## Lessons Learned

- `uniq` needs sorted input — always pipe through `sort` first.
- `sort | uniq -u` is the standard one-liner for "find the line that appears only once".

## Mitigation

- Hiding a secret among repeated filler does not protect it; anyone who can read the file can run this one-liner.
- Store credentials in a secrets manager, never in data files on disk.
