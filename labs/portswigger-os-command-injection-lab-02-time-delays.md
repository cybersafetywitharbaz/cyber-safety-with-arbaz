# OS Command Injection Lab 2 — Blind OS command injection with time delays — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OS Command Injection (blind)
**Lab:** Blind OS command injection with time delays
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~03:42 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Blind OS Command Injection — the shop's feedback form passes the `email` field into a server-side shell command, but the command's output is never returned in the response. Execution can only be confirmed by making the server pause (time delay).

## Solution

1. Fetched the feedback form (`GET /feedback`) and took its CSRF token.
2. Submitted feedback with the injection in the `email` field:
   ```
   POST /feedback/submit
   Content-Type: application/x-www-form-urlencoded

   csrf=<token>&name=Test&email=x||ping -c 10 127.0.0.1||&subject=Test&message=Test
   ```
3. The response took **10.6 seconds** to return (normally ~1 second) — the `ping -c 10` delay proving the command ran.
4. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why `||` on both sides:** the application runs its own command first; the leading `||` runs our command only when that command "fails", and the trailing `||` keeps the rest of the original command line as a harmless `OR …` branch so nothing breaks.
- **Why `ping -c 10`:** on Linux, `ping -c 10 127.0.0.1` waits ~10 seconds. The response time IS the signal — a 10-second delay means the injection executed; a fast response means it didn't.
- **Why blind matters:** the response body was just `{}` — no output at all. Without the timing signal there would be no way to know the command ran.

## Lessons Learned

- Blind injection is still full command execution — the attacker just needs a different signal (time, output redirection, or an out-of-band callback).
- Time delays are the simplest blind confirmation: `ping`/`sleep` turn an invisible vulnerability into a measurable one.
- Fix is the same as Lab 1: never pass user input to a shell; validate the email format and use fixed commands.
