# SQL injection with filter bypass via XML encoding

**Platform:** PortSwigger Web Security Academy (free lab)
**Vulnerability type:** SQL Injection
**Difficulty:** Apprentice
**Solved:** Solved on this account before 15 November 2026

## Objective

Bypass a filter that blocks SQL keywords by encoding part of the payload, so the stock-check query still runs the injected SELECT.

## Method

1. Send a stock-check request in XML, with the keyword encoded — the `S` in SELECT written as the XML escape `&#x53;`:
2. ```xml
3. <stockCheck>
4.     <productId>123</productId>
5.     <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
6. </stockCheck>
7. ```

## Result

The injected query runs and the lab shows solved (this lab was already marked solved on this account).

## Why it works

The filter looks for the keyword `SELECT` in the raw request, but the application XML-decodes the input before putting it into the query — so `&#x53;ELECT` passes the filter and becomes `SELECT` at the database.

## Lessons learned

- Filters that inspect the raw input fail when the application decodes or transforms input afterwards.
- Encoding tricks only work because the check happens before the decode — the fix is to check after, or better, not to concatenate at all.

## Mitigation

Use parameterized queries. Validate and filter input after all decoding, never before.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
