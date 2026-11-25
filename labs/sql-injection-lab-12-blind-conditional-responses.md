# Blind SQL injection with conditional responses

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Practitioner
**Solved:** 18 November 2026

## Objective

Extract the administrator's password character by character from a blind injection point, using a true/false signal on the page.

## Method

1. In the `TrackingId` cookie, inject an OR-form conditional payload (the AND form does not work here because the TrackingId value `x` does not exist in the database).
2. Use the page's `Welcome back` message as the true/false signal: it appears when the injected condition is true, and not when it is false.
3. Binary-search each of the 20 password characters one by one through that signal.
4. Take the login form's CSRF token and log in as administrator with the recovered password.

## Result

Administrator password (20 characters) recovered and login succeeded. Lab solved.

## Why it works

Even when the query result is never shown, a single true/false difference on the page is a one-bit channel. Repeating it with different conditions (binary search) leaks the whole password, one character at a time.

## Lessons learned

- Blind injection is slower, but the same data leaks — the signal just changes from visible data to a yes/no difference.
- The browser could not edit the cookie, so this was solved with a script — the same steps done by hand in Burp Repeater.

## Mitigation

Parameterized queries stop blind injection. Also, make error and welcome messages identical whether a condition is true or false.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
