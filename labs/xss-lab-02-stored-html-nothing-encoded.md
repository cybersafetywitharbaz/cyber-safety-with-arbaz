# Stored XSS into HTML context with nothing encoded

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Store a script in a blog comment so it runs for everyone who views the post.

## Method

1. Post a comment containing `<script>alert(1)</script>`.

## Result

The alert runs as soon as the comment is submitted and displayed. Lab solved.

## Why it works

The comment is saved and then rendered as HTML without encoding, so the stored `<script>` tag executes in every visitor's browser.

## Lessons learned

- Stored XSS is worse than reflected — the attacker only plants it once, and every viewer is hit.
- Comments, profiles and any saved user content must be encoded on output.

## Mitigation

HTML-encode stored user content on output. Apply a Content Security Policy.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
