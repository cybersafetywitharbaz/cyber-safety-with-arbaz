# Insecure Deserialization Lab 2 — Modifying serialized data types — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Insecure Deserialization
**Lab:** Modifying serialized data types
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~09:40 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0acd00ae030e60f080f60dc400440051.web-security-academy.net/
**Exploit server:** None (no exploit server shown for this lab)

## Vulnerability Type

Insecure deserialization combined with PHP 7 loose comparison — the session cookie holds a Base64-encoded serialized PHP `User` object, and the application compares the deserialized `access_token` with the loose `==` operator. In PHP 7, comparing an integer `0` with any non-numeric string using `==` returns true (`0 == "any-string"` is true), so an integer token of `0` passes for any user, including the administrator.

## Solution

1. Logged in as `wiener:peter` and inspected the session cookie. Decoded (Base64-decode) it is a serialized PHP object:
   ```
   O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"dsqnhk416x2mi3d2luc21c9emdhqw2ie";}
   ```
2. Built a forged object that changes the username to `administrator` and the token's *type* from string (`s:32:...`) to integer `0`:
   ```
   O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
   ```
   The username length changes from `s:6` to `s:13`, and the token becomes `i:0` — the exact serialized form matters, including the string lengths.
3. Base64-encoded the object, URL-encoded the result, and replaced the session cookie with it (script `~/workspace/scripts/ps-deserialization.py 2`).
4. Sent `GET /my-account` with the forged cookie — the page showed the `/admin` link, confirming the loose comparison accepted `0` as the administrator's token.
5. Sent `GET /admin/delete?username=carlos` to delete the user Carlos.
6. Verified on the instance home page: **"Congratulations, you solved the lab!"**

## Practical Explanation

PHP's `==` operator does type juggling: when comparing an integer with a string, PHP 7 converts the string to a number, and a non-numeric string becomes `0` — so `0 == "dsqnhk..."` is true. Combined with insecure deserialization, the attacker controls both the *value* and the *type* of the token, so they can submit an integer `0` where the application expects a 32-character string token. The fix is twofold: use strict comparison (`===`, which also checks the type) for security-critical checks, and never unserialize untrusted input — sign serialized session data and verify the signature first.

## Lessons Learned

- In serialized PHP objects, the *type* marker is part of the attack surface: `s:32:"..."` (string) vs `i:0` (integer) changes how the value is compared.
- PHP 7 loose comparison (`==`) between an integer and a non-numeric string is always true for `0` — a classic type-juggling primitive.
- String lengths in the serialized form (`s:13:"administrator"`) must be exact, or unserialization fails.
