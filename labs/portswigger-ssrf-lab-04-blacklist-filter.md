# SSRF Lab 4 — SSRF with blacklist-based input filter — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** SSRF (Server-Side Request Forgery)
**Lab:** SSRF with blacklist-based input filter
**Difficulty:** Practitioner
**Solved:** 26 November 2026 ~22:02 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-Side Request Forgery (SSRF) with a blacklist filter — the stock checker blocks `127.0.0.1` and the string `/admin`, but the blacklist is incomplete.

## Solution

Two defences, two bypasses, one request:

1. `127.0.0.1` is blocked — use the alternative IP representation `127.1` (which means the same loopback address).
2. `/admin` is blocked — double-URL-encode the letter "a": `%2561` decodes to `%61`, which decodes again to `a`.
3. `stockApi=http://127.1/%2561dmin/delete?username=carlos` — sent as a RAW pre-encoded form body (a normal encoded body would re-encode the `%25` and break the bypass).
4. The server decoded it to `http://127.1/admin/delete?username=carlos`, fetched it, and deleted carlos.
5. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the filter checks the raw input string, but the server decodes the URL before fetching it. What the filter sees (`%2561dmin`) is not what the fetcher requests (`admin`) — the check and the use happen at different layers.
- **Alternative IP forms:** `127.1`, `127.0.1`, `2130706433` (decimal), `0x7f000001` (hex) all mean 127.0.0.1 — a blacklist that only blocks one spelling blocks nothing.

## Lessons Learned

- Blacklists fail because there is always another spelling: encoding, alternative IP formats, case changes, Unicode.
- When a payload contains `%25` sequences, send the form body raw — a library that re-encodes the body destroys the double-encoding.
- Defence: parse the URL first, normalise it completely (decode, resolve the IP), then allowlist the result — never blacklist strings.

## Next Steps

- SSRF Lab 5 (whitelist filter) — solved the same evening.
- Lab 6 (open-redirection bypass) — first instance was broken (stock checker rejected every URL); fresh instance launching.
- Labs 3 and 7 (blind SSRF) need an out-of-band receiver — Burp Collaborator on his own computer.
