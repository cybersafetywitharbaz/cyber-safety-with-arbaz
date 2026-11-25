# Blind SQL injection with time delays

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Practitioner
**Solved:** 20 November 2026

## Objective

Prove the injection point works by making the database pause before answering — the delay itself is the goal of this lab.

## Method

1. In the `TrackingId` cookie, inject a PostgreSQL time-delay payload using `CASE WHEN (1=1) THEN pg_sleep(10)` in the same shape as the earlier blind labs.
2. Watch the response time.

## Result

The server's answer arrived about 10 seconds late, and the lab showed solved on the home page.

## Why it works

`pg_sleep(10)` makes the database sleep for 10 seconds, but only when the injected condition is true. A late answer proves the injected SQL actually ran — the timing difference is the signal.

## Lessons learned

- Time delays prove injection even when the page shows nothing different at all.
- The well-known stacked-query payload did not work on this instance; the `CASE WHEN` form did — payloads must be adapted to the database in front of you.

## Mitigation

Parameterized queries stop blind injection. Also, keep database response times uniform where possible.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
