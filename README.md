# Announcr — voice announcements for your agents

[Announcr](https://announcr.fm) turns short text announcements into spoken audio on your devices. This plugin gives any AI agent a voice: when a long build finishes, a deploy goes live, or a task needs your attention, the agent announces it out loud instead of waiting for you to check back.

## Components

- **1 skill — `announce`** ([skills/announce/SKILL.md](skills/announce/SKILL.md)): teaches the agent *when* to announce (task finished, blocked on your input, error halted progress, or you asked) and *how* to write for the ear.
- **1 hosted MCP server** — `https://announcr.fm/api/mcp` (Streamable HTTP, OAuth sign-in), declared in [mcp.json](mcp.json).

## Tools

| Tool | Argument | Required | Default | Description |
|---|---|---|---|---|
| `send_announcement` | `message` | yes | — | Text that will be spoken (max 500 characters) |
| | `event` | no | `"announce"` | Event name used for subscription matching |
| | `service` | no | `"mcp"` | Service/app label (lets one webhook multiplex) |

Example call:

```json
{
  "name": "send_announcement",
  "arguments": {
    "message": "Deploy finished successfully."
  }
}
```

## Install

### Cursor

- **Marketplace**: search for **announcr** in the Cursor Marketplace and install (once listed).
- **One click**: your webhook page at [announcr.fm](https://announcr.fm) → Webhooks has an **Add to Cursor** button.
- **Manual**: in Cursor's MCP settings, add a server with URL `https://announcr.fm/api/mcp`.

### Claude Code

```
/plugin marketplace add BloxxOnline/announcr-mcp-plugin
```

Then install **announcr** from the plugin list (`/plugin`).

### Grok Build

Grok Build reads Claude Code marketplaces automatically — add this repo (`BloxxOnline/announcr-mcp-plugin`) as a marketplace and the plugin appears.

### Grok chat / Grok Bot

Open [grok.com/connectors](https://grok.com/connectors) → **New Connector** → **Custom** and paste `https://announcr.fm/api/mcp`.

### Skills CLI

Install just the skill (agent instructions + CLI fallback, no MCP connection needed):

```bash
npx skills add BloxxOnline/announcr-mcp-plugin
```

### Any other MCP host

Point it at `https://announcr.fm/api/mcp`. Hosts without OAuth support can send the header `Authorization: Bearer <webhook-secret>` — get the secret from [announcr.fm](https://announcr.fm) → Webhooks.

## Authentication

After installing, the server shows as needing sign-in ("Needs login" in Cursor; Grok opens a sign-in window). Click it, sign in at announcr.fm, and pick which webhook "voice" the agent speaks through. That's it — no keys to paste.

- Hosts without OAuth support can use the bearer header instead: `Authorization: Bearer <webhook-secret>`.
- `?secret=` URLs still work for existing connectors but are deprecated — prefer OAuth sign-in or the bearer header.

## Privacy & data handling

- The plugin sends **only the announcement text** you or your agent compose (max 500 characters) to Announcr, where it becomes speech on **your** devices.
- It never reads your code, files, or repository.
- The credential is scoped to speaking announcements only — it cannot read account data.
- Revoke access anytime at announcr.fm → **Webhooks** (rotate or revoke the webhook) or **Settings → Connected apps**.

## Local development

Symlink the repo into Cursor's local plugins directory and reload:

```bash
ln -s "$(pwd)" ~/.cursor/plugins/local/announcr
```

Then run **Developer: Reload Window** in Cursor. This repo is the canonical home of the `announce` skill — the Announcr monorepo references it.

## Support

- Feedback: https://announcr.fm/feedback
- Docs: https://announcr.fm/docs/ai-agents

## License

MIT
