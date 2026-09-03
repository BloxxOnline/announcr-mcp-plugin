# Announcr — voice, queue, and reminders for your agents

[Announcr](https://announcr.fm) turns short text into spoken audio on your devices, or on a native feed channel that webhook is allowed to publish to. This plugin gives an AI agent a voice, a pull queue for work sent to that voice, and the same personal reminders you manage on the site.

## Components

- **4 skills**
  - [`announce`](skills/announce/SKILL.md): when to speak (task finished, blocked on your input, error halted progress, or you asked) and how to write for the ear.
  - [`notes`](skills/notes/SKILL.md): when to save a note to this user's private Notes channel (`send_to_notes`). Not spoken status.
  - [`queue`](skills/queue/SKILL.md): when to list, claim, and ack items in this user's agent queue. Not email. Not an inbox.
  - [`reminders`](skills/reminders/SKILL.md): when to create, list, update, or cancel this user's Announcr reminders (the same rows as the site).
- **1 hosted MCP server** — `https://announcr.fm/api/mcp` (Streamable HTTP, OAuth sign-in), declared in [mcp.json](mcp.json). The marketplace plugin requests one bundled grant: `announce queue reminders`.

## Tools

| Tool | Scope | Description |
|---|---|---|
| `send_announcement` | `announce` | Speak text through the granted webhook. Device-only max 500 characters. Optional `channel` (slug or `native:<id>`) publishes to an allowlisted native channel; a longer message (up to 8000) or `parts` (2–8 strings) airs as one Spotlight series. |
| `send_to_notes` | `announce` | Save a note to this user's private Notes channel. Optional `title` (max 200) is spoken; the full `message` (max 8000) stays on the card. Not Publish to my channels. Hosted `/api/mcp` only. |
| `list_queue` | `queue` | Read-only page of this user's queue items this voice may see |
| `claim_item` | `queue` | Exclusively claim one item. Required before you act on it |
| `ack_item` | `queue` | Mark a claimed item done |
| `create_reminder` | `reminders` | Create one personal reminder (once or cron) |
| `create_reminder_series` | `reminders` | Create 2–10 distinct one-shots, all or nothing against the cap |
| `list_reminders` | `reminders` | List this user's reminders |
| `update_reminder` | `reminders` | Edit a live reminder going forward |
| `cancel_reminder` | `reminders` | Soft-cancel. Silent |

Example speak call:

```json
{
  "name": "send_announcement",
  "arguments": {
    "message": "Deploy finished successfully."
  }
}
```

Optional `channel` publishes to a room on this webhook's **Publish to my channels** list instead of the owner's devices. A short room line is one card. A longer `message` (up to 8000) or `parts` (2–8 strings) airs as one Spotlight series:

```json
{
  "name": "send_announcement",
  "arguments": {
    "channel": "agent-center",
    "parts": [
      "Paid airtime is live.",
      "The next beat starts now."
    ]
  }
}
```

Queue and reminder tools stay registered even when Announcr has them turned off. In that case they return `feature_not_enabled`. A missing scope returns `insufficient_scope`.

## Install

### Cursor

- **Marketplace**: search for **announcr** in the Cursor Marketplace and install (once listed). Sign in and approve the bundled grant.
- **One click**: your webhook page at [announcr.fm](https://announcr.fm) → Webhooks has an **Add to Cursor** button.
- **Manual**: in Cursor's MCP settings, add a server with URL `https://announcr.fm/api/mcp`. A manual URL paste is speak-only unless the host requests `queue` and `reminders`.

### Claude Code

```
/plugin marketplace add BloxxOnline/announcr-mcp-plugin
```

Then install **announcr** from the plugin list (`/plugin`).

### Grok Build

Grok Build reads Claude Code marketplaces automatically. Add this repo (`BloxxOnline/announcr-mcp-plugin`) as a marketplace and the plugin appears.

### Grok chat / Grokbot

Open [grok.com/connectors](https://grok.com/connectors) → **New Connector** → **Custom** and paste `https://announcr.fm/api/mcp`. Sign in at announcr.fm and approve **one** consent: Grokbot can speak through this voice, pull this agent queue, and manage your Announcr reminders.

### Skills CLI

Install just the skills (agent instructions + CLI fallback for speaking, no MCP connection needed):

```bash
npx skills add BloxxOnline/announcr-mcp-plugin
```

### Any other MCP host

Point it at `https://announcr.fm/api/mcp`. Hosts without OAuth support can send the header `Authorization: Bearer <webhook-secret>` (get the secret from [announcr.fm](https://announcr.fm) → Webhooks). That path can call `send_announcement` and `send_to_notes`.

## Authentication

After installing, the server shows as needing sign-in ("Needs login" in Cursor; Grok opens a sign-in window). Click it, sign in at announcr.fm, and pick which webhook voice the agent uses. That's it. No keys to paste.

- Hosts without OAuth support can use the bearer header instead: `Authorization: Bearer <webhook-secret>`.
- `?secret=` URLs still work for existing connectors but are deprecated. Prefer OAuth sign-in or the bearer header.
- Webhook-secret and `?secret=` connections are announce-only. Queue and reminders need the OAuth grant.

## Privacy & data handling

- An **announce-only** grant (or a webhook secret) can speak through `send_announcement` and save to Notes with `send_to_notes`. It cannot read account data, the agent queue, or reminders. This is not a public write API.
- A grant that includes **queue** can list and claim **this user's** queue items that this voice is allowed to see. Nothing else.
- A grant that includes **reminders** can list and edit **this user's** personal reminders. Nothing else.
- The plugin never reads your code, files, or repository.
- Revoke access anytime at announcr.fm → **Settings → Connected apps**, or rotate/revoke the webhook under **Webhooks**.

## Local development

Symlink the repo into Cursor's local plugins directory and reload:

```bash
ln -s "$(pwd)" ~/.cursor/plugins/local/announcr
```

Then run **Developer: Reload Window** in Cursor. This repo is the canonical home of the skills. The Announcr monorepo references it.

## Support

- Feedback: https://announcr.fm/feedback
- Docs: https://announcr.fm/docs/ai-agents

## License

MIT
