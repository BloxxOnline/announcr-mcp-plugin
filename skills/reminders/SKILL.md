---
name: reminders
description: Create, list, update, and cancel this user's Announcr personal reminders. Use when they ask to be reminded, change a reminder, see upcoming reminders, or cancel one. Same rows as the Announcr site. Do NOT invent a second reminder store. Do NOT use a series of one-shots for a repeating schedule (use cron).
when-to-use: Set, list, change, or cancel an Announcr personal reminder when the user asks. These are the same reminders as the site. Use create_reminder with cron for repeating schedules. Use create_reminder_series only for 2 to 10 distinct one-shots.
license: MIT
metadata:
  author: Announcr
  version: "1.0.0"
---

# Announcr — reminders

> Personal reminders live on announcr.fm. MCP writes the same rows the site
> and voice create. After a successful create or update, Announcr speaks a
> short canned line through this grant's webhook. Cancel is silent.

## Tools

| Tool | When |
|---|---|
| `create_reminder` | One reminder, once or cron |
| `create_reminder_series` | 2–10 distinct one-shots in one transaction |
| `list_reminders` | This user's live reminders. `include_terminal` adds canceled/completed from the last 30 days |
| `update_reminder` | Edit a live reminder going forward |
| `cancel_reminder` | Soft-cancel. The row stays listable with `include_terminal` for 30 days |

### `create_reminder`

Required: `message` (1–500, spoken text), `schedule_kind` (`once` or `cron`).

Once: `fire_at` (ISO-8601 with offset, future) or `relative_minutes` (1–5256000).
Cron: five-field `cron`, cadence at least 5 minutes (`*/5 * * * *` is ok;
`*/4 * * * *` is not). Optional `timezone` (IANA). Default is the user's
stored timezone, else UTC.

Do not send a reminder key. The server generates it. Do not insert occurrences
yourself.

### `create_reminder_series`

`items` is 2–10 once-payloads. All or nothing against the 50-reminder cap
(active + paused). If the batch would exceed 50, nothing is created.

A repeating weekday 9am reminder is `create_reminder` with `schedule_kind=cron`,
not ten one-shots.

### `update_reminder`

`reminder_id` plus at least one field. Message-only keeps the clock. A schedule
change invalidates undelivered work from the old version. Cannot edit another
user. Cannot resurrect `canceled` or `completed`. Optional `expected_updated_at`
from a prior list/create/update; mismatch returns `reminder_conflict` with the
current row.

### Caps and spoken lines

Cap is 50 occupying reminders. On success, create speaks "Reminder created."
or "Two reminders created." (through Ten). Update speaks "Reminder updated."
Cancel does not speak.

## Errors

| Tool text | Meaning | What to do |
|---|---|---|
| `feature_not_enabled` | Reminder tools are registered but the host has them off | Tell the user Announcr has not enabled reminder tools yet |
| `insufficient_scope` | This grant is speak-only | Tell them to reconnect and approve reminders. Never ask for a secret |
| `personal_reminder_limit_reached` | 50 live reminders already | Ask which one to cancel. Do not retry the same create |
| `reminder_time_in_past` / `reminder_cadence_too_frequent` / `invalid_reminder_*` | Schedule failed validation | Fix the time or cron |
| `reminder_not_editable` | Canceled or completed | Create a new reminder |
| `reminder_conflict` | Stale `expected_updated_at` | Use the `current` row in the error and retry if the user still wants the edit |
| `not_found` | Wrong id or another user | Stop |

If the server shows **Needs login**, tell the user to click it and sign in at
announcr.fm. **Never ask the user to paste a secret into chat.**

## When to write a reminder

- The user asks to be reminded, to see upcoming reminders, to change one, or
  to cancel one.

## When NOT to

- Speaking a finished job. That is `send_announcement`.
- Pulling queued X or Pocket work. That is the queue skill.
- Inventing a local reminder list or a calendar event Announcr cannot speak.

## Privacy

A grant with `reminders` can list and edit **this user's** personal reminders.
It cannot see another user's reminders. An announce-only grant or a webhook
secret cannot read or write reminders.
