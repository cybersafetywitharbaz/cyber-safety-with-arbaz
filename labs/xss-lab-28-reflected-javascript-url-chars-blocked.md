# Reflected XSS in a JavaScript URL with some characters blocked

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Practitioner
**Solved:** 15 November 2026

## Objective

Bypass a character filter in a JavaScript URL by throwing an error and hijacking toString.

## Method

1. Open the search with `'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+',{x:'` and click "Back to blog".

## Result

alert(1337) runs. Lab solved.

## Why it works

The payload breaks out of the JavaScript string/object, defines an arrow function that throws, and sets `window.toString` — when the thrown error is handled, `onerror=alert` fires with 1337.

## Lessons learned

- Blocked characters only force creativity — the payload routes around them with throw, comments and function tricks.
- JavaScript URLs built from user input are dangerous by design.

## Mitigation

Never build javascript: URLs from user input. Encode for the JS context and enforce a Content Security Policy.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
