# Host Header Lab 2 — Host header authentication bypass — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** HTTP Host Header Attacks
**Lab:** Host header authentication bypass
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~04:58 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Host header authentication bypass — the application decides whether a request comes from a local (trusted) user by reading the HTTP `Host` header, so anyone who sets `Host: localhost` is treated as an administrator.

## Solution

1. Fetched the home page normally (`GET /`) to obtain the session cookies. The Academy edge routes requests by the `_lab` cookie when the `Host` header does not name a known lab, so every forged-Host request must carry those cookies exactly.
2. Requested the admin panel with a forged Host header:
   ```
   GET /admin
   Host: localhost
   Cookie: <session cookies from GET />
   ```
   The response was the admin panel (users list with Delete links).
3. Deleted carlos with the same forged Host:
   ```
   GET /admin/delete?username=carlos
   Host: localhost
   Cookie: <session cookies>
   ```
   Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the app checks `Host == localhost` to decide the request is local and therefore privileged. The `Host` header is fully attacker-controlled, so the check is no check at all.
- **Why the cookies matter:** with `Host: localhost` and no cookies, the Academy edge cannot route the request and answers 403 "Client Error: Forbidden". With the `_lab`/`session` cookies from a normal visit, the edge routes by cookie and the app sees `Host: localhost`.
- **A tooling gotcha:** Python `requests`' cookie jar mangles the `_lab` cookie value when sending it (the edge then fails to route and returns 403), while the same cookies sent as a raw `Cookie` header work. All Host-override requests in this topic were therefore sent with raw HTTP over a proxy CONNECT tunnel instead of through the `requests` session.

## Lessons Learned

- Never use the `Host` header (or any client-controlled header) for authentication or authorization decisions — trust only the network-level source of the request.
- If a panel must be local-only, enforce it at the network/proxy layer (e.g. firewall rules, binding to loopback), not in application code reading a header.
