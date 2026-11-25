# Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Break out of a JavaScript string by doubling the backslash the application adds.

## Method

1. Open the search with `\'-alert(document.domain)//`.

## Result

The alert runs. Lab solved.

## Why it works

Backslashes are not escaped by the application, so the payload's backslash plus the application's own backslash become `\\` — an escaped backslash — and the following quote then closes the string, letting `-alert(document.domain)` run as code.

## Lessons learned

- When the application adds a backslash, the attacker can supply one more and change what is escaped.
- Escaping must be done by a correct encoder for the exact context, not by ad-hoc string fixes.

## Mitigation

JavaScript-encode data placed inside script strings. Better: avoid embedding user data in scripts at all.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
