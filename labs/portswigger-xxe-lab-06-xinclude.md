# XXE Lab 6 — Exploiting XInclude to retrieve files — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** XXE / XInclude
**Lab:** Exploiting XInclude to retrieve files
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~03:13 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

XInclude injection — the stock-check value is embedded into a server-side XML document, and the parser processes XInclude directives, so an `<xi:include>` tag reads a local file.

## Solution

1. This lab's stock check is a normal form POST (`productId` + `storeId`, urlencoded) — no DOCTYPE can be injected, because the server builds its own XML document from the value.
2. Sent this `productId` value:
```
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```
3. The response echoed the file: `Invalid product ID: root:x:0:0:root:/root:/bin/bash...`
4. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the server drops the `productId` value into an XML document and parses it with XInclude processing on. The `<xi:include>` directive tells the parser to fetch `file:///etc/passwd` and inline it — the application's own error message then prints it back.
- **`parse="text"` is required** because `/etc/passwd` is not valid XML; without it the parser would try to parse the file as XML and fail.

## Lessons Learned

- Blocking DOCTYPE does not stop XXE-style attacks: XInclude is a second door into the same parser.
- The difference from Lab 1 matters: when you cannot inject a DOCTYPE, look for places where your input is embedded into XML the server builds itself — XInclude rides along.

## Next Steps

- Lab 9 (repurposing a local DTD) and Lab 8 (error-message retrieval via a malicious external DTD) use the same stock-check XML channel.
- Labs 3 and 4 (blind XXE) need Burp Collaborator on his own computer.
