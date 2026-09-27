---
name: announce
description: Speak to the user out loud through their Announcr speakers. Use when a long-running build, test run, deploy, or research task finishes, when you are blocked waiting on the user's input, when an error halts progress, or when the user asks to announce, say, or tell them something audibly. Do NOT announce routine progress, every tool call, or anything containing secrets.
when-to-use: Announce out loud when a long build/test/deploy/research task finishes, when you are blocked waiting on the user's input, when an error halts progress, or when the user asks to hear something spoken. Never for routine progress, every tool call, or anything containing secrets.
license: MIT
metadata:
  author: Announcr
  version: "1.3.0"
---

# Announcr — speak to the user out loud

> Announcr (https://announcr.fm) plays spoken audio on the user's devices. This
> skill lets you send an announcement — one short sentence of text that is
> turned into speech and played on their speakers within a few seconds.

## Send an announcement

### Preferred: the `send_announcement` MCP tool

This plugin connects an MCP server (`https://announcr.fm/api/mcp`). For speaking,
use `send_announcement` with just the message:

```json
{"message": "The deploy finished and all checks passed."}
```

Optional Spotlight CTA (do not put the URL in spoken text):

```json
{
  "message": "The deploy finished and all checks passed.",
  "link": { "url": "https://example.com/deploy/42", "caption": "Open deploy" }
}
```

- `event` and `service` are optional filtering labels — omit them unless the
  user has set up event filtering (defaults `announce` / `mcp`).
- `link` is optional: `{ "url": "https://…", "caption": "Open" }`. http(s)
  only. Spotlight shows it as a new-tab CTA. A top-level `link` applies to
  card 0 unless that card already has a part-level link.
- `channel` is optional. List first with `list_channels`. If you need a
  series and no matching publishable slug, `create_private_channel` with a
  name, then pass that slug here (or `native:<id>`). A short room line is
  max 500 characters. Longer channel copy (up to 8000) is packed into a
  Spotlight series, or send `parts` as 2–8 strings or `{text, link}` objects
  (each max 900). Without `channel`, the line speaks on the owner's devices
  as usual (or to the one allowlisted channel when that webhook has exactly
  one).
- If the server shows as **needing login** or the tool returns unauthorized,
  tell the user to click **Needs login** (or reconnect the server) and sign in
  at announcr.fm. **Never ask the user to paste a secret into chat.**

### Fallback: the `say` CLI (no MCP connection)

```bash
npx -y @announcr/mcp say "Deploy to production finished."
```

- Reads the env vars below; the request is HMAC-signed so the secret never
  travels.
- Exit code `0` = accepted and will be spoken. Non-zero = not spoken; stderr
  says why.
- Optional filtering labels: `--event <name>` and `--service <label>`
  (letters, numbers, `.` `-` `_`; defaults `announce` / `cli`). Omit them
  unless the user has set up event filtering.
- Optional `--channel <slug>` to publish onto an allowlisted native channel
  instead of the owner's devices.

#### Credentials for the CLI

The CLI authenticates with a **webhook URL + secret** from the user's Announcr
account, read from environment variables:

```bash
ANNOUNCR_WEBHOOK_URL=https://announcr.fm/hooks/in/<publicId>
ANNOUNCR_WEBHOOK_SECRET=<64-char hex secret>
```

Check for them before sending (e.g. `printenv ANNOUNCR_WEBHOOK_SECRET` — check
existence, do not print the value). If they are missing, tell the user:

> Open **announcr.fm → Webhooks**, create (or open) a webhook, and copy the
> URL and secret from **Connect an AI or app** into the environment variables
> `ANNOUNCR_WEBHOOK_URL` and `ANNOUNCR_WEBHOOK_SECRET` (shell profile, `.env`,
> or your agent's environment). They can view or rotate these values there
> any time.

**Never print, echo, log, commit, or speak the secret or the URL.** Do not
paste them into code, config files under version control, or chat replies.
If the user pastes the secret into chat, use it, suggest they move it into
the environment variables, and never repeat it back.

### Last resort: plain HTTP (no MCP, no Node/npx)

```bash
curl -sS -X POST "$ANNOUNCR_WEBHOOK_URL" \
  -H "Authorization: Bearer $ANNOUNCR_WEBHOOK_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"message": "Deploy to production finished."}'
```

A `202` response with `{"accepted":true}` means it will be spoken. Optional
body fields: `"event"`, `"service"`, `"channel"`, `"link"` (`{url, caption?}`),
and `"parts"` (2–8 strings or `{text, link}` objects for a channel series).

## When to announce

- A long-running task the user assigned finishes with an outcome — build green
  or broken, deploy live, test run done, research ready.
- You are blocked waiting on the user's input and they may have stepped away.
- An error halts progress and nothing more can happen until they return.
- The user explicitly asks you to announce, say, speak, or tell them
  something out loud.

## When NOT to announce

- Ordinary chat replies or questions — answer in text.
- Routine progress: not every tool call, never mid-task progress narration.
  One announcement per outcome — announce the result once, not the steps.
- Retries or intermediate failures you are still handling yourself.
- Anything containing credentials, tokens, file paths, or code. Put URLs on
  `link`, not in spoken text.

## Message style

The message is read aloud by text-to-speech — write for the ear:

- One or two short, natural spoken sentences. Device-only maximum 500
  characters. A channel fire may pack a longer message (up to 8000) or use
  `parts` (2–8 strings or `{text, link}` objects).
- No URLs, no code, no markdown, no emoji, no secrets in spoken text. Pass
  an http(s) URL on `link` (optional `caption`) so Spotlight can show a CTA.
- Say what happened and what it means, not the log line: prefer
  `"The deploy finished and all checks passed."` over
  `"deploy.sh exited 0 (47 passed)"`.

## Errors

| Response | Meaning | What to do |
|---|---|---|
| `202` / exit 0 / tool success | Accepted, will be spoken | Done |
| MCP tool returns unauthorized / server shows "Needs login" | OAuth session missing or expired | Tell the user to click **Needs login** / reconnect the server and sign in at announcr.fm — never ask for a secret in chat |
| `401 missing_auth` / `bad_secret` | Wrong or missing secret (CLI/HTTP path) | Ask the user to re-copy both values from the webhook's page (they may have rotated the secret) |
| `404 unknown_webhook` | URL wrong, or webhook revoked/parked | Ask the user to check the webhook on announcr.fm → Webhooks |
| `409 replay` | Identical signed request re-sent | Already delivered once — do not resend |
| `400 invalid_body` | Malformed JSON or field limits | Fix the body; device-only `message` ≤ 500 chars, packed channel `message` ≤ 8000, `parts` 2–8 × ≤ 900, `link.url` http(s), caption ≤ 40, `event`/`service` `[\w.-]+` |
| `400 sequence_requires_channel` | `parts` or a long packed message without a channel target | Pass `channel` or omit `parts` / shorten the message |
| `400 parts_limit` | More than 8 spoken cards | Send fewer `parts` or a shorter message |
| `400 channel_required` | Webhook allows more than one channel | Pass `channel` with one allowlisted slug |
| `403 channel_forbidden` | Channel not on this webhook's allowlist | `list_channels` / `create_private_channel`, then pass that slug |
| `403 channel_unavailable` | Channel is a story, parked, or suspended | Do not retry as a device announcement |
| `429` | Channel broadcast throttle | Wait `retryAfterMs` and retry |
| Accepted but user hears nothing | Their webhook's listening filter excludes this event, or their audio is off | Ask them to set the webhook's listening to **Everything** and check audio is enabled on an open Announcr tab or the desktop app |

## Related tools on the same server

`send_announcement` does not read the account. Use `list_channels` to find a
publishable slug and `create_private_channel` to mint a private feed (hosted
MCP only). Use `send_to_notes` when the user asks to save something to Notes.
Queue pull and reminder write are separate tools (`list_queue`, `claim_item`,
`ack_item`, `create_reminder`, and the other reminder tools). Use those only
when the user asks, and only when the grant includes `queue` or `reminders`.
A webhook-secret connection can speak, save notes, list channels, and mint a
private room, but cannot use queue or reminder tools.

## More

- Install this skill standalone: `npx skills add BloxxOnline/announcr-mcp-plugin`
- Setup with per-tool instructions (Grok, ChatGPT, Claude, Zapier):
  https://announcr.fm/docs/ai-agents
- Webhook reference (auth modes, signing, filtering):
  https://announcr.fm/docs/webhooks
- npm package (stdio MCP server + `say` CLI):
  https://www.npmjs.com/package/@announcr/mcp
