# Business Logic Lab 1 — Excessive Trust in Client-Side Controls

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Apprentice
**Solved:** 24 November 2026
**Lab instance:** `0aba008a04b6bd6880c2a32b001d00b9.web-security-academy.net`

## Objective

Buy the Lightweight "l33t" Leather Jacket ($1,337.00) using only the $100.00 store credit on the test account — by abusing the shop's trust in a price value supplied by the client.

## Method

1. **Log in** with the lab's test account `wiener:peter`, taking the `csrf` token from the login page.
2. **Add the jacket to the cart** with `POST /cart` — `productId=1`, `quantity=1`. The request carries a hidden `price` parameter (in cents). The shop normally sends `price=133700` ($1,337.00).
3. **Tamper with the price:** send the same `POST /cart` with `price=100` (i.e. $1.00). The server accepts the client-supplied price without checking it against its own catalogue.
4. **Check out:** `POST /cart/checkout` with the cart page's `csrf` token. The cart total is $1.00, which the $100.00 store credit covers.

The solve was done with a Python `requests` script (login → tampered add-to-cart → checkout), then verified as "Solved" in the browser.

## Result

✅ **Solved.** The lab confirmed: "Congratulations, you solved the lab!" — jacket purchased for $1.00 instead of $1,337.00.

## Why it works

The server trusted a price that came from the client. Anything the browser sends — hidden form fields, prices, quantities, roles — is attacker-controlled input. A hidden field is not a security control.

## Mitigation

- Never trust prices (or any business-critical value) sent by the client. Look the price up server-side from the product database on every request.
- Recompute the cart total server-side from trusted prices before charging or checking out.
- Validate every parameter server-side, even ones the UI never lets a normal user change.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
