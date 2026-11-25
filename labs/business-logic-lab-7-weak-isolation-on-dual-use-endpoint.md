# Business Logic Lab 7 — Weak isolation on dual-use endpoint

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Practitioner
**Solved:** 25 November 2026

## Objective

Access the `administrator` account and delete the user `carlos`, by exploiting a password-change endpoint that trusts the `username` parameter sent by the client and never asks for the current password.

## Method

1. **Log in** with the lab's test account `wiener:peter`.
2. **Open the account page** (`GET /my-account`) and read the change-password form. The form posts to `POST /my-account/change-password` and carries a hidden `username` field (set to `wiener`) plus `current-password`, `new-password-1` and `new-password-2` fields.
3. **Change the administrator's password instead:** send the change-password POST with `username=administrator`, `new-password-1` and `new-password-2` set to a new password — and **omit the `current-password` parameter entirely**. The server accepts it and replies "Password changed successfully!" — it never checks whose password is being changed, and never asks for the current one.
4. **Log in as `administrator`** with the new password. The login succeeds and the admin panel appears.
5. **Delete `carlos`** from the admin panel (`/admin/delete?username=carlos`). The lab shows "Congratulations, you solved the lab!"

The solve was done with a free Python `requests` script (no Burp Pro): login → parse the change-password form dynamically → forged change-password POST → admin login → delete carlos. Solver script: `ps-lab7-weak-isolation-dual-use.py`.

## Result

✅ **Solved.** The administrator's password was changed without ever knowing it, carlos was deleted from the admin panel, and the lab confirmed the solve ("Congratulations, you solved the lab!").

## Why it works

The change-password endpoint is "dual-use": the same endpoint serves every user, and it decides *whose* password to change from the `username` value the client sends — instead of from the logged-in session. It also treats the current password as optional: removing the `current-password` parameter entirely still passes. So any logged-in user can point the request at `administrator`, set a password of their choosing, and walk in as admin. The flaw is weak isolation between "change my own password" and "change someone else's password" — the endpoint does both, and trusts the client to say which one.

## Lessons from this lab (worth remembering)

- **A hidden form field is not a security boundary.** The `username` field was hidden in the form, but hidden fields are sent by the client and can be changed to anything — here, to `administrator`.
- **Omitting a parameter can be stronger than forging one.** Removing `current-password` entirely bypassed the check that would have blocked a wrong value — the server only validated the parameter when it was present.
- **Authorisation must come from the session, not from request parameters.** The server should change the password of the user in the session, and ignore (or reject) any `username` the client sends.
- **Parse forms dynamically in scripts.** This lab's form fields were read from the live `/my-account` page at run time, so the script did not depend on hard-coded field names.

## Mitigation

- Determine the target account from the authenticated session only — never from a client-supplied `username` parameter on a password-change endpoint.
- Always require the current password when changing a password, and reject the request if it is missing or wrong.
- Bind the CSRF token to the session and the action, and re-authenticate the user for sensitive account changes.
- Log and alert on password changes for privileged accounts (especially `administrator`).

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
