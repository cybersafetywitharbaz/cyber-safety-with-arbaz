# Reflected XSS into attribute with angle brackets HTML-encoded

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Escape from an HTML attribute using a quote, even though angle brackets are encoded.

## Method

1. Open `/?search=" autofocus onfocus=alert(1) x="`.

## Result

The alert runs when the input receives focus. Lab solved.

## Why it works

Angle brackets are encoded, but the double-quote is not. The quote closes the attribute, and `autofocus onfocus=alert(1)` adds a new event-handler attribute that fires as soon as the element is focused.

## Lessons learned

- Encoding angle brackets alone is not enough — quotes let the attacker out of the attribute.
- Every context (element, attribute, script) needs its own encoding.

## Mitigation

HTML-encode quotes as well as angle brackets in attribute contexts. Encode for the exact context the data lands in.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
