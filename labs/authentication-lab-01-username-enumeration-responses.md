# Username enumeration via different responses

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Authentication
**Difficulty:** Apprentice
**Solved:** 20 November 2026

## Objective

Find a valid username by comparing the login error messages, then find that user's password from the candidate list.

## Method

1. Try the lab's candidate usernames with a wrong password. Every one returns "Invalid username" — except `asterix`, which returns "Incorrect password".
2. Try the candidate passwords for `asterix` — `ranger` logs in and opens My Account.

## Result

Logged in as asterix. Lab solved.

## Why it works

"Invalid username" means the account does not exist; "Incorrect password" means it does — so the different messages hand the attacker a valid username, and then only the password is left to guess.

## Lessons learned

- Different error messages for bad usernames vs bad passwords are an information leak.
- Enumeration halves the attacker's work: username first, then password only.

## Mitigation

Return the exact same message for a bad username and a bad password: "Invalid username or password".

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
