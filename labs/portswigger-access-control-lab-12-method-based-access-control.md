# PortSwigger Lab — Method-Based Access Control Can Be Circumvented (Access Control)

**Lab:** Method-based access control can be circumvented
**Category:** Access control
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented
**Instance:** https://0a190073034e5edd81f148fe00f20052.web-security-academy.net/
**Solved:** 26 November 2026 — `wiener` promoted to admin by switching the request method; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Broken access control — the authorization check only applies to POST requests to `/admin-roles`; the same endpoint with a GET request is not checked at all.

## Objective
Promote `wiener` to admin.

## Solution
1. Logged in as `administrator:admin` and opened `/admin` to see the role-change form (POSTs to `/admin-roles` with `username` and `action=upgrade`).
2. Logged in as `wiener:peter` and sent the same parameters as a GET instead of a POST: `GET /admin-roles?username=wiener&action=upgrade`.
3. The server accepted it — `wiener` was promoted to admin. The lab solved; the home page banner confirmed it.

## Practical Explanation
The access-control rule was written for one HTTP method only (POST), but the endpoint itself accepts other methods and performs the same action. An attacker just changes the method and the check never runs. Authorization must be enforced per endpoint (per route), regardless of method — or the endpoint must reject methods it does not expect. The same pattern appears with HEAD, PUT, PATCH and other methods on other apps.

## Lessons Learned
- When a POST endpoint is access-controlled, always try the same request as GET (and other methods) — the check may only cover one method.
- Authorization belongs on the route/function, not on a single method handler.
- A 401/403 on POST is not proof the function is protected — verify every method the endpoint accepts.
