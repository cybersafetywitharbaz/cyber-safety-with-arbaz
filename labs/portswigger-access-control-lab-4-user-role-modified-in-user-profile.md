# PortSwigger Lab — User Role Can Be Modified in User Profile (Access Control)

**Lab:** User role can be modified in user profile
**Category:** Access control
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile
**Instance:** https://0a6f006e0319cdd980326c85002f005e.web-security-academy.net/
**Solved:** 26 November 2026 — user `carlos` deleted from the admin panel; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Broken access control / mass assignment — the change-email request accepts a JSON body, and adding `"roleid": 2` to it promotes the logged-in user to admin.

## Objective
Promote `wiener` to admin and delete the user `carlos`.

## Solution
1. Logged in as `wiener:peter`. (Note: this lab instance's login form has no CSRF token at all — the login works with just username and password.)
2. Opened `/my-account`. The change-email form has no `action` attribute — it submits via `onsubmit='jsonSubmit(this, event, "/my-account/change-email")'`, sending a JSON body.
3. Posted to `/my-account/change-email` with the JSON body `{"email": "wiener@wiener-mail.com", "roleid": 2}`. The server accepted the extra field and set the role to 2 (admin).
4. Opened `/admin` — the admin panel loaded, listing `wiener` and `carlos`.
5. Followed the delete link for `carlos` (`/admin/delete?username=carlos`). The lab solved; the home page banner confirmed it.

## Practical Explanation
The server binds the whole JSON body the client sends to the user object, including fields the UI never shows. The client controls the request, so any field the server accepts can be set by the attacker — here `roleid`. This is mass assignment: the fix is to whitelist exactly the fields an endpoint may change (only `email` here) and ignore or reject everything else, and to never let a self-service endpoint change a role.

## Lessons Learned
- Always test whether an API/JSON endpoint accepts extra fields the UI does not send — especially `role`, `roleid`, `isAdmin`, `admin`.
- Mass assignment is the server trusting the shape of the client's request; endpoints must whitelist allowed fields.
- A missing `action` attribute or a JS submit handler is not a barrier — read the handler to find the real endpoint and send the request directly.
