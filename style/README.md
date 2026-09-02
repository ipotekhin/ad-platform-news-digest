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
both sit one level below repo root). The header shows the brand logo
`../assets/aidigital-icon-blue.svg` and the page favicon is
`../assets/stickers/blob-face-pink.png` — both local files, so they add no new request
hosts. It also loads **Google Tag Manager** (container `GTM-M2BTR42L`) for analytics and
pushes a `link_click` event to the dataLayer on every `<a>` click (for GTM triggers).
Don't add anything beyond fonts + local assets + that GTM container — no other analytics
or third-party JS. There is no dark-mode variant;
this skin is a light/cream editorial look by design.

### Interactive & responsive (built into the template)

These are template chrome — they work automatically for every edition, no
deck-data changes needed:

- **Team filter.** The header holds an **All news / Search / Social** segmented
  control. Selecting a team shows/hides platform sections and recomputes the header
  stats. Teams are derived from the platform name (`teamOf`): Meta / TikTok / LinkedIn
  → *Social*, everything else (Google Ads, Microsoft/Bing, ChatGPT Ads) → *Search*.
- **Language pill.** A dropdown next to the filter offers 🇺🇸 EN / 🇷🇺 RU / 🇪🇸 ES /
  🇷🇸 SR (Serbian in **Latin** script). Picking one re-renders the content from the
  translations baked into the deck data — see *Translations* below. **The URL is the only
  source of language**, exactly like `?team=`: `?lang=ru|es|sr` selects one, and a link
  with no param always opens in English. Nothing is stored, so a shared link renders the
  same for whoever opens it. Both header controls collapse to dropdown pills below 640px,
  so the header keeps a fixed footprint however many controls we add later.
  The open dropdown panel is moved onto `<body>` and positioned `fixed` under its pill:
  the sticky header sets its own `backdrop-filter`, and a `backdrop-filter` nested inside
  another one samples an empty backdrop, so in place the panel rendered with **no blur at
  all**. It returns to its control on close, and scroll/resize close it.
- **Editions strip.** "Other editions" lists every edition except the one being viewed,
  **newest first**, so an older deck links forward as well as back. It reads
  `data/editions.json` at view time (same origin — not a third-party request), which is
  what lets an already-published, frozen deck show editions released after it without
  anything being rewritten. The `archive` array in the deck data is the fallback for when
  that read can't happen (opened from disk, offline) and holds prior editions only.
  Links in this strip carry the reader's current `?lang=` so a language survives moving
  between editions — the language is written into the link's own URL, not stored, so the
  "a bare link always opens in English" rule is untouched.
  The strip **collapses to one row** with a *View all N editions* toggle beneath it, so a
  long run of editions never becomes a wall of cards. Which cards fit is measured rather
  than a fixed count (a desktop row holds ~4–6, a phone row one), re-measured on resize,
  with a floor of `ARCHIVE_MIN` = 3 so a phone never collapses to a single lonely card.
  The toggle only appears when something is actually hidden — with a handful of editions
  the strip behaves exactly as before.
- **Motion.** A typewriter reveal on the sign-off heading, sticker entrance
  (fade + scale, staggered on scroll), and section reveals. Platform-heading icons
  (`.pf-icon`) and the Past-editions sticker (`.stk-static`) are excluded from the
  entrance; decorative stickers carry `stk-in` in the markup so they never flash on
  load. All motion is disabled under `prefers-reduced-motion`.
- **Mobile.** A `≤640px` media query hides the large floating stickers (they overlap
  text on narrow screens), centers the footer, keeps the brand left / controls right,
  drops the TL;DR to one column, and shrinks the sign-off heading. Desktop (≥641px) is
  unaffected.

### Translations

The deck is **pre-translated at build time** by Routine 2 — there is no runtime
translation API and no key to leak. Only **four** things are translated: the hero
subtitle, card titles, card summaries and TL;DR bullets. Everything else is fixed English chrome baked into
this template and is never regenerated per edition. Switching language re-runs the
content render from the same deck data, so stickers, layout and stats stay identical
across languages (the sticker RNG is re-seeded from a language-independent sub-seed).
Decks built before this feature have no translations and simply stay English.

### Stickers (randomized per edition)

Sticker PNGs live in `assets/stickers/`. Each edition shuffles which PNG lands in each
fixed slot (seeded by the edition date, so a given deck always looks the same). Which
stickers may appear **where** is controlled by pools in the template's `<script>`:

- `POOL_GENERAL` — hero / TL;DR / archive / **cards**. Excludes `arrows-grid`, `flag`,
  `folder-star`, `paperclip`.
- `POOL_HERO` = `POOL_GENERAL` + `paperclip` — the hero (`section:nth-child(2)`) may use
  paperclip.
- `POOL_WRAP` — the dark-blue "THAT'S A WRAP" block only. A hand-curated set that reads
  well on blue (the only place `folder-star` and `paperclip-green-pink` appear).
- Platform-heading icons stay a meaningful shape per platform (globe / notepad / doc)
  with a random colour — see `FAM` / `PF_FAM`.

`arrows-grid` and `flag` PNGs were removed from the repo. To change what appears where,
edit the pool arrays; positions/sizes are fixed in the HTML so nothing shifts.

## Deck-data schema

Required fields are unchanged (`period_label`, `generated`, `tldr`, `platforms[].items[]`
— see the schema comment inside `deck-template.html`). The Zine skin also accepts
optional fields that degrade gracefully if Routine 2 doesn't supply them:
`edition_no`, `subtitle`, `read_minutes`, `author`, `archive`, plus the translation
fields `subtitle_i18n`, `tldr_i18n` and per-item `i18n` (see the schema comment in the
template). None of these require
changes to `routines/routine-2-presentation.md` — the template falls back sensibly
(e.g. no "No. N" prefix without `edition_no`, an estimated read time without
`read_minutes`) so the existing automation keeps working unmodified.

## Deck content rules (applied by Routine 2)

- Order platforms **Search block first, then Social**: **Google Ads → Microsoft/Bing →
  ChatGPT Ads → Meta → TikTok → LinkedIn** (drop empty ones). The deck renders in
  deck-data order.
- Within a platform, order items **high → medium → low** impact.
- TL;DR: **3–4 bullets** (4 is the ceiling — 5 leaves an odd gap in the two-column grid),
  the single most important change first. **One sentence each, max
  ~20 words / ~140 characters** — it's a skim block; detail belongs on the card. On phones
  the TL;DR grid drops to a single column (two columns squeeze the text unreadably).
  Length limits are defined **in English** (the base language); translations match the
  English in meaning and length rather than having their own character caps.
- No overall size cap; **per-platform cap 6** (up to 8 to fit all `high`s) keeps Search
  vs Social balanced. Overflow carries to a future edition (see `criteria.md`).
- Every item links to its **canonical** source URL (official over aggregator).
