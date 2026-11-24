# TryHackMe — SOC Fundamentals

| | |
| --- | --- |
| **Platform** | TryHackMe |
| **Room** | SOC Fundamentals |
| **Status** | ✅ Completed |
| **Profile** | [arbazchuahan](https://tryhackme.com/p/arbazchuahan) |
| **Context** | Introductory blue-team room; part of the early learning path toward a SOC Analyst role. |

## Overview

A Security Operations Centre (SOC) is the team that watches an
organisation's systems around the clock, detects attacks, and responds to
them. This room explains what a SOC is, who works in it, and how alerts
flow from detection to response. It is the conceptual foundation for the
later, hands-on SOC work (log analysis, SIEM, incident response).

## What I learned

- **What a SOC does** — continuous monitoring, detection, triage, and
  incident response, rather than one-off security checks.
- **SOC roles and tiers** — Tier 1 analysts triage alerts, Tier 2 analysts
  investigate in depth, Tier 3 analysts hunt threats and handle the hardest
  incidents.
- **SIEM** — Security Information and Event Management: one place where logs
  from servers, firewalls, and endpoints are collected, correlated, and
  turned into alerts.
- **Threat intelligence** — using knowledge about known attackers, malware,
  and indicators of compromise to recognise attacks faster.
- **Incident response lifecycle** — preparation, detection and analysis,
  containment, eradication, recovery, and lessons learned.
- **Common SOC tools** — log sources, ticketing/queueing of alerts, and the
  difference between true positives, false positives, and benign activity.

## Why it matters

My goal is a SOC Analyst role. This room maps the job itself: what a shift
looks like, what "triage an alert" actually means, and which skills to build
next (Linux, networking, log analysis). It connects directly to the other
completed rooms — Linux Fundamentals for working on the systems, and the
web-security labs for understanding what the attacks in the alerts look like.

## What is next

- **TryHackMe — SOC Level 1 path** (hands-on SIEM and log-analysis rooms).
- **Cisco Networking Academy — Network Defense** (free course).
- **Microsoft Learn — SC-900 Security Fundamentals** (free learning path).
- Continue PortSwigger web-security labs, so the attacks behind SOC alerts
  are understood from the attacker's side as well as the defender's.
