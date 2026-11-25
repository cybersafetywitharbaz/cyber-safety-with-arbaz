# File path traversal, simple case

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Path Traversal
**Difficulty:** Apprentice
**Solved:** 21 November 2026

## Objective

Read a system file outside the web folder by walking up the directory tree in the image filename parameter.

## Method

1. Request a product image and change the filename parameter: `GET /image?filename=../../../etc/passwd`.

## Result

The server returns the contents of `/etc/passwd`. Lab solved.

## Why it works

Each `../` moves one directory up. Three of them climb out of the images folder to the filesystem root, and the server opens `/etc/passwd` because it never checked the filename.

## Lessons learned

- Path traversal is the file-system version of injection: user input becomes a path.
- The fix is to never trust the filename — resolve it and check it stays inside the allowed folder.

## Mitigation

Resolve the requested path to its canonical form and verify it stays inside the base directory. Use an allowlist of permitted files.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
