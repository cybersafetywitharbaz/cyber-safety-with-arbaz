# SSRF Lab 5 — SSRF with whitelist-based input filter — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** SSRF (Server-Side Request Forgery)
**Lab:** SSRF with whitelist-based input filter
**Difficulty:** Expert
**Solved:** 26 November 2026 ~22:02 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-Side Request Forgery (SSRF) with a whitelist filter — the validator parses the URL, extracts the hostname, and checks it against a whitelist, but it mishandles embedded credentials and double-encoded characters.

## Solution

The official bypass payload, sent as a RAW pre-encoded form body:

`stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos`

How it works:

1. The validator parses the URL and reads the hostname as `stock.weliketoshop.net` (after the `@` — the part before it is treated as credentials) — that host is whitelisted, so the check passes.
2. The fetcher decodes `%2523` twice: first to `%23`, then to `#`. So the real request becomes `http://localhost:80` with the fragment `#@stock.weliketoshop.net/admin/delete?username=carlos` — no wait: the fragment is stripped before the request, and the path after decoding is `/admin/delete?username=carlos` on `localhost:80`.
3. The server fetched the local admin delete URL and deleted carlos.
4. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the validator and the fetcher disagree about what the URL means. The validator sees the whitelisted hostname; the fetcher, after decoding, sees localhost. Any time two components parse the same URL differently, an attacker can craft a URL that is safe for one and dangerous for the other.
- **The `#` trick:** in a URL, `#` starts a fragment that is never sent to the server — but here the double-encoded `#` survives validation as data, then becomes a real `#` during fetching, changing where the request goes.

## Lessons Learned

- Whitelists are stronger than blacklists, but only if the URL is parsed exactly once, fully decoded, and the parsed result is what gets fetched.
- Embedded credentials (`user@host`) and encoded delimiters (`%23`, `%40`, `%2F`) are the classic parser-confusion tools.
- Defence: use one well-tested URL parser, decode completely before validating, and fetch exactly the URL that was validated — never re-parse it.

## Next Steps

- Lab 6 (open-redirection bypass) — first instance was broken (stock checker rejected every URL); fresh instance launching.
- Labs 3 and 7 (blind SSRF) need an out-of-band receiver — Burp Collaborator on his own computer.
