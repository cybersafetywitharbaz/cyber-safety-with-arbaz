# SSRF Lab 1 — Basic SSRF against the local server — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** SSRF (Server-Side Request Forgery)
**Lab:** Basic SSRF against the local server
**Difficulty:** Apprentice
**Solved:** 26 November 2026 ~22:02 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-Side Request Forgery (SSRF) — the shop's stock-check feature fetches a URL supplied by the user (`stockApi` parameter) from the server side, with no validation of where that URL points.

## Solution

The shop checks product stock with `POST /product/stock` and a `stockApi` parameter. The server fetches that URL itself:

1. `stockApi=http://localhost/admin` — the server fetched its own local admin panel (local requests bypass the access check) and returned the admin interface, including the user list with carlos.
2. `stockApi=http://localhost/admin/delete?username=carlos` — the server fetched the delete URL for carlos.
3. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the stock checker runs on the server, so `localhost` means the server itself. The admin panel trusts requests that come from localhost, but it cannot tell the difference between its own legitimate request and one an attacker tricked it into making.
- **SSRF in real life:** the same flaw lets an attacker reach internal services (admin panels, metadata endpoints like cloud instance metadata, internal APIs) that are never exposed to the internet — the server becomes the attacker's proxy into the private network.

## Lessons Learned

- Any feature that fetches a user-supplied URL is an SSRF candidate: stock checks, image previews, webhooks, URL shorteners, PDF generators.
- "Only localhost can access the admin panel" is not a control when another feature can make localhost requests on an attacker's behalf.
- Defence: validate and allowlist the URLs a server-side fetcher may request, and never let it reach internal addresses (localhost, 127.0.0.1, 192.168.x.x, 169.254.169.254).

## Next Steps

- SSRF Lab 2 (another back-end system), Lab 4 (blacklist filter), Lab 5 (whitelist filter) — solved the same evening.
- Labs 3 and 7 (blind SSRF) need an out-of-band receiver — Burp Collaborator on his own computer.
