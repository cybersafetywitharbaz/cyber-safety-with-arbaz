# PortSwigger Lab — Unprotected Admin Functionality with Unpredictable URL (Access Control)

**Lab:** Unprotected admin functionality with unpredictable URL
**Category:** Access control
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url
**Instance:** https://0a11005f03ba8c2680c7992000450095.web-security-academy.net/
**Solved:** 26 November 2026 — user `carlos` deleted from the admin panel; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Broken access control — the admin panel is at a random, unguessable URL, but it has no access check, and the URL itself is disclosed in the home page JavaScript.

## Objective
Find the admin panel and delete the user `carlos`.

## Solution
1. Opened the lab home page and read its JavaScript. The script disclosed the admin panel URL: `/admin-<random string>`.
2. Opened that URL — the admin panel loaded with no login required, listing `wiener` and `carlos`.
3. Submitted the delete action for `carlos`. The lab solved; the home page banner confirmed it.

## Practical Explanation
A random URL is only a secret if nobody can read it. Here the home page itself hands the URL out in its JavaScript, so the "unpredictable" part is useless — and even without that leak, the panel has no authorization check at all. Anyone who learns the URL (from JS, from a proxy log, from a shared link) gets full admin access. The fix is the same as Lab 1: check the user's role server-side on every admin request.

## Lessons Learned
- Read the page source and JavaScript — apps often leak admin paths, API endpoints and secrets there.
- A random URL is not an access control: it is obscurity, and obscurity fails the moment the URL is disclosed anywhere.
- Every admin function must verify the caller's role, regardless of how the URL was reached.
