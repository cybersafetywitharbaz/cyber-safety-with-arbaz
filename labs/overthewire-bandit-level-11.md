# OverTheWire Bandit — Level 11

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / ROT13 encoding
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit11@bandit.labs.overthewire.org -p 2220` (password from Level 10)

## Objective

Find the password for bandit12 in the file `data.txt` — it is ROT13 encoded.

## Solution

1. Log in as `bandit11` (password from Level 10).
2. Read the file:
   ```
   cat data.txt
   ```
   Output: `Gur cnffjbeq vf TEBbmJCB8DlA0zTewHxVQ0JPLxMvDkeA`
   (Readable-ish words with shifted letters — the classic sign of ROT13.)
3. Decode it with `tr` (ROT13 = shift each letter 13 places; applying it twice returns the original):
   ```
   cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
   ```
   Output: `The password is GROozWPO8QyN0mGrjUkID0WCYkZiQxrN`

## Result

Password for bandit12: `GROozWPO8QyN0mGrjUkID0WCYkZiQxrN` — verified by logging in as bandit12 (`whoami` → `bandit12`).

## Practical Explanation

- **ROT13 is a substitution cipher, not encryption.** Each letter is replaced by the letter 13 places after it in the alphabet (`A`→`N`, `B`→`O`, …). Because the alphabet has 26 letters, applying ROT13 twice restores the original text — that is why the same `tr` command both encodes and decodes.
- **`tr 'A-Za-z' 'N-ZA-Mn-za-m'`** maps `A–M`→`N–Z`, `N–Z`→`A–M`, and the same for lowercase. Digits and punctuation pass through unchanged.
- ROT13 was historically used to hide spoilers and punchlines on Usenet — it hides text from a casual glance, but anyone who recognises it can reverse it instantly.

## Lessons Learned

- Recognise ROT13 on sight: English-looking words with every letter shifted (e.g. `Gur` = `The`, `cnffjbeq` = `password`).
- `tr` with a rotated alphabet is the one-command decoder.

## Mitigation

- ROT13 is obfuscation, not a control — never use it (or base64, or hex) to "protect" a secret. Use real encryption for secrets and proper hashing for stored passwords.
