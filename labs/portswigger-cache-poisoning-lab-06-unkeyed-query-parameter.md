# Web Cache Poisoning Lab 6 — Web cache poisoning via an unkeyed query parameter — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Web cache poisoning via an unkeyed query parameter
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:15 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0a840018046d919d80e003b6008e0089.web-security-academy.net/

## Vulnerability Type

Web cache poisoning via an unkeyed query parameter (`utm_content`) leading to reflected XSS — the query string *is* part of the cache key here, but the `utm_content` parameter is specifically excluded from it (analytics parameters are commonly unkeyed), and its value is reflected in the home-page response.

## Solution

1. Confirmed the home page is a cache oracle: changing the query string gave a cache miss (query string is keyed). Probing with `utm_content` showed it is excluded from the key — adding it still returned `X-Cache: hit`.
2. Sent the payload in `utm_content` with a cache-buster parameter until the cached response contained it:
   ```
   GET /?utm_content='/><script>alert(1)</script> HTTP/1.1
   Host: 0a840018046d919d80e003b6008e0089.web-security-academy.net
   ```
3. Re-poisoned the plain `/` entry in a loop (the cache expires after ~30 seconds); the victim visited while poisoned and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 6).

## Practical Explanation

- **Why it works:** the cache key is the query string *minus* `utm_content`. The back end, however, reflects the whole query string — including `utm_content` — into the page. So a response built from my `utm_content` payload is stored under the key for `/`, and every visitor gets it.
- **Why `utm_*` parameters are a classic target:** caches routinely exclude marketing/analytics parameters (`utm_source`, `utm_content`, …) from keys for performance, which makes them ideal unkeyed inputs when the app reflects them.
- **Difference from Lab 5:** there the *whole* query string was unkeyed; here only one named parameter is excluded — so the probe must find that specific parameter (here, by testing `utm_content` and watching for a cache hit).

## Lessons Learned

- Excluding a parameter from the cache key is only safe if the application never reflects that parameter — "analytics only" parameters get reflected more often than people think.
- The cache oracle test (change one input, watch hit vs miss) maps the key precisely: keyed inputs cause misses, unkeyed inputs keep returning hits.
- A ~30-second TTL means the poison must be re-sent continuously until the victim arrives; one shot is rarely enough.
