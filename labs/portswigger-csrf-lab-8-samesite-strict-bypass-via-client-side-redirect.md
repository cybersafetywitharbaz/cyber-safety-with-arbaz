# PortSwigger Lab — SameSite Strict Bypass via Client-Side Redirect

**Lab:** SameSite Strict bypass via client-side redirect
**Category:** Cross-Site Request Forgery (CSRF)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-client-side-redirect
**Instance:** https://0ad5006c03eeb6338086037f00c300cb.web-security-academy.net/
**Solved:** 26 November 2026 — the victim's (carlos's) email was changed to `pwned@web-security-academy.net`; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site Request Forgery bypassing SameSite=Strict via a client-side redirect gadget — the session cookie is Strict, so no cookies are sent on cross-site requests, but a redirect that happens on the lab's own domain is a same-site navigation, so the cookie is sent.

## Objective
Change the email address of the victim (carlos) by making the victim's browser navigate to the email-change endpoint from inside the lab's own domain.

## Solution
1. Logged in as `wiener:peter` and confirmed the email change works via GET (`/my-account/change-email?email=<addr>&submit=1`).
2. Found the redirect gadget: `/post/comment/confirmation?postId=x` runs `/resources/js/commentConfirmationRedirect.js`, which builds a redirect path from the `postId` parameter. A path traversal in `postId` reaches any endpoint on the same site.
3. Stored this exploit HTML on the lab's exploit server:
   `<script>document.location = "https://<instance>/post/comment/confirmation?postId=1/../../my-account/change-email?email=pwned%40web-security-academy.net%26submit=1";</script>` (the `&` before `submit=1` is encoded as `%26` so it stays inside `postId`).
4. Clicked "Deliver exploit to victim". The victim's browser was sent to the confirmation page, which redirected same-site to the email-change endpoint with the victim's Strict cookie attached, and the email changed. The home page banner confirmed the lab solved.

## Practical Explanation
SameSite=Strict only stops cookies on requests that start cross-site. A redirect gadget on the target site turns the attack into a same-site navigation: the browser follows the redirect within the site, and the cookie goes along. Open redirects and client-side redirect gadgets that accept attacker-controlled paths are therefore CSRF bypasses, not just phishing helpers.

## Lessons Learned
- Strict cookies do not help if the target site has a redirect gadget the attacker can steer.
- Always check comment-confirmation and similar flows for path parameters that build redirect URLs.
- The defence is a CSRF token on the state-changing request — cookie policies alone can be routed around.
