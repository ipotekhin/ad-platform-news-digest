# Routine 1 — Collect

**Cadence:** 1×/week (set manually in the routine UI; the schedule can change at any
time without affecting this logic — the collection window stays a fixed 2 weeks).
**Job:** discover new, marketing-relevant updates from the ad platforms and append
only genuinely new items to `data/updates.json`.

The routine UI holds only a short pointer to this file. All logic lives here.

---

## Routine UI call (what to paste into the trigger prompt)

```
You are Routine 1 (Collect) for the ad-platform-news-digest repo.
Read routines/routine-1-collect.md and follow it exactly. Branch from main to a
claude/collect-run branch, commit the updated data/updates.json and
data/state.json, push, then open a pull request into main and merge it.
Do not build any deck.
```

> **Branch note:** `main` is the mainline. Routines can only push to `claude/`-prefixed
> branches, so each run branches from `main`, pushes to `claude/collect-run`, then
> merges into `main` via pull request.

---

## Preconditions

- **Network:** the collection environment MUST allow outbound HTTPS to every source
  domain in `sources.yaml`. If CONNECT is denied (403 policy denial) the fetch fails —
  this is an environment config issue, not a code bug. See repo `README.md` → Network.
- **Fetch method:** fetch with an **in-session client** (`curl` / `requests` / a
  browser), **not** the managed `WebFetch` tool. Validated 2026-07-21: with egress
  open, curl gets HTTP 200 from all domains, but `WebFetch`'s fetcher is blocked by the
  sites' anti-bot (403) even so. Read dates/titles from each article's JSON-LD
  (`"datePublished"`) and `og:` meta tags — most reliable across sites.
- Repo is checked out; `sources.yaml`, `data/updates.json`, `data/state.json` exist.

---

## Steps

### 1. Load state and set the window
- Read `data/updates.json` → the existing items (this is the base you dedup against —
  **check the base first, before visiting any source**).
- Read `sources.yaml` → the source list. Read `data/state.json` (a record of the last
  successful collection date per source; informational).
- **Collection window = the last 2 weeks.** `window_start = today − 14 days`, minus a
  **2-day safety margin** (so effectively today − 16 days). **Ignore anything published
  before `window_start`.** The window is always these ~2 weeks — never an open-ended or
  earlier period. Dedup (below) prevents re-adding items already collected in a prior
  run whose window overlaps this one.

### 2. Visit each source
For each source in `sources.yaml` (a source marked `validation: pending` — e.g. a newly
added one — should still be fetched: read it, and if it returns readable dated content,
flip its `validation` to `readable` with a short note; if not, mark it accordingly and
flag). Handling depends on the source's `cadence` (default `feed`):

**`cadence: feed`** (dated article stream — Google blog, Search Engine Land, Social
Media Today, Bing, etc.):
- Fetch the entry-point `url` with an in-session client (curl / requests), not WebFetch.
- Parse the index for **article links with dates**; keep only links **published within
  the window** (step 1).
- **Always open the article page itself** — never judge relevance or write a summary
  from the index headline alone. Read the body, then decide (per `criteria.md`) and, if
  it belongs, extract `url`, `published` (JSON-LD `datePublished` / `og:`), and enough
  body text for the title + summary.
- If a headline has no date on the index, still open the article to get its real date
  before applying the window.
- **`platform: mixed` sources (PPC Land tag pages).** These aggregate several platforms
  on one page, so there is no single source platform. For each in-window article: open
  it, decide relevance per `criteria.md`, and **determine which platform it's about**;
  keep it **only** if that platform is in our pipeline **and** within the source's
  `scope` note — `ppcland_search` → **Google Ads / Microsoft (Bing)** only,
  `ppcland_social` → **Meta / TikTok / LinkedIn** only. Set the item's `platform` to that
  specific value (not `mixed`). Drop everything else the page carries (SEO / zero-click /
  organic-social / creator / consumer stories) — these pages run heavy on non-ads news,
  so filter hard. Dedup (Layers 1–2) still applies, and since these are aggregators,
  prefer an official post as canonical when the same story is already stored.

