---
name: notes
description: Save a note to the user's private Announcr Notes channel. Use when they ask to save, send, or put something in their notes, or when a longer write-up should be kept readable instead of spoken in full. Do NOT use for spoken status (that is send_announcement). Do not include secrets.
when-to-use: Save to Notes when the user asks to save/send something to their notes, or when a longer write-up should stay readable on the Notes channel. Never for spoken status, and never with secrets.
license: MIT
metadata:
  author: Announcr
  version: "1.0.0"
---

# Announcr — save to Notes

> Announcr (https://announcr.fm) has one private **Notes** channel per user.
> Pocket recordings and MCP notes share it. This skill writes a note there.

## Send a note

Use the hosted MCP tool `send_to_notes` on `https://announcr.fm/api/mcp`:

```json
{"title": "Meeting notes", "message": "Meeting notes\n\nShipped the docs and the MCP tool."}
```

- `message` is required (1–8000 characters). This is the full note on the card and in the reader.
- `title` is optional (max 200). If omitted, Announcr speaks the first line of `message`.
- Speech is the short title/first line only. Do not also call `send_announcement` for the same note.
- This is **not** Publish to my channels. Do not pass `channel`.
- Same `announce` scope as `send_announcement`. A webhook-secret bearer can call it. Queue/reminder grants are not required.
- The stdio CLI (`npx @announcr/mcp say`) cannot write Notes. Hosted `/api/mcp` is the Notes path.

If the server shows as **needing login** or the tool returns unauthorized, tell the user to click **Needs login** (or reconnect) and sign in at announcr.fm. **Never ask the user to paste a secret into chat.**

## When to save a note

- The user asks to save, send, or put something in their notes.
- A longer write-up they will want to copy later (research, meeting recap) rather than hear in full.

## When NOT to

- Spoken status for a finished job, a blocker, or “tell me out loud” — that is `send_announcement`.
- Ordinary chat replies.
- Anything containing credentials, tokens, or secrets.

## Related tools

`send_announcement` speaks a short status line. Queue and reminder tools are separate (`list_queue`, `create_reminder`, …) and need those scopes.
