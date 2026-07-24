# BUILD_LOG — how this system was built & how it works

> Durable handoff/context doc. Captures every decision and rule so a fresh session can
> pick up without re-deriving. Operational quick-ref: [`CLAUDE.md`](../CLAUDE.md).
> Design/overview: [`README.md`](../README.md).

## 1. What this is
Two scheduled Claude **routines** + this GitHub repo as **persistent memory**, producing
a bi-weekly ad-platform news digest as a self-contained HTML "Zine" deck on GitHub Pages,
announced via a Slack DM. Everything is **data-driven and repo-controlled**: all tuning
happens by editing repo files, never by rewriting the routine trigger prompts.

- **Routine 1 — Collect** (weekly): fetch sources → filter → dedup → append to
  `data/updates.json`. Never builds a deck.
- **Routine 2 — Presentation** (every 2 weeks): select un-presented items → build deck →
  publish to Pages → Slack DM → mark `presented:true` + append `data/editions.json`.

Each run branches from `main` to a `claude/*` branch, commits, opens a PR, and merges.

## 2. Sources (`sources.yaml`, 10 — all `readable`)
Domains that MUST be in the run environment's egress allowlist (exact hosts):
`blog.google`, `support.google.com`, `searchengineland.com`, `about.fb.com`,
`socialbee.com`, `www.socialmediatoday.com`, `blogs.bing.com`.

| key | platform | type | cadence |
|---|---|---|---|
| google_ads_commerce_blog | google_ads | official | feed |
| google_ads_announcements | google_ads | official | feed |
| searchengineland_google | google_ads | aggregator | feed |
| meta_newsroom_tech | meta | official | feed |
| searchengineland_meta | meta | aggregator | feed |
| searchengineland_tiktok | tiktok | aggregator | feed |
| socialbee_linkedin_updates | linkedin | aggregator | **roundup** |
| socialmediatoday_linkedin | linkedin | aggregator | feed |
| bing_blogs | bing | official | feed |
| searchengineland_microsoft | bing | aggregator | feed |

Sources that couldn't be read (official Meta business news, TikTok blog, LinkedIn Ads
blog, Google Search blog) were **dropped** and are intentionally not tracked.

## 3. Collection rules (Routine 1)
- **Fetch with an in-session client** (curl/requests/browser), NOT the managed WebFetch
  tool (its fetcher is anti-bot-blocked even when egress is open).
- **Window = fixed last 2 weeks** (`today − 14d`, ~2-day margin). Earlier periods are
  never revisited. This holds even though R1 runs weekly (dedup handles overlap).
- **cadence:feed** — parse dated article links; keep those in-window; **always open the
  article** to read the body + real date before judging relevance/writing the summary.
- **cadence:roundup** (SocialBee) — a monthly digest that surfaces items late, so the
  strict window does NOT apply. Read only the **current-month section** (+ previous month
  only in the first 7 days of a month); take ads items not already in the base
  (dedup id = `source + month + sha1(title)`); date them by the roundup month.
- **URL rule (critical):** store the source href **verbatim** — never construct/guess a
  slug — and **verify it resolves (2xx/3xx)** before saving; else fall back to a working
  link (canonical article / source index / roundup page). (A hallucinated LinkedIn slug
  once shipped a 400 link — this rule prevents recurrence.)
- **Dedup:** Layer 1 canonical-URL exact (`sha1(canonical_url)[:12]`, strip utm/#/trailing
  slash); Layer 2 semantic (LLM), preferring official over aggregator.
- **Item fields:** clear `title` (headline or a better one we write), 1–3 sentence
  `summary` (what changed + why it matters), `category`, `impact`, `published`.
- **state.json** advances `last_collected` only for sources actually read (informational;
  window is fixed, not derived from it).

## 4. Relevance (`criteria.md`)
Only **ad-account / ads-manager** changes (features, formats, reporting, targeting,
bidding, algorithms, policy/eligibility, deprecations, ads API). Excludes consumer/PR,
case studies, generic tips, organic-social. Per-platform GRAB/IGNORE table included.
R1 applies it loosely (lean-include); R2 applies it strictly.

## 5. Presentation rules (Routine 2)
- **Cadence gate (step 0):** trigger fires **every Wednesday** (`0 7 * * 3`); if the
  newest `editions.json` entry is < 12 days old, the run does nothing → digest lands
  exactly every 2 weeks and self-corrects after a miss.
