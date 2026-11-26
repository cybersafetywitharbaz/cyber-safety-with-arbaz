# OS Command Injection Lab 1 — OS command injection, simple case — PortSwigger

**Platform:** PortSwigger Web Security Academy (free lab)
**Topic:** OS Command Injection
**Lab:** OS command injection, simple case
**Difficulty:** Apprentice
**Solved:** 27 November 2026 ~03:40 PKT (banner verified on the instance home page)
**Account:** Arbaz Ahmed Naveed

## Vulnerability Type

OS Command Injection (shell injection) — the shop's stock-check feature passes user-supplied product and store IDs straight into a server-side shell command (`stockreport.pl <productId> <storeId>`), so a shell metacharacter in the input runs an arbitrary command and its output comes back in the response.

## Solution

1. Baseline: `POST /product/stock` with `productId=1&storeId=1` returned the normal stock count (`62`).
2. Injected into the trailing `storeId` parameter (safest place — end of the command line):
   ```
   POST /product/stock
   Content-Type: application/x-www-form-urlencoded

   productId=1&storeId=1|whoami
   ```
3. The response came back as `peter-KGy1WX` instead of a stock number — the `whoami` output.
4. Home page then showed "Congratulations, you solved the lab!"

## Practical Explanation

- **Why it works:** the server builds a shell command from the two IDs. The `|` (pipe) feeds the stock command's output into `whoami`, and since `whoami` ignores its input and prints the current user, the response is purely the injected command's output.
- **Why `storeId` and not `productId`:** injecting at the end of the command line is least likely to break the command's own syntax before the payload runs.
- **The key detail:** the lab user is a random `peter-XXXXXX` name (here `peter-KGy1WX`) — each instance gets its own, so the answer is never guessable, only retrievable.

## Lessons Learned

- Any input concatenated into a shell command is full remote code execution: one pipe character turned a stock lookup into a `whoami`.
- Never build OS commands from user input — use fixed commands with an allowlist, or call the underlying function directly instead of shelling out.
