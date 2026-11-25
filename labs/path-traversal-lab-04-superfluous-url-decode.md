# File path traversal, traversal sequences stripped with superfluous URL-decode

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Path Traversal
**Difficulty:** Practitioner
**Solved:** 21 November 2026

## Objective

Read /etc/passwd by double-encoding the traversal so it only becomes `../` after the application's extra decode step.

## Method

1. Request the image with a double-encoded path: `GET /image?filename=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd`.

## Result

The server returns `/etc/passwd`. Lab solved.

## Why it works

The application URL-decodes the filename more than once. The first decode turns `%252e` into `%2e`, the second turns `%2e` into `.` — so `../../../etc/passwd` only exists after the check has already passed.

## Lessons learned

- Decoding after validation means the validated text is not the text that gets used.
- Decode fully, exactly once, before any check — then verify the canonical path.

## Mitigation

URL-decode input completely, exactly once, before validating it. Then verify the canonical path stays inside the base directory.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
