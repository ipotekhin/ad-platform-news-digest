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

## 2. Sources (`sources.yaml`, 13 — all `readable`, all `feed`)
Domains that MUST be in the run environment's egress allowlist (exact hosts):
`blog.google`, `support.google.com`, `searchengineland.com`, `about.fb.com`,
`www.socialmediatoday.com`, `blogs.bing.com`, `ppc.land`, `www.seroundtable.com`.

| key | platform | type | cadence |
|---|---|---|---|
| google_ads_commerce_blog | google_ads | official | feed |
| google_ads_announcements | google_ads | official | feed |
| searchengineland_google | google_ads | aggregator | feed |
| meta_newsroom_tech | meta | official | feed |
| searchengineland_meta | meta | aggregator | feed |
| searchengineland_tiktok | tiktok | aggregator | feed |
| socialmediatoday_linkedin | linkedin | aggregator | feed |
| bing_blogs | bing | official | feed |
| searchengineland_microsoft | bing | aggregator | feed |
| seroundtable_chatgpt_ads | chatgpt | aggregator | feed |
| searchengineland_chatgpt | chatgpt | aggregator | feed |
| ppcland_search | mixed (google_ads/bing/chatgpt) | aggregator | feed |
| ppcland_social | mixed (meta/tiktok/linkedin) | aggregator | feed |

**SocialBee removed (2026-08-04):** its monthly-`roundup` cadence let month-old LinkedIn
items past the 2-week window; Social Media Today covers the same LinkedIn announcements as
a dated feed, so SocialBee was dropped. **No `roundup` source remains — the 2-week window
now applies to every source.**

**ChatGPT Ads added (2026-08-17):** a new Search-team platform (`platform: chatgpt`,
deck name "ChatGPT Ads"). **Two tracked sources**, both validated readable via curl that
day: Search Engine Roundtable's dedicated **ChatGPT Ads** category (highest signal) and
SEL **/platforms/openai/chatgpt**. The SEL page mixes real Ads news with SEO/organic and
evergreen how-to content, so filter hard per `criteria.md`. SEL **/library/ppc** was
evaluated too (readable) but **dropped**: it is a broad PPC feed whose ChatGPT items also
appear on the two tracked sources, so it only added duplicates plus evergreen content.
`teamOf()` needs no change (anything not Meta/TikTok/LinkedIn ⇒ Search); an explicit
`chatgpt`/`openai` icon family was added to `PF_FAM`.

**Mixed sources (PPC Land tag pages):** one page carries several platforms + lots of
non-ads/SEO news. The collector reads each article, sets the item's real `platform`, and
keeps only pipeline platforms within scope (`ppcland_search` → Google Ads/Bing/ChatGPT
Ads — that feed does carry ChatGPT Ads news; `ppcland_social` → Meta/TikTok/LinkedIn).
Filter hard for ads-manager relevance.

Sources that couldn't be read (official Meta business news, TikTok blog, LinkedIn Ads
blog, Google Search blog) were **dropped** and are intentionally not tracked.

## 3. Collection rules (Routine 1)
- **Fetch with an in-session client** (curl/requests/browser), NOT the managed WebFetch
  tool (its fetcher is anti-bot-blocked even when egress is open).
- **Window = fixed last 2 weeks** (`today − 14d`, ~2-day margin). Earlier periods are
  never revisited. This holds even though R1 runs weekly (dedup handles overlap).
- **cadence:feed (all sources)** — parse dated article links; keep those in-window;
  **always open the article** to read the body + real date before judging relevance/
  writing the summary. Every source is a feed now, so the 2-week window applies to all —
  nothing is exempt (the SocialBee `roundup` exemption was removed with that source).
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
bidding, algorithms, policy/eligibility, deprecations, ads API) **on the official
platforms we run** — plus the **ad-analytics stack** that feeds them (GA4, GTM, conversion
tracking, UET / Meta CAPI / LinkedIn Insight Tag, consent mode) and **external ecosystem
changes** that concretely hit ads or tracking there (iOS/Safari/Chrome privacy changes).
Excludes consumer/PR, case studies, generic tips, organic-social.
**Third-party vendors/resellers/tools are always out** (added 2026-08-20): news about
someone else's product is not our news even when it names our platforms — e.g. a vendor
gaining ad-buying access ("AdRoll opens a ChatGPT ad-buying pilot") or a clean-room
ingesting platform data ("LiveRamp now ingests Meta campaign data"). This one exclusion
applies at **both** stages, so R1 doesn't collect vendor news that R2 would always drop.
Per-platform GRAB/IGNORE table included. Otherwise R1 applies the rules loosely
(lean-include); R2 applies them strictly.

