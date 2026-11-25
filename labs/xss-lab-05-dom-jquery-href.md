# DOM XSS in jQuery anchor href attribute sink using location.search source

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Make the page's 'Back' link run JavaScript by controlling its href through the returnPath parameter.

## Method

1. Open `/feedback?returnPath=javascript:alert(document.cookie)`.
2. Click the "Back" link.

## Result

The alert runs with the page's cookies. Lab solved.

## Why it works

jQuery sets the Back link's href from the `returnPath` parameter without checking it. A `javascript:` URL in an href executes as code when the link is clicked.

## Lessons learned

- A javascript: URL is code, not a link — any place a URL is reflected must be validated.
- The attack needs one click, but the link looks like an ordinary Back button.

## Mitigation

Validate redirect/return URLs against an allowlist. Never put user input into an href without checking its scheme.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
