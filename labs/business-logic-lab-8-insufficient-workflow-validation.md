# Business Logic Lab 8 — Insufficient workflow validation

**Platform:** PortSwigger Web Security Academy (free lab)
**Level:** Practitioner
**Solved:** 25 November 2026

## Objective

Buy the "Lightweight 'l33t' Leather Jacket" ($1337.00) even though the store credit is only $100.00, by exploiting the purchasing workflow — the application assumes the checkout steps always happen in the intended order, and never checks that they did.

## Method

1. **Log in** with the lab's test account `wiener:peter` (store credit $100.00).
2. **Find a cheap item you can afford.** Product prices vary per lab instance, so the price was *measured*, not trusted from the homepage (the homepage price parse is shifted by one — see Lessons). Probing 1 item at a time and reading the cart-total delta: productId 6 (Inflatable Holiday Home) costs **$0.49** and productId 5 (Giant Pillow Thing) costs **$1.95** on this instance.
3. **Buy the cheap items.** The cart held the two probed items (Giant Pillow Thing ×2 + Inflatable Holiday Home ×1) for a total of **$4.39**, and `POST /cart/checkout` (with the cart's CSRF token) completed the order. The checkout returned HTTP 303 redirecting to `GET /cart/order-confirmation?order-confirmed=true` — this is the order-confirmation request. Store credit after this order: **$95.61** (the $4.39 was properly deducted).
4. **Add the leather jacket to the cart.** `POST /cart` with productId=1, quantity=1 — the cart total became **$1337.00**, far above the remaining $95.61 credit.
5. **Replay the order-confirmation request.** Send the same `GET /cart/order-confirmation?order-confirmed=true` again, with the jacket in the cart. The server completes the order without ever checking the cart total against the store credit — and the lab shows "Congratulations, you solved the lab!"

The solve was done with a free Python `requests` script (no Burp Pro): login → probe-and-measure the cheap item → cheap checkout (capture the confirmation URL) → add jacket → replay confirmation. Solver script: `ps-lab8-insufficient-workflow-validation.py`.

## Result

✅ **Solved.** The order confirmation page shows "LAB Solved" and "Congratulations, you solved the lab!" with the order listed as Lightweight "l33t" Leather Jacket ×1, Total: $1337.00 — while the store credit is still **$95.61**. The $1337 was never deducted. (The first, legitimate order shows "LAB Not solved" — only the replayed confirmation solved it.)

## Why it works

The purchasing workflow is a sequence of steps the application *assumes* will happen in order: add items → checkout (which checks the credit) → order confirmation. But the confirmation step does not re-validate anything — it just completes whatever is in the cart. The checkout's credit check and the confirmation's order completion are not tied together, so replaying the confirmation request with a different (much more expensive) cart skips the payment check entirely. The application trusts the *sequence of events* instead of validating the *state* at each step: "if the user reached the confirmation page, they must have paid." Replaying the request breaks that assumption.

## Lessons from this lab (worth remembering)

- **Every step must validate the state, not assume the previous step did.** The confirmation endpoint should re-check the cart total against the store credit (and that the order was actually paid for) before completing the order.
- **Replaying a request is a real attack.** A request that was legitimate once (the first confirmation) becomes an exploit when sent again with different state — replay protection (one-time order tokens, nonces) matters for state-changing requests.
- **Measure prices, don't trust the page.** On this instance the homepage price parse was shifted by one (the price shown after productId N really belonged to product N+1), and the empty-cart "total" read was actually the $100.00 store-credit line. Real prices were measured by adding 1 item and reading the cart-total delta.
- **Note the exact confirmation parameter:** the official solution text says `order-confirmation=true`, but this instance's real redirect used `order-confirmed=true` (`/cart/order-confirmation?order-confirmed=true`) — the script captured the actual `Location` header instead of guessing.
- **Quantity=0 does not empty the cart here.** Setting a probed item's quantity to 0 did not remove it — both probed items stayed in the cart and were bought in the first order ($4.39). The solve still worked because the total stayed under $100.

## Mitigation

- Validate the order at the confirmation step itself: re-check the cart total against the user's store credit (or payment) before completing any order — never trust that an earlier step already did it.
- Make order-confirmation requests one-time: bind each confirmation to a single order ID / one-time token that is consumed on first use, so replaying it does nothing.
- Enforce the workflow server-side with an explicit order state machine (e.g. an order can only move CART → PAID → CONFIRMED, and CONFIRMED can only follow a PAID order for the same items and total).
- Never let a client-controlled sequence drive a payment decision — the server must re-derive the price and the balance at the moment of completion.

---

*Note: this lab is one of PortSwigger Web Security Academy's intentionally vulnerable free training labs. It was solved legally, for learning, on PortSwigger's own lab environment.*