## 5. Presentation rules (Routine 2)
- **Cadence gate (step 0):** trigger fires **every Wednesday** (`0 7 * * 3`); if the
  newest `editions.json` entry is < 12 days old, the run does nothing → digest lands
  exactly every 2 weeks and self-corrects after a miss.
- **Empty-edition guard:** if nothing un-presented+relevant, build/post nothing.
- **Expiry:** un-presented items older than **28 days** (`first_seen`) are retired
  (`expired:true`) so backlog can't pile up forever.
- **Select:** `presented:false` and not `expired`.
- **Order:** Search block first, then Social — Google Ads → Microsoft/Bing → ChatGPT Ads
  → Meta → TikTok → LinkedIn; within a platform, high→med→low. Deck renders in deck-data
  order.
- **Balance (no overall cap):** **per-platform cap 6** (up to **8** to fit all `high`s),
  so no platform dominates and both teams (Search: Google/Bing/ChatGPT Ads · Social:
  Meta/TikTok/LinkedIn) stay represented. Overflow carries to a future edition.
- **Backlog status note:** after delivery, Routine 2 DMs Ivan (`U065VBRHYV7`) a short
  deferred/expired-by-platform summary (internal, always to Ivan even once the digest
  goes to a team channel).
- **Deck:** copy `style/deck-template.html` → `decks/deck-YYYY-MM-DD.html`; fill the
  `#deck-data` JSON. Publish to GitHub Pages (only delivery — no Slack Canvas). Each
  edition is a **new permanent URL**; old ones stay live.
- **Past editions:** pass prior editions as `archive` in deck-data (from
  `data/editions.json`), url = **bare `deck-YYYY-MM-DD.html`** (sibling). After a
  successful post, append the new edition to `data/editions.json`.
