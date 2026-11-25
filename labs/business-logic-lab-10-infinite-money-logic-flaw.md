# Business Logic Lab 10 — Infinite money logic flaw

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Practitioner
**Solved:** 25 November 2026

## Objective

Buy the Lightweight "l33t" Leather Jacket ($1,337.00) by exploiting the shop's gift-card logic: a $10 gift card bought with the `SIGNUP30` coupon costs only $7.00, but redeeming it gives back the full $10.00 in store credit — a net gain of $3.00 every cycle. Repeat the cycle until there is enough credit for the jacket.

## Method

1. **Log in** with the lab's test account `wiener:peter` (starting store credit $100.00).
2. **Get the coupon:** sign up for the newsletter (`POST /sign-up` with an email address) to obtain the coupon code `SIGNUP30` (30% off).
3. **One cycle, in one session** (the cart is session-scoped):
   - `POST /cart` — add the Gift Card (productId=2, $10.00) to the cart.
   - `POST /cart/coupon` — apply `SIGNUP30` (cart total $10.00 → $7.00).
   - `POST /cart/checkout` — pay $7.00 from store credit; the order-confirmation page lists the new gift card code.
   - `POST /gift-card` — redeem the code (`gift-card` parameter + CSRF token from `/my-account`) for $10.00 store credit. Net gain per cycle: **+$3.00**.
4. **Batch it:** buy **10 gift cards in a single order** (quantity=10, coupon 30% off the whole order: pay $70.00, redeem 10 codes for $100.00) — net gain **+$30.00 per order**. The gift card codes were read from the lab's **email client inbox** (one "Gift card" email per code: "Your gift card code is: …"), which works for any batch size.
5. **Repeat 40 orders** — credit grew $145.00 → $1,345.00 — then add the jacket (productId=1, $1,337.00) to the cart and check out. Final store credit: **$8.00** ($1,345.00 − $1,337.00).

The solve was done with a free Python `requests` script (no Burp Pro). Solver script: `ps-lab10-infinite-money-v2.py`.

## Result

✅ **Solved.** The jacket order-confirmation contains "Congratulations, you solved the lab!". The final credit of $8.00 confirms the arithmetic exactly: $1,345.00 of accumulated credit minus the $1,337.00 jacket.

## Why it works

The shop treats buying and redeeming a gift card as two independent, honest transactions — but their values don't match. The coupon discounts the *purchase* price ($10 → $7) while the *redemption* value stays at the full face value ($10). The application never checks that the money coming back is less than (or equal to) the money that went out, so every buy-and-redeem round prints $3.00 of free credit. Repeating a small, legitimate-looking profit is exactly how "infinite money" flaws work in real stores: the logic is correct for one transaction and broken across the pair.

## What went wrong first — and the recovery (worth remembering)

The first solver attempt (`ps-lab10-infinite-money.py`) **failed and drained the credit from $103.00 to $5.00** before dying on a checkout timeout. The bug: it parsed gift card codes from the **raw HTML** of the confirmation page with the regex `Code\s+([A-Za-z0-9]{8,})` — but the code sits inside a table (`<th>Code</th> … <td>CODE</td>`), so the regex matched **nothing**. Every order paid $7.00 and redeemed $0. The script never checked whether credit actually rose, so it kept draining.

The recovery: the shop **emails every gift card code** to the lab's email client inbox. All 14 lost codes were sitting there as "Gift card" emails — redeeming them restored the credit from $5.00 to $145.00, and the fixed solver (v2) reads codes from the inbox instead of the confirmation HTML.

Other dead ends tested on this instance (all verified, not assumed):
- **Coupon re-application does not stack here** — applying `SIGNUP30` four times left the total at $7.00 (unlike Lab 4's instance, where alternating coupons stacked).
- **`NEWCUST5` does not apply** — the total stayed $10.00 (wiener already has order history, so he is not a "new customer").
- **Negative quantities are rejected here** — adding a product with quantity=-1, or updating an existing line to -1, silently left the cart unchanged (unlike Lab 2's instance).
- **No registration on this lab** — `GET /register` returns 404, so a fresh account (and fresh $100 credit) is not an option.
- **Double-redeem is blocked** — redeeming the same code twice returns "Invalid gift card".

## Lessons from this lab (worth remembering)

- **Parse data from the representation you actually verified.** The code was visible in the *text* of the confirmation page but invisible to a regex run on the *raw HTML* — table markup sat between "Code" and the value. Always test the parser against the real response, not the rendered page.
- **Verify the money moved after every step.** A loop that never checks its own result will happily drain an account 14 times. Check the balance after each cycle and stop the moment it doesn't rise.
- **The email client is a second source of truth.** Anything the shop emails (gift card codes, confirmations) can be recovered there even when the web flow loses it.
- **Batching multiplies both profit and risk.** Ten cards per order turned 412 single cycles into 40 orders — but a bug in one batch costs ten times as much, so the per-order balance check matters even more.
- **The same shop behaves differently per lab instance.** Coupons stacking, negative quantities, and registration all worked on other Business Logic instances and failed on this one — never assume a trick carries over; test it on the live instance.

## Mitigation

- Make redemption value follow the *amount actually paid*, not the face value: a gift card bought at a discount should redeem for the discounted price (or the discount should not apply to gift cards at all).
- Track gift cards as liabilities with their purchase price recorded; redeeming must never return more than was paid for that specific card.
- Reconcile purchase and redemption as one transaction: if the redeemed value exceeds the purchase price, flag or block it.
- Exclude gift cards (and other stored-value products) from percentage coupons, or cap the coupon so the total can never go below the card's face value minus the discount logic.
- Add a per-order and per-account anomaly check: rapid repeated gift-card buy/redeem cycles are a classic infinite-money signature.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
