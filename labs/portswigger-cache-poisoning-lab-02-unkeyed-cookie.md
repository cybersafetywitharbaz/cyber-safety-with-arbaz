# Web Cache Poisoning Lab 2 — Web cache poisoning with an unkeyed cookie — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Web cache poisoning with an unkeyed cookie
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:15 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0ac0002c0340efa28000d600000b00c0.web-security-academy.net/

## Vulnerability Type

Web cache poisoning via an unkeyed cookie (`fehost`) leading to reflected XSS — the first response sets `fehost=prod-cache-01`; on later requests the cookie value is reflected inside a double-quoted JavaScript object in the home page, but the cookie is not part of the cache key.

## Solution

1. Loaded the home page and noted the `Set-Cookie: fehost=prod-cache-01` header and that the cookie value is reflected inside a double-quoted JS string in the response.
2. Sent the home page with a cache buster and a crafted cookie, then replayed until `X-Cache: hit` showed the payload in the cached response:
   ```
   GET /?cb=1234 HTTP/1.1
   Host: 0ac0002c0340efa28000d600000b00c0.web-security-academy.net
   Cookie: fehost=someString"-alert(1)-"someString
   ```
   The `"` breaks out of the JS string, `-alert(1)-` executes, and the trailing `"` re-opens the string so the page stays valid.
3. Removed the cache buster and re-poisoned the plain `/` entry in a loop; the victim visited while poisoned and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 2).

## Practical Explanation

- **Why it works:** cookies are normally excluded from cache keys (otherwise every user would get their own cached copy). The application, however, reflects a cookie value into JavaScript without encoding it — so the poisoned response is served to everyone under the shared home-page key.
- **Why the payload is shaped that way:** the value lands inside `"..."` in a JS object, so the payload must close the quote, inject the call, and re-open the quote (`someString"-alert(1)-"someString`).
- **Cache buster first:** probing with `?cb=…` keeps the test response under a throwaway key; only the final poison (no buster) lands on the real `/` key.

## Lessons Learned

- Unkeyed does not mean harmless — cookies, headers and query parameters that are reflected anywhere in a cacheable response are all poisoning inputs.
- Reflecting any user-controlled value into JavaScript without encoding is XSS; reflecting it into a *cached* page turns one XSS into a site-wide one.
- Always check `Set-Cookie` values: the server hands you the unkeyed input on the first response.
