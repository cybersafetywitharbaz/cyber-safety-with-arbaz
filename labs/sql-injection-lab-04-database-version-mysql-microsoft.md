# SQL injection attack, querying the database type and version on MySQL and Microsoft

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Find out the database type and version on a MySQL/Microsoft database using a UNION attack.

## Method

1. Use the category parameter with a UNION payload: `Gifts' UNION SELECT @@version, NULL-- ` (note the trailing space after `--`).

## Result

The page returns the version: "8.0.42-0ubuntu0.20.04.1". Lab solved.

## Why it works

`@@version` is MySQL's built-in version variable. UNION brings its value into the page, which also confirms the database is MySQL.

## Lessons learned

- The trailing space after `--` matters — SQL needs it to treat `--` as a comment.
- Version strings immediately reveal the database family and patch level.

## Mitigation

Parameterized queries stop UNION injection. Do not expose version details in application responses.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
