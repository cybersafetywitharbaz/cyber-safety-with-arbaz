# TryHackMe — Linux Fundamentals Part 1

| | |
| --- | --- |
| **Platform** | TryHackMe |
| **Room** | Linux Fundamentals Part 1 |
| **Path** | Pre Security |
| **Status** | ✅ Completed |
| **Profile** | [arbazchuahan](https://tryhackme.com/p/arbazchuahan) |
| **Context** | First Linux room in the Pre Security path; foundation for everything that follows (SOC work, penetration testing, Hack The Box). |

## Overview

Linux runs most of the servers, security tools, and SOC infrastructure a
cyber security professional will touch. This room is the starting point: it
teaches how to move around a Linux machine, work with files, and use the
terminal instead of a graphical interface. It was completed before
Linux Fundamentals Part 2, which continues from here.

## What I learned

- **How Linux is organised** — the file system starts at root (`/`), with
  standard directories for system files, user home folders, and configuration.
- **Navigating the terminal** — moving between directories, seeing where you
  are, and listing what a directory contains.
- **Working with files** — creating, viewing, copying, moving, renaming, and
  deleting files and directories from the command line.
- **Searching for files** — finding files by name and content instead of
  browsing by hand.
- **Shell operators** — chaining commands together (`&&`, `;`, `|`, `>`, `>>`)
  so the output of one command becomes the input of the next.
- **Getting help** — using built-in manual pages and `--help` instead of
  guessing flags.

## Key commands practised

| Command | What it does |
| --- | --- |
| `pwd` | Shows the current directory |
| `ls` | Lists files and directories |
| `cd` | Changes directory |
| `cat` | Displays a file's contents |
| `touch` | Creates an empty file |
| `mkdir` | Creates a directory |
| `cp` / `mv` | Copies / moves (renames) files |
| `rm` | Deletes files and directories |
| `echo` | Prints text, or writes it to a file |
| `find` / `grep` | Searches for files / searches inside files |
| `man` | Opens the manual page for a command |

## Why it matters

Almost every security tool — Nmap, Burp Suite, Metasploit, SIEM agents,
log analysis — runs on Linux or is driven from a Linux terminal. A SOC
analyst reads logs, hunts through file systems, and runs triage commands on
Linux hosts every day. These fundamentals are also the base for the
TryHackMe Linux Fundamentals Part 2 and 3 rooms and for Hack The Box
Academy Linux modules.

## What is next

- **Linux Fundamentals Part 2** — users, permissions, processes, and
  services (in progress).
- **Linux Fundamentals Part 3** — then the Web Fundamentals rooms in the
  Pre Security path.
