# TryHackMe — Progress (updated 26 November 2026)

**Platform:** TryHackMe (free rooms only — no Premium)
**Profile:** arbazchuahan (joined April 2026)
**Level:** 34 — Silver league — Top 35%
**Questions answered:** 252 — **Streak:** 3 days
**Rooms completed at 100%:** 42

## Rooms completed (100%)

### First 13 rooms
- Linux Modules
- SOC Fundamentals
- Offensive Security Intro
- Inside a Computer System
- Defensive Security Intro
- Defensive Security Intro (second room of the same name in the path)
- Careers in Cyber
- Overheard at Breakfast
- Linux Fundamentals Pt 1
- Operating Systems: Introduction
- Search Skills
- Windows Fundamentals 1
- Windows Fundamentals 2

### Next 21 rooms (24 November 2026 sessions)
- Starting Out In Cyber Sec
- Junior Security Analyst Intro
- Intro to LAN
- Introductory Networking (Task 7 whois checks finished 24 Nov ~21:23 PKT)
- HTTP in Detail — badge earned: **Webbed**
- Pentesting Fundamentals
- DNS in Detail
- Learning Cyber Security
- Web Application Security
- Cryptography Basics
- Web Application Basics
- Training Impact on Teams — badge earned: **Cyber Ready**
- OWASP Top 10 2025: IAAA Failures
- Security Principles
- Data Representation
- Data Encoding
- The CIA Triad
- Cryptography Concepts
- Introduction to SIEM
- CyberChef: The Basics
- Windows Basics (completed via the room's Windows Server 2019 lab machine, terminated after)

**Paths:** Pre Security path 43% complete. Cyber Security 101 path in progress.

## Detailed write-ups (already on GitHub)
- `labs/tryhackme-linux-fundamentals-part-1.md`
- `labs/tryhackme-soc-fundamentals.md`

## Blocked / not done (free-plan limits)
- **Windows Command Line** — 9%, Task 1 done (cmd.exe). Remaining tasks need AttackBox + lab machine running together, but the Free plan allows only 1 active machine at a time. (Can be done from a personal computer with OpenVPN + SSH.)
- **Blue** (free room) — 0%. Needs AttackBox + lab machine together; free AttackBox is limited to 1 hour a day. Tested 24 Nov ~21:23, lab machine terminated.
- **OWASP Top 10 2025: Application Design Flaws** and **Networking Concepts Task 7** — need AttackBox + lab machine together (blocked on Free).
- **Premium-only rooms** (tested and skipped, not attempted): Linux Fundamentals Pt 2, Computer Types, What is Networking?, Client-Server Basics, Virtualisation Basics, Windows Fundamentals 3, Public Key Cryptography Basics, Hashing Basics, Digital Forensics Fundamentals, JavaScript Essentials, SQL Fundamentals.

### 2 more rooms (25 November 2026, round 4)
- **AI Models & Data** (AI Security path, Medium) — 100%, 7 tasks, 96 points. Model-card audit: 6 red flags in a simulated Hugging Face page (5/6 at correct severity — PII filtering High; undocumented base model, sparse evaluation, file-size discrepancy, custom licence Medium). Flag: THM{A_m0del_Stud3nt}
- **Prompt Engineering** (AI Security path, Easy) — 100%, 7 tasks. PromptSec agent challenge completed 45/40 (few-shot and chain-of-thought prompts 9–10/10). Flag: THM{Pr0mpt_3ng1neer}

### 2 more rooms (25 November 2026, round 3)
- **The Building Blocks of AI** (AI Security path) — 100%, 8 tasks, 104 points, NEURON-1 practical 100% accuracy
- **Tutorial** — 100%, 1 task (free AttackBox + lab machine, flag{connection_verified}; both machines terminated after)
- NOTE: AI Security Threats started (26%) but abandoned — the room's in-room agent exercise kept tripping a safety check, so it was left unfinished rather than forced.

### 2 more rooms (25 November 2026, round 2)
- **AI Security Path Ticketing Event** — 100% (info room, 1 task)
- **Security Engineer Intro** (Security Engineer path) — 100%, 7 tasks, badge earned: **Introduction to Security Engineering**. Included an in-browser VAPT/audit simulation (flag THM{S3CUR1TY_3NG1N33R5_R0CK})

### 2 more rooms (25 November 2026)
- **SOC Role in Blue Team** (SOC Level 1 path) — 100%, 6 tasks, 56 points. SOC roles, CISO, Blue Team, CIRT, MSSP, SOC L1/L2 duties; final challenge matching analysts to incidents. Flag: THM{trysecureme_is_secured!}
- **Defensive Security Intro** (Jr Penetration Tester path — a different room from the earlier same-named one) — 100%. FakeBank attack investigation: source IP 32.122.195.63, malicious URL https://fakebank.thm/admin, Block/Slow/Protect deployed. Flag: THM{threat_stopped}
- **Pickle Rick** — 100% (26 Nov 2026, 90 points). Free room outside the exhausted paths. nmap (22 SSH, 80 HTTP) → gobuster found /login.php + /portal.php → username `R1ckRul3s` in an HTML comment, password `Wubbalubbadubdub` in robots.txt → login → Command Panel RCE as www-data → ingredients: `mr. meeseek hair` (Sup3rS3cretPickl3Ingred.txt), `1 jerry tear` (/home/rick/second ingredients), `fleeb juice` (/root/3rd.txt via passwordless sudo). Full write-up: `labs/tryhackme-pickle-rick.md`.

## Where things stand
Free rooms in the Pre Security + Cyber Security 101 paths are **exhausted** (verified 24 Nov ~22:58 PKT) — every remaining room in those paths is Premium-only or needs AttackBox + lab machine together. SOC Level 1 path moved from 2% to 3% (25 Nov). Next rooms checked and Premium-only: Humans as Attack Vectors (SOC Level 1), Putting it all together (Web Fundamentals), Dive Into Pentesting (Jr Penetration Tester). More free rooms are being tried in other paths (Jr Penetration Tester, Security Engineer).

---

*TryHackMe rooms are TryHackMe's own authorised training environments, solved legally for learning.*
