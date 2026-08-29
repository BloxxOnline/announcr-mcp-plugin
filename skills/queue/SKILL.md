---
name: queue
description: Pull work from this user's Announcr agent queue. Use when the user asks to check, list, pull, claim, or finish items queued for this voice (an X mention or Pocket clip sent to the agent). Do NOT treat this as email or an inbox. Do NOT claim on list. Do NOT read another user's queue.
when-to-use: Pull or claim items from this user's Announcr agent queue when they ask to check queued work, pick up something sent to this voice, or mark a claimed item done. Never call this email or an inbox. Never auto-claim on list. Never read another account.
license: MIT
metadata:
  author: Announcr
  version: "1.0.0"
---

# Announcr — agent queue

> Announcr parks work for **this user** on the hosted MCP server
> (`https://announcr.fm/api/mcp`). You pull it. You do not receive a push into
> chat. Durable noun: **queue**. Do not say inbox, mailbox, or mail.

## Tools

| Tool | When |
|---|---|
| `list_queue` | Read-only page of items this grant's voice may see |
| `claim_item` | Take exclusive hold of one item before you act |
| `ack_item` | Mark a claimed item done after you finish it |

`list_queue` never claims. Two agents on one account cannot both hold the same
item. If `claim_item` returns `already_claimed`, pick a different row.

### `list_queue`

Optional arguments: `status` (`open` default = queued + claimed), `limit`
(1–20), `cursor`, `target_label`, `source` (`x_mention` or `pocket`).

Use `next_cursor` to page. Occupancy counts (`queued_count`, `claimed_count`,
`occupying_count`, `cap`) are for the whole user. An item targeted at another
webhook is hidden from this grant and still counts toward the cap.

### `claim_item` / `ack_item`

Both take `item_id`. Claim is required before you treat the payload as yours.
Ack only works for the grant that holds the claim, during the lease plus a
short grace. Retrying ack for the same grant is safe. A sibling grant gets
`not_yours`.

No spoken line on claim or ack. The user already heard the queue status when
the source landed.

## Errors

| Tool text | Meaning | What to do |
|---|---|---|
| `feature_not_enabled` | Queue tools are registered but the host has them off | Tell the user Announcr has not enabled the agent queue yet |
| `insufficient_scope` | This grant is speak-only | Tell them to reconnect and approve queue access. Never ask for a secret |
| `grant_required` | Webhook-secret / `?secret=` auth | Queue needs OAuth sign-in |
| `not_found` | Wrong id, other user, or targeted at another voice | Stop. Do not guess another id |
| `already_claimed` | Another grant holds it | List again and pick a different item |
| `not_claimable` | Expired, acked, rejected, or lease plus grace lapsed | Leave it |
| `not_yours` | You are not the claiming grant | Do not ack |

If the server shows **Needs login**, tell the user to click it and sign in at
announcr.fm. **Never ask the user to paste a secret into chat.**

## When to pull

- The user asks what is queued, to pick up work sent to this voice, or to
  finish a claimed item.
- You are the connected agent they named (for example Grokbot) and they said
  they sent something to you.

## When NOT to pull

- Ordinary chat. Speaking a finished build is `send_announcement`, not the queue.
- Creating or editing reminders. That is the reminders skill.
- Reading another account, even if you have an id.

## Privacy

A grant with `queue` can list and claim **this user's** items that this voice
is allowed to see. It cannot read another user's queue. An announce-only grant
or a webhook secret cannot read the queue at all.
