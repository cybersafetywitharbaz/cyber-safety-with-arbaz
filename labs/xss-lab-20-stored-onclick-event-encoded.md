# Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Store a website value that breaks out of an onclick handler when the comment is viewed.

## Method

1. Post a comment with the Website field set to `http://foo?&apos;-alert(document.domain)-&apos;`.

## Result

The alert runs when the comment is viewed. Lab solved.

## Why it works

The value lands inside an `onclick` attribute where `&apos;` is decoded back to a quote as the attribute is parsed — so the `'` closes the JS string inside the handler and `-alert(document.domain)-` runs as code.

## Lessons learned

- HTML entities are decoded during attribute parsing, so &apos; becomes a real quote exactly where it hurts.
- Encoding that is undone by the browser's own parsing is no encoding at all.

## Mitigation

Encode for the innermost context (JS inside an HTML attribute needs both). Validate URL fields against an allowlist.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
