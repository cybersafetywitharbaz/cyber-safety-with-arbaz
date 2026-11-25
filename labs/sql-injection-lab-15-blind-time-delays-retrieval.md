# Blind SQL injection with time delays and information retrieval

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Practitioner
**Solved:** 20 November 2026

## Objective

Extract the administrator's password character by character using response time as the only signal.

## Method

1. In the `TrackingId` cookie, inject a conditional time-delay payload: when the condition is true the answer is delayed by `pg_sleep` (about 3 seconds), when false it comes back immediately.
2. Use the timing difference as the true/false signal and binary-search each of the 20 password characters one by one.
3. Take the login form's CSRF token and log in as administrator with the recovered password.

## Result

Administrator password (20 characters) recovered and login succeeded. Lab solved.

## Why it works

A slow answer means 'true', a fast answer means 'false'. That one-bit timing channel is enough to test every possible character at every position until the whole password is known.

## Lessons learned

- Timing is the slowest signal, but it works where the page shows no visible difference at all.
- Binary search keeps the request count low — about 5 requests per character instead of trying every character.

## Mitigation

Parameterized queries stop blind injection. Return generic errors and keep response times uniform.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
