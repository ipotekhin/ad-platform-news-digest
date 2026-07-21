# Routine 1 — Collect

**Cadence:** 2×/week (e.g. Wed + Fri).
**Job:** discover new, marketing-relevant updates from the ad platforms and append
only genuinely new items to `data/updates.json`.

The routine UI holds only a short pointer to this file. All logic lives here.

---

## Routine UI call (what to paste into the trigger prompt)

```
You are Routine 1 (Collect) for the ad-platform-news-digest repo.
Read routines/routine-1-collect.md and follow it exactly. Work on branch
claude/collect-run (create from the default branch), commit the updated
data/updates.json and data/state.json, and push. Do not build any deck.
```

> **Branch note:** routines can push to `claude/`-prefixed branches by default. Ivan
> runs a separate automation that merges these to `main`. If your setup lets the
> routine commit to `main` directly, it may do so — confirm before relying on it.

---

## Preconditions

- **Network:** the collection environment MUST allow outbound HTTPS to every source
  domain in `sources.yaml`. If CONNECT is denied (403 policy denial) the fetch fails —
  this is an environment config issue, not a code bug. See repo `README.md` → Network.
- Repo is checked out; `sources.yaml`, `data/updates.json`, `data/state.json` exist.

---

## Steps

### 1. Load state
- Read `data/state.json` → for each source `key`, note `last_collected` (a date).
- Read `data/updates.json` → the existing items (for dedup).
- Read `sources.yaml` → the source list.
- Compute the **collection window start** = `min(last_collected)` minus a **2-day
  safety margin**. You will ignore items older than each source's own
  `last_collected` (minus margin).

### 2. Visit each source
For each source in `sources.yaml` (skip `validation: broken`):
- Fetch the entry-point `url`.
- **If the fetch returns thin/empty content** (heavy-JS pages — Meta business news,
  TikTok are pre-flagged): try the browser path (Playwright) if available; otherwise
  fall back to that platform's `aggregator` source and set this source's
  `validation` to `needs-browser` in `sources.yaml` (commit the note).
- Parse the index for **article links with dates**. Keep only items **newer than
  that source's `last_collected` minus the 2-day margin**.
- For each candidate, fetch the article page and extract: `title`, `url`,
  `published` date, and enough body text to write a 2–3 sentence summary.

**Record per source, for this run:** readable / thin / needs-browser / broken. If a
source's real status differs from `sources.yaml`, update the file.

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
  "title": "...",
  "url": "https://...(canonical)",
  "published": "2026-07-14",
  "first_seen": "<now, ISO8601 UTC>",
  "summary": "2–3 sentences you wrote from the article body.",
  "category": "feature|beta|policy|deprecation|measurement|api|other",
  "impact": "high|medium|low",
  "collected": true,
  "presented": false
}
```
Then update `data/state.json`: set each source's `last_collected` to the **newest
`published` date you actually processed** for that source (not "today" — so a source
that published nothing keeps its old date and stays inside the window next run).

### 7. Commit & push
- Commit `data/updates.json`, `data/state.json`, and any `sources.yaml` validation
  edits with a message like `collect: +N new items (YYYY-MM-DD)`.
- Push to the `claude/`-prefixed branch. Do **not** build a deck; that's Routine 2.

---

## Guardrails
- **Never** set `presented` — that is Routine 2's job, after a successful Slack post.
- **Idempotent:** re-running the same day must not create duplicates (Layer 1 guards this).
- **Fail safe:** if a source errors, log it, skip it, and keep going — one bad source
  must not abort the whole run or corrupt `state.json` (only advance a source's date
  if you actually read it).
- Keep summaries factual and specific (what changed, who it affects) — no fluff.
