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
All sources in `sources.yaml` are validated `readable` via curl (see the file header).
For each source:
- Fetch the entry-point `url` with an in-session client (curl / requests), not WebFetch.
- Parse the index for **article links with dates**. Keep only links **published within
  the window** (step 1). For SocialBee (monthly roundup), parse the newest month
  section(s) and treat each bullet as a candidate.
- **Always open the article page itself** — never judge relevance or write a summary
  from the index headline alone. Fetch the article, read the body, then decide whether
  it belongs (per `criteria.md`) and, if so, extract `url`, `published` date, and enough
  body text to write the title + summary.
- If an index shows a headline with no date, still open the article to get its
  `datePublished` (JSON-LD / og:) before applying the window.

If a source's real behavior changes (e.g. stops returning readable content), note it
and flag to a human — do not silently drop items.

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
