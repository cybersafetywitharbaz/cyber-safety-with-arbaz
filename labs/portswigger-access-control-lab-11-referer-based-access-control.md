# PortSwigger Lab — Referer-Based Access Control (Access Control)

**Lab:** Referer-based access control
**Category:** Access control
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/access-control/lab-referer-based-access-control
**Instance:** https://0aff001d042cf5fc817d4d48001f009f.web-security-academy.net/
**Solved:** 26 November 2026 — `wiener` promoted to admin via a forged Referer header; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Broken access control — the role-change endpoint decides whether the request is allowed by checking the `Referer` header, which the client fully controls.

## Objective
Promote `wiener` to admin.

## Solution
1. Logged in as `administrator:admin` and opened `/admin` to see the role-change form (posts to `/admin-roles` with `username` and `action=upgrade`).
2. Logged in as `wiener:peter` and sent the same request with a forged header: `GET /admin-roles?username=wiener&action=upgrade` with `Referer: <instance>/admin`.
3. The server accepted it — `wiener` was promoted to admin. The lab solved; the home page banner confirmed it.

## Practical Explanation
The `Referer` header is sent by the browser to say which page the request came from — but any client can set it to any value. The server treated "came from /admin" as proof the requester is an admin, which is exactly backwards: the header is a claim, not evidence. Authorization must come from the session's server-side role, never from a request header.

## Lessons Learned
- Any header (`Referer`, `X-Original-URL`, `X-Forwarded-For`) is attacker-controlled input — it must never be the basis of an authorization decision.
- "The request came from the admin page" only proves the requester knows the admin page's URL, which is public knowledge.
- Test every protected endpoint with the headers the app checks — forging one header is often the whole exploit.
