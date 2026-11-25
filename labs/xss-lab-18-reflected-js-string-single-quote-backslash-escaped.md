# Reflected XSS into a JavaScript string with single quote and backslash escaped

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Break out of a JavaScript string by closing the script tag itself.

## Method

1. Open the search with `</script><script>alert(document.domain)</script>`.

## Result

The alert runs. Lab solved.

## Why it works

Only quotes and backslashes are escaped — but the payload does not need a quote at all. `</script>` closes the script block, and the new `<script>` tag runs as ordinary page code.

## Lessons learned

- Escaping quotes is useless when the attacker can close the whole script element.
- The HTML parser ends a script block at </script> no matter what the JS escaping does.

## Mitigation

Never embed user data inside script blocks. Pass data via safe DOM APIs or JSON.parse on a separate element.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
