# Reflected XSS with some SVG markup allowed

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Practitioner
**Solved:** 15 November 2026

## Objective

Run a script through an allowed SVG animation event.

## Method

1. Open the search with payload `<svg><animatetransform onbegin=alert(document.domain)>`.

## Result

The alert runs. Lab solved.

## Why it works

SVG tags and their animation events are not blocked. When the animation begins, `onbegin` fires the injected handler.

## Lessons learned

- SVG is XML with its own event model — allowing SVG markup means allowing script-like behaviour.
- Filters must cover SVG and MathML contexts, not just HTML tags.

## Mitigation

Do not allow raw SVG markup from users. Encode output for its context and enforce a Content Security Policy.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
