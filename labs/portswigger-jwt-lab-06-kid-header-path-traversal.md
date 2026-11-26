# JWT Lab 6 — JWT authentication bypass via kid header path traversal — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** JSON Web Tokens (JWT)
**Lab:** JWT authentication bypass via kid header path traversal
**Difficulty:** Practitioner
**Solved:** 27 November 2026 ~04:24 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

JWT authentication bypass — the server uses the token's `kid` header as a file path to load the verification key, so `kid` can be traversed to a file whose contents the attacker knows (here `/dev/null`, which is empty) and the token signed with that known content.

## Solution

1. Logged in as `wiener:peter` and got the session JWT (RS256, `kid` was a UUID).
2. Forged a token with header `{"alg":"HS256","kid":"../../../../../dev/null"}`, payload `sub` changed to `administrator`, signed HS256 with an **empty** HMAC secret (the contents of `/dev/null`). The signature was computed manually with HMAC-SHA256 because PyJWT refuses empty HMAC keys.
3. Sent it as the `session` cookie: `GET /my-account` showed the administrator account.
4. Deleted carlos: `GET /admin/delete?username=carlos` — home page then showed "Congratulations, you solved the lab!"

Tried and failed first: `kid` values `/dev/null`, `../../dev/null`, `dev/null` and `../../../../dev/null` (wrong traversal depth — the key directory sits five levels deep), and HMAC secret `null` instead of empty.

## Practical Explanation

- **Why it works:** the server builds a file path from `kid` and reads that file as the verification key. Traversing to `/dev/null` makes the key empty, and the attacker knows the empty key, so they can sign a valid HS256 token with it.
- **Why the depth matters:** the traversal has to climb out of the key directory first — five `../` segments were needed on this instance; fewer left the path inside a non-existent directory and verification failed.
- **The key detail:** `kid` is attacker-controlled input, but the server treats it as a trusted file path — a path traversal in a key lookup is as powerful as a path traversal anywhere else.

## Lessons Learned

- Never build file paths from token headers (or any client input) — map key IDs to keys server-side with a fixed allowlist.
- Key loading must not follow user-supplied paths; a `kid` should be an opaque identifier, not a filename.
