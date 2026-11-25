# File path traversal, traversal sequences blocked with absolute path bypass

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Path Traversal
**Difficulty:** Apprentice
**Solved:** 21 November 2026

## Objective

Read /etc/passwd even though `../` sequences are blocked, by giving the full absolute path instead.

## Method

1. Request the image with an absolute path: `GET /image?filename=/etc/passwd`.

## Result

The server returns `/etc/passwd`. Lab solved.

## Why it works

The defence only blocks `../` sequences — it never checks absolute paths. Giving the full path from the root skips the blocked pattern entirely.

## Lessons learned

- Blocking one pattern is not validation — the attacker just uses a different shape of the same input.
- Defences must check the final resolved path, not a list of forbidden substrings.

## Mitigation

Resolve the requested path to its canonical form and verify it stays inside the base directory.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
