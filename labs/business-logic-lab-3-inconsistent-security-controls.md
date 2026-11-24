# Business Logic Lab 3 — Inconsistent Security Controls

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Practitioner
**Solved:** 24 November 2026

## Objective

Delete the user `carlos` by abusing security controls that are enforced inconsistently — the admin panel is shown to accounts whose email address ends in `@dontwannacry.com`, but the email-change flow does not verify the new address.

## Method

1. **Register a new account** with username `arbaz1`, using an email address hosted on the lab's exploit server.
2. **Confirm the account** via the confirmation link in the Email client.
3. **Change the account email** to `arbaz@dontwannacry.com`. The application lets a logged-in user change their email address without verifying the new one.
4. **Open the Admin panel.** Because the account's email now ends in `@dontwannacry.com`, the application treats the account as an administrator and shows the Admin panel.
5. **Delete the user `carlos`** from the Admin panel.

The solve was done in the browser (registration → email confirmation → email change → admin panel), then verified as "Solved".

## Result

✅ **Solved.** User `carlos` was deleted through the Admin panel, and the lab confirmed the solve.

## Why it works

Two different parts of the application disagreed about who is an admin. The admin check looked at a user-controlled field (the email domain), while the email-change flow never verified that the user actually owned that address. Each control looked reasonable on its own; together they handed out admin access to anyone who asked.

## Mitigation

- Never base authorization on a user-controlled attribute such as an email domain. Assign admin rights explicitly, server-side.
- Verify a new email address (confirmation link/code) before it takes effect — especially when it influences access.
- Enforce the same authorization check on every path that reaches admin functionality, not just on the page that links to it.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
