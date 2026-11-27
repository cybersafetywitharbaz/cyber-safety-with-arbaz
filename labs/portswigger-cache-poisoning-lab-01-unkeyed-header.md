# Web Cache Poisoning Lab 1 — Web cache poisoning with an unkeyed header — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Web cache poisoning with an unkeyed header
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:15 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0a6200fd03a9ec5583fd0a96009400d4.web-security-academy.net/
**Exploit server:** https://exploit-0adf008203adeccb831809b901e00084.exploit-server.net

## Vulnerability Type

Web cache poisoning via an unkeyed header (`X-Forwarded-Host`) leading to stored XSS — the header is not part of the cache key, but the application uses it to build the absolute URL of the JavaScript file `/resources/js/tracking.js` imported by the home page. Poisoning the cache makes every visitor load the attacker's script instead.

## Solution

1. Confirmed the home page imports `<script src="https://<host>/resources/js/tracking.js">` and that `X-Forwarded-Host` changes that URL. Sent a probe with a cache buster:
   ```
   GET /?cb=1234 HTTP/1.1
   Host: 0a6200fd03a9ec5583fd0a96009400d4.web-security-academy.net
   X-Forwarded-Host: example.com
   ```
   The response reflected `https://example.com/resources/js/tracking.js` and returned `X-Cache: hit` on replay — the header is unkeyed.
2. Stored the exploit file on the lab's exploit server at `/resources/js/tracking.js` with body `alert(document.cookie)` (Content-Type `application/javascript`).
3. Poisoned the cache by replaying the home-page request with `X-Forwarded-Host: exploit-0adf008203adeccb831809b901e00084.exploit-server.net` (no cache buster) every ~5 seconds until the cached home page's `tracking.js` `src` pointed at the exploit server.
4. The victim visited while the cache was poisoned; the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 1).

## Practical Explanation

- **Why it works:** the cache key is (method, path, Host, …) but not `X-Forwarded-Host`. The back end, however, trusts that header to build absolute URLs. So one crafted response — built from my header — is stored under the normal home-page key and served to everyone.
- **Why the cache buster matters:** the first probe uses `?cb=1234` so the poisoned response is cached under a throwaway key and only I see it. Removing the buster for the real poison puts the payload under the plain `/` key every visitor uses.
- **TTL:** the cache expires after ~30 seconds, so the poison must be re-sent in a loop until the victim visits.

## Lessons Learned

- Any input that is reflected into a response but excluded from the cache key is a poisoning vector — headers are the classic case.
- `X-Forwarded-Host` (and friends) should never be trusted to build URLs unless the cache key includes them.
- When a response is cacheable, one request can compromise every visitor — poisoning is a distribution mechanism, not just a bug.
