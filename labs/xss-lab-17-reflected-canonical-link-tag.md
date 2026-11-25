# Reflected XSS in canonical link tag

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Escape from a canonical link's href and plant an accesskey + click handler, then trigger it from the keyboard.

## Method

1. Open the URL with `?%27accesskey=%27x%27onclick=%27alert(document.domain)`.
2. Press Alt+X on the keyboard.

## Result

The alert runs. Lab solved.

## Why it works

The quote escapes the canonical link's href attribute, and `accesskey` plus `onclick` are added to the link element. Pressing Alt+X activates the access key, which clicks the link and fires the handler.

## Lessons learned

- Access keys turn a hidden element into a keyboard-triggered attack — no mouse click needed.
- Link tags are not inert: their attributes are attacker-reachable.

## Mitigation

Encode quotes in attribute contexts. Validate URLs and strip event-handler attributes from reflected data.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
