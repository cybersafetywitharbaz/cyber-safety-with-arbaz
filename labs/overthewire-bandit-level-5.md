# OverTheWire Bandit — Level 5

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / find with filters
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit5@bandit.labs.overthewire.org -p 2220` (password from Level 4)

## Objective

Find the password for bandit6 in a file inside `inhere/` that is human-readable, 1033 bytes in size, and not executable.

## Solution

1. Log in as `bandit5` (password `6C7h9GD8M6ai5nr7wo1RonrzFjj9yIrG`).
2. The `inhere/` directory contains 20 subdirectories (`maybehere00`–`maybehere19`), each with several files — too many to open one by one.
3. Filter with `find` using all three given properties at once:
   ```
   find inhere/ -type f -size 1033c ! -executable -exec file {} +
   ```
   Output: `inhere/maybehere07/.file2: ASCII text, with very long lines (1000)`
4. Read it:
   ```
   cat inhere/maybehere07/.file2
   ```
   Output: `pXa26xhMWaC2SvDotA4r9EgZkulOeSBW`

## Result

Password for bandit6: `pXa26xhMWaC2SvDotA4r9EgZkulOeSBW` — verified by logging in as bandit6 (`whoami` → `bandit6`).

## Practical Explanation

- **`find` beats manual browsing.** With 20 directories and dozens of files, opening each one is slow and error-prone. One `find` with the exact criteria (`-type f`, `-size 1033c`, `! -executable`) narrowed it to a single file.
- **`-size 1033c`** means exactly 1033 bytes (`c` = bytes). Size is a strong filter when you know it.
- **`-exec file {} +`** runs `file` on the matches to confirm the result is really human-readable text, not a binary that happens to be 1033 bytes.
- **Hidden files again:** the answer was `.file2` — a dotfile inside `maybehere07`. `find` sees dotfiles automatically, unlike plain `ls`.

## Lessons Learned

- Translate every clue in the task ("human-readable, 1033 bytes, not executable") into a `find` predicate — one command, one result.
- `find` + `-exec` is the standard way to search a filesystem by properties (owner, group, size, permissions, type) instead of by name.

## Mitigation

- Do not scatter secrets across a filesystem with predictable sizes/permissions — attackers use exactly these `find` filters.
- Restrict directory permissions so other users cannot enumerate the whole tree.
