# DOM XSS in innerHTML sink using source location.search

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Run a script through the search parameter into an innerHTML sink.

## Method

1. Open `/?search=<svg onload=alert(1)>`.

## Result

The alert runs. Lab solved.

## Why it works

The search term is inserted with innerHTML, which parses it as HTML. The injected `<svg>` element is created and its `onload` handler fires immediately.

## Lessons learned

- innerHTML parses strings as HTML — anything user-controlled in it becomes code.
- Use textContent for user data, or sanitize with a trusted library before using innerHTML.

## Mitigation

Do not assign user-controlled data to innerHTML. Use textContent or a vetted sanitizer.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
