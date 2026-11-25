# SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Retrieve data that the application normally hides (unreleased products) by changing the SQL query in the product-category parameter.

## Method

1. Open the shop and click the **Gifts** category.
2. Change the category parameter to `Gifts' OR 1=1--` and reload the page.

## Result

All 20 products are shown, including the 8 unreleased ones. Lab solved.

## Why it works

The query is `SELECT * FROM products WHERE category = 'Gifts' AND released = 1`. Adding `OR 1=1` makes the WHERE clause true for every row, so the `released = 1` filter stops hiding anything — the database returns all products.

## Lessons learned

- Never build SQL by concatenating user input — use parameterized queries (prepared statements).
- Treat every parameter (category, search, filters) as untrusted input.

## Mitigation

Use parameterized queries everywhere and never concatenate user input into SQL. Apply the same access rules in the query that the UI applies.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