- **Empty-edition guard:** if nothing un-presented+relevant, build/post nothing.
- **Select:** all `presented:false` items (R1 already bounded the window).
- **Order:** platforms Google Ads → Meta → TikTok → LinkedIn → Bing; within, high→med→low.
- **>12 rule:** if the deck would exceed ~12 items, drop/group `low`-impact ones to keep
  it tight. Dropped items stay `presented:false` and carry to a future edition (not lost).
- **Deck:** copy `style/deck-template.html` → `decks/deck-YYYY-MM-DD.html`; fill the
  `#deck-data` JSON. Publish to GitHub Pages (only delivery — no Slack Canvas). Each
  edition is a **new permanent URL**; old ones stay live.
- **Past editions:** pass prior editions as `archive` in deck-data (from
  `data/editions.json`), url = **bare `deck-YYYY-MM-DD.html`** (sibling). After a
  successful post, append the new edition to `data/editions.json`.
- **Slack:** compose per `style/slack-summary.md` (greeting → intro w/ date range →
  3–4 highlights → summary → link), send via `slack_send_message` to **Ivan's DM
  `channel_id = U065VBRHYV7`** (posts as Ivan). Change the channel_id in `routine-2`
  Open config to redirect later.
- **presented:true** is set ONLY after a confirmed delivery; then commit + PR + merge.

## 6. Deck design (`style/deck-template.html`, "Zine")
- Data-driven: only the `#deck-data` JSON changes per edition. Loads Google Fonts
  (Playfair Display, Courier Prime) + local PNG stickers from `assets/stickers/` — the
  only external/local requests. Light/cream editorial look, no dark mode.
- **Footer:** "Made for Performance Team with 💜 by Ivan Potekhin".
- **Stickers** (`assets/stickers/`, 35 PNGs; `arrows-grid` and `flag` were removed):
  randomized per edition, seeded by the edition date (deterministic). Pools by slot —
  `POOL_GENERAL` (hero/tldr/archive/cards; no arrows-grid/flag/folder-star/paperclip),
  `POOL_HERO` = general + paperclip, `POOL_WRAP` = curated set that reads on the dark-blue
  "THAT'S A WRAP" block (only place folder-star & paperclip-green-pink appear). Cards
  never use paperclip. Positions/sizes fixed so nothing shifts. Documented in
  `style/README.md`. (Open question: whether folder-star should stay in the wrap block or
  be removed entirely — currently kept per Ivan's wrap allow-list.)

## 7. Schedule
- Routine 1: weekly, e.g. Monday `0 7 * * 1`.
- Routine 2: every Wednesday `0 7 * * 3` (cadence gate makes it bi-weekly).
Schedules are set manually in the routine UI and do not affect any logic.

## 8. Environment prerequisites (Ivan-managed)
- Egress allowlist: the 7 hosts in §2 (exact host, incl. `www.` where the site
  canonicalizes to www — e.g. Social Media Today).
- Slack connector enabled in the routine environment (posts as Ivan).
- GitHub Pages serving `main` (deck URLs: `https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html`).
- Model: run routines on **Opus 4.8** (judgment-heavy; low volume).

## 9. Testing (2026-07-24) — all passed
- R1 run 1: collected 11. Run 2 (after roundup fix): +3 incl. LinkedIn → 14. Run 3
  (after SMT domain fix): +0, no dupes → **idempotency confirmed**. All 10 sources read.
- R2: cadence gate passed (first edition), Zine deck built (12 items after strict filter
  + >12 trim), DM delivered, `presented` set, edition 1 registered. Sticker pools +
  Past-editions block verified via render.
- Fixes made during testing (by PR): roundup windowing (#11/#12), Social Media Today host
  mismatch → readable (#14), sticker pools + reset (#19), broken LinkedIn URL + verify
  guardrail (#22).

## 10. Current state & next step
- `data/updates.json`: 14 items (12 `presented:true`, 2 `false`); `editions.json`: edition 1;
  `decks/deck-2026-07-24.html` live; a temporary `decks/deck-mock-pastedition.html` exists
  only to test the Past-editions link (to be removed in cleanup).
- **PENDING — production cleanup** (Ivan to confirm): reset for the first real weekly run —
  `updates.json → []`, `editions.json → []`, remove the test/mocks decks, reset
  `state.json` `last_collected → null`. Then the system runs hands-off on schedule.
- Open question: folder-star sticker (keep in wrap block vs remove entirely).