- **Slack (two teams):** compose per `style/slack-summary.md` (greeting → intro w/ date
  range → **one highlight per platform** → summary → link; **bold via `**double**`** —
  a single `*x*` renders italic in this connector), and send **one message per team** via
  `slack_send_message`: **Search** (Google Ads + Bing + ChatGPT Ads) → `search_channel_id`, **Social**
  (Meta + TikTok + LinkedIn) → `social_channel_id`. Each message covers only its team's
  platforms and links the deck with `?team=search|social` appended (opens on that team's
  filtered view; reader can switch to *All news*). A team with no items gets no message.
  Channels are in `routine-2` Open config — **`search_channel_id = C04PJUZMN91`
  (#paid-search-team)** and **`social_channel_id = C04AU6G17GT` (#paid-social-team)**, both
  private (Ivan must be a member); posts as Ivan. The step-8 backlog note still goes to
  Ivan's DM `U065VBRHYV7`.
- **presented:true** is set ONLY after a confirmed delivery, **per team** (a team whose
  send failed/was skipped keeps its items un-presented); then commit + PR + merge.

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
- **Interactive + responsive (template chrome, edition-agnostic):** a header **All news /
  Search / Social** filter (shows/hides platform sections + recomputes stats; team from
  `teamOf` — Meta/TikTok/LinkedIn = Social, else Search); a **deep link** `?team=search|
  social` opens the deck pre-filtered (Routine 2 sends each team its own link); motion
  (sign-off typewriter, staggered sticker entrance, section reveals) with
  `prefers-reduced-motion` fallback; decorative stickers carry `stk-in` in the markup so
  they never flash on load; a `≤640px` mobile pass (hide big floating stickers, center
  footer, brand-left/filter-right header, smaller sign-off). Desktop ≥641px unaffected.
- **Analytics:** every deck loads **Google Tag Manager** (`GTM-M2BTR42L`, in the template
  `<head>` + `<noscript>`) and pushes a `link_click` dataLayer event on every `<a>` click
  with `{link_url, link_text, link_domain, link_team, link_platform, link_location,
  active_filter}` so GTM can trigger on outbound clicks. It's in the template, so every
  generated deck inherits it — don't strip it out.

## 7. Schedule
- Routine 1: weekly, e.g. Monday `0 7 * * 1`.
- Routine 2: every Wednesday `0 7 * * 3` (cadence gate makes it bi-weekly).
Schedules are set manually in the routine UI and do not affect any logic.

## 8. Environment prerequisites (Ivan-managed)
- Egress allowlist: the hosts in §2 (exact host, incl. `www.` where the site
  canonicalizes to www — e.g. Social Media Today; `ppc.land` for the PPC Land tags).
  (GTM needs no egress here — it loads in end-users' browsers, not during the routine.)
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

## 10. Current state — LIVE (as of 2026-08-17)
- **Released.** Both routines run on schedule; **edition 1 shipped 2026-08-05**
  (`decks/deck-2026-08-05.html`, 10 items) to the real team channels.
- **Slack targets:** `search_channel_id = C04PJUZMN91` (**#paid-search-team**),
  `social_channel_id = C04AU6G17GT` (**#paid-social-team**) — both **private**, so Ivan
  must be a member of each (the connector posts as Ivan). The step-8 backlog/system note
  goes to Ivan's DM `U065VBRHYV7`.
- **Cadence in practice:** R1 collects weekly (e.g. +18 items on 2026-08-11); R2 fires
  every Wednesday but the step-0 gate (min 12 days) makes the digest land bi-weekly.
- **ChatGPT Ads onboarded 2026-08-17** — 2 sources (13 total), `platform: chatgpt`,
  routed to the **Search** team and ordered after Microsoft/Bing. Nothing collected from
  them yet; the next R1 run is the first that reads them.

## 11. Fixes from the first release test (2026-08-04)
- **period_label** now = the digest window (prev edition +1 → generated; first edition
  generated−14 → generated), not the min/max of item dates — so a stray old item can't
  stretch it to a whole month.
- **Card stickers** appear on **every 3rd card** (`i % 3 === 1`), not on all high-impact
  cards. Which PNG lands there is still seeded-random per edition.
- **Platform order:** Search block first, then Social — Google Ads → Microsoft/Bing →
  Meta → TikTok → LinkedIn (deck renders in deck-data order).
- **Slack message:** one highlight line **per platform** (the platform's single hottest
  item), not 3–4 updates. Bold uses **`**double asterisks**`** — this connector renders a
  single `*word*` as italic (the italic-platform-names bug).
- **SocialBee removed** — its monthly-roundup exemption let month-old LinkedIn items past
  the window; Social Media Today covers the same announcements as a dated feed. No
  `roundup` source remains, so the 2-week window applies to every source.
- Open question: folder-star sticker (keep in wrap block vs remove entirely).

## 12. Multi-language support (2026-08-31)
The deck now ships in **four languages** behind a header language pill —
🇺🇸 EN (base) / 🇷🇺 RU / 🇪🇸 ES / 🇷🇸 SR (**Latin** script).

**Approach: pre-translate at build time.** GitHub Pages is static, so there is no
backend and no place to hold a translation-API key. Routine 2 writes the translations
into the deck data when it builds the edition (new step 3a); the page just re-renders
from that data. No new external request, no key, no runtime dependency.

**Scope — exactly three things are translated:** card titles, card summaries and TL;DR
bullets. Everything else is fixed English chrome baked into the template and never
regenerated per edition: brand, filters, hero headline/subtitle, dates, counters,
platform names, category/impact chips, card meta, Past editions, sign-off, footer.
That keeps the interface identical edition to edition and keeps Courier Prime (which we
only use for chrome) off any non-Latin text.

**Data schema** (both optional; a missing language/item/field falls back to English, so
the two already-published editions keep rendering):
```
"tldr_i18n": { "ru":[…], "es":[…], "sr":[…] }      // same count & order as "tldr"
item.i18n:   { "ru":{"title","summary"}, "es":{…}, "sr":{…} }
```

**Behaviour:** English by default · `?lang=ru|es|sr` deep-links a language and wins over
the reader's stored choice · the pick is remembered in `localStorage` (`apd_lang`) ·
`<html lang>` is updated · `?lang=` and `?team=` compose, so a Slack team link keeps its
filter when the reader switches language.

**Template changes:** the render is now a re-callable `renderContent(lang)` that rebuilds
`#d-tldr` + `#d-body` and fires a `deck:rendered` event; the chrome script re-applies the
current team filter and shows the new nodes at once (replaying the scroll entrance would
blank the page mid-read). The sticker RNG is re-seeded from a language-independent
sub-seed at the top of every render, so all four languages get an identical sticker and
icon layout.

**Header controls** share one dropdown component: the language pill is always a dropdown
(frosted-glass popup); the news filter is the original segmented slider on desktop and
collapses to the same pill below 640px — a fixed header footprint that stays workable if
more filters arrive.

**Length rule:** limits stay defined **in English** (the base language). Translations
match the English in meaning and length; no per-language character cap.

Verified on `decks/deck-i18n-test.html` (a copy of edition 2, not in the archive) at
1280px and 390px: all four languages, Cyrillic and Serbian/Spanish diacritics, dropdowns,
filter + language composing, localStorage persistence, no card overflow, no console
errors. **Still English:** the two published editions — they get translated separately.

### Review fixes + rollout (2026-08-31)
Two rounds of fixes on the test page, then the rollout.

**Round 1.** The hero **subtitle** is translated too (new `subtitle_i18n`) — four things
now, not three. TL;DR capped at **4 bullets** for future editions (5 left an odd gap in
the two-column grid); the two published editions keep their 5. The language pill was
6px shorter than the filter slider. And the stored language preference was dropped:
language comes from `?lang=` **only**, exactly like `?team=`, so a plain link always
opens in English and a shared link renders the same for whoever opens it.

**Round 2.**
- *Pill height.* Still ~1px off with the real webfont: the pill used `line-height:1`
  while the filter buttons used `line-height:normal`, so the two heights diverged once
  Courier Prime loaded. Both controls now carry an **explicit height** (36px / 30px),
  which no font can shift.
- *No blur in the dropdown.* Proved by rendering the panel with a transparent
  background — content behind stayed razor sharp. The sticky header sets its own
  `backdrop-filter`, making it a **backdrop root**; a `backdrop-filter` nested inside one
  samples an empty backdrop. The panel is now **moved onto `<body>` while open**
  (positioned `fixed` under its pill, right-aligned) and returned to its control on
  close; scroll and resize close it, since the placement is a one-shot measurement.
  Click handling matches `.opts` rather than `.dd`, because an open panel is no longer a
  descendant of its control.
- *Slack labels* are language codes (`SR`, not `RS`); only the flag shortcode stays
  country-based (`:flag-rs:`).
- *Translation quality.* The first pass produced calques that only parse if you know the
  English — e.g. "goes video-only and on by default" → "станет только видео", where
  "видео" reads as the medium rather than the ad format. Routine 2 step 3a gained a
  **"How to translate"** block: translate the fact rather than the word order, expand
  English's compressed noun stacks and dropped words, use the platform's own localized
  terminology (but keep the borrowed term the industry actually says), never invent a
  name for a UI control, and read it back cold.

**Rollout.** Both published editions were rebuilt on the new template and translated
(edition 1: 10 items, edition 2: 15). Their TL;DR bullet counts are unchanged at 5 —
the 3–4 rule applies to future editions only. The test page `decks/deck-i18n-test.html`
is deleted. Slack messages now carry four language links embedded next to flag emoji.

## 13. Edition 3 run notes (2026-09-02)
Edition 3 shipped clean (14 items, both team channels, four language links per message).
The run surfaced two config gaps, both now closed in the docs:

- **`ipotekhin.github.io` was not in the run environment's egress allowlist**, so step 5
  could not verify the deck URL and the link was announced unchecked
  (`connect_rejected`). Ivan added the domain. The preconditions now name this domain
  explicitly, and step 5 says to verify a 2xx before posting — and, if the refusal is the
  egress policy rather than Pages, to post anyway but flag it in the step-8 note.
- **The stored trigger prompt had drifted from the doc.** It still read "deliver the Slack
  message to the target in the doc's Open config (currently Ivan's DM)" — a stale
  parenthetical from before the two-channel split, and it also predated the backlog
  expiry, per-platform caps and per-team messages. CLAUDE.md makes the doc the source of
  truth, so the run correctly posted to the two team channels, but the contradiction was
  live. Root cause: `routines/SETUP.md` carried that same old prompt, and it is what got
  pasted into the UI. Both files now hold one identical, destination-free prompt, plus a
  note that the stored copy must be re-pasted whenever the doc's block changes.

Not changed: **Google Ads backlog pressure** (6 shipped, 27 deferred, 14 retired unseen at
the 28-day expiry). The lever is its per-platform cap in Open config — left as-is pending
a decision.

### Trigger prompts made expiry-proof (2026-09-02, follow-up)
The first fix for the drifted Routine 2 prompt replaced one stale prompt with another
detailed one — it still enumerated the caps, the translations, `?team=`, the four
language links and `editions.json`, every one of which would rot the next time the repo
changed. Wrong shape, same failure mode.

Both prompts are now **contentless pointers**. They carry only what cannot expire:
which routine it is, which file to obey, that the file outranks the prompt if they ever
disagree, and the `claude/`-branch + PR mechanic the platform forces. Routine 2 also
carries "check the stop conditions first and do nothing if they say stop" — phrased
without naming which gates, because silently building and posting an edition that should
not exist is the one failure worth a line in the prompt itself.

Everything else — destinations, thresholds, caps, languages, outputs, steps — lives in
`routines/*.md`. The rule is now written into `CLAUDE.md`, both routine docs and
`SETUP.md`: **never put a specific in a prompt**; wanting to is the signal it belongs in
the routine's file. Paste the prompts once; after that, tuning the system means editing
the repo, never the routine UI.

## 14. Editions navigation + language carry-over (2026-09-02)

**Forward navigation between editions.** The strip used to be built once, at build time,
from the `archive` array — so it could only ever list editions *older* than the deck
holding it. Land on edition 2 and edition 3 was unreachable; with ten editions, opening
No. 3 stranded you with no route to 4–10.

A deck can't know at build time about an edition that doesn't exist yet, so the strip now
reads the **live registry** (`data/editions.json`, same origin on Pages) at **view time**
and lists every *other* edition, newest first. Already-published decks pick up new
editions on their own — nothing is rewritten, no git churn, and `editions.json` stays the
single source of truth. The baked `archive` array remains as the fallback for when that
read can't happen (opened from disk, offline) and still holds prior editions only, so
nothing regressed for offline copies.

Section heading is now **"Other editions"** rather than "Past editions", since it lists
both directions. Same-origin fetch of our own JSON — not a third-party request, so the
"fonts + local assets + GTM only" rule is intact.

**Language carries across internal edition links.** Links in the strip are built with the
reader's current `?lang=`, and are rebuilt when the language pill changes. This does *not*
reintroduce the stored-preference behaviour that was deliberately removed: nothing is
persisted, and the language travels because it is written into the link's own URL. A bare
link — from Slack, a bookmark, anywhere outside the page — still opens in English. The URL
remains the only source of language.

`?team=` is deliberately **not** carried: a past edition may have no platforms for that
team, which would land the reader on an empty filtered view. Language has no such failure
mode.

All three published decks were rebuilt on the updated template. Verified over HTTP:
edition 1 now lists Nos. 3 and 2, edition 2 lists 3 and 1, edition 3 lists 2 and 1;
clicking through from `?lang=ru` keeps Russian; switching language in-page updates the
hrefs; a plain link still opens English; and over `file://` each deck falls back to its
baked archive with no errors.

### Editions strip collapses to one row (2026-09-02)
With a dozen editions the strip would grow into a wall of cards, so it now shows a single
row plus a **View all N editions** toggle; expanding shows the full grid, and the toggle
flips to *Show fewer editions*.

How many cards that is gets **measured, not fixed**: the code lays them all out, reads the
first card's `offsetTop`, and hides everything sitting on a lower row. A desktop row holds
about 4–6 and a phone row exactly one, so a fixed count would behave differently on each.
Re-measured (debounced) on resize. `ARCHIVE_MIN` = 3 is the floor — collapsing a phone to
one lonely card reads as broken rather than tidy — so the three most recent editions
always stay visible even if that spills onto extra rows. The toggle appears **only** when
something is actually hidden, so with today's three editions nothing changes at all.

One regression caught in review while building this: hiding/showing cards by clearing
`style.display` wiped the inline `display:flex` that stacks each card's two lines, so
restored cards rendered "No. 11" and its date on one row. Hiding now uses an `.ed-hidden`
class, leaving the inline display alone. All three decks rebuilt and re-verified.
