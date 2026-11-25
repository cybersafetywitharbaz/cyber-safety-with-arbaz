# Stored XSS into anchor href attribute with double quotes HTML-encoded

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Store a javascript: URL as a comment author's website link so it runs when clicked.

## Method

1. Post a comment with the website field set to `javascript:alert(1)`.

## Result

The author's name becomes a link; clicking it runs the alert. Lab solved.

## Why it works

The comment author's name is linked with the stored website value. Double quotes are encoded, but the href value itself is not validated — a `javascript:` URL in an href is code, and it runs on click.

## Lessons learned

- Stored XSS can hide in a profile field as easily as in a comment body.
- URL fields need scheme validation, not just quote encoding.

## Mitigation

Validate URLs against an allowlist of schemes (http/https) before storing or rendering them.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
