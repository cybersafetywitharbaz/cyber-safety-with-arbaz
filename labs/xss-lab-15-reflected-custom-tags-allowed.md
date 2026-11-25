# Reflected XSS with all tags blocked except custom ones

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Practitioner
**Solved:** 15 November 2026

## Objective

Bypass a filter that blocks all standard tags by using a custom tag with a focus handler.

## Method

1. Open the search with payload `<xss id=x onfocus=alert(document.cookie) tabindex=0>` followed by the fragment `#x`.

## Result

The custom tag receives focus and the alert runs. Lab solved.

## Why it works

The filter blocks known HTML tags but allows unknown (custom) ones like `<xss>`. With `tabindex=0` the element can receive focus, and the `#x` fragment focuses it — firing `onfocus`.

## Lessons learned

- A filter that blocks a list of tags cannot block tags it has never heard of.
- Focus-based handlers need no click — the URL fragment alone triggers them.

## Mitigation

Do not rely on tag blocklists. Encode output for its context and enforce a Content Security Policy.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
