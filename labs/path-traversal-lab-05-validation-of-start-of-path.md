# File path traversal, validation of start of path

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Path Traversal
**Difficulty:** Practitioner
**Solved:** 21 November 2026

## Objective

Read /etc/passwd even though the filename must start with the images folder path, by starting there and walking back up.

## Method

1. Request the image with: `GET /image?filename=/var/www/images/../../../etc/passwd`.

## Result

The server returns `/etc/passwd`. Lab solved.

## Why it works

The check only looks at the start of the path — and the payload does start with `/var/www/images/`, so it passes. The `../` sequences after it then climb back out to the root and open `/etc/passwd`.

## Lessons learned

- A prefix check says nothing about where the path ends up.
- Only the final canonical path matters — check that, not the first characters.

## Mitigation

Resolve the requested path to its canonical form and verify the whole path stays inside the base directory.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
