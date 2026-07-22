# Routine 2 — Presentation

**Cadence:** every 2 weeks.
**Job:** turn the not-yet-presented updates into a readable deck, post a summary to
Slack, then mark those items `presented: true` — **only after** a successful post.

The routine UI holds only a short pointer to this file. All logic lives here.

---

## Routine UI call (what to paste into the trigger prompt)

```
You are Routine 2 (Presentation) for the ad-platform-news-digest repo.
Read routines/routine-2-presentation.md and follow it exactly. Branch from main to
a claude/present-run branch. Build the deck, post the summary to Slack channel
#<CHANNEL>, then set presented:true on the included items, commit, push, open a
pull request into main and merge it.
```

Set `#<CHANNEL>` to the target Slack channel (Ivan to fill in — see "Open config").

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
From `data/updates.json`, take every item where `presented == false` **and**
`published` (or `first_seen`) falls in the **last 2 weeks**. Older un-presented items:
include them too (better late than dropped) unless clearly stale.

### 2. Strict relevance filter
Apply `criteria.md` **strictly** now. Drop items that don't clearly match an "Include"
rule or that hit an "Exclude" rule. For survivors, confirm/adjust `category` and
`impact` per `criteria.md`.

### 3. Order & trim
- Group by platform: **Google Ads → Meta → TikTok → LinkedIn → Bing** (omit empty).
- Within each platform, order **high → medium → low** impact.
- If > ~12 items total, group or drop `low`-impact items to keep the deck tight.
- Write a **TL;DR** of 3–5 bullets, most important change first.

### 4. Build the deck
- Copy `style/deck-template.html` → `decks/deck-YYYY-MM-DD.html` (today's date).
- Replace the `<script id="deck-data">` JSON block with the real data (schema is
  documented inside the template). Fill `period_label`, `generated`, `tldr`, and
  `platforms[].items[]`. Each item links to its **canonical** URL.
- Sanity check: open/parse the file; ensure valid JSON in the data block.

### 5. Publish the deck (GitHub Pages)
- Commit the deck to `main` (via the PR flow); the Pages URL is
  `https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html`.
- Requires Pages enabled once (Settings → Pages, serve `main` / root — see
  `routines/SETUP.md`). Give Pages a moment to build before posting the link.
- This is the **only** delivery format. Do not use Slack Canvas.

### 6. Post to Slack
- Build the message from `style/slack-summary.md` (top 3–5 items + `deck_link`
  embedded in the text), standard Markdown.
- **Test / first runs (default until Ivan flips it live):** do NOT post publicly.
  Either create a **draft** with `slack_send_message_draft`, or send the message to
  **Ivan's DM** (`slack_send_message` with `channel_id` = his user id) so he can review.
- **Live mode:** post with `slack_send_message` to the target channel `#<CHANNEL>`.
- **Verify it succeeded** (tool returned ok / message link). If it failed, STOP —
  do not mark anything presented; leave the items for the next run.

### 7. Mark presented (only after success)
- For every item included in the deck, set `presented: true` in `data/updates.json`.
- Commit `data/updates.json` + `decks/deck-YYYY-MM-DD.html` with a message like
  `present: deck YYYY-MM-DD, M items -> Slack`.
- Push to `claude/present-run`, then open a pull request into `main` and merge it
  (Pages serves `main`, so the deck link resolves after merge).

---

## Open config
- **Target Slack channel** for the digest — Ivan to fill into the trigger prompt (only
  needed for live mode; test runs use a draft or Ivan's DM).
- **Delivery mechanism:** DECIDED → **GitHub Pages only** (no Slack Canvas).
- **Deck format:** DECIDED → self-contained HTML (zero-dependency, links cleanly,
  prints to PDF on demand). PPTX not used.

---

## Guardrails
- **`presented` is set ONLY after a confirmed Slack post.** A failed post must leave
  items un-presented so nothing is silently lost.
- **Idempotent:** re-running before a successful post just rebuilds; re-running after
  finds nothing new to present.
- Never invent updates — the deck contains only what's in `updates.json`.
- Keep the channel message short; depth lives in the deck / canvas.
