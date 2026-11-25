# Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Run code inside a JavaScript template literal using ${} expression interpolation.

## Method

1. Open the search with `${alert(document.domain)}`.

## Result

The alert runs. Lab solved.

## Why it works

The search term lands inside a template literal (backticks). Everything else is escaped, but `${...}` inside a template literal is evaluated as an expression — so the injected expression runs.

## Lessons learned

- Template literals evaluate ${} — user input inside backticks is code, not text.
- Unicode-escaping quotes does not stop expression interpolation.

## Mitigation

Never put user data inside template literals. Use safe DOM APIs or JSON.parse.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
