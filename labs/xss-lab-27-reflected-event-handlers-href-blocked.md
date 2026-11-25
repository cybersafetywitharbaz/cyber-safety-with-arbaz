# Reflected XSS with event handlers and href attributes blocked

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Practitioner
**Solved:** 15 November 2026

## Objective

Bypass a filter that blocks event handlers and href attributes by using an SVG animation to set a javascript: URL.

## Method

1. Open the search with `<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>`.

## Result

Clicking the text runs the alert. Lab solved.

## Why it works

`<animate>` is neither an event handler nor an href attribute, so the filter lets it through — but it can set the anchor's href to a `javascript:` URL, which runs when the link is clicked.

## Lessons learned

- Filters check names, not behaviour — an animation element that sets an href is as dangerous as an href itself.
- SVG has many elements that can change attributes over time.

## Mitigation

Do not rely on attribute blocklists. Encode output for its context and enforce a Content Security Policy.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
