# DOM XSS in jQuery selector sink using a hashchange event

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice
**Solved:** 15 November 2026

## Objective

Use the URL hash to make jQuery build an image element that runs code, delivered through the exploit server.

## Method

1. Build an exploit-server page with an iframe pointing at the lab, with the hash set to `#<img src=x onerror=print()>` so the hashchange handler fires.

## Result

The print dialog opens. Lab solved.

## Why it works

The page reads the URL hash and passes it to jQuery's `$()` selector. jQuery treats an HTML-looking string as markup and creates the `<img>` element — its `onerror` fires because `src=x` fails to load.

## Lessons learned

- jQuery's $() is a sink when it receives user input — it builds HTML, not just selects elements.
- URL fragments are attacker-controlled input too.

## Mitigation

Never pass location.hash (or any user input) to jQuery's $() as a selector. Validate and encode first.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
