# Ad Platform News Digest

Automatically collect marketing-relevant updates from the major ad platforms
(Google Ads, Microsoft/Bing, Meta, LinkedIn, TikTok), keep only what matters, and
every two weeks build an easy-to-read mini-deck that gets posted to Slack.

Discovery and relevance filtering are done by an **LLM agent** (two Claude routines),
not a fixed scraper — article URLs are unknown in advance and "is this relevant" is a
judgment call. The routines start from stable blog **index** pages and follow the fresh
links. This GitHub repo is the system's **persistent memory**.

---

## How it works

Two scheduled **routines** + this repo as state:

| Routine | Cadence | What it does |
|---|---|---|
| **[Routine 1 — Collect](routines/routine-1-collect.md)** | 2×/week | Visits each source, finds new items, dedupes them, appends to `data/updates.json`. |
| **[Routine 2 — Presentation](routines/routine-2-presentation.md)** | every 2 weeks | Filters the un-presented items, builds the deck, posts a summary + link to Slack, then marks them `presented`. |

Each routine's trigger prompt is just a short pointer to its instruction doc in
`routines/`; all logic lives in those docs.

---

## Repository layout

```
sources.yaml          # validated entry points (blog index pages) per platform
criteria.md           # relevance rules + impact scoring (human-edited)
style/
  deck-template.html  # data-driven HTML deck ("Zine" skin)
  slack-summary.md    # Slack channel message format
  README.md           # style/deck notes
assets/
  stickers/           # colorized sticker PNGs the deck template references
data/
  updates.json        # all collected updates + collected/presented flags
  state.json          # last-collected date per source (Routine 1's window)
decks/                # generated presentations (deck-YYYY-MM-DD.html)
routines/
  routine-1-collect.md
  routine-2-presentation.md
README.md / CLAUDE.md # repo context
```

### `updates.json` record schema
```json
{
  "id": "a3f9c2b1e004",
  "platform": "google_ads",
  "source": "Google Ads & Commerce Blog",
  "title": "...",
  "url": "https://...",
  "published": "2026-07-14",
  "first_seen": "2026-07-15T06:03:00Z",
  "summary": "2–3 sentences, written by the LLM",
  "category": "feature | beta | policy | deprecation | measurement | api | other",
  "impact": "high | medium | low",
  "collected": true,
  "presented": false
}
```

---

## Deduplication (why two layers)

URL-only dedup catches ~10% of real-world duplicates — the same news appears on
different sites with different URLs/wording. Heavy web-scale machinery (MinHash/LSH,
embeddings, vector DBs) is overkill for ~a dozen items twice a week, so the LLM does
the semantic dedup itself.

- **Layer 1 — exact (no LLM):** canonicalize the URL (strip `utm_*`, `#fragment`,
  trailing slash); ID = `sha1(canonical_url)[:12]`. Fallback ID when no per-item URL:
  `sha1(source + published + title)[:12]`.
- **Layer 2 — semantic (LLM):** ask the model whether a candidate is the same story as
  anything already stored in the current window. Prefer the **official** source over an
  **aggregator** (Search Engine Land) as the canonical record.

## Collection window
- Routine 1 collects over a **fixed last-2-weeks window** (`today − 14 days`, ~2 days
  margin) on every run. Earlier periods are never revisited.
- Dedup against `data/updates.json` (checked first) prevents re-adding items already
  collected in an overlapping earlier run.

## State management
- `collected` and `presented` flags live **inside each update record**.
- `data/state.json` records the last-collected date **per source** (informational — the
  window is the fixed 2 weeks above, not derived from this file).
- `presented` is set **only after** the deck is successfully posted to Slack — a failed
  run never silently loses updates.

---

## Delivery

The connected Slack MCP has **no file-upload tool**, so the deck is **not** attached.
Routine 2 posts a short text summary of the top items with the **GitHub Pages link**
embedded in the message:
- **GitHub Pages** — enable once in *Settings → Pages* (serve `main` / root); deck URL is
  `https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html`.
- Delivery is **GitHub Pages only** — no Slack Canvas.
- The Slack message posts **from Ivan's own identity** (the connector is authorized under
  his account), not a separate bot.

Deck format is a single HTML file per edition, still portable and prints cleanly to
PDF. It is not zero-request: the "Zine" skin loads Playfair Display / Courier Prime
from the Google Fonts CDN and sticker PNGs from `assets/stickers/` (repo-relative) —
both are static, low-risk, and GitHub Pages serves them fine alongside the deck.

---

## Network requirement (important)

A routine runs as a full cloud Claude Code session. **Outbound HTTPS to every source
domain in `sources.yaml` must be allowed by the run environment's egress policy**, or
the fetch fails with a `403 CONNECT` policy denial (not a code bug).

**Fetch with an in-session client (`curl` / `requests` / browser), not the managed
`WebFetch` tool.** Validated 2026-07-21: once egress is open, curl gets HTTP 200 from
every source domain, but `WebFetch`'s fetcher is still blocked by the sites' anti-bot
(403). `sources.yaml` lists **9 sources**, all confirmed readable via curl; sources that
could not be read were dropped and are not tracked.

## Status / next actions
- [x] 1. Architecture / workflow decided
- [x] 2. Repo created + sources gathered
- [x] 3. **Validate sources** — done 2026-07-21 (in-session curl). 9 sources confirmed
      `readable`; unreadable ones dropped. Recorded in `sources.yaml`.
- [x] 4. Repo scaffolded
- [x] 5. Routine instruction docs written
- [ ] 6. **Test + schedule:** manual run of Routine 1 → inspect `updates.json`; then
      Routine 2; then put both on schedule. Full runbook in
      [`routines/SETUP.md`](routines/SETUP.md).

See [`routines/SETUP.md`](routines/SETUP.md) to put the routines into operation, and
[`CLAUDE.md`](CLAUDE.md) for the operational quick-reference.
