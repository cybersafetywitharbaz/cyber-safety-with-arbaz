# Username enumeration via subtly different responses

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Authentication
**Difficulty:** Apprentice
**Solved:** 20 November 2026

## Objective

Find a valid username from a one-character difference in the error message, then find that user's password.

## Method

1. Try the lab's candidate usernames with a probe password. All error messages look identical — except for the real user, whose message is missing the final period (.) and has a trailing space instead.
2. Try the candidate passwords for that user — one logs in and the lab shows solved.

## Result

Valid username found and login succeeded. Lab solved.

## Why it works

The messages look the same, but a period replaced by a trailing space is still a difference — and any consistent difference is a signal. The attacker reads the valid username from that tiny difference, then only has to guess the password.

## Lessons learned

- Even a single character of difference leaks the answer — responses must be byte-identical.
- Subtle differences are easy to miss by eye and easy to find with a script.

## Mitigation

Return byte-identical responses for existing and non-existing usernames, including punctuation and spacing.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
