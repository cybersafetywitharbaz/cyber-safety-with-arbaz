# SQL Injection Labs

| | |
| --- | --- |
| **Platform** | PortSwigger Web Security Academy |
| **Topic** | SQL Injection (SQLi) |
| **Status** | ✅ 15 of 18 labs solved |
| **Scope** | Legal training labs only — PortSwigger's own lab domains. No real websites were tested. |
| **Detailed log** | [portswigger-labs-progress.md](portswigger-labs-progress.md) — step-by-step record of every solved lab. |

## Overview

SQL Injection is one of the most common and most dangerous web
vulnerabilities (OWASP Top 10). It happens when an application builds a
database query from user input without properly separating code from data,
so an attacker can change the query itself. These labs were solved on
PortSwigger's intentionally vulnerable training applications, where solving
a lab means retrieving hidden data or logging in as another user inside
that isolated lab environment.

> **Note on scope:** the labs documented here are the PortSwigger Web
> Security Academy SQL Injection labs, recorded in the progress file linked
> above. The CV also lists SQL Injection labs completed on intentionally
> vulnerable targets; the PortSwigger set is the documented set. If labs on
> any other platform were completed, they are not recorded here yet — that
> detail is still to be confirmed, not assumed.

## Techniques covered (15 labs solved)

1. **Retrieving hidden data** — changing a `WHERE` clause so the query
   returns rows the application never intended to show.
2. **Login bypass** — commenting out the password check so a login succeeds
   with a known username and any password.
3. **UNION-based data retrieval** — working out the number and type of
   columns, then using `UNION SELECT` to pull data from other tables
   (e.g. usernames and passwords from a `users` table).
4. **Database fingerprinting** — identifying the database engine (Oracle,
   MySQL, PostgreSQL) and its version, because the exact syntax depends on it.
5. **Examining the database** — listing tables and columns through the
   database's own metadata views (`information_schema`, Oracle's
   `all_tables` / `all_tab_columns`) before extracting data.
6. **Blind SQL injection — conditional responses** — when the page shows no
   data, using a true/false signal (a "Welcome back" message appearing or
   not) to extract a password one character at a time.
7. **Blind SQL injection — conditional errors** — using a database error as
   the true/false signal instead, e.g. triggering a divide-by-zero only
   when a condition is true.
8. **Blind SQL injection — time delays** — using `pg_sleep` so a true
   condition makes the server answer late; the delay itself is the signal.
9. **Blind SQL injection — time delays with data retrieval** — combining
   the timing signal with character-by-character extraction to recover a
   full password, then logging in with it.

## How the blind labs were solved

Blind injection gives no data back, only a signal. The method in each case
was the same: ask the database one yes/no question at a time ("is the
first character of the password greater than 'm'?"), use the signal
(message, error, or delay) to answer it, and narrow it down character by
character with a binary search. Once the password was recovered, the login
form's CSRF token was obtained and the administrator account was used to
complete the lab — all inside the training lab.

## Remaining labs (3 of 18)

| Lab | Status |
| --- | --- |
| Visible error-based SQL injection (Lab 13) | Concept only — no practical solver run yet |
| Blind SQL injection with out-of-band interaction (Lab 16) | Needs Burp Collaborator (out-of-band channel) |
| Blind SQL injection with out-of-band data exfiltration (Lab 17) | Needs Burp Collaborator (out-of-band channel) |

## Defences — what developers should do

- **Parameterized queries / prepared statements** — never build SQL by
  concatenating user input.
- **Least privilege** — the application's database account should only be
  able to do what the application needs.
- **Input validation** — validate on the server side, not just in the browser.
- **Safe error handling** — don't leak database errors to the user; log them
  server-side instead.

## Why it matters

SQL Injection is the vulnerability class a penetration tester and a SOC
analyst both meet constantly: testers look for it, defenders detect it in
logs and WAF alerts. Understanding UNION-based and blind techniques — and
why parameterized queries stop them — is core to web application security,
vulnerability assessment, and bug bounty work (always inside an authorised
scope).
