# PortSwigger Lab — Insecure Direct Object References (Access Control)

**Lab:** Insecure direct object references
**Category:** Access control
**Difficulty:** APPRENTICE
**Lab page:** https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references
**Instance:** https://0afc001a036513d382aa7ed70069009d.web-security-academy.net/
**Solved:** 26 November 2026 — logged in as `carlos` with the password from the chat transcript; verified "Congratulations, you solved the lab!" banner on the instance home page.

## Vulnerability Type
Insecure direct object reference (IDOR) — live-chat transcripts are saved as numbered text files (`/download-transcript/1.txt`, `2.txt`, …) with no access control, so anyone can read other users' transcripts.

## Objective
Find carlos's password in a chat transcript and log in as him.

## Solution
1. Logged in as `wiener:peter` and opened `/chat` (Live chat). The chat form's `action` is a WebSocket URL: `wss://<instance>/chat` — the chat runs over WebSocket, not HTTP.
2. Connected to the WebSocket, sent `READY`, then sent a message (`{"message": "hello, I need help with my account"}`). The support agent Hal Pline replied.
3. Posted the transcript (built from the chat messages, as the page's `viewTranscript.js` does) to `/download-transcript`. The server saved it as `/download-transcript/3.txt` — which showed transcripts are numbered files.
4. Opened the earlier transcript `/download-transcript/1.txt` (pre-seeded on the instance). It contained a conversation where a user tells Hal Pline "my password is dbd683yg1ol847o3g77k" and Hal confirms "Yes it is!".
5. Logged in as `carlos` with that password. The lab solved; the home page banner confirmed it.

## Practical Explanation
Two flaws chain together: the chat is a direct-object reference (transcripts are just numbered files anyone can request), and the transcript itself contains a password a user shared with support. Numbered, predictable URLs with no ownership check mean every transcript — including ones holding credentials — is public. The fix: store transcripts per user, require the session owner to match, and never let users share secrets in chat (support should never ask for or confirm a password in plain text).

## Lessons Learned
- Predictable numbered files (`1.txt`, `2.txt`) are always worth enumerating — an earlier file may already hold the secret.
- Read the page's JavaScript (`chat.js`, `viewTranscript.js`) to learn the real protocol: here it revealed the WebSocket chat and the transcript-save endpoint.
- A form `action` pointing at `wss://` is a WebSocket endpoint, not an HTTP one — POSTing to it fails; connect with a WebSocket client instead.
- Passwords must never be typed into live chat, and support must never confirm one in a transcript.
