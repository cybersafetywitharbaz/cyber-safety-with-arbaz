# OverTheWire Bandit — Level 6

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / find by owner, group and size
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit6@bandit.labs.overthewire.org -p 2220` (password from Level 5)

## Objective

Find the password for bandit7 in a file somewhere on the server that is owned by user bandit7, owned by group bandit6, and 33 bytes in size.

## Solution

1. Log in as `bandit6` (password `pXa26xhMWaC2SvDotA4r9EgZkulOeSBW`).
2. Search the whole filesystem with all three properties as `find` predicates, silencing permission errors:
   ```
   find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
   ```
   Output: `/var/lib/dpkg/info/bandit7.password`
3. Read it:
   ```
   cat /var/lib/dpkg/info/bandit7.password
   ```
   Output: `Bmnnvf82KzQlfxgAI2d1zYbr1u9pr3E3`

## Result

Password for bandit7: `Bmnnvf82KzQlfxgAI2d1zYbr1u9pr3E3` — verified by logging in as bandit7 (`whoami` → `bandit7`).

## Practical Explanation

- **Ownership is a searchable property.** `find` can filter by `-user` and `-group`, not just by name or size. On a real system this is how you hunt for files a particular user left behind, or for files with wrong ownership after an incident.
- **`2>/dev/null` hides the noise.** Searching `/` hits directories you cannot read; without redirecting stderr, hundreds of "Permission denied" lines bury the real result.
- **The file was outside the home directory** (`/var/lib/dpkg/info/` — a system package directory). The lesson: secrets and misplaced files can live anywhere on a filesystem; searching only the home directory misses them.
- The password length (32 chars + newline = 33 bytes) matched `-size 33c` exactly.

## Lessons Learned

- Combine several `find` predicates to get a unique match from an entire filesystem.
- Always silence permission errors with `2>/dev/null` when searching `/`.
- Check the result with `ls -l` before trusting it — confirm owner, group and size match the clue.

## Mitigation

- Never store passwords in files on a shared server, especially not in world-readable system directories.
- Audit file ownership regularly; a file owned by another user in a system directory is a red flag.
