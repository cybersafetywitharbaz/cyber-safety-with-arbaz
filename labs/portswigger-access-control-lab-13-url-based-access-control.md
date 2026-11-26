# PortSwigger Lab — URL-Based Access Control Can Be Circumvented (Access Control)

**Lab:** URL-based access control can be circumvented
**Category:** Access control
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented
**Instance:** https://0aa4004903348c63806a9e4200be0094.web-security-academy.net/
**Solved:** 26 November 2026 — user `carlos` deleted from the admin panel; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Broken access control — the app checks authorization against the request URL, but it trusts the client-supplied `X-Original-URL` header to decide which URL was requested, so the check can be bypassed.

## Objective
Access the admin panel and delete the user `carlos`.

## Solution
1. Sent `GET /` with the header `X-Original-URL: /admin`. The app treated the request as if it were for `/admin` and returned the admin panel (the URL check passed because `/` is public, while the real target was `/admin`).
2. Deleted `carlos` with `GET /?username=carlos` and the header `X-Original-URL: /admin/delete`.
3. The lab solved; the home page banner confirmed it. (One delete request returned 403 during the run, but the banner check confirmed the lab solved.)

## Practical Explanation
This is the same class of flaw as the Referer lab (Lab 11) and the Information Disclosure Lab 4 (`X-Custom-IP-Authorization`): a header the client controls is used to make a security decision. The app's access-control logic looks at `X-Original-URL` instead of the actual request path, so the attacker asks for a public URL while telling the app (via the header) that the request is really for the admin page — or vice versa — and the check passes. Headers are input, not evidence; authorization must use the server's own record of the request and the session's role.

## Lessons Learned
- `X-Original-URL` (and `X-Rewrite-URL`) exist for proxies/load balancers — if the app itself honors them, an attacker can rewrite the request path the access control sees.
- Never let a client-controlled header override the real request path in a security check.
- This pattern repeats across topics: Lab 11 (Referer), Info Disclosure Lab 4 (IP header) — whenever a header decides access, forging it is the exploit.
