# Web Cache Poisoning Lab 8 — Web cache poisoning via a fat GET request — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Web cache poisoning via a fat GET request
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:15 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0af90092043d11ec8304001d0035001c.web-security-academy.net/

## Vulnerability Type

Web cache poisoning via a fat GET request — a GET request with a body. The cache key is derived only from the request line, so the body is unkeyed, but the back end reads the `callback` parameter from the body of `/js/geolocate.js` and reflects it into the JavaScript response.

## Solution

1. The home page imports `/js/geolocate.js?callback=setCountryCookie`, which returns `setCountryCookie({...})`. The `callback` in the URL is keyed, so changing it cannot poison others.
2. Sent a GET request whose *body* carries a second `callback` — the request line keeps the innocent value, so the cache key is unchanged, but the back end prefers the body value:
   ```
   GET /js/geolocate.js?callback=setCountryCookie HTTP/1.1
   Host: 0af90092043d11ec8304001d0035001c.web-security-academy.net
   Content-Length: 18

   callback=alert(1)
   ```
   Response: `alert(1)({"country": "United Kingdom"})`.
3. Replayed the fat GET in a loop until the cached `geolocate.js` entry contained `alert(1)`; the victim loaded a page importing it and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 8).

## Practical Explanation

- **Why it works:** caches assume GET requests have no body, so the body is never part of the cache key. This back end, however, parses POST-style body parameters on a GET request. The request line says `callback=setCountryCookie` (the key), the body says `callback=alert(1)` (the payload) — the cache stores the back end's body-driven response under the innocent key.
- **Why the victim is affected:** every page imports the same `geolocate.js?callback=setCountryCookie` URL, so the poisoned entry is served to everyone who loads any page.
- **The same pattern as Lab 7:** both labs hide the real `callback` from the cache key — Lab 7 uses a semicolon inside an unkeyed parameter, Lab 8 uses the request body. The underlying bug is identical: the cache key does not represent every input that changes the response.

## Lessons Learned

- GET requests with bodies ("fat GETs") are legal HTTP but widely mishandled — caches ignore the body, some back ends do not. That gap is a poisoning vector.
- Defences: never accept a body on GET, and never let body parameters override query-string parameters.
- When probing a cache, test *every* part of the request — line, headers, cookies, body — because any part the key ignores is a candidate input.
