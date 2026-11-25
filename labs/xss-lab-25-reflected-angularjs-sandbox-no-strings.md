# Reflected XSS with AngularJS sandbox escape without strings

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Practitioner
**Solved:** 15 November 2026

## Objective

Escape the AngularJS sandbox without using string literals, by overwriting a built-in method.

## Method

1. Open the search with the payload built from `toString().constructor.prototype.charAt=[].join` plus `orderBy`/`fromCharCode` to assemble `x=alert(1)`.

## Result

The alert runs. Lab solved.

## Why it works

Overwriting `charAt` changes how AngularJS's sandbox check reads characters, so the sandbox test is bypassed; `fromCharCode` then builds the code without any string literal appearing in the payload.

## Lessons learned

- Sandboxes that inspect the payload text can be beaten by changing the very functions they rely on.
- Client-side sandboxes are not security boundaries.

## Mitigation

Do not rely on the AngularJS sandbox. Upgrade AngularJS or use a framework with proper escaping, plus CSP.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
