# SSTI Lab 3 — Server-side template injection using documentation — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Server-side template injection (SSTI)
**Lab:** Server-side template injection using documentation
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~03:29 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-side template injection (SSTI) in a Freemarker template — a content-manager can edit product description templates, and the template engine's `new()` built-in can instantiate arbitrary Java classes, including one that executes shell commands.

## Solution

1. Logged in as `content-manager:C0nt3ntM4n4g3r` and opened a product's "Edit template" page (`/product/template?productId=1`).
2. Identified the engine as Freemarker: the template syntax is `${someExpression}`, and an invalid expression (`${foobar}`) produced an error naming Freemarker.
3. The Freemarker documentation warns that the `new()` built-in can create arbitrary Java objects implementing `TemplateModel`. The JavaDoc lists `freemarker.template.utility.Execute` as an implementing class that runs shell commands.
4. Submitted this payload as the product description template (action `save`):
```
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
```
5. Viewed the product page to render the template — the command ran and the home page showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** Freemarker's `?new` lets a template instantiate any class that implements `TemplateModel`. `Execute` is such a class and its whole purpose is running a shell command, so a template author gets command execution with one line.
- **The key detail:** the vulnerability is not a bug in Freemarker — it is a documented feature that becomes dangerous the moment untrusted users can edit templates. The documentation itself flags `new()` as a security concern.

## Lessons Learned

- Letting users edit templates is equivalent to letting them run code — only trusted users should ever have that power, and even then the engine should be locked down.
- When the engine is unknown, an error message from invalid syntax usually names it; from there the engine's own documentation is the exploit manual.
- Sandboxing or removing dangerous built-ins (like `new()`) is essential if templates must be user-editable.
