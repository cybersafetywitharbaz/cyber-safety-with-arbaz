# Business Logic Lab 9 — Authentication bypass via flawed state machine

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Practitioner
**Solved:** 25 November 2026

## Objective

Bypass the lab's authentication, access the admin interface, and delete the user `carlos` — by exploiting the application's flawed assumptions about the sequence of events in the login process.

## Method

1. **Log in** with the lab's test account `wiener:peter`. The login form needs its CSRF token (taken from `GET /login`).
2. **Watch the redirect.** `POST /login` returns HTTP 302 redirecting to `GET /role-selector` — after logging in, the user is supposed to pick a role before reaching the home page.
3. **Skip the role-selector step.** Instead of requesting `/role-selector` at all, go straight to the home page (`GET /`). The home page now shows the **Admin panel** link and the account link reads `My account?id=administrator` — the role has **defaulted to `administrator`** because the role-selection step never ran.
4. **Open the admin panel** (`GET /admin`). It lists the users `wiener` and `carlos`, each with a `Delete` link (`/admin/delete?username=...`) — a plain GET link, no form and no CSRF token.
5. **Delete carlos** with `GET /admin/delete?username=carlos`. The lab shows "Congratulations, you solved the lab!" and the banner reads "Solved" (carlos no longer appears in the admin list).

The solve was done with a free Python `requests` script (no Burp Pro): login → capture the 302 to `/role-selector` → skip it, request `/` directly → open `/admin` → delete carlos. Solver script: `ps-lab9-flawed-state-machine.py`.

## Result

✅ **Solved.** The delete response contains "Congratulations, you solved the lab!" with the lab banner showing "Solved". The login POST's 302 Location was exactly `/role-selector`, and the home page after skipping it confirmed the defaulted administrator role (Admin panel link visible, account id `administrator`).

## Why it works

The application treats the login process as a fixed sequence — login → pick a role → home page — and *assumes* every user passes through the role-selector step. But the role is only set when that step actually runs; the server never checks that it happened. Skipping the step leaves the session in its default state, and the default role happens to be `administrator`. The application trusts the *sequence of events* instead of validating the *state* at each step: "if the user reached the home page, they must have chosen a role." Dropping one request in the middle breaks that assumption and hands over admin rights.

## Lessons from this lab (worth remembering)

- **Every step must validate the state, not assume the previous step ran.** The home page and admin panel should check that a role was actually selected and assigned — never rely on a default role, and never let the default be the most privileged one.
- **Defaults must be the least privileged option.** A session with no role chosen should get the lowest-privilege role (or no access at all), never `administrator`.
- **Skipping a step is an attack, not just replaying one.** Unlike Lab 8 (replaying a confirmation request), this exploit *removes* a step from the workflow — the server must enforce that required steps cannot be bypassed, e.g. by tracking an explicit login state machine server-side and refusing requests that arrive out of order.
- **The 302 Location header tells you the workflow.** The login response pointed straight at `/role-selector` — that redirect is the map of the intended sequence, and the step it names is exactly the one to test skipping.
- **Check the account link after login.** `My account?id=administrator` on the home page was the immediate proof the bypass had worked, before even opening `/admin`.

## Mitigation

- Enforce the workflow server-side with an explicit state machine: a session must be in the "role selected" state before the home page or admin panel will serve it; requests arriving out of order are rejected.
- Default to the least-privileged role (or deny access) when no role has been explicitly assigned — never default to administrator.
- Re-check authorization at every step: the admin panel must verify the user's actual role from the server-side session, not from the fact that the user reached the page.
- Make role assignment an explicit, server-recorded event (not a client-driven page visit) and require it to complete before any authenticated page is served.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
