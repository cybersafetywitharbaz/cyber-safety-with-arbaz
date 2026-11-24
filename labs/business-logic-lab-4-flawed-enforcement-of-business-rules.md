# Business Logic Lab 4 — Flawed Enforcement of Business Rules

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Practitioner
**Solved:** 24 November 2026

## Objective

Buy the Lightweight "l33t" Leather Jacket ($1,337.00) for $0.00 by abusing the shop's coupon rules — the application lets the same coupons be applied repeatedly instead of enforcing them once per order.

## Method

1. **Log in** with the lab's test account `wiener:peter`.
2. **Get a coupon:** sign up for the newsletter with `arbaz@example.com` to receive the coupon code `SIGNUP30`.
3. **Add the jacket to the cart** (productId=1, $1,337.00).
4. **Apply coupons in turn, repeatedly:** alternate the two coupons — `NEWCUST5` (−$5.00) and `SIGNUP30` (−$401.10) — applying each one **4 times**. Each application deducts its full value again, with no check that the coupon was already used on this order.
5. **Place the order** once the cart total reaches **$0.00**.

The solve was done in the browser (login → newsletter signup → cart → repeated coupon applications → order), then verified as "Solved".

## Result

✅ **Solved.** The jacket was ordered for $0.00, and the lab confirmed the solve.

## Why it works

The coupon logic was enforced in the UI, not on the server: nothing stopped the same discount being applied over and over to the same order. Business rules that only exist in the interface are not rules at all — the server has to enforce them.

## Mitigation

- Enforce coupon rules server-side: one coupon per order (or an explicit, tracked limit), and record which coupons have already been applied to an order.
- Never let a discount drive the order total below zero; clamp the total at $0.00 and reject further discounts.
- Treat every discount as a server-side transaction with its own state, not as a client-side toggle.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
