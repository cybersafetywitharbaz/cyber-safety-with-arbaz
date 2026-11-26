# OAuth Lab 3 — Forced OAuth profile linking — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OAuth 2.0 authentication
**Lab:** Forced OAuth profile linking
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:49 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Cross-site request forgery (CSRF) in the OAuth profile-linking flow — the authorization request that links a social profile to a blog account carries no `state` parameter, so an attacker can start the linking flow with their own social account and trick a logged-in victim into completing it, binding the victim's account to the attacker's social profile.

## Solution

1. Logged in to the blog as `wiener:peter` and clicked "Attach a social profile". The link goes to the OAuth provider:
   ```
   GET https://oauth-<id>.oauth-server.net/auth?client_id=<id>&redirect_uri=https://<lab>/oauth-linking&response_type=code&scope=openid%20profile%20email
   ```
   Note: **two credential sets exist in this lab** — blog account `wiener:peter`, social media profile `peter.wiener:hotdog` (`wiener:peter` is rejected by the OAuth provider with "Invalid username/email or password").
2. At the provider, logged in with the social account `peter.wiener:hotdog` and confirmed consent. The provider redirected to the linking callback — **captured the URL and did NOT request it**, so the code stayed valid:
   ```
   https://<lab>/oauth-linking?code=<stolen-code>
   ```
3. Stored this exploit on the lab's own exploit server and delivered it to the victim (the admin, who has an active blog session):
   ```html
   <iframe src="https://<lab>/oauth-linking?code=<stolen-code>"></iframe>
   ```
4. The victim's browser loaded the iframe, completing the OAuth linking flow with the attacker's social profile — the admin's blog account was now bound to the attacker's social account.
5. Logged in via "Log in with social media" with `peter.wiener:hotdog` — the blog instantly logged in as the admin (`administrator@normal-user.net`).
6. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the linking flow's only protection against cross-user completion is supposed to be the `state` parameter (a CSRF token bound to the session that started the flow). With no `state`, the `code` in the callback URL is a bearer credential — whoever's browser presents it gets their account linked to the social profile it belongs to.
- **Why the code must not be used first:** an authorization code is single-use; requesting the callback URL in the attacker's own (wiener) session would consume it and link wiener's own account instead. Capturing it without requesting it keeps it live for the victim.
- **The key detail:** the victim never types anything — the iframe silently fires the callback in the victim's logged-in browser, which is exactly why a missing `state` parameter is a CSRF hole and not just a theoretical weakness.

## Lessons Learned

- Always send a `state` parameter in OAuth authorization requests and verify it on callback — it is the CSRF protection for the whole flow, including profile linking.
- Treat authorization codes as single-use, short-lived secrets and bind them to the session/client that requested them.
- Two credential sets in one lab is a deliberate trap: blog and social accounts are different users (`wiener:peter` vs `peter.wiener:hotdog`) — read the lab description before assuming one login works everywhere.
