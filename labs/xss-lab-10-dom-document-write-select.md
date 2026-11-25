# DOM XSS in document.write sink using source location.search inside a select element

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Escape from a select/option element and run a script through the storeId parameter.

## Method

1. Send the storeId value `</option></select><img src=x onerror=alert(1)>`.

## Result

The alert runs. Lab solved.

## Why it works

The value is written inside `<select><option>`, so both tags must be closed first: `</option></select>` gets out of the select, and the injected `<img onerror>` then fires.

## Lessons learned

- The payload must match the HTML context it lands in — here two closing tags are needed before the attack tag.
- DOM sinks with document.write need the same care as server-side reflection.

## Mitigation

Avoid document.write. Encode user data for the exact HTML context it lands in.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
