# SSTI Lab 2 — Basic server-side template injection (code context) — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Server-side template injection (SSTI)
**Lab:** Basic server-side template injection (code context)
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~03:27 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-side template injection (SSTI) in a code context — the "preferred name" setting (`blog-post-author-display`) is placed inside a Tornado template expression `{{...}}`, so breaking out of the expression lets an attacker inject arbitrary template syntax.

## Solution

1. Logged in as `wiener:peter` and posted a comment on a blog post (the author name above the comment comes from the `blog-post-author-display` setting).
2. The setting is changed with `POST /my-account/change-blog-post-author-display`. The normal values are `user.name`, `user.first_name`, `user.nickname`.
3. Broke out of the template expression with `}}` and injected a maths test: `user.name}}{{7*7}}` — reloading the blog post showed the author as `Peter Wiener49}}`, confirming SSTI in a code context.
4. Tornado executes Python inside `{% ... %}` blocks. Sent the exploit payload:
```
blog-post-author-display=user.name}}{% import os %}{{os.system('rm /home/carlos/morale.txt')
```
5. Reloaded the blog post page to render the template — the command ran and the home page showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the setting value is embedded inside `{{...}}` in the template. Closing the expression with `}}` ends the intended statement, and everything after it is parsed as new template code — including `{% import os %}` and the `os.system()` call.
- **The key detail:** code-context SSTI is easy to miss because it does not look like XSS — the injected HTML is not rendered, the value is just a variable name. The `}}` breakout plus a `7*7` test is the reliable detector.

## Lessons Learned

- Any user-controlled value placed inside a template expression is an injection point, even if it looks like a harmless dropdown choice.
- Tornado's `{% %}` blocks run raw Python — once you can inject template syntax, you are one `import os` away from command execution.
- Prefer allow-listed values for settings like display names; never build a template from a request parameter.
