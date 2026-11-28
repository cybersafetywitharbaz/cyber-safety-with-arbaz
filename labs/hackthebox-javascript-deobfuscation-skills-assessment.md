# HTB Academy — JavaScript Deobfuscation: Skills Assessment (module completed)

**Platform:** Hack The Box Academy (free Tier 0 module)
**Module:** JavaScript Deobfuscation
**Section:** Skills Assessment (final section, 11/11)
**Vulnerability Type:** Information disclosure via client-side code — hardcoded secrets and keys in obfuscated JavaScript
**Difficulty:** Easy (Tier 0)
**Solved:** 28 November 2026 (Q4 + Q5, module completed 100%)
**Account:** Arbaz Ahmed Naveed (@cybersafetywitharbaz)

> No screenshots available from this run — the module was solved from a
> PwnBox terminal during browser task 49aec676 (28 Nov 2026, 2h16m).

## Task

The Skills Assessment gives a live target running an obfuscated
JavaScript application. The JavaScript (api.min.js) talks to a
`/keys.php` endpoint. Find the per-spawn key, decode it, and use it to
retrieve the flag. The earlier sections were already solved: Q1–Q3
(api.min.js; `HTB{j4v45cr1p7_3num3r4710n_15_k3y}`;
`HTB{n3v3r_run_0bfu5c473d_c0d3!}`). Q4 (the key) and Q5 (the flag)
were the two remaining questions.

## Solution

Spawned a free PwnBox (UK, `htb-cncgkztalb.htb-cloud.com`). Module
target: `154.57.164.75:32897`.

1. Fetched the application's JavaScript from the live target and ran
   it with node:

   ```bash
   curl http://154.57.164.75:32897/api.min.js -o api.min.js   # 719 bytes
   node api.min.js
   # console output: HTB{j4v45cr1p7_3num3r4710n_15_k3y}
   ```

2. Read the deobfuscated logic: the script POSTs to `/keys.php`.
   Replicated that POST from the PwnBox. The endpoint returned a
   per-spawn hex-encoded key:

   ```
   4150495f70336e5f37333537316e365f31355f66756e
   ```

3. Hex-decoded the key:

   ```
   4150495f70336e5f37333537316e365f31355f66756e
   → API_p3n_73571n6_15_fun
   ```

4. POSTed the decoded key back to the endpoint:

   ```bash
   curl -X POST http://154.57.164.75:32897/keys.php -d "key=API_p3n_73571n6_15_fun"
   # → HTB{r34dy_70_h4ck_my_w4y_1n_2_HTB}
   ```

**Answers: Q4 = the per-spawn hex key
`4150495f70336e5f37333537316e365f31355f66756e` — Success. Q5 = flag
`HTB{r34dy_70_h4ck_my_w4y_1n_2_HTB}` — Success.**

Module result: **JavaScript Deobfuscation 100% (11/11 sections)**,
badge earned, **+15 XP**. Hack The Box Academy total moved to
**10 / 413 modules completed, Level 14, 125 cubes**.

## Practical Explanation

- **Never run obfuscated JavaScript blind.** Q3's flag
  (`HTB{n3v3r_run_0bfu5c473d_c0d3!}`) says it plainly — but the safe
  way to learn what the code does is to read it / run it in a
  disposable environment (here, the PwnBox), not on your own machine.
  The console output alone gave the Q1 flag; the network calls it made
  gave the rest.
- **The key is PER-SPAWN.** The hex key returned by `/keys.php`
  belongs to that one live target instance. Copying a key from a
  write-up, a forum post, or an earlier spawn will not work — every
  spawn generates a fresh one. The correct flow is always: fetch from
  the live target → decode → POST the decoded value back.
- **Hex encoding is not encryption.** The "key" was just hex of a
  readable string (`API_p3n_73571n6_15_fun`). `xxd -r -p`, `printf`,
  or any hex decoder reveals it in one step. When a value looks like
  long hex, decode it before assuming it is a hash.
- **Replicate the request, don't guess it.** The breakthrough was
  copying the exact POST the JavaScript itself makes (same endpoint,
  same parameter name `key`), first with no key (to get the hex key)
  and then with the decoded key (to get the flag). Browser DevTools
  Network tab shows the same calls if you prefer the browser.

## Lessons Learned

- Client-side code is never secret: anything the browser must run, an
  attacker can read. Secrets, keys and flags must live server-side.
- Obfuscation slows reading; it does not hide behaviour. Run the code
  in a sandbox and watch its network traffic — that is usually faster
  than deobfuscating by hand.
- Per-spawn / per-instance values must be fetched live every time;
  hardcoding a value from a previous run is the classic trap in these
  assessments.
- Hex-decode first, ask questions later: `echo <hex> | xxd -r -p`
  turns an opaque blob into a readable string in seconds.

## Status

Module **COMPLETED 28 Nov 2026 — 100%, 11/11 sections, badge earned,
+15 XP**. Nothing further to do in this module. No answers were
submitted from this write-up session — Q4/Q5 were submitted during
the live browser run.
