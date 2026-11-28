# Hack The Box Academy — Progress (checked 28 November 2026)

**Platform:** Hack The Box Academy (free Tier 0 modules only — no purchases)
**Profile:** @cybersafetywitharbaz — https://profile.hackthebox.com/profile/019d69b3-3719-727d-ab6e-fb4ea34e5a95
**Rank:** Beginner — **Level:** 14 (125 cubes) — **Grade:** 3/3
**Total XP (last year):** 1605 — **Active:** 5 weeks
**Academy modules completed:** 10 / 413
**Labs:** 0 / 283 — **Paths / Missions / Exams:** none completed

## Badges earned (9)
- **Philomath** — Learning Process module
- **Academician** — Introduction to Academy module
- **Our favorite seabird** — Linux Fundamentals module
- **Unwavering User** — first weekly streak
- **Your request is my demand** — Web Requests module
- **Fuzzing guru** — Web Fuzzing module
- **Hunt the bug** — Bug Bounty Hunting Process module
- **Humanoid** — Brief Intro to Hardware Attacks module
- **JavaScript Deobfuscation module badge** — earned 28 Nov 2026 (badge name not recorded in the run notes — verify on the Academy dashboard)

## Modules completed (10)
- Learning Process
- Introduction to Academy
- Linux Fundamentals
- Introduction to Web Applications — 100%, 17/17 sections (completed 24 Nov 2026)
- Web Requests — completed 24 Nov 2026; badge earned: **Your request is my demand**
- Web Fuzzing — 100% (completed 25 Nov 2026); Recursive Fuzzing flag solved; badge earned: **Fuzzing guru**
- Bug Bounty Hunting Process — 100% (completed 25 Nov 2026, theory-only, 6 sections); badge earned: **Hunt the bug**
- Brief Intro to Hardware Attacks — 100% (completed 25 Nov 2026, theory-only, 8/8 sections); badge earned: **Humanoid**
- JavaScript Deobfuscation — **100%, 11/11 sections (completed 28 Nov 2026)**; Skills Assessment solved via PwnBox: per-spawn hex key decoded and POSTed to /keys.php, flag `HTB{r34dy_70_h4ck_my_w4y_1n_2_HTB}`; **+15 XP**; badge earned. Write-up: `labs/hackthebox-javascript-deobfuscation-skills-assessment.md`
- One further completed module is counted in the 10/413 total but its name is not recorded in the progress notes — to be verified on the Academy dashboard before it is listed here.

All completed modules are free Tier 0. No purchases made, no cubes spent.

## In progress
- **Brief Intro to Hardware Attacks (Mini, Tier 0) — COMPLETED 25 Nov 2026 (100%, 8/8 sections)** — answers: BlueSmacking, 8 BlueBorne zero-days, Timing Attacks (side-channel), speculative execution (Spectre), out-of-order execution (Meltdown). Badge earned: **Humanoid**. No PwnBox needed, 122 cubes unchanged.
- **Introduction to Bash Scripting — 90%** — last question SOLVED 27 Nov 2026: Flow Control – Loops salt = **34071**, verified by running the module's exact exercise script in the shell (decrypt → flag `HTBL00p5r0x`; negative control: 34070 and 34072 both give 'bad decrypt'). Why the 26+ earlier values failed: the script computes salt with `echo $var | wc -c` (= ${#var} + 1, the newline echo adds) — 34070 is wrong, 34071 is right. **Answer submission still pending — needs a signed-in browser session on HTB Academy.** Write-up: `labs/hackthebox-bash-scripting-flow-control-loops.md`. Once submitted, module → 100% (11/413).
- **JavaScript Deobfuscation — COMPLETED 28 Nov 2026 (100%, 11/11 sections)** — Skills Assessment solved in PwnBox: per-spawn hex key `4150495f70336e5f37333537316e365f31355f66756e` decoded to `API_p3n_73571n6_15_fun`, POSTed to /keys.php → flag `HTB{r34dy_70_h4ck_my_w4y_1n_2_HTB}`. Badge earned, +15 XP. Write-up: `labs/hackthebox-javascript-deobfuscation-skills-assessment.md`.
- **SQL Injection Fundamentals — 29.41% (5/17)** — theory done (Intro to SQL Injections, Mitigating SQL Injection). Remaining MySQL, interactive SQLi, Exploitation and Skills Assessment sections need PwnBox. **Not attempted on 28 Nov — time + rate limits.**
- **File Inclusion — 18.18% (2/11)** — Local File Inclusion complete (28 Nov 2026): `/etc/passwd` via LFI gave `barry`; `/usr/share/flags/flag.txt` via LFI gave `HTB{n3v3r_tru$t_u$3r_!nput}`. PHP Filters (Section 4/11) started: gobuster found `/configure.php` (302), but `php://filter/convert.base64-encode/resource=configure.php` returned only the homepage — question still open. Write-up: `labs/hackthebox-file-inclusion-lfi.md`.
- **Network Enum With Nmap — 50%** — Host and Port Scanning, Service Enumeration and Nmap Scripting Engine complete (28 Nov 2026): hostname NIX-NMAP-DEFAULT; 7 open ports, highest 31337, banner `HTB{pr0F7pDv3r510nb4nn3r}`; `/robots.txt` flag `HTB{873nniuc71bu6usbs1i96as6dsv26}`. Write-up: `labs/hackthebox-nmap-network-enumeration.md`.

## Run notes — 28 November 2026 (browser task 49aec676, 2h16m)
- Free PwnBox spawned (UK, `htb-cncgkztalb.htb-cloud.com`); left **Online** at end of run with 30–40 min remaining — expires on its own. File Inclusion target (`154.57.164.74:37607`) left running — expires on its own.
- **6 HTB rate-limit blocks** during the run, each cleared after a 5–10 min wait. No cubes spent, nothing bought; cubes 122 → 125 from XP rewards.
- Bash Scripting salt question is **Arbaz's step** — do NOT submit any salt value.

## Blockers (as of 28 November 2026)
- Free PwnBox: 1 spawn per day. The daily spawn was used on 28 Nov — interactive sections resume after the next daily reset.
- HTB Academy rate-limited the cloud browser during the 28 Nov run ("Take it Slow") — 6 blocks, each cleared after 5–10 min waits. The device browser (own Chrome, Google sign-in) reaches the dashboard — it loads blank at first, then renders after ~30s waits.
- Bash Scripting salt question: SOLVED 27 Nov 2026 (34071, decrypt-verified) — only the answer submission is pending (Arbaz's step, browser sign-in).

## Next (free Tier 0, in order)
1. Bash Scripting — Arbaz submits the verified salt (34071) himself via browser, then module → 100% (11/413).
2. File Inclusion — PHP Filters (Section 4/11) question still open: retry the `php://filter` read with a different file/approach in a fresh PwnBox session.
3. Network Enum With Nmap — remaining sections after 50%.
4. SQL Injection Fundamentals — MySQL, interactive SQLi, Exploitation, Skills Assessment (29.41% now).

---

*Hack The Box Academy modules are HTB's own authorised training environments, completed legally for learning. No machines owned yet — owned machines/challenges are not shown on the public profile.*
