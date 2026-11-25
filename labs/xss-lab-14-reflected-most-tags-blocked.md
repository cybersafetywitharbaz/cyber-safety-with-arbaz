# Reflected XSS with most tags and attributes blocked

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Practitioner
**Solved:** 15 November 2026

## Objective

Bypass a tag/attribute blocklist by injecting a body tag whose resize handler fires.

## Method

1. Deliver an exploit-server iframe with `?search="><body onresize=print()>` and resize the iframe.

## Result

The print dialog opens. Lab solved.

## Why it works

Most tags are blocked, but `<body>` gets through. The injected body merges with the page's real body, and resizing the iframe fires its `onresize` handler.

## Lessons learned

- Blocklists always miss something — here the body tag itself became the attack.
- Event handlers on unusual tags are easy to overlook when writing filters.

## Mitigation

Do not rely on blocklists. Encode output for its context and enforce a Content Security Policy.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
