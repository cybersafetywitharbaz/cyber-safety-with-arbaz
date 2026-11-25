# Reflected XSS with AngularJS sandbox escape and CSP

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Practitioner
**Solved:** 15 November 2026

## Objective

Escape the AngularJS sandbox and get past the Content Security Policy using a focus handler and the event path.

## Method

1. Deliver an exploit-server redirect with the `#x` fragment to focus the target element and trigger its `ng-focus` handler.
2. Use `orderBy` with `$event.composedPath()` to reach the window object and run the alert.

## Result

The alert runs. Lab solved.

## Why it works

The focus event fires `ng-focus` (which runs AngularJS code), and `composedPath()` walks the event's path up to the window object — giving the payload a route to code execution that the CSP does not block.

## Lessons learned

- CSP and sandbox together still fall when a handler gives the attacker a path to the window object.
- Event handlers are code entry points too.

## Mitigation

Do not rely on the AngularJS sandbox or on CSP alone. Upgrade the framework and fix the underlying injection.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
