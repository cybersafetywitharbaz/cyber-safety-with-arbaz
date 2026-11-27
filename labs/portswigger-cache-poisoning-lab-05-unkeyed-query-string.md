# Web Cache Poisoning Lab 5 — Web cache poisoning via an unkeyed query string — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Web cache poisoning via an unkeyed query string
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:15 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0a9a002c03e2b39e83fdaf66004500c5.web-security-academy.net/

## Vulnerability Type

Web cache poisoning via an unkeyed query string leading to reflected XSS — the entire query string is ignored by the cache key but reflected in the home-page response, so any parameter can carry the payload.

## Solution

1. Confirmed the home page is a cache oracle: changing the query string still returned `X-Cache: hit`, so the query string is unkeyed. The `Origin` header *is* keyed, so it works as the cache buster.
2. Sent the payload as an arbitrary query parameter with a fresh `Origin` buster until the response was cached with the payload reflected:
   ```
   GET /?evil='/><script>alert(1)</script> HTTP/1.1
   Host: 0a9a002c03e2b39e83fdaf66004500c5.web-security-academy.net
   Origin: https://buster-1.example
   ```
3. Verified the poison: re-sent the plain `/` request with the same `Origin` buster and the cached response still contained the payload. Then removed the buster and replayed the payload request in a loop so the plain `/` entry stayed poisoned; the victim visited and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 5).

## Practical Explanation

- **Why it works:** the cache key ignores the query string entirely, so `/?evil=<payload>` and `/` are the *same* cache entry — but the back end reflects the query string into the page. The payload rides in a parameter the cache never sees.
- **Why `Origin` is the buster here:** the query string cannot bust the cache (it is unkeyed), so a *keyed* header has to do it. `Origin` is part of this cache's key, so changing it forces a miss and a fresh back-end response.
- **Two-stage poisoning:** first poison under a throwaway `Origin` (to prove the reflection caches), then poison the no-buster entry that normal visitors actually get.

## Lessons Learned

- If the query string is unkeyed, *every* parameter is a potential injection point — there is no need to find a "supported" parameter.
- A cache oracle (a page that shows `X-Cache: hit/miss`) is the fastest way to map what is keyed and what is not.
- When the usual buster (query string) is unkeyed, look for a keyed header to bust with instead.
