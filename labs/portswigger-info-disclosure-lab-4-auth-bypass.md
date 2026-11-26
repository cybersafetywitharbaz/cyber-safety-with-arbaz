# PortSwigger Lab — Authentication Bypass via Information Disclosure (Information Disclosure)

**Lab:** Authentication bypass via information disclosure
**Category:** Information disclosure
**Difficulty:** PRACTITIONER
**Lab page:** https://portswigger.net/web-security/information-disclosure/lab-authentication-bypass-via-information-disclosure
**Instance:** https://0a410032035c82e080a3cb6500640074.web-security-academy.net/
**Solved:** 26 November 2026 — user `carlos` deleted from the admin panel; verified the panel afterwards lists only `wiener`.

## Vulnerability Type
Information disclosure leading to authentication bypass — the application leaks the name of a custom request header it uses to decide whether a request comes from a local user, and then trusts that client-supplied header for access control.

## Objective
Access the admin panel (restricted to local users) and delete the user `carlos`.

## Solution
1. Opened the lab and requested `/admin` in the browser. The page replied: "Admin interface only available to local users".
2. Sent a `TRACE` request to `/admin` (the lab's intended tool is Burp Repeater; here it was done with curl because Burp was not available). The TRACE response echoed the request and revealed a custom header the server adds: `X-Custom-IP-Authorization` (value = the client's own IP).
3. Requested `/admin` again, this time sending the header `X-Custom-IP-Authorization: 127.0.0.1`. The admin panel loaded, listing `wiener` and `carlos`, each with a Delete link.
4. Requested `/admin/delete?username=carlos` with the same header. Carlos was deleted.
5. Verified by re-opening `/admin` with the header: only `wiener` remains. Lab solved.

Standard Burp method (same idea): send the TRACE request in Repeater to learn the header name, then use Burp Match & Replace to add `X-Custom-IP-Authorization: 127.0.0.1` to every request, open `/admin`, and delete `carlos`.

## Practical Explanation
The server decides "is this request local?" by reading a request header instead of the real connection source. A header is client-controlled data — anyone can send any value in it. Setting it to `127.0.0.1` makes the server believe the request came from its own machine, so the local-only admin check passes. The TRACE method made this possible because the server echoes the full request (including the internal header it added), disclosing the header's exact name.

## Lessons Learned
- Never use a client-supplied header as proof of where a request came from — check the actual TCP connection source server-side.
- The TRACE method can leak internal headers and request details; disable TRACE (and TRACK) on production servers.
- Whenever an error or debug response shows internal details (header names, versions, paths), treat it as a lead: the next step is usually to replay the request with the disclosed detail changed in the attacker's favour.
- The same pattern (disclosed detail → forged request → access) appears across the whole Information Disclosure topic: error messages, debug pages, backup files, version control.
