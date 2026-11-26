# OverTheWire Bandit — Level 12

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / file types and (de)compression
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit12@bandit.labs.overthewire.org -p 2220` (password from Level 11)

## Objective

Find the password for bandit13 in the file `data.txt` — it is a hexdump of a file that has been compressed over and over again.

## Solution

1. Log in as `bandit12` (password from Level 11).
2. `data.txt` (2,641 bytes) starts with `1f8b 0808 ...` — a hexdump, not the data itself. Reverse it:
   ```
   mkdir -p /tmp/b12 && cp data.txt /tmp/b12/ && cd /tmp/b12
   xxd -r data.txt > data
   file data
   ```
   Output: `gzip compressed data, was "data2.bin" ...`
3. Decompress layer by layer, checking the type with `file` each time (all in one SSH session — see the note below):
   ```
   for i in 1 2 3 4 5 6 7 8 9 10; do
     t=$(file -b data); echo step $i: $t
     case "$t" in
       *gzip*)  mv data data.gz; gunzip data.gz;;
       *bzip2*) mv data data.bz2; bunzip2 data.bz2;;
       *tar*)   mkdir -p x; tar xf data -C x; f=$(ls x); mv x/$f data; rmdir x;;
       *ASCII*) break;;
     esac
   done
   cat data
   ```
   The chain was: hexdump → gzip → bzip2 → gzip → tar → tar → bzip2 → tar → gzip → ASCII text.
   Final output: `The password is qQYQiHOBPR8zR61qxYqX45quvihF2uzk`

## Result

Password for bandit13: `qQYQiHOBPR8zR61qxYqX45quvihF2uzk` — verified by logging in as bandit13 (`whoami` → `bandit13`).

## Practical Explanation

- **`xxd -r` reverses a hexdump** back into the original binary bytes. `file` then tells you what that binary actually is.
- **Each layer must be peeled with the right tool:** `gunzip` for gzip, `bunzip2` for bzip2, `tar xf` for tar archives. `file -b` (brief mode) gives just the type, which is easy to test in a loop.
- **Work in one session:** the game servers are load-balanced across backends (`gibson-0` / `gibson-1`) and `/tmp` is local to each backend — a working directory created in one login can vanish on the next. The whole chain was therefore run as a single command in a single SSH session. (First attempt failed exactly this way: `/tmp/b12` was gone on the next login.)

## Lessons Learned

- Never guess a file's format from its name — run `file` on it.
- When a task is a chain of unknown layers, loop: identify → unpack → identify again.
- On shared/multi-backend systems, keep multi-step work in a single session or a directory that persists.

## Mitigation

- Nested compression does not protect a secret — every layer is reversible with standard tools.
- Do not leave sensitive files lying around in world-readable temp locations; use proper access control and encryption.
