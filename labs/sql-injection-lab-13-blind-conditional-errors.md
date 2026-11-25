# Blind SQL injection with conditional errors

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Practitioner
**Solved:** 18 November 2026

## Objective

Extract the administrator's password character by character, using a server error page as the true/false signal.

## Method

1. In the `TrackingId` cookie, inject a conditional payload that divides by zero only when the condition is true, for example using `CASE WHEN (condition) THEN 1/0 ELSE 'a' END`.
2. Use the error page as the signal: condition true → server error page; condition false → normal page.
3. Binary-search each password character one by one through that signal.
4. Take the login form's CSRF token and log in as administrator with the recovered password.

## Result

Administrator password recovered and login succeeded. Lab solved.

## Why it works

Dividing by zero (`1/0`) only errors when the injected condition is true, so the error page itself becomes the true/false signal. The rest is the same character-by-character extraction as the conditional-response lab.

## Lessons learned

- An error page is just as good a signal as a welcome message — any consistent difference leaks data.
- This lab differs from the previous one only in the signal: error page here, `Welcome back` there.

## Mitigation

Parameterized queries stop blind injection. Return generic error pages that reveal nothing about what went wrong.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
