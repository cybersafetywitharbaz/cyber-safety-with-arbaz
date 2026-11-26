# PortSwigger Lab — Bypassing Access Controls Using Email Address Parsing Discrepancies (Business Logic)

**Lab:** Bypassing access controls using email address parsing discrepancies
**Category:** Business Logic flaws — access control via email domain
**Difficulty:** EXPERT
**Lab page:** https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-bypassing-access-controls-using-email-address-parsing-discrepancies
**Instance:** https://0aae00f20349f4c68145b1c4005c0092.web-security-academy.net/
**Solved:** 25 November 2026 — verified "Congratulations, you solved the lab!" banner, "User deleted successfully!" on /admin, and Solved badge.

## Vulnerability Type
Inconsistent email parsing between the registration validator and the mail server — the app grants admin rights based on the email domain it *thinks* the user has, while the mail server delivers to a different address than the one the app recorded.

## Objective
Register an account that the app treats as a `@ginandjuice.shop` user (which gets admin access), while the confirmation email actually goes to the attacker's exploit-server inbox, then delete `carlos` from the admin panel.

## Solution
1. Launched a fresh instance from the lab page (the earlier instance was unreachable from this network).
2. Registered username `attacker` with an email that the validator accepts as a `ginandjuice.shop` address but that encodes a different delivery address using an RFC 2047 encoded-word the mail server decodes and the validator does not detect (UTF-7 encoded-word form). Registration was accepted: "Please check your emails for your account registration link".
3. Opened the exploit-server email client, found the "Account registration" email, clicked the confirmation link — "Your account registration was successful!"
4. Logged in as `attacker`. The Admin panel was visible because the stored account still looks like a `ginandjuice.shop` user. Deleted `carlos` from /admin.

## Practical Explanation
Two components parse the same email string differently. The registration validator checks the raw string and sees a `ginandjuice.shop` address, so it grants the account admin privileges. The mail server decodes the encoded-word first and delivers the confirmation to the attacker's inbox instead. The app never notices the mismatch, so the attacker confirms an account the app believes belongs to the privileged domain.

## Lessons Learned
- When two systems parse the same input, an attacker will find the parse difference — validation must happen on the canonical, decoded value, not the raw string.
- Email domain checks are access control: they must be enforced on the normalised address the mail system will actually use.
- Encoded-words and other encodings must be decoded *before* validation, and re-validated after decoding.

## Mitigation
- Canonicalise and fully decode email addresses (including RFC 2047 encoded-words) before any domain or access check.
- Validate the address again after the mail system parses it, and bind admin rights to a verified, server-side record — not to a string the user supplied.
- Use a strict allowlist parser for privileged domains instead of substring/domain checks on raw input.

## Progress after this lab
- Business Logic: 12/12 — topic complete
- PortSwigger total: 64/272
