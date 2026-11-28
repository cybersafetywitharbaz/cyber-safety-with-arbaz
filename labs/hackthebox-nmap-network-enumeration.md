# HTB Academy — Network Enumeration With Nmap (50%)

**Platform:** Hack The Box Academy (free Tier 0 module)
**Module:** Network Enumeration With Nmap
**Sections completed:** Host and Port Scanning, Service Enumeration, Nmap Scripting Engine
**Vulnerability Type:** N/A — enumeration / reconnaissance (no vulnerability exploited; information gathering on an authorised target)
**Difficulty:** Easy (Tier 0)
**Progress:** 25% → **50%** on 28 November 2026
**Account:** Arbaz Ahmed Naveed (@cybersafetywitharbaz)

> No screenshots available from this run — the module was solved from a
> PwnBox terminal during browser task 49aec676 (28 Nov 2026, 2h16m).

## Task

Enumerate the module target with Nmap: find open ports and services,
identify the host, grab service banners, and use Nmap Scripting Engine
(NSE) scripts to find hidden web content and flags.

## Solution

### Host and Port Scanning

Target: `10.129.62.37`.

- Q1 answer was already known from the earlier session: **7** (open
  ports found).
- Q2 — hostname: `nmap -sV` Service Info on the target reported the
  hostname **NIX-NMAP-DEFAULT**. Answer submitted: Success.

### Service Enumeration

Full-port scan of the target:

```bash
nmap -p- 10.129.62.37
```

- **7 open ports**, highest open port **31337**.
- Saved the scan as XML and converted it to a readable HTML report:

  ```bash
  xsltproc target.xml -o target.html
  ```

- Connected to the highest port with netcat and read its banner:

  ```bash
  nc 10.129.62.37 31337
  # banner: HTB{pr0F7pDv3r510nb4nn3r}
  ```

  **Answer submitted: Success.**

### Nmap Scripting Engine (NSE)

Ran web-enumeration scripts against the target's web service:

```bash
nmap --script http-enum,http-methods,http-robots.txt <target>
```

- The scripts found `/robots.txt`, which contained the flag:

  ```
  HTB{873nniuc71bu6usbs1i96as6dsv26}
  ```

  **Answer submitted: Success.**

- Note: the first attempt submitted the banner flag from the 31337
  service (`HTB{pr0F7pDv3r510nb4nn3r}`) for this question and it was
  **Incorrect** — the module hint pointed at the web servers, and the
  correct flag was the one in `/robots.txt`. Each question has its own
  flag; a flag from one service does not answer another section's
  question.

## Practical Explanation

- **`-p-` scans all 65,535 ports.** A default Nmap scan only checks
  the top 1,000 ports — the interesting service here (31337) only
  shows up with a full scan. Always confirm the port count with `-p-`
  when a question asks "how many ports are open".
- **Banners leak information.** A `nc` connection to an open port
  often prints a banner with the service name, version — and in
  training labs, a flag. Banner grabbing is the simplest form of
  service enumeration and it costs one command.
- **`-sV` gives more than versions.** Service detection also reports
  the target's hostname (here `NIX-NMAP-DEFAULT` in the Service Info
  line) — read the whole output, not just the port table.
- **NSE scripts automate the boring parts.** `http-enum`,
  `http-methods` and `http-robots.txt` check common web paths and
  files in seconds; `/robots.txt` is a classic place to hide (or
  accidentally expose) sensitive paths and flags.
- **`xsltproc` turns Nmap XML into a readable report** — useful when
  you need to review or share a full scan result.

## Lessons Learned

- One flag per question: don't reuse a flag found in one section for
  another — read the hint and check what the question actually asks.
- High ports (31337 and above) are easy to miss; `-p-` is not
  optional when the question is about completeness.
- `robots.txt` is public by design — never put secrets in it, and
  always check it during web enumeration.
- Save scan output (`-oA` / `-oX`) so you can re-read it and convert
  it later instead of re-scanning.

## Status

Module at **50%** after the 28 Nov 2026 run (Host and Port Scanning,
Service Enumeration, Nmap Scripting Engine complete). Remaining
sections still open — continue with the next sections in a future
PwnBox session. No answers were submitted from this write-up
session — all answers were submitted during the live browser run.
