# SQL injection UNION attack, finding a column containing text

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Find which of the query's columns can hold text, so stolen data can be displayed through it.

## Method

1. Use the category parameter with a UNION payload that puts a test string in each column in turn, for example: `Gifts' UNION SELECT NULL, 's91gAp', NULL--`.

## Result

The test string `s91gAp` appears on the page — the second column is the text column. Lab solved.

## Why it works

Only a text column will display a string value. By placing a unique marker string in one column at a time, the attacker learns exactly which column can carry stolen data to the screen.

## Lessons learned

- Finding the text column is what makes the later data-theft labs possible.
- A random marker string is used so the result is unmistakable on the page.

## Mitigation

Parameterized queries stop UNION injection entirely.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
