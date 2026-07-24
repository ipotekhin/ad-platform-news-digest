# Routine 2 — Presentation

**Cadence:** every 2 weeks (set manually in the routine UI; changing the schedule does
not affect this logic).
**Job:** turn the not-yet-presented updates into a readable deck, post a summary to
Slack, then mark those items `presented: true` — **only after** a successful post.

The routine UI holds only a short pointer to this file. All logic lives here.

---

## Routine UI call (what to paste into the trigger prompt)

```
You are Routine 2 (Presentation) for the ad-platform-news-digest repo.
Read routines/routine-2-presentation.md and follow it exactly. If nothing qualifies,
stop and do nothing (empty-edition guard). Otherwise branch from main to a
claude/present-run branch, build the deck, deliver the Slack message to the target in
the doc's Open config, then set presented:true on the included items, commit, push,
open a pull request into main and merge it.
```

The Slack destination lives in the doc (Open config → currently Ivan's DM), so the
prompt never needs editing when the destination changes.

---

## Preconditions
- `data/updates.json`, `criteria.md`, `style/deck-template.html` exist.
- Slack MCP is connected (tools: `slack_send_message`, `slack_send_message_draft`,
  `slack_search_channels`). **No file-upload tool exists** — the deck is **linked**, not
  attached. The message posts **from Ivan's own Slack identity** (the connector is
  authorized under his account), not a separate bot.
- **Delivery is GitHub Pages only.** Do not create or format anything in Slack Canvas.

---

## Steps

### 1. Select candidates
From `data/updates.json`, take every item where `presented == false` (all un-presented
items — no time filter here; Routine 1 already bounded what got collected).

### 2. Strict relevance filter
Apply `criteria.md` **strictly** now. Drop items that don't clearly match an "Include"
rule or that hit an "Exclude" rule. For survivors, confirm/adjust `category` and
`impact` per `criteria.md`.

### 2a. Empty-edition guard (IMPORTANT — stop if nothing qualifies)
If **no items** survive selection + filtering (i.e. there are zero un-presented,
relevant updates across **all** platforms), **STOP the whole run**: do **not** build a
deck, do **not** post anything to Slack, do **not** commit. This edition simply does
not happen. (Optional: leave a one-line note in the run log.) Only continue to step 3
when there is at least one qualifying update.

### 3. Order & trim
- Group by platform: **Google Ads → Meta → TikTok → LinkedIn → Bing** (omit empty).
- Within each platform, order **high → medium → low** impact.
- If > ~12 items total, group or drop `low`-impact items to keep the deck tight.
- Write a **TL;DR** of 3–5 bullets, most important change first.

### 4. Build the deck
- Copy `style/deck-template.html` → `decks/deck-YYYY-MM-DD.html` (today's date).
- Replace the `<script id="deck-data">` JSON block with the real data (schema is
  documented inside the template). Fill `period_label`, `generated`, `author`
  ("Ivan Potekhin"), `tldr`, and `platforms[].items[]`. Each item links to its
  **canonical** URL. `edition_no` = this run's number (see editions registry below).
- **Past editions block:** read `data/editions.json` (the registry of prior editions)
  and pass its entries as `archive` in the deck data — each as
  `{ "no": <n>, "label": "<period_label>", "url": "deck-YYYY-MM-DD.html" }`. Use the
  **bare filename** for `url` (decks are siblings in `decks/`, so `deck-….html`
  resolves correctly; do NOT prefix `decks/`). If the registry is empty, omit `archive`
  and the section stays hidden.
- Sanity check: open/parse the file; ensure valid JSON in the data block.

### 5. Publish the deck (GitHub Pages)
- Commit the deck to `main` (via the PR flow); the Pages URL is
  `https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html`.
- Requires Pages enabled once (Settings → Pages, serve `main` / root — see
  `routines/SETUP.md`). Give Pages a moment to build before posting the link.
- This is the **only** delivery format. Do not use Slack Canvas.

### 6. Post to Slack
- Compose the message by following **`style/slack-summary.md`** exactly (greeting →
  digest intro with the date range → highlights intro → 3–4 platform lines → digest
  summary → link line with the deck URL). Rotate the wording per that file; standard
  Slack formatting (`*bold*`, `:emoji:`).
- **Delivery target (current):** send via `slack_send_message` to **Ivan's DM** —
  `channel_id = U065VBRHYV7`. This is the configured destination; it posts from Ivan's
  own Slack identity. To change where the digest goes (e.g. a team channel), edit the
  `channel_id` in this line — nothing else changes.
- **Verify it succeeded** (tool returned ok / message link). If it failed, STOP —
  do not mark anything presented; leave the items for the next run.

### 7. Mark presented (only after success)
- For every item included in the deck, set `presented: true` in `data/updates.json`.
- **Append this edition to `data/editions.json`** so the next run lists it under Past
  editions: `{ "no": <n>, "date": "YYYY-MM-DD", "period_label": "<period_label>",
  "url": "deck-YYYY-MM-DD.html" }` (`no` = previous max + 1).
- Commit `data/updates.json` + `data/editions.json` + `decks/deck-YYYY-MM-DD.html`
  with a message like `present: deck YYYY-MM-DD, M items -> Slack`.
- Push to `claude/present-run`, then open a pull request into `main` and merge it
  (Pages serves `main`, so the deck link resolves after merge).

---

## Open config (single source of truth — edit here, routines pick it up)
- **Slack delivery target:** Ivan's DM, `channel_id = U065VBRHYV7` (step 6). Change this
  value here to redirect the digest to a channel later.
- **Delivery mechanism:** GitHub Pages only (no Slack Canvas).
- **Deck format:** the "Zine" HTML template in `style/` (Google Fonts + local sticker
  PNGs). PPTX not used.
- **Author credit (footer):** "Ivan Potekhin" (`author` in deck data).
- **Empty edition:** if nothing qualifies, no deck and no Slack post (step 2a).

---

## Guardrails
- **`presented` is set ONLY after a confirmed Slack post.** A failed post must leave
  items un-presented so nothing is silently lost.
- **Idempotent:** re-running before a successful post just rebuilds; re-running after
  finds nothing new to present.
- Never invent updates — the deck contains only what's in `updates.json`.
- Keep the channel message short; depth lives in the deck / canvas.
