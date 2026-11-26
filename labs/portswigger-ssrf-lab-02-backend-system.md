# SSRF Lab 2 — Basic SSRF against another back-end system — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** SSRF (Server-Side Request Forgery)
**Lab:** Basic SSRF against another back-end system
**Difficulty:** Apprentice
**Solved:** 26 November 2026 ~22:02 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-Side Request Forgery (SSRF) — the shop's stock-check feature (`stockApi` parameter) fetches a user-supplied URL from the server side, reaching other machines on the internal network.

## Solution

1. The admin panel is not on the shop server itself this time — it runs on another back-end host in the `192.168.0.x` range, port 8080.
2. Scanned the range through the stock checker: `stockApi=http://192.168.0.1:8080/admin` … `http://192.168.0.255:8080/admin` (255 requests, in parallel).
3. Exactly one host answered with the admin interface: **192.168.0.86** (the response contained the "Admin panel" link and user delete links).
4. `stockApi=http://192.168.0.86:8080/admin/delete?username=carlos` — carlos deleted.
5. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the server-side fetcher is inside the private network, so it can reach internal hosts that no outside user can. The attacker uses it as a scanner: each `stockApi` value is one probe, and the response tells whether that internal host exists.
- **In real life:** this is how attackers map a company's internal network from the outside — one SSRF endpoint becomes a port scanner for every internal service (databases, admin panels, Jenkins, Kubernetes metadata).

## Lessons Learned

- SSRF scanning is slow one request at a time — parallel requests (25 at a time here) find the live host in seconds instead of minutes.
- The signal matters: the admin page's "Admin panel" link only appears on the one live host; every other host returns an error.
- Defence: the fetcher must never be able to reach private IP ranges (RFC 1918, localhost, link-local) — block them at the network layer, not just in application code.

## Next Steps

- SSRF Lab 4 (blacklist filter) and Lab 5 (whitelist filter) — solved the same evening.
- Labs 3 and 7 (blind SSRF) need an out-of-band receiver — Burp Collaborator on his own computer.
