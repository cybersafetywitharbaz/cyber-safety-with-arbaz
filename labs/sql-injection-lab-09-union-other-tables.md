# SQL injection UNION attack, retrieving data from other tables

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Read usernames and passwords from the users table through the product-category parameter.

## Method

1. Use the category parameter with: `Gifts' UNION SELECT username, password FROM users--`.

## Result

The administrator's username and password appear on the page. Lab solved.

## Why it works

UNION appends a completely different table's rows to the original product query. Because the users table has username and password columns, those values are returned and displayed like product data.

## Lessons learned

- UNION can reach any table the database user can read — not just the one the page was built for.
- This is why the database account used by a web app should have the fewest privileges possible.

## Mitigation

Parameterized queries stop UNION injection. Give the application's database account minimal privileges, and store passwords hashed.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
