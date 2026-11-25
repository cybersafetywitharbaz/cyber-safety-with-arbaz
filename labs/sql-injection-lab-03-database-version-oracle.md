# SQL injection attack, querying the database type and version on Oracle

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Find out which database the application runs on, and its version, using a UNION attack on Oracle.

## Method

1. Use the category parameter with a UNION payload: `Gifts' UNION SELECT banner, NULL FROM v$version--`.

## Result

The page returns the banner: "Oracle Database 11g Express Edition Release 11.2.0.2.0". Lab solved.

## Why it works

UNION appends a second SELECT to the original query. Oracle's `v$version` view holds the version banner, and UNION brings those rows into the page output.

## Lessons learned

- Knowing the exact database version tells the attacker which syntax and tricks will work next.
- UNION attacks only work when the injected query returns the same number of columns as the original.

## Mitigation

Parameterized queries stop UNION injection. Also, return as little database detail as possible in errors and responses.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
