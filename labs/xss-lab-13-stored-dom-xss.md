# Stored DOM XSS

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Store a comment that bypasses the sanitizer and runs when the post is viewed again.

## Method

1. Post a comment containing `<><img src=1 onerror=alert(1)>`.

## Result

On the next view of the post, the alert runs. Lab solved.

## Why it works

The leading `<>` is consumed by the sanitizer's encoding step, and the real `<img>` tag survives behind it — when the stored comment is rendered, the image fails to load and `onerror` fires.

## Lessons learned

- Sanitizers that encode only the first suspicious sequence can be bypassed by hiding the payload behind a sacrificial tag.
- Sanitization must be done by a tested library, not a hand-rolled filter.

## Mitigation

Use a well-tested HTML sanitizer (e.g. DOMPurify) for stored content. Encode on output as well.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
