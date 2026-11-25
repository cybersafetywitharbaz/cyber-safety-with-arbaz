# File path traversal, traversal sequences stripped non-recursively

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** Path Traversal
**Difficulty:** Practitioner
**Solved:** 21 November 2026

## Objective

Read /etc/passwd even though `../` is stripped, by nesting the sequence so the strip itself rebuilds it.

## Method

1. Request the image with a nested sequence: `GET /image?filename=....//....//....//etc/passwd`.

## Result

The server returns `/etc/passwd`. Lab solved.

## Why it works

The application strips `../` once. Each `....//` contains `../` hidden inside it — after one strip, `....//` becomes `../`, so the traversal reappears from the stripped text.

## Lessons learned

- A single non-recursive strip can be defeated by nesting the banned sequence inside itself.
- Stripping is the wrong approach — reject bad input instead of trying to clean it.

## Mitigation

Reject input containing traversal sequences instead of stripping them. Verify the canonical path stays inside the base directory.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
