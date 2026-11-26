# SSRF Lab 6 — SSRF with filter bypass via open redirection — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** SSRF (Server-Side Request Forgery)
**Lab:** SSRF with filter bypass via open redirection
**Difficulty:** Practitioner
**Solved:** 26 November 2026 ~22:09 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-Side Request Forgery (SSRF) via an open redirect — the stock checker only allows the local application, but the shop has an open-redirect endpoint that the checker follows to an internal admin server.

## Solution

1. The stock check form in this lab is different from the other SSRF labs: its `stockApi` options are RELATIVE paths (`/product/stock/check?productId=1&storeId=1`), not full URLs. The checker only allows the local application and rejects absolute URLs with `400 "Invalid external stock check url 'Invalid URL'"` — even the instance's own full URL, and even `http://localhost/admin`.
2. The shop has an open redirect: `GET /product/nextProduct?path=<url>` returns `302` to `<url>` (verified directly: `?path=http://192.168.0.12:8080/admin` → `302 → http://192.168.0.12:8080/admin`).
3. The working payload is a RELATIVE URL through the redirect: `stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos`
4. The checker fetched the local redirect, followed it to the internal admin server at `192.168.0.12:8080`, and deleted carlos.
5. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the checker validates only the first hop (a local path — allowed), but it follows redirects without re-validating the destination. The redirect is the bridge from "local only" to "any internal host".
- **The debugging lesson:** the first four attempts all failed with the same 400, including a plain `http://localhost/admin` that works in Lab 1. The clue was in the form itself — reading the product page's `<select>` options showed this lab's checker expects relative paths. When an exploit fails uniformly, read what the application itself sends.

## Lessons Learned

- Open redirects turn a "local only" SSRF filter into full internal access — the redirect target is never validated.
- Always read the form's real default values before crafting a payload: the expected input format (relative vs absolute URL) is written in the page.
- A 400 on every input (even known-good ones) means the input format is wrong, not the exploit idea — compare against the application's own requests.

## Next Steps

- SSRF Labs 3 and 7 (blind SSRF) need an out-of-band receiver — Burp Collaborator on his own computer (no exploit server is provided for these labs).
- Next topic in the roadmap after SSRF: XXE.
