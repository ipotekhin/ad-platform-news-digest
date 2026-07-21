# Slack summary format (Routine 2 output)

> The Slack MCP has **no file-upload tool**, so the deck is **not** attached. Routine 2
> posts this short text summary with a **link** to the deck (GitHub Pages) and/or
> mirrors the full digest into a **Slack Canvas**. Keep the channel message short —
> detail lives in the deck / canvas.

## Channel message template (Slack mrkdwn)

```
:newspaper: *Ad Platform News Digest — {period_label}*
{N} updates across {platform_count} platforms. Top items:

• *{Platform}* — {title}  _({impact})_
• *{Platform}* — {title}  _({impact})_
• *{Platform}* — {title}  _({impact})_

:point_right: Full deck: {deck_link}
```

Rules:
- List only the **top 3–5** items (highest impact first) in the channel message.
- `{deck_link}` = the GitHub Pages URL of `decks/deck-YYYY-MM-DD.html`, or the Slack
  Canvas link if Pages is not set up.
- Use Slack *mrkdwn* (`*bold*`, `_italic_`, `•` bullets) — not full Markdown.

## Canvas fallback / mirror

If posting a Canvas (via `slack_create_canvas` / `slack_update_canvas`), include the
full per-platform list there using the same ordering rules as the deck. The channel
message then links to the Canvas instead of (or in addition to) the Pages deck.
