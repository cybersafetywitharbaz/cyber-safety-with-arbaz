# DOM XSS in document.write sink using source location.search

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Break out of an image tag and run a script through the search parameter.

## Method

1. Open `/?search="><svg onload=alert(1)>`.

## Result

The alert runs. Lab solved.

## Why it works

The search term is written into the page inside an img tag, as the value of its src attribute. The double-quote closes that attribute, the angle bracket closes the img tag, and the injected `<svg onload=...>` then runs when it loads.

## Lessons learned

- DOM XSS happens entirely in the browser — the payload may never reach the server.
- Sinks like document.write must never receive unsanitized input.

## Mitigation

Avoid document.write. Encode data before inserting it into the DOM, or use safe DOM APIs like textContent.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
