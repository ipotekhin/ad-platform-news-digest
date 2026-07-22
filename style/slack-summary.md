# Slack summary format (Routine 2 output)

> The Slack MCP has **no file-upload tool**, so the deck is **not** attached. Routine 2
> posts a short text summary with the **GitHub Pages link** to the deck embedded in the
> text. Delivery is GitHub Pages only — no Slack Canvas. The message posts **from Ivan's
> own Slack identity** (the connector is authorized under his account).

## Channel message template (standard Markdown)

The connector renders **standard Markdown** (`**bold**`, `_italic_`, `[text](url)`,
bullet lists) — not legacy Slack mrkdwn.

```
📰 **Ad Platform News Digest — {period_label}**
{N} ad-account updates across {platform_count} platforms. Top items:

- **{Platform}** — {title} _({impact})_
- **{Platform}** — {title} _({impact})_
- **{Platform}** — {title} _({impact})_

👉 Full deck: {deck_link}
```

Rules:
- List only the **top 3–5** items (highest impact first) in the message.
- `{deck_link}` = the GitHub Pages URL of `decks/deck-YYYY-MM-DD.html`, embedded in the
  text (not a query-param-laden link).
- Keep it short — depth lives in the deck.

## Test vs live delivery

- **Test / first runs (default):** do not post to the public channel. Either
  `slack_send_message_draft` (a draft Ivan reviews in Slack) or `slack_send_message`
  to **Ivan's DM** (`channel_id` = his user id). Ivan approves before going live.
- **Live:** `slack_send_message` to the target channel `#<CHANNEL>`.
- Only after a **confirmed** post (any mode Ivan accepts as the delivery) does Routine 2
  set `presented: true`.
