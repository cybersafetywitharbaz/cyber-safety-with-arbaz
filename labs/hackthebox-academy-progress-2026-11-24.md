# Hack The Box Academy — Progress (checked 25 November 2026)

**Platform:** Hack The Box Academy (free Tier 0 modules only — no purchases)
**Profile:** @cybersafetywitharbaz — https://profile.hackthebox.com/profile/019d69b3-3719-727d-ab6e-fb4ea34e5a95
**Rank:** Beginner — **Level:** 12 (122 cubes) — **Grade:** 3/3
**Total XP (last year):** 1605 — **Active:** 5 weeks
**Academy modules completed:** 9 / 413
**Labs:** 0 / 283 — **Paths / Missions / Exams:** none completed

## Badges earned (8)
- **Philomath** — Learning Process module
- **Academician** — Introduction to Academy module
- **Our favorite seabird** — Linux Fundamentals module
- **Unwavering User** — first weekly streak
- **Your request is my demand** — Web Requests module
- **Fuzzing guru** — Web Fuzzing module
- **Hunt the bug** — Bug Bounty Hunting Process module
- **Humanoid** — Brief Intro to Hardware Attacks module

## Modules completed (9)
- Learning Process
- Introduction to Academy
- Linux Fundamentals
- Introduction to Web Applications — 100%, 17/17 sections (completed 24 Nov 2026)
- Web Requests — completed 24 Nov 2026; badge earned: **Your request is my demand**
- Web Fuzzing — 100% (completed 25 Nov 2026); Recursive Fuzzing flag solved; badge earned: **Fuzzing guru**
- Bug Bounty Hunting Process — 100% (completed 25 Nov 2026, theory-only, 6 sections); badge earned: **Hunt the bug**
- Brief Intro to Hardware Attacks — 100% (completed 25 Nov 2026, theory-only, 8/8 sections); badge earned: **Humanoid**
- One further completed module is counted in the 9/413 total but its name is not recorded in the progress notes — to be verified on the Academy dashboard before it is listed here.

All completed modules are free Tier 0. No purchases made, no cubes spent.

## In progress
- **Brief Intro to Hardware Attacks (Mini, Tier 0) — COMPLETED 25 Nov 2026 (100%, 8/8 sections)** — answers: BlueSmacking, 8 BlueBorne zero-days, Timing Attacks (side-channel), speculative execution (Spectre), out-of-order execution (Meltdown). Badge earned: **Humanoid**. No PwnBox needed, 122 cubes unchanged.
- **Introduction to Bash Scripting — 90%** — last question SOLVED 27 Nov 2026: Flow Control – Loops salt = **34071**, verified by running the module's exact exercise script in the shell (decrypt → flag `HTBL00p5r0x`; negative control: 34070 and 34072 both give 'bad decrypt'). Why the 26+ earlier values failed: the script computes salt with `echo $var | wc -c` (= ${#var} + 1, the newline echo adds) — 34070 is wrong, 34071 is right. **Answer submission still pending — needs a signed-in browser session on HTB Academy.** Write-up: `labs/hackthebox-bash-scripting-flow-control-loops.md`. Once submitted, module → 100% (10/413).
- **JavaScript Deobfuscation — 90.91% (10/11)** — Q1–Q3 solved (api.min.js; HTB{j4v45cr1p7_3num3r4710n_15_k3y}; HTB{n3v3r_run_0bfu5c473d_c0d3!}). **Q4/Q5 remain — the key is PER-SPAWN: fetch from the live target via PwnBox, hex-decode, POST key=DECODED_KEY to /keys.php.** Needs a fresh PwnBox spawn (daily reset 27 Nov).
- **SQL Injection Fundamentals — 29.41% (5/17)** — theory done (Intro to SQL Injections, Mitigating SQL Injection). Remaining MySQL, interactive SQLi, Exploitation and Skills Assessment sections need PwnBox.
- **File Inclusion — 9.09% (1/11)** — Introduction (the only theory section) complete. 10 interactive sections need PwnBox.
- **Network Enum With Nmap — 25%** — in progress.

## Blockers (as of 25 November 2026)
- Free PwnBox: 1 spawn per day. The daily spawn was used up on 25 Nov (0/1 left) — interactive sections resume after the next daily reset (26 Nov).
- HTB Academy rate-limited / Cloudflare-challenged the cloud browser on 24–25 Nov ("Take it Slow", Turnstile). The device browser (own Chrome, Google sign-in) reaches the dashboard — it loads blank at first, then renders after ~30s waits.
- Bash Scripting salt question: SOLVED 27 Nov 2026 (34071, decrypt-verified) — only the answer submission is pending (browser sign-in).

## Next (free Tier 0, in order)
1. Spawn PwnBox (26 Nov) — Bash Scripting Loops salt, then the interactive sections of JavaScript Deobfuscation, SQL Injection Fundamentals, File Inclusion.
2. Bash Scripting — submit the verified salt (34071) via browser, then the interactive sections of JavaScript Deobfuscation (Q4/Q5, per-spawn key), SQL Injection Fundamentals, File Inclusion.
3. Network Enum With Nmap.

---

*Hack The Box Academy modules are HTB's own authorised training environments, completed legally for learning. No machines owned yet — owned machines/challenges are not shown on the public profile.*
