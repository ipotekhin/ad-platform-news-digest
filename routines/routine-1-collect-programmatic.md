# Routine 1 — Collect — Programmatic

**Cadence:** 1×/week (set manually in the routine UI; the schedule can change at any
time without affecting this logic — the collection window stays a fixed 2 weeks).
**Job:** discover new updates useful to a programmatic trader on **our programmatic
platforms** — The Trade Desk, DV360, Amazon DSP, Microsoft programmatic, CTV — and
append only genuinely new items to `data/updates.json` with `"track": "programmatic"`.

**Two collect routines share this repo.** This one reads `sources-programmatic.yaml` and
filters by `criteria-programmatic.md`. Its twin, **Routine 1 — Collect — Performance**
(`routines/routine-1-collect-performance.md`), reads `sources.yaml`, filters by
`criteria.md` and writes `"track": "performance"` into the same `data/updates.json`.
Performance-account news — Google Ads, the Microsoft Advertising account (Search,
Shopping, PMax, Audience ads), ChatGPT Ads, Meta, TikTok, LinkedIn — is **its** job:
drop it here. Some pages are read by both routines (PPC Land, the Microsoft Advertising
blog); each keeps only its own track's stories.

**Schedule it at least a couple of hours apart from the performance routine** — both
append to `data/updates.json`, and two runs merging at once would conflict.

The routine UI holds only a short pointer to this file. All logic lives here.

---

## Routine UI call (what to paste into the trigger prompt)

```
You are Routine 1 — Collect — Programmatic, for the ad-platform-news-digest repo.
Read routines/routine-1-collect-programmatic.md and follow it exactly. That file is the
source of truth for this run — its scope, rules, outputs and stop conditions are kept
current, and if anything in this prompt ever disagrees with it, the file wins. Work on a
claude/-prefixed branch off main and land the result through a pull request into main,
as the file specifies. Stay inside this routine's job — collect the programmatic track
only; do not do the performance collection or Routine 2's work.
```

> **Why this prompt says so little.** The stored prompt in the routine UI is **not**
> version-controlled and nobody re-reads it — so anything concrete written here silently
> rots the moment this file changes. It happened once already: the Routine 2 prompt still
> named Ivan's DM as the destination months after the two-channel split. So the prompt
> carries only what can never go out of date — which routine this is, which file to obey,
> that the file outranks the prompt, and the branch/PR mechanic the platform forces. Every
> destination, threshold, cap, language, output and step lives **in this file**, where it
> can be edited and reviewed. **Never add a specific to the prompt** — add it here instead;
> if you find yourself wanting to, that is the signal it belongs in this file.

> **Branch note:** `main` is the mainline. Routines can only push to `claude/`-prefixed
> branches, so each run branches from `main`, pushes to its `claude/*` branch (the
> session's assigned name, or `claude/collect-run`), merges into `main` via pull
> request; GitHub then deletes the branch automatically (step 8).

---

## Preconditions

- **Network:** the collection environment MUST allow outbound HTTPS to every source
  domain in `sources-programmatic.yaml`. If CONNECT is denied (403 policy denial) the fetch fails —
  this is an environment config issue, not a code bug. See repo `README.md` → Network.
- **Fetch method:** fetch with an **in-session client** (`curl` / `requests`), **not**
  the managed `WebFetch` tool — with egress open, curl gets HTTP 200 from the source
  domains, but `WebFetch`'s fetcher is blocked by the sites' anti-bot. The full fetch
  rules (headers, timeout, retries, where dates come from) are in the header of
  `sources-programmatic.yaml` → FETCHING.
- **Slack** connector (used only for the step-9 alert to Ivan). If it is unavailable,
  the run still collects and lands its results; say in the PR body that the alert could
  not be sent.
- Repo is checked out; `sources-programmatic.yaml`, `criteria-programmatic.md`,
  `data/updates.json`, `data/state.json`, `docs/SOURCES_LOG.md` exist.

---

## Steps

### 1. Load state and set the window
- Read `data/updates.json` → the existing items (this is the base you dedup against —
  **check the base first, before visiting any source**).
- Read `sources-programmatic.yaml` → the source list (ignore `sources.yaml`, it is the
  performance track's). Read `data/state.json` (a record of the last successful
  collection date per source, shared by both tracks; informational).
- **Collection window = the last 2 weeks.** `window_start = today − 14 days`, minus a
  **2-day safety margin** (so effectively today − 16 days). **Ignore anything published
  before `window_start`.** The window is always these ~2 weeks — never an open-ended or
  earlier period. Dedup (below) prevents re-adding items already collected in a prior
  run whose window overlaps this one.

### 2. Visit each source
Read the header of `sources-programmatic.yaml` first (STATUS, FETCHING, DOMAIN NOTES),
then go
through every source. **What you do depends on its `validation`:**

- **`readable` / `pending`** — fetch and read it normally (below), with the retries
  FETCHING allows.
- **`blocked` / `broken`** — make **one** request to `url`. If it fails the same way,
  **skip the source and move on**: no other URLs, feeds, sitemaps, user agents or
  headless browsers, and nothing written about it anywhere (its status is already on
  record). If it reads, the source has recovered — read it normally.

As you go, note each source's **outcome** for step 7: `readable` (the entry page was
fetched and a dated article list parsed from it — even if nothing falls in the window),
`blocked` (the entry page could not be fetched) or `broken` (fetched, but no dated
article list could be parsed). Keep the failure detail — HTTP code, `cf-mitigated`
header, curl error — for the log and the alert.

