# Insecure Deserialization Lab 4 — Arbitrary object injection in PHP — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Insecure Deserialization
**Lab:** Arbitrary object injection in PHP
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~09:35 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0a39006e046e16ef84bf247500ed0030.web-security-academy.net/
**Exploit server:** None (no exploit server shown for this lab)

## Vulnerability Type

PHP object injection — the session cookie holds a Base64 + URL-encoded serialized PHP object that the server unserializes without checking the class. Any class available to the application can be instantiated, including one whose `__destruct()` magic method performs a dangerous action when the object is destroyed.

## Solution

1. Logged in as `wiener:peter` and inspected the session cookie. Decoded (URL-decode, then Base64-decode) it is a serialized PHP object:
   ```
   O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"e66kw6bvr4jcnurl1npztwj3ieop1944";}
   ```
2. The site references `/libs/CustomTemplate.php`. Appending a tilde (`~`) retrieves an editor-generated backup file, `/libs/CustomTemplate.php~`, which leaks the source code. The `CustomTemplate` class has a `__destruct()` magic method that calls `unlink()` on its `lock_file_path` attribute — so destroying a `CustomTemplate` object deletes whatever file its path points to.
3. Built a malicious serialized object pointing `lock_file_path` at Carlos's file:
   ```
   O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
   ```
4. Base64-encoded the object, URL-encoded the result, and replaced the session cookie with it:
   ```python
   import base64, urllib.parse
   obj = b'O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}'
   cookie = urllib.parse.quote(base64.b64encode(obj).decode(), safe="")
   ```
5. Sent `GET /my-account` with the forged cookie. The server unserialized the `CustomTemplate` object and, at the end of the request, `__destruct()` ran `unlink("/home/carlos/morale.txt")` and deleted the file. (The response itself was HTTP 500 — the object is not a `User`, so the account page errors — but the file was already deleted by then.)
6. Verified on the instance home page: **"Congratulations, you solved the lab!"**

## Practical Explanation

Serialization turns an object into a flat string so it can be stored (here, in a cookie) and rebuilt later. If the application unserializes data the user controls, the attacker can hand it an object of *any* class the application knows — not just the expected one. PHP runs magic methods automatically at certain points in an object's life: `__destruct()` runs when the object is destroyed (end of request). If a class's `__destruct()` does something dangerous with an attribute the attacker controls (here, `unlink()` on a file path), the attacker gets that action for free just by submitting the object. The fix is to never unserialize untrusted input — use a safe format (e.g. JSON) with strict type checks, or sign the serialized data and verify the signature *before* unserializing.

## Lessons Learned

- A `~` backup file (`file.php~`) is a classic source-code leak — check for it whenever a site references a source file.
- Magic methods (`__destruct`, `__wakeup`, `__toString`, `__get`) are the gadget in PHP object injection: the dangerous code already exists in the application, the attacker only has to trigger it.
- PHP serialized format is fully hand-writable: `O:<class-name-length>:"<Class>":<prop-count>:{s:<len>:"<name>";<value>;}` — length indicators must match exactly.
- A 500 error after the exploit does not mean failure: the destructive magic method runs during request cleanup, so the lab can be solved even when the page itself errors. Always check the banner.
