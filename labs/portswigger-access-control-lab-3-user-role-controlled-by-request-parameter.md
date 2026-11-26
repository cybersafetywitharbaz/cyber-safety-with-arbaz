# PortSwigger Lab — User Role Controlled by Request Parameter (Access Control)

**Lab:** User role controlled by request parameter
**Category:** Access control
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter
**Instance:** https://0ac7000c04018a258029a3ab002000c2.web-security-academy.net/
**Solved:** 26 November 2026 — user `carlos` deleted from the admin panel; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Broken access control — the user's role is stored in a client-side cookie (`Admin=false`) and the server trusts it, so changing the cookie makes the user an admin.

## Objective
Become an administrator and delete the user `carlos`.

## Solution
1. Logged in as `wiener:peter`. The login response set a cookie `Admin=false`.
2. Changed the cookie to `Admin=true`.
3. Opened `/admin` — the admin panel loaded, listing `wiener` and `carlos`.
4. Submitted the delete action for `carlos`. The lab solved; the home page banner confirmed it.

## Practical Explanation
The server decides "is this user an admin?" by reading a cookie the browser sends — and cookies are fully controlled by the client. Anyone can edit them in the browser or in a proxy. Role and privilege information must live server-side (in the session on the server), and the client must only ever send an opaque session identifier. Anything the client can change is not a security decision.

## Lessons Learned
- Never trust client-supplied data (cookies, hidden fields, request parameters) for authorization decisions.
- Roles must be looked up server-side from the session, never read from a cookie value.
- After login, always inspect the cookies the app sets — a readable `Admin=false` / `role=user` cookie is a classic privilege-escalation lead.
