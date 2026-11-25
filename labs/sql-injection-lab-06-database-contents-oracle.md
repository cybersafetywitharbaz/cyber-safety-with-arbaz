# SQL injection attack, listing the database contents on Oracle

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

List the database's tables and columns on Oracle, then read the administrator's password from the users table.

## Method

1. First explore the schema with Oracle's views: `all_tables` and `all_tab_columns`, via UNION, to find the users table and its columns.
2. Then read the credentials with the exact names found, for example: `Gifts' UNION SELECT USERNAME_TTIDXF, PASSWORD_TKNTFR FROM USERS_DCAWFW--`.
3. Log in as administrator with the retrieved password.

## Result

Administrator password retrieved and login succeeded. Lab solved.

## Why it works

Oracle keeps its schema in `all_tables` / `all_tab_columns` instead of `information_schema`. The technique is the same: discover the table, then UNION its contents into the page.

## Lessons learned

- The schema-discovery step is identical in idea on every database — only the system views change.
- Oracle folds unquoted identifiers to upper case, which is why the names are upper-case here.

## Mitigation

Parameterized queries stop UNION injection. Store passwords hashed (bcrypt/Argon2), never in plain text.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