**Reading a source** (every source is a dated feed; per-source structure is in its
`note` and in DOMAIN NOTES):
- Fetch the entry-point `url` with an in-session client (curl / requests), not WebFetch.
- Parse the index for **article links with dates**; keep only links **published within
  the window** (step 1). Page back where the note says to, until the cards precede
  `window_start`.
- **Index without dates** (The Trade Desk Resource Desk): the list is newest first, so
  open the cards in order, read each article's date, and stop at the first card older
  than `window_start`. You never need the catalogue's last page.
- **Always open the article page itself** — never judge relevance or write a summary
  from the index headline alone. Read the body, then decide (per
  `criteria-programmatic.md`) and, if
  it belongs, extract `url`, `published` (JSON-LD `datePublished` / `og:`), and enough
  body text for the title + summary. The one exception is a high-volume page whose note
  says to screen titles (PPC Land): a card that is plainly about none of our five
  programmatic platforms may be dropped unread; anything that *might* be ours is opened.
- If a headline has no date on the index, still open the article to get its real date
  before applying the window.
- **Set every item's `platform` by where the change lands** (criteria-programmatic.md →
  product test): `ttd`, `dv360`, `amazon_dsp`, `microsoft_programmatic` or `ctv`. For a
  source with a fixed platform that is usually its own value; for `platform: mixed`
  sources (AdExchanger, Digiday, PPC Land, MediaPost, Adweek) decide per article — a CTV
  story that names one of our DSPs takes that DSP, `ctv` is for a publisher's own change.
  Never write `mixed`. Drop everything that is not about our five platforms — other
  DSPs / SSPs / vendors, business news, performance-account news — filter hard. Dedup
  (Layers 1–2) still applies, and since the trade press repeats official posts, prefer
  an official post as canonical when the same story is already stored.

The 2-week window applies to **every** source — nothing is exempt.

### 3. Relevance gate (light, at collection time)
Apply `criteria-programmatic.md` **loosely** here — *lean include*. The strict filter
happens in Routine 2. Drop only obvious non-fits (business news, case studies, research
pieces, vendor products, developer-only API notes).

### 4. Dedup — Layer 1 (exact, no LLM)
For each surviving candidate, compute the **canonical URL** — exactly these steps, in
this order, so every run gets the same string:
1. lowercase the scheme and host;
2. drop `utm_*` and other tracking params (`ref_`, `ref`, `fbclid`, `gclid`, `mc_cid`,
   `mc_eid`); keep every other query param;
3. drop the `#fragment` — **unless** the source's note says its items are sections of
   one page (e.g. DV360 API release notes): then the fragment *is* the item, keep it;
4. drop the trailing slash (`…/slug/` → `…/slug`).

