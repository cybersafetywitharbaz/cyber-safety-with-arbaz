# Insecure Deserialization Lab 9 — Developing a custom gadget chain for PHP deserialization — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Insecure Deserialization
**Lab:** Developing a custom gadget chain for PHP deserialization
**Difficulty:** Expert
**Solved:** 27 November 2026 ~09:40 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0ac9006c04d5f3ee8009e971007000eb.web-security-academy.net/
**Exploit server:** None (no exploit server shown for this lab)

## Vulnerability Type

PHP object injection with a custom gadget chain — chaining two classes' magic methods (`__wakeup` → `__get` → `call_user_func`) to reach `exec()` and achieve remote code execution from a serialized object in the session cookie.

## Solution

1. Logged in as `wiener:peter` and inspected the session cookie: a Base64 + URL-encoded serialized PHP object.
2. The site references `/cgi-bin/libs/CustomTemplate.php`. Requesting the backup file `/cgi-bin/libs/CustomTemplate.php~` leaks the source code. Two classes matter:
   - `CustomTemplate::__wakeup()` — runs automatically when the object is unserialized. It creates a new `Product`, passing in the template's `default_desc_type` and `desc` attributes.
   - `DefaultMap::__get($name)` — runs when code reads an attribute the object does not have. It calls `call_user_func($this->callback, $name)` — it executes whatever function is stored in `callback`, passing the requested attribute name as the argument.
3. Traced the data flow: the `Product` constructor tries to read `default_desc_type` from the `desc` object. If `desc` is a `DefaultMap` (which has no `default_desc_type` attribute), PHP invokes `DefaultMap::__get("default_desc_type")`, which runs `call_user_func(callback, "default_desc_type")`. So setting `callback = "exec"` and `default_desc_type = "rm /home/carlos/morale.txt"` makes the chain execute `exec("rm /home/carlos/morale.txt")`.
4. Built the chained object (note: `CustomTemplate->desc` holds the `DefaultMap` object, and `default_desc_type` holds the shell command):
   ```
   O:14:"CustomTemplate":2:{s:17:"default_desc_type";s:26:"rm /home/carlos/morale.txt";s:4:"desc";O:10:"DefaultMap":1:{s:8:"callback";s:4:"exec";}}
   ```
5. Base64-encoded the object, URL-encoded the result, and replaced the session cookie with it:
   ```python
   import base64, urllib.parse
   obj = b'O:14:"CustomTemplate":2:{s:17:"default_desc_type";s:26:"rm /home/carlos/morale.txt";s:4:"desc";O:10:"DefaultMap":1:{s:8:"callback";s:4:"exec";}}'
   cookie = urllib.parse.quote(base64.b64encode(obj).decode(), safe="")
   ```
6. Sent `GET /my-account` with the forged cookie. During unserialization `__wakeup()` built the `Product`, the constructor read the missing attribute, `__get()` fired `exec()`, and Carlos's `morale.txt` was deleted. (The response was HTTP 500 — the page itself errors on the unexpected object — but the command had already run.)
7. Verified on the instance home page: **"Congratulations, you solved the lab!"**

## Practical Explanation

A gadget chain links together harmless-looking pieces of existing application code so that one magic method leads to another until a dangerous sink is reached. Here the chain is short: `__wakeup` (automatic on unserialize) → `Product` constructor reads an attribute → `__get` (automatic on a missing attribute) → `call_user_func` with an attacker-controlled function name → `exec()` with an attacker-controlled command. No single class is "the vulnerability" — the danger is the *combination*, and the attacker controls every link through the serialized object's attributes. This is why deserializing untrusted input is considered unfixable by validation: the gadget chain is built from the application's own code, and no allow-list of classes can reliably predict which combinations are dangerous.

## Lessons Learned

- Read the leaked source first and map the magic methods: `__wakeup` is the entry point, `__get`/`__call`/`__toString` are the pivots, `call_user_func`/`unlink`/`exec` are the sinks.
- A chain can be built purely by nesting serialized objects — `CustomTemplate` holds a `DefaultMap` as its `desc` attribute — no code needs to be uploaded.
- Length indicators in PHP serialization must be exact (`s:17:"default_desc_type"` = 17 chars, `s:26:"rm /home/carlos/morale.txt"` = 26 chars); one wrong count breaks the whole object.
- As in Lab 4, a 500 response after the exploit does not mean failure — the command runs during deserialization, before the page renders. Always check the banner.
