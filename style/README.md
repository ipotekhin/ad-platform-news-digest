# style/ — deck template & presentation style

## Files

- **`deck-template.html`** — data-driven HTML deck (the "Zine" skin, from the Claude
  Design handoff). Routine 2 copies it to `decks/deck-YYYY-MM-DD.html` and replaces
  the `<script id="deck-data">` JSON block with the real digest. Safe for GitHub Pages
  and prints cleanly to PDF.
- **`slack-summary.md`** — the text format Routine 2 posts to Slack (the connected
  Slack MCP has **no file upload**, so the deck is linked, not attached).

## Editing the look

All styling lives in the `<style>` block of `deck-template.html`. Change the CSS
variables under `:root` to re-skin. It is **not** zero-request: it loads Playfair
Display / Courier Prime from the Google Fonts CDN and sticker PNGs from
`../assets/stickers/` (repo-relative — works from both `style/` and `decks/`, since
both sit one level below repo root). Don't add anything beyond fonts + local sticker
assets — no analytics or other third-party JS. There is no dark-mode variant; this
skin is a light/cream editorial look by design.

## Deck-data schema

Required fields are unchanged (`period_label`, `generated`, `tldr`, `platforms[].items[]`
— see the schema comment inside `deck-template.html`). The Zine skin also accepts
optional fields that degrade gracefully if Routine 2 doesn't supply them:
`edition_no`, `subtitle`, `read_minutes`, `author`, `archive`. None of these require
changes to `routines/routine-2-presentation.md` — the template falls back sensibly
(e.g. no "No. N" prefix without `edition_no`, an estimated read time without
`read_minutes`) so the existing automation keeps working unmodified.

## Deck content rules (applied by Routine 2)

- Order platforms: **Google Ads → Meta → TikTok → LinkedIn → Bing** (drop empty ones).
- Within a platform, order items **high → medium → low** impact.
- TL;DR: 3–5 bullets, the single most important change first.
- If the deck exceeds ~12 items, drop or group `low`-impact items (see `criteria.md`).
- Every item links to its **canonical** source URL (official over aggregator).
