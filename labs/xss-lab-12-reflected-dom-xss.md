# Reflected DOM XSS

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Break out of a JSON string that is passed to eval() and run code.

## Method

1. Open `/?search=\"-alert(1)}//`.

## Result

The alert runs. Lab solved.

## Why it works

The search term is placed inside a JSON string that the page then runs through `eval()`. The backslash-quote escapes the JSON string, and `-alert(1)` becomes code inside the evaluated expression.

## Lessons learned

- eval() on anything containing user input is effectively remote code execution.
- JSON data must be parsed with JSON.parse, never eval.

## Mitigation

Never use eval() on user-influenced data. Parse JSON with JSON.parse and encode data for its context.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
