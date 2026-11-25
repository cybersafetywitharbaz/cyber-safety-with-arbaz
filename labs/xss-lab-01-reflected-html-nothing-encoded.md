# Reflected XSS into HTML context with nothing encoded

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Make the search page run an injected script by putting it in the search term.

## Method

1. Open the search page and enter `<script>alert(1)</script>` as the search term.

## Result

The alert runs. Lab solved.

## Why it works

The search term is reflected into the page without any encoding, so the browser treats the injected `<script>` tag as real page code and runs it.

## Lessons learned

- Reflected XSS needs the victim to open a crafted link — but the payload itself is trivial when nothing is encoded.
- Any user input reflected into HTML must be HTML-encoded.

## Mitigation

HTML-encode all user input on output. Set a Content Security Policy as defence in depth.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
