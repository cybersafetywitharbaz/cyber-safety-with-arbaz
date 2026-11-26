# HTB Academy — Introduction to Bash Scripting: Flow Control – Loops (salt question)

**Platform:** Hack The Box Academy (free Tier 0 module)
**Module:** Introduction to Bash Scripting
**Section:** Flow Control – Loops
**Difficulty:** Easy (Tier 0)
**Solved:** 27 November 2026 (salt verified by successful decryption — see below)
**Account:** Arbaz Ahmed Naveed (@cybersafetywitharbaz)

## Task

Create a `for` loop that encodes the variable `var` ("9M") 28 times in
base64. The number of characters in the 28th hash is the value that must
be assigned to the `salt` variable. The salt decrypts the exercise flag
with the module's `decrypt` function (a sed-chain over a static hash,
then `openssl enc -aes-128-cbc -a -d -salt -pass pass:$salt`).

## Solution

Run the exercise script exactly as written in the module:

```bash
var="9M"
salt=""
hash="VTJGc2RHVmtYMTl2ZnYyNTdUeERVRnBtQWVGNmFWWVUySG1wTXNmRi9rQT0K"
for i in {1..28}
do
    var=$(echo $var | base64)
    if [[ $i == 28 ]]
    then
        salt=$(echo $var | wc -c)
    fi
done
decrypt   # sed-chain + openssl, as in the module script
echo $flag
```

**Answer: salt = 34071.**

Verification (run 27 Nov 2026, two independent runs + negative control):
- 28th hash length `${#var}` = **34070**; `echo $var | wc -c` = **34071**.
- Decrypt with salt **34071** → flag `HTBL00p5r0x` (exit 0). Only the
  deprecation warning about key derivation appears, no error.
- Negative control: salt 34070 → `bad decrypt` (exit 1); salt 34072 →
  `bad decrypt` (exit 1). The correct value is uniquely 34071.

## Practical Explanation

- `echo $var` prints the value **plus a trailing newline**, so
  `echo $var | wc -c` counts one more character than `${#var}`. That
  single `+1` is the whole trap: `${#var}` = 34070 is *wrong* as the
  salt, `wc -c` = 34071 is *right* — because the module's own script
  computes the salt with `wc -c`, and the flag was encrypted with that
  exact value.
- The earlier 26+ rejected values all came from computing the length
  with a different method than the script uses (`${#var}`, `echo -n`,
  `base64 -w0`, hand-computed length chains). The fix was never a new
  number — it was running the script itself, exactly as written, and
  letting the decrypt prove the value.
- GNU `base64` wraps output at 76 columns, so from about the 10th
  iteration the hash contains embedded newlines; unquoted `echo $var`
  word-splits them back to spaces on the next pass. That is why the
  length chain cannot be worked out by hand from the 4/3 growth rule
  alone — the script must be executed.

## Lessons Learned

- When a task gives you a script, run the script — don't re-implement
  its arithmetic from first principles. `wc -c` vs `${#var}` differs by
  exactly the newline `echo` adds.
- A decrypt that only succeeds for one value is a perfect oracle: test
  the neighbours (34070 / 34072) to prove the answer is unique instead
  of submitting blind.
- `echo` adds `\n`; `echo -n` doesn't; `$(...)` strips trailing
  newlines but keeps embedded ones; unquoted variables word-split.
  These four facts explain most "my computed value is off by one"
  shell bugs.

## Status

Salt solved and verified 27 Nov 2026. **Submitting the answer on HTB
Academy still needs a signed-in browser session** (parent's browser
task) — nothing was submitted from here. Once submitted, Introduction
to Bash Scripting moves to 100% (10/413 modules).
