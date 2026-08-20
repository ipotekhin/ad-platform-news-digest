# Routine 2 — Presentation

**Cadence:** bi-weekly. Set the trigger to fire **every Wednesday** (`0 7 * * 3`); the
step-0 cadence gate (min 12 days between editions) makes the digest land every 2 weeks
and self-corrects after any missed run. Changing the schedule doesn't affect the logic.
**Job:** turn the not-yet-presented updates into a readable deck, post a summary to
Slack, then mark those items `presented: true` — **only after** a successful post.

The routine UI holds only a short pointer to this file. All logic lives here.

---

## Routine UI call (what to paste into the trigger prompt)

```
You are Routine 2 (Presentation) for the ad-platform-news-digest repo.
Read routines/routine-2-presentation.md and follow it exactly. First apply the step-0
cadence gate and the empty-edition guard — if either says stop, do nothing. Otherwise
branch from main to a claude/present-run branch, expire stale backlog, select with the
per-platform caps, build the deck, deliver the per-team Slack messages (Search / Social,
each with its own `?team=` deep link) to the targets in the doc's Open config, then set
presented:true on the delivered items, DM Ivan the backlog status note, commit, push,
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

### 0. Cadence gate (enforces "every 2 weeks" even if the trigger fires weekly)
Read `data/editions.json`. If it has entries, take the **most recent** edition date. If
**fewer than 12 days** have passed since that date, **STOP the whole run** — do nothing
(no deck, no Slack, no commit). This makes the digest land exactly bi-weekly even when
the schedule is set to fire every Wednesday. If the registry is empty, continue (first
edition). Threshold lives in Open config.

### 1. Expire stale backlog, then select candidates
- **Expiry pass first.** For every item with `presented == false` and no `expired`
  flag, if its `first_seen` is **older than 28 days** (from today), set `expired: true`.
  Expired items are permanently out of the running — a digest is about recent news, and
  this stops an un-shown item (esp. high-volume Google) from piling up forever. Threshold
  in Open config.
- **Select:** take every item where `presented == false` **and** `expired` is not true.

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

### 3. Balance & per-platform cap (keeps the deck fair across teams)
The digest serves two audiences — **Search** (Google Ads, Microsoft/Bing, ChatGPT Ads) and
**Social** (Meta, TikTok, LinkedIn) — so no single platform may dominate.
- Group by platform, **Search block first, then Social block** (so All-news reads
  Search → Social): **Google Ads → Microsoft/Bing → ChatGPT Ads → Meta → TikTok →
  LinkedIn** (omit empty). Emit the `platforms` array in the deck data in exactly this
  order — the deck renders platforms in data order, no re-sorting.
- Within each platform, order **high → medium → low** impact.
- **No overall cap.** **Per-platform cap = 6** items: take the top 6 by impact.
- **`high`-impact items are always included** — if a platform has more than 6 highs, raise
  its cap to fit them, up to an absolute **max of 8** per platform. (So a platform ships
  6 normally, up to 8 only when highs demand it.)
- Items beyond the cap stay `presented: false` and carry to a future edition (subject to
  the 28-day expiry in step 1). Caps live in Open config.
- Write a **TL;DR** of 3–5 bullets, most important change first (across platforms).
  **Keep each bullet short — this block is skimmed, not read.** Hard rules:
  - **One sentence, max ~20 words / ~140 characters.** If it needs a semicolon, an em-dash
    aside or a second clause to fit, it is too long — cut it.
  - **Lead with the change itself**, in plain words: *what changed + who it hits*. Drop
    the supporting detail (exact dates, version numbers, rollout mechanics, caveats) —
    that lives on the card below, and the reader opens the card for it.
  - No preamble ("Google has announced that…") — start at the change.
  - Example of the right length: *"Local Services Ads fold into Performance Max as
    pay-per-lead from August; historical LSA reports don't carry over."*

### 4. Build the deck
- Copy `style/deck-template.html` → `decks/deck-YYYY-MM-DD.html` (today's date).
- Replace the `<script id="deck-data">` JSON block with the real data (schema is
  documented inside the template). Fill `period_label`, `generated`, `author`
  ("Ivan Potekhin"), `tldr`, and `platforms[].items[]`. Each item links to its
  **canonical** URL. `edition_no` = this run's number (see editions registry below).
- **`period_label` = the digest window, NOT the min/max of the items' `published`
  dates.** Compute it as: **start** = the previous edition's date + 1 day (from
  `editions.json`); for the **first** edition, `generated − 14 days`. **end** =
  `generated`. Format `"Mon D – Mon D, YYYY"` (e.g. `Jul 20 – Aug 3, 2026`). This keeps
  the label a clean ~2-week span independent of individual item dates.
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

### 6. Post to Slack — one message per team (Search / Social)
The digest is announced to **two teams**, each in its own channel, with a message adapted
to that team and a link that opens the deck **already filtered to their view**. The deck
itself is a single file with all platforms; the two links only differ by a `?team=` param.

- **Split the included (capped) items into two teams by platform:**
  - **Search** = Google Ads + Microsoft/Bing + **ChatGPT Ads**.
  - **Social** = Meta + TikTok + LinkedIn.
- **For each team that has at least one included item**, compose a message by following
  **`style/slack-summary.md`** exactly (greeting → digest intro with the date range →
  highlights intro → **one highlight line per platform** → digest summary → link). The
  highlights and the summary count use **only that team's platforms**. Rotate the wording
  per that file; use **standard-markdown** Slack formatting — **`**bold**`** (double
  asterisks; a single `*word*` renders as *italic* in this connector) and `:emoji:`.
- **One highlight per platform (not a list of updates).** For each platform in the team
  that has items, pick its **single hottest** item (highest impact; break ties by
  recency) and write one line for it. Don't list several updates from the same platform —
  the deck holds the full detail; the message is a teaser of the top change per platform.
- **Deep-linked deck URL:** append `?team=search` or `?team=social` to the deck URL so the
  link opens straight to that team's filtered view (readers can switch to *All news* in the
  header). E.g. `https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html?team=social`.
- **Send** each team's message via `slack_send_message` to that team's channel from Open
  config (`search_channel_id` / `social_channel_id`). Posts from Ivan's own Slack identity.
- **Empty team = skip:** if a team has **no** included items this edition, **do not post to
  that team's channel** — its digest simply doesn't happen this time. (If *both* teams are
  empty, the run already stopped at step 2a.)
- **Verify each send** (tool returned ok / message link). If a team's send fails, leave
  **that team's** items un-presented for the next run; a success for the other team still
  counts. Only mark presented (step 7) for items whose team message was confirmed.

### 7. Mark presented (only after success)
- For every item **whose team message was confirmed delivered** in step 6, set
  `presented: true` in `data/updates.json`. Items in a team whose send failed — or a team
  skipped for being empty — stay `presented:false` for the next run. (Items marked
  `expired: true` in step 1 keep that flag.)
- **Append this edition to `data/editions.json`** (once at least one team's message was
  delivered) so the next run lists it under Past editions: `{ "no": <n>, "date":
  "YYYY-MM-DD", "period_label": "<period_label>", "url": "deck-YYYY-MM-DD.html" }`
  (`no` = previous max + 1).
- Commit `data/updates.json` (presented + expired flags) + `data/editions.json` +
  `decks/deck-YYYY-MM-DD.html` with a message like `present: deck YYYY-MM-DD, M items -> Slack`.
- Push to `claude/present-run`, then open a pull request into `main` and merge it
  (Pages serves `main`, so the deck link resolves after merge).

### 8. Backlog status note to Ivan (internal — always to Ivan's DM)
After the digest is delivered, send a **separate short** `slack_send_message` to
**Ivan's DM (`channel_id = U065VBRHYV7`)** — this is an internal ops note and always goes
to Ivan regardless of where the digest itself is sent. Summarize the backlog pressure so
Ivan can decide whether to raise a platform's cap later:
- **Included:** total shipped this edition.
- **Deferred** (still `presented:false`, hit the per-platform cap): count **by platform**.
- **Expired this run** (aged past 28 days, retired): count **by platform**.
If nothing was deferred or expired, one line ("no backlog — everything relevant shipped")
is enough. Keep it compact; this note is for Ivan only, not the teams.

---

## Open config (single source of truth — edit here, routines pick it up)
- **Slack delivery targets (two teams):** the digest goes to the team channels —
  `search_channel_id = C04PJUZMN91` (**#paid-search-team**) and
  `social_channel_id = C04AU6G17GT` (**#paid-social-team**). Both are **private** channels,
  so Ivan (whose identity the connector posts under) must be a member of each. Change an ID
  here to redirect a team later (nothing else changes). Team→platform mapping:
  **Search** = Google Ads + Microsoft/Bing + ChatGPT Ads · **Social** = Meta + TikTok +
  LinkedIn.
- **Deep-linked digest URL:** each team message links to the *same* deck with
  `?team=search` / `?team=social` appended, so it opens on that team's filtered view; the
  reader can switch to *All news* in the header (step 6).
- **Empty team:** a team with no items this edition gets **no post** (step 6). If both
  teams are empty, no edition at all (step 2a).
- **Delivery mechanism:** GitHub Pages only (no Slack Canvas).
- **Deck format:** the "Zine" HTML template in `style/` (Google Fonts + local sticker
  PNGs). PPTX not used.
- **Author credit (footer):** "Ivan Potekhin" (`author` in deck data).
- **Empty edition:** if nothing qualifies, no deck and no Slack post (step 2a).
- **Cadence gate:** min **12 days** between editions (step 0). Set the trigger to fire
  every Wednesday (`0 7 * * 3`); the gate makes the actual digest land bi-weekly.
  Change the 12-day threshold here to adjust cadence.
- **Per-platform cap:** **6** items (up to **8** to fit all `high`-impact); **no overall
  cap** (step 3). Raise a platform's cap here if its backlog note shows steady pressure.
- **Backlog expiry:** un-presented items older than **28 days** (`first_seen`) are retired
  (`expired:true`, step 1). Raise this if too much is aging out.
- **Backlog status note:** step 8 always DMs Ivan (`U065VBRHYV7`) a short deferred/expired
  summary by platform, even when the digest later goes to a team channel.

---

## Guardrails
- **`presented` is set ONLY after a confirmed Slack post.** A failed post must leave
  items un-presented so nothing is silently lost.
- **Idempotent:** re-running before a successful post just rebuilds; re-running after
  finds nothing new to present.
- Never invent updates — the deck contains only what's in `updates.json`.
- Keep the channel message short; depth lives in the deck / canvas.
