# SQL injection UNION attack, determining the number of columns returned by the query

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Find how many columns the original query returns, so a UNION attack can be built with the right shape.

## Method

1. Use the category parameter with a UNION payload of NULLs, adding one NULL at a time: `Gifts' UNION SELECT NULL,NULL,NULL--`.

## Result

With three NULLs the page loads normally — the query has 3 columns. Lab solved.

## Why it works

UNION only works when both SELECTs return the same number of columns. NULL is a safe placeholder for any column type, so adding NULLs one by one finds the count: when the count matches, the page stops erroring.

## Lessons learned

- Column-count discovery is the first step of every UNION attack.
- NULL works as a placeholder because it fits any column type.

## Mitigation

Parameterized queries stop UNION injection entirely.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
