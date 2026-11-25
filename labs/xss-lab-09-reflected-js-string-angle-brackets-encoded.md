# Reflected XSS into a JavaScript string with angle brackets HTML encoded

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Break out of a JavaScript string in the page and run code, using a single quote.

## Method

1. Open `/?search='-alert(1)-'`.

## Result

The alert runs. Lab solved.

## Why it works

The search term lands inside a JavaScript string, and angle brackets are encoded — but the single quote is not. The `'` closes the string, `-alert(1)-` becomes code, and the trailing `'` is absorbed.

## Lessons learned

- Breaking out of a JS string needs only a quote — angle brackets are irrelevant inside script code.
- Data in script contexts needs JavaScript encoding, not HTML encoding.

## Mitigation

JavaScript-encode data placed inside script strings. Better: avoid embedding user data in scripts at all.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
