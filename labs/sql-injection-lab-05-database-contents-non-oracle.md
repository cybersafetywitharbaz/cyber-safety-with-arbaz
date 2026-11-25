# SQL injection attack, listing the database contents on non-Oracle databases

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

List the database's tables and columns, then read the administrator's password from the users table.

## Method

1. First explore the schema with `information_schema`: `Gifts' UNION SELECT table_name, NULL FROM information_schema.tables--` (and the matching columns query) to find the users table and its columns.
2. Then read the credentials with the exact table/column names found, for example: `Gifts' UNION SELECT username_kttagt, password_qwpqpr FROM users_vvbeiz--`.
3. Log in as administrator with the retrieved password.

## Result

Administrator password retrieved and login succeeded. Lab solved.

## Why it works

`information_schema` is the database describing itself — it lists every table and column. Once the attacker knows the users table name, UNION reads the usernames and passwords straight into the page.

## Lessons learned

- Schema discovery is the step that turns a UNION hole into a full account takeover.
- Table and column names are random per lab instance, so they must be discovered each time, never hard-coded.

## Mitigation

Parameterized queries stop UNION injection. Store passwords hashed (bcrypt/Argon2), never in plain text.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
