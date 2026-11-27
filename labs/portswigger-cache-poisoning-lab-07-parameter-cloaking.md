# Web Cache Poisoning Lab 7 — Parameter cloaking — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Parameter cloaking
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:15 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0ab200ec03906bfc81b0c06500160096.web-security-academy.net/

## Vulnerability Type

Parameter cloaking — a Rails-style quirk where the cache and the back end parse the query string differently. The `callback` parameter of `/js/geolocate.js` controls the JavaScript function name in the response, and it *is* keyed — but appending a second `callback` to the unkeyed `utm_content` parameter with a semicolon (`;`) hides it from the cache key while the back end still honours it.

## Solution

1. The home page imports `/js/geolocate.js?callback=setCountryCookie`, which returns `setCountryCookie({"country": "United Kingdom"})`. Changing `callback` changes the function name, but the parameter is keyed, so it cannot poison other users directly. Duplicate `callback` parameters also stay keyed.
2. The cloaked request — the second `callback` rides inside the unkeyed `utm_content` value, separated by a semicolon:
   ```
   GET /js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=alert(1) HTTP/1.1
   Host: 0ab200ec03906bfc81b0c06500160096.web-security-academy.net
   ```
   The cache sees one parameter (`utm_content`, unkeyed) and keys the response as `callback=setCountryCookie`; the back end parses the semicolon as a parameter separator, takes the *last* `callback` value, and returns `alert(1)({...})`.
3. Replayed the cloaked request in a loop until the cached `geolocate.js` entry contained `alert(1)`; the victim loaded a page importing it and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 7).

## Practical Explanation

- **Why it works:** the cache and the application disagree about what a parameter is. The cache treats `utm_content=foo;callback=alert(1)` as one unkeyed value; the back-end framework (Rails-style parsing) splits on `;` and sees a second `callback` parameter that overwrites the first. The poisoned response is stored under the innocent-looking key.
- **Why the semicolon matters:** `&` would create a real second parameter that the cache *would* key on; `;` is treated as a separator by the back end but as ordinary data by the cache — that mismatch is the whole vulnerability.
- **The gadget:** `geolocate.js` reflects the `callback` name into executable JavaScript, so controlling it is already XSS; cloaking is what makes that XSS cacheable for everyone.

## Lessons Learned

- Cache poisoning is often a *parser differential*: whenever two components parse the same request differently, the gap can hide a payload from the cache key.
- Semicolons (and other "alternative" separators) are a classic cloaking tool in Rails/PHP-style stacks — always test them when a parameter is keyed but a sibling parameter is not.
- The reflected value does not have to be in the page you poison — poisoning a shared JS resource (`geolocate.js`) hits every page that imports it.
