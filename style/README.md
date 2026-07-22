# style/ — deck template & presentation style

## Files

- **`deck-template.html`** — self-contained, data-driven HTML deck. Routine 2 copies
  it to `decks/deck-YYYY-MM-DD.html` and replaces the `<script id="deck-data">` JSON
  block with the real digest. No external requests → safe for GitHub Pages and prints
  cleanly to PDF.
- **`slack-summary.md`** — the text format Routine 2 posts to Slack (the connected
  Slack MCP has **no file upload**, so the deck is linked, not attached).

## Editing the look

All styling lives in the `<style>` block of `deck-template.html`. It is theme-aware
(light/dark via `prefers-color-scheme`). Change the CSS variables under `:root` to
re-skin. Do not add external fonts/CSS/JS — the single-file, zero-request property is
what makes the deck portable and safe to serve from GitHub Pages.

## Deck content rules (applied by Routine 2)

- Order platforms: **Google Ads → Meta → TikTok → LinkedIn → Bing** (drop empty ones).
- Within a platform, order items **high → medium → low** impact.
- TL;DR: 3–5 bullets, the single most important change first.
- If the deck exceeds ~12 items, drop or group `low`-impact items (see `criteria.md`).
- Every item links to its **canonical** source URL (official over aggregator).
