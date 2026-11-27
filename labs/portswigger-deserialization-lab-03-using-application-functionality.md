# Insecure Deserialization Lab 3 — Using application functionality to exploit insecure deserialization — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Insecure Deserialization
**Lab:** Using application functionality to exploit insecure deserialization
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~09:41 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0ad600d604bff58880b5625a001b001f.web-security-academy.net/
**Exploit server:** None (no exploit server shown for this lab)

## Vulnerability Type

Insecure deserialization — the session cookie holds a Base64-encoded serialized PHP `User` object with an `avatar_link` attribute that the application uses as a file path. The application lets the user update their avatar link, and deleting your own account also deletes the avatar file — so pointing `avatar_link` at another user's file and deleting your own account deletes that file.

## Solution

1. Logged in as `wiener:peter` and inspected the session cookie. Decoded (Base64-decode) it is a serialized PHP object:
   ```
   O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"ne8vcf3zwzib6wyvkrswd2x96rdchwt4";s:11:"avatar_link";s:19:"users/wiener/avatar";}
   ```
2. Replaced the `avatar_link` value with Carlos's file, keeping the serialized format exact (string length `s:19` → `s:23`):
   ```
   O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"ne8vcf3zwzib6wyvkrswd2x96rdchwt4";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
   ```
3. Base64-encoded the object, URL-encoded the result, and replaced the session cookie with it (script `~/workspace/scripts/ps-deserialization.py 3`).
4. Sent `POST /my-account/delete` (with `Referer: /my-account`) to delete my own account. The application deleted the avatar file — which was now `/home/carlos/morale.txt`.
5. Verified on the instance home page: **"Congratulations, you solved the lab!"**

## Practical Explanation

This lab needs no gadget chain and no magic-method trick: the dangerous behaviour is the application's own feature. The `User` object's `avatar_link` is a file path, and the account-deletion feature calls `unlink()` (or equivalent) on it. Because the serialized object lives in a cookie the attacker controls, the attacker can point that path at any file the application can delete — here, Carlos's `morale.txt`. The lesson is that deserialized data flows into *all* the application's functionality, so any attribute that ends up as a file path, a command, or a database value is attacker-controlled. The fix is to never unserialize untrusted input: keep session data server-side, or sign the serialized data and verify the signature before unserializing — and never let a user-supplied path reach the filesystem unchecked.

## Lessons Learned

- Read the whole serialized object: every attribute is a potential attack primitive, not just the obvious `admin` flag.
- Application features (here, account deletion) become the exploit when they act on deserialized attributes.
- String lengths in the serialized form must be updated to match the new value (`s:19` → `s:23`), or unserialization fails silently.
