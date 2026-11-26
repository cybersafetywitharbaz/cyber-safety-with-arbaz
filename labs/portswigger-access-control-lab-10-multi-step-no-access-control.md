# PortSwigger Lab — Multi-Step Process with No Access Control on One Step (Access Control)

**Lab:** Multi-step process with no access control on one step
**Category:** Access control
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step
**Instance:** https://0a0b00e403cde0d3817a4340005e0040.web-security-academy.net/
**Solved:** 26 November 2026 — `wiener` promoted to admin via the unprotected confirmation step; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Broken access control in a multi-step workflow — the role-change has two steps, and only the first is access-controlled; the confirmation step can be replayed by any logged-in user.

## Objective
Promote `wiener` to admin.

## Solution
1. Logged in as `administrator:admin` and opened `/admin`. The role form posts to `/admin-roles` with a `<select name='username'>` and `Upgrade user` / `Downgrade user` buttons.
2. Submitted step 1 as admin: `POST /admin-roles` with `username=carlos&action=upgrade`. The response showed the confirmation form: hidden fields `action=upgrade`, `confirmed=true`, `username=carlos`, and a "Yes" button.
3. Logged in as `wiener:peter` and replayed the confirmation step with the username changed: `POST /admin-roles` with `action=upgrade&confirmed=true&username=wiener`.
4. The server accepted it — `wiener` was promoted to admin. The lab solved; the home page banner confirmed it. (Replaying step 1 itself as wiener returns 401 Unauthorized — only the confirmation step is unprotected.)

## Practical Explanation
The app checks authorization on the first step of the workflow but assumes that reaching the second step means the first was properly done. An attacker can skip the protected step and call the confirmation directly with their own username. Every step of a sensitive workflow must independently verify the caller's authorization — "the user got this far" is not proof they were allowed to.

## Lessons Learned
- Map every step of a multi-step flow and test each one separately as a low-privileged user — the unprotected step is often the second one.
- A confirmation/replay step must re-check authorization and re-validate the data; hidden fields like `confirmed=true` are attacker-controlled.
- Note the form details: step 1 uses a `<select>` and `<button>` elements, not `<input>`s — automation that only parses inputs misses the fields entirely.
