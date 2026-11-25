# Business Logic Lab 5 — Low-level logic flaw

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Practitioner
**Solved:** 24 November 2026

## Objective

Buy the Lightweight "l33t" Leather Jacket ($1,337.00) using only the $100.00 store credit, by overflowing the shop's 32-bit cart total so it wraps negative — then nudging it back up to a small positive amount the store credit can cover.

## Method

1. **Log in** with the lab's test account `wiener:peter` (store credit $100.00).
2. **Overflow the total:** add 99 jackets (productId=1, $1,337.00) 323 times, then 47 more — 32,024 jackets in total, worth $42,816,088. The cart total is stored as a 32-bit signed integer of cents, so $4,281,608,800 cents wraps past 2³² and the total becomes **−$133,584.96**.
3. **Fix the total back up:** add 99 jackets once — the total rises by $132,363.00 to **−$1,221.96**.
4. **Measure the cheapest product:** add 1 item of productId 14 and read the cart total — it moves from −$1,221.96 to −$1,220.36, so its real price is **$1.60**.
5. **Land inside the credit:** add productId 14 in batches of 99 (7 batches), then 70 more — 763 items × $1.60 = $1,220.80 — taking the total to **$0.44**.
6. **Check out** with the cart's CSRF token. The order goes through on the $100.00 store credit and the lab shows "Congratulations, you solved the lab!"

The solve was done with a free Python `requests` script (no Burp Pro): login → repeated cart adds → cart-total reads after each phase → checkout. Solver script: `ps-lab5-low-level-logic.py` (v7).

## Result

✅ **Solved.** The jacket was ordered for $0.44 of store credit, and the lab confirmed the solve (`order-confirmation?order-confirmed=true`).

## Why it works

The cart total is a 32-bit signed integer, so a large enough total wraps around into negative numbers — a low-level integer overflow. Once the total is negative, adding cheap items back in small, controlled steps lands it just above zero, inside the store credit. The shop never checks that the total makes sense at any point; it just trusts its own arithmetic.

## Lessons from this lab (worth remembering)

- **Product prices vary per lab instance.** On this instance the Eco Boat (productId=16) cost $86.87, not $1.83 as on the Lab 2 instance — so never hard-code a price from another lab.
- **Don't trust the shop homepage's prices.** The first `$` amount after a product link on the homepage belongs to the *next* product (parsed[id] = real[id+1]). Measure a product's real price instead: add 1 item and read how much the cart total moves.
- **The server charges its real price** no matter what price is sent in the add-to-cart POST — the price parameter is ignored.
- **Quantity is capped at 99 per request**, so big fix-ups must be sent in batches of 99.
- **Parse the cart table's "Total" row**, not the last `$` amount on the page — the $100.00 store credit appears first and looks like a total.
- **Do the fix-up math in integer cents**, not floating-point dollars, so the final total lands exactly where intended.

## Mitigation

- Store money totals in a 64-bit integer (or a decimal type), and reject any order whose total overflows, goes negative, or doesn't match a server-side recomputation.
- Recompute the cart total server-side from trusted product prices and quantities on every change and at checkout; never trust client-sent prices or quantities.
- Validate quantities server-side (positive, and within a sane per-request maximum) and clamp the total at $0.00.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