- **Stable ID** = `sha1(canonical_url)` truncated to 12 hex chars. Hash the canonical
  string, never the raw href (the record's `url` stays the raw href — see step 6).
- If a source has no per-item URL: `id = sha1(source + published + title)[:12]`.
- **Already collected → skip** if the ID exists in `data/updates.json` **or** any stored
  record's `url`, canonicalised the same way, equals this candidate's canonical URL.
  (Some older records were hashed from the raw href, so the ID check alone can miss
  them; the URL check cannot.)
- Either track counts: one URL is one record, so a story the performance routine
  already holds is not added again here.

### 5. Dedup — Layer 2 (semantic, LLM)
Take the candidates that survived Layer 1 **plus** the titles+summaries already stored
in `updates.json` within the current 2-week window (both tracks — a story the
performance routine already stored is not new here either). For each new candidate ask:
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
> canonical URL or the source's index page — rather than storing a broken deep link.

For each genuinely-new item, append a record to `data/updates.json`:
```json
{
  "id": "a3f9c2b1e004",
  "track": "programmatic",
  "platform": "dv360",
  "source": "Display & Video 360 Help — Announcements",
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
you actually read this run** (a record, so a persistently failing source is visible;
a source read for the first time gets its key added). This is informational — the
collection window is always the fixed last-2-weeks span from step 1, not derived from
`state.json`.

### 7. Source health — record status changes
Compare each source's **outcome** (step 2) with its stored `validation` in
`sources-programmatic.yaml`:

| Stored → outcome | `sources-programmatic.yaml` | `docs/SOURCES_LOG.md` | Alert (step 9) |
|---|---|---|---|
| same as before (incl. still `blocked`) | no change | nothing | no |
| `readable` / `pending` → `blocked` / `broken` | set `validation`, set `blocked_since: <today>` | one line | **yes** |
| `blocked` / `broken` → `readable` | set `readable`, remove `blocked_since` | one line | **yes** (recovered) |
| `blocked` ↔ `broken` | set `validation` (keep `blocked_since`) | one line | **yes** |
| `pending` → `readable` | set `readable` | one line | no |

- A log line follows the format at the top of `docs/SOURCES_LOG.md`: under a
  `## <today>` heading (create it at the top if missing), `` - `<key>`: <old> → <new> —
  <what you saw> `` (HTTP code / `cf-mitigated` / curl error / what the page showed).
- **Never** write "still blocked" or "nothing changed" — not in
  `sources-programmatic.yaml`, not in the log. The record is the status plus `blocked_since`.
- If a source still reads but its `note` no longer matches the page (new markup, a
  different date format, a different page size), correct the `note` and add one log
  line. Don't change a source's `url` on your own — a source whose entry page moved is
  `blocked` / `broken`, and Ivan decides on the replacement.

### 8. Commit & push
- Commit `data/updates.json`, `data/state.json`, and — when step 7 changed them —
  `sources-programmatic.yaml` and `docs/SOURCES_LOG.md`, with a message like
  `collect (programmatic): +N new items (YYYY-MM-DD)`.
- Push to your `claude/*` branch, then open a pull request into `main` and merge it.
  Do **not** build a deck; that's Routine 2.
- **Branch cleanup is GitHub's job:** the repo's "Automatically delete head branches"
  setting removes the branch when the PR merges. Don't run `git push --delete` — the
  session's git proxy refuses it (HTTP 403). If the merge did not happen, the branch
  stays and holds the run's results (step 9 alerts Ivan).

### 9. Alert Ivan — only when something needs his attention
Send **one** short `slack_send_message` to Ivan's DM (`alert_channel_id` in Open config)
**only if** this run has at least one **alert** row from step 7, or step 8 failed (push,
PR or merge — the results did not reach `main`). Otherwise send **nothing**: no "all
good" messages, no repeats of a status already reported.

**In Russian.** Keep source titles, domains, URLs, HTTP codes and product names as they
are. Standard markdown: `**bold**` (double asterisks), `:emoji:` shortcodes. One line per
change, then one line with what Ivan should do when there is something for him to do:

```
:rotating_light: **Сбор новостей (программатик): изменился статус источников** (DD.MM.YYYY)

:red_circle: **<source title>** — <что случилось, коротко>.
→ <что сделать>
:large_green_circle: **<source title>** — снова работает.

Источников в работе: <readable> из <total>.
```

What to tell Ivan, by cause:
- **Egress refusal** (curl `CONNECT tunnel failed, response 403`): "домен закрыт сетевой
  политикой окружения" → "добавь `<host>` в Allowed domains окружения
  Ad-platform-news-digest".
- **The site refuses** (403 / anti-bot challenge / 4xx / 5xx / timeout): name the code →
  "проверь, открывается ли сайт у тебя; рутина будет пробовать его раз в запуск".
- **`broken`** (page opens, no dated article list): "страница открывается, но список
  статей не распознаётся — похоже, сайт поменял вёрстку или адрес" → "пришли новый URL или
  попроси обновить описание источника в sources-programmatic.yaml".
- **Step 8 failed:** `:warning:` line — "собранные новости не попали в main: <ошибка>" →
  "нужен ручной merge PR <link>" (or what blocked it).

If the send fails, say so in the run's final report; don't retry in a loop.

---

## Open config (single source of truth — edit here, the routine picks it up)
- **Alert destination:** `alert_channel_id = U065VBRHYV7` (Ivan's DM — the same DM
  Routine 2 sends its backlog note to). Alert language: **Russian**.
- **Alert triggers:** a source's status changes into or out of `blocked` / `broken`, or
  the run fails to land its results on `main` (step 9). Nothing else.
- **Collection window:** last 14 days + 2-day margin (step 1).
- **Retries:** up to 2 extra attempts on a transient failure of a readable / pending
  source; exactly one request per run for a blocked / broken one
  (`sources-programmatic.yaml` → STATUS / FETCHING).

---

## Guardrails
- **Never** set `presented` — that is Routine 2's job, after a successful Slack post.
- **Idempotent:** re-running the same day must not create duplicates (Layer 1 guards this)
  or repeat an alert (the status is already recorded on `main`).
- **Fail safe:** if a source errors, record its outcome, skip it, and keep going — one bad
  source must not abort the whole run or corrupt `state.json` (only advance a source's
  date if you actually read it).
- `sources-programmatic.yaml` holds the **current** state only; history goes to
  `docs/SOURCES_LOG.md`, and only when something changed. Never edit `sources.yaml` —
  it belongs to the performance routine.
- Write `"track": "programmatic"` on every record you add, and never edit a record of
  the performance track.
- Keep summaries factual and specific (what changed, who it affects) — no fluff.
