# Broken brute-force protection, IP block

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Authentication
**Difficulty:** Practitioner
**Solved:** 21 November 2026

## Objective

Brute-force the victim carlos's password without ever triggering the IP block, by resetting the failed-attempt counter with a valid login.

## Method

1. Send wrong-password attempts for `carlos` in pairs.
2. After every pair, log in once with the valid account `wiener:peter` — the successful login resets the IP-based failed-attempt counter to zero.
3. Repeat until all candidate passwords are tried — `12345678` logs in as carlos.

## Result

Logged in as carlos. Lab solved.

## Why it works

The block counts failed attempts per IP address, and any successful login from that IP resets the count. Two failures plus one success, repeated, never reaches the limit — so the whole password list can be tried.

## Lessons learned

- A counter that resets on any success is not a limit at all.
- Rate limits must be per account, not only per IP, and a success on one account must not reset another account's failures.

## Mitigation

Rate-limit per account as well as per IP. Do not reset failure counters on an unrelated successful login.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
