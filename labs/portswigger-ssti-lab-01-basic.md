# SSTI Lab 1 — Basic server-side template injection — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Server-side template injection (SSTI)
**Lab:** Basic server-side template injection
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~03:25 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-side template injection (SSTI) — the shop renders the `message` GET parameter inside an ERB (Ruby) template, so template syntax in the parameter is evaluated server-side.

## Solution

1. Viewing a product uses `GET /?message=...` to render "Unfortunately this product is out of stock" on the home page.
2. Confirmed the injection with a maths test: `GET /?message=<%= 7*7 %>` rendered `49` — the expression was evaluated server-side (ERB syntax `<%= ... %>`).
3. From the Ruby documentation, `system()` executes arbitrary OS commands. Sent the exploit payload (URL-encoded):
```
GET /?message=<%= system("rm /home/carlos/morale.txt") %>
```
4. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** user input is concatenated directly into the template instead of being passed in as data, so `<%= ... %>` is treated as template code. The ERB engine evaluates it on the server, and `system()` hands the string to the operating system.
- **The key detail:** the `message` parameter is reflected inside a template, not just echoed as HTML — a `7*7` test returning `49` (instead of the literal text) is the proof that the value is being executed, not just displayed.

## Lessons Learned

- SSTI is far more dangerous than XSS: the same reflected input that looks like a display bug can become full remote code execution.
- Always test reflected parameters with `{{7*7}}` / `<%= 7*7 %>` style maths — a computed answer proves server-side evaluation.
- Never concatenate user input into templates; pass it in as data instead.