**`cadence: roundup`** (periodic digest that surfaces items late — SocialBee):
- Do **not** apply the strict 2-week window, and do **not** date items by the underlying
  official post (those are often weeks old and would be dropped every run).
- **Scope (hard bound — do not scan the whole archive):** read only the **current
  calendar month** section relative to the run date; additionally read the **previous
  month** section *only* if the run date is within the first 7 days of a new month.
  Ignore all older sections.
- From that scope, take every **ads-manager-relevant** item **not already in
  `updates.json`** (dedup id = `source + month + sha1(title)`); set `published` to the
  roundup month (or an in-text date if given). Dedup guarantees no repeats across runs.
  Follow the item's source link if you need context for the summary, but keep the
  roundup date.

If a source's real behavior changes (e.g. stops returning readable content), note it and
flag to a human — do not silently drop items.

### 3. Relevance gate (light, at collection time)
Apply `criteria.md` **loosely** here — *lean include*. The strict filter happens in
Routine 2. Drop only obvious non-fits (pure PR, consumer news, inspiration posts).

### 4. Dedup — Layer 1 (exact, no LLM)
For each surviving candidate, compute the **canonical URL**:
- lowercase host, strip `utm_*` and other tracking params, strip `#fragment`, strip
  trailing slash.
- **Stable ID** = `sha1(canonical_url)` truncated to 12 hex chars.
- If a source has no per-item URL: `id = sha1(source + published + title)[:12]`.
- If the ID already exists in `data/updates.json` → **skip** (already collected).

### 5. Dedup — Layer 2 (semantic, LLM)
Take the candidates that survived Layer 1 **plus** the titles+summaries already stored
in `updates.json` within the current 2-week window. For each new candidate ask:
> "Is this the **same story** as any of these existing/other-candidate items?"

- If yes and one is an **official** source and the other an **aggregator** → keep the
  **official** as canonical; drop the aggregator duplicate.
- If yes and both are the same tier → keep the first / more complete; drop the rest.
- If no → it is genuinely new.

### 6. Write results

> **URL rule (critical — a broken link ships to the reader).** Set `url` to the EXACT
> link as it appears in the source — **copy the href verbatim; never construct,
> shorten, paraphrase, or guess any part of it (especially the slug).** Then **verify
> the URL resolves** (a request returns 2xx/3xx, following redirects) before saving. If
> it does not resolve (4xx/5xx), fall back to a link that does — the article's own
> canonical URL, the source's index page, or (for roundups) the roundup page — rather
> than storing a broken deep link. For roundup items whose cited source link is dead,
> the roundup page URL is an acceptable fallback.

For each genuinely-new item, append a record to `data/updates.json`:
```json
{
  "id": "a3f9c2b1e004",
  "platform": "google_ads",
  "source": "Google Ads & Commerce Blog",
  "title": "clear title — the article's own headline, or one you write if it reads better",
  "url": "https://...(canonical)",
  "published": "2026-07-14",
  "first_seen": "<now, ISO8601 UTC>",
  "summary": "1–3 sentences you wrote from the article body: what changed + why it matters.",
  "category": "feature|beta|policy|deprecation|measurement|api|other",
  "impact": "high|medium|low",
  "collected": true,
  "presented": false
}
```
Then update `data/state.json`: set `last_collected` to today's date **only for sources
you actually read this run** (a record, so a persistently failing source is visible).
This is informational — the collection window is always the fixed last-2-weeks span
from step 1, not derived from `state.json`.

### 7. Commit & push
- Commit `data/updates.json`, `data/state.json`, and any `sources.yaml` validation
  edits with a message like `collect: +N new items (YYYY-MM-DD)`.
- Push to `claude/collect-run`, then open a pull request into `main` and merge it.
  Do **not** build a deck; that's Routine 2.

---

## Guardrails
- **Never** set `presented` — that is Routine 2's job, after a successful Slack post.
- **Idempotent:** re-running the same day must not create duplicates (Layer 1 guards this).
- **Fail safe:** if a source errors, log it, skip it, and keep going — one bad source
  must not abort the whole run or corrupt `state.json` (only advance a source's date
  if you actually read it).
- Keep summaries factual and specific (what changed, who it affects) — no fluff.
