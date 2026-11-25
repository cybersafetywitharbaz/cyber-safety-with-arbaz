# SQL injection vulnerability allowing login bypass

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Log in as the administrator without knowing the password, by commenting out the password check in the login query.

## Method

1. On the login page, enter username `administrator'--` and any password (for example `x`).
2. Submit the form.

## Result

Logged in as administrator. Lab solved.

## Why it works

The login query is `SELECT * FROM users WHERE username = 'administrator'--' AND password = '...'`. The `--` comments out the password check, so the query only checks the username — and `administrator` exists.

## Lessons learned

- A comment sequence in a username field can delete the password check entirely.
- Login forms need the same parameterized-query protection as every other query.

## Mitigation

Use parameterized queries for authentication. Never compare passwords inside a concatenated SQL string.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
