# PortSwigger Lab — Unprotected Admin Functionality (Access Control)

**Lab:** Unprotected admin functionality
**Category:** Access control
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality
**Instance:** https://0ab600bb0446e40c82b01f720037006d.web-security-academy.net/
**Solved:** 26 November 2026 — user `carlos` deleted from the admin panel; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Broken access control — the admin panel has no access check at all, so any user (even unauthenticated) can open it and delete users.

## Objective
Access the admin panel and delete the user `carlos`.

## Solution
1. Opened the lab home page and fetched `/robots.txt`. It contained: `Disallow: /administrator-panel`.
2. Opened `/administrator-panel` — the admin panel loaded with no login required, listing `wiener` and `carlos`, each with a Delete action.
3. Submitted the delete action for `carlos`. The lab solved; the home page banner confirmed it.

## Practical Explanation
`robots.txt` is meant to tell search engines what not to crawl — it is public to everyone, so it must never be used to hide sensitive URLs. The real flaw is that the admin panel itself checks nothing: it assumes nobody will find the URL. Security through obscurity is not access control — every sensitive endpoint must check the user's role server-side, no matter how the URL was found.

## Lessons Learned
- Always check `robots.txt` — it often lists the very paths an app wants to keep hidden.
- An admin page with no authorization check is fully open to anyone who guesses or discovers the URL.
- Authorization must be enforced on the server for every request to a protected function, not just hidden behind an unlinked URL.
