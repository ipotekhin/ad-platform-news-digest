# Routine 2 — Presentation

**Cadence:** bi-weekly. Set the trigger to fire **every Wednesday** (`0 7 * * 3`); the
step-0 cadence gate (min 12 days between editions) makes the digest land every 2 weeks
and self-corrects after any missed run. Changing the schedule doesn't affect the logic.
**Job:** turn the not-yet-presented updates of **both tracks** — performance and
programmatic — into one readable deck for three teams (Search, Social, Programmatic), post
one summary per team to Slack, then mark those items `presented: true` — **only after** a
successful post.

The routine UI holds only a short pointer to this file. All logic lives here.

---

## Routine UI call (what to paste into the trigger prompt)

```
You are Routine 2 (Presentation) for the ad-platform-news-digest repo.
Read routines/routine-2-presentation.md and follow it exactly. That file is the source of
truth for this run — its scope, rules, outputs, delivery targets and stop conditions are
kept current, and if anything in this prompt ever disagrees with it, the file wins. Check
its stop conditions first and do nothing at all if they say stop. Work on a
claude/-prefixed branch off main and land the result through a pull request into main,
as the file specifies. Stay inside this routine's job — do not do Routine 1's work.
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

> **Branch note:** each run branches from `main`, pushes to its `claude/*` branch (the
> session's assigned name, or `claude/present-run`), merges into `main` via pull request
> (the deck must be on `main` before step 6, since that is what GitHub Pages serves);
> GitHub then deletes the branch automatically (step 7).

---

## Preconditions
- `data/updates.json`, `criteria.md`, `criteria-programmatic.md`,
  `style/deck-template.html`, `style/slack-summary.md` exist.
- Slack MCP is connected (tools: `slack_send_message`, `slack_send_message_draft`,
  `slack_search_channels`). **No file-upload tool exists** — the deck is **linked**, not
  attached. The message posts **from Ivan's own Slack identity** (the connector is
  authorized under his account), not a separate bot.
- **Delivery is GitHub Pages only.** Do not create or format anything in Slack Canvas.
- **Egress:** the run environment's network policy must allow
  **`ipotekhin.github.io`**, or step 5 cannot verify the deck URL and the link goes out
  unchecked (`connect_rejected` — this happened on 2026-09-02 and was fixed by adding the
  domain). This is an environment setting, not a code bug — same class of issue as
  the collect routines' source domains (README → Network requirement).

## Teams, tracks and platforms (used by every step below)

| Team (`?team=`, deck `team`) | Track (`updates.json`) | `platform` value → deck name, in deck order | Criteria |
|---|---|---|---|
| **Search** (`search`) | `performance` | `google_ads` → **Google Ads**, `bing` → **Microsoft Advertising**, `chatgpt` → **ChatGPT Ads** | `criteria.md` |
| **Social** (`social`) | `performance` | `meta` → **Meta**, `tiktok` → **TikTok**, `linkedin` → **LinkedIn** | `criteria.md` |
| **Programmatic** (`programmatic`) | `programmatic` | `ttd` → **The Trade Desk**, `dv360` → **DV360**, `amazon_dsp` → **Amazon DSP**, `microsoft_programmatic` → **Microsoft Programmatic**, `ctv` → **CTV** | `criteria-programmatic.md` |

Teams appear in this order — Search, Social, Programmatic — and platforms within a team
in the order listed. Slack channels per team are in Open config.

---

## Steps

### 0. Cadence gate (enforces "every 2 weeks" even if the trigger fires weekly)
Read `data/editions.json`. If it has entries, take the **most recent** edition date. If
**fewer than 12 days** have passed since that date, **STOP the whole run** — do nothing
(no deck, no Slack, no commit). This makes the digest land exactly bi-weekly even when
the schedule is set to fire every Wednesday. If the registry is empty, continue (first
edition). Threshold lives in Open config.

### 1. Expire stale backlog, then select candidates
Both tracks, the same way.
- **Expiry pass first.** For every item with `presented == false` and no `expired`
  flag, if its `first_seen` is **older than 28 days** (from today), set `expired: true`.
  Expired items are permanently out of the running — a digest is about recent news, and
  this stops an un-shown item (esp. high-volume Google) from piling up forever. Threshold
  in Open config.
- **Select:** take every item where `presented == false` **and** `expired` is not true.

### 2. Strict relevance filter — each track by its own criteria
- `"track": "performance"` items → apply **`criteria.md`** strictly.
- `"track": "programmatic"` items → apply **`criteria-programmatic.md`** strictly.

Drop items that don't clearly match an "Include" rule or that hit an "Exclude" rule. For
survivors, confirm/adjust `category` and `impact` per the item's criteria file.

- **Re-check the product test, and don't trust the `track` / `platform` fields.** The
  collect routines set them; a mis-filed item looks right while it is not. Read each
  item's own title and summary and ask which product actually changes:
  - A **performance** item that is really about DV360, The Trade Desk, Amazon DSP,
    Microsoft's programmatic stack or CTV bought through those DSPs is not a performance
    item. If it passes `criteria-programmatic.md`, **move it**: set `"track":
    "programmatic"` and the right programmatic `platform` in `data/updates.json`, then
    treat it as programmatic. Otherwise drop it. **The account a campaign is bought in
    decides** — YouTube, Video, Demand Gen and Masthead bought in Google Ads (and Reach
    Planner) stay `google_ads`, and CTV placements of Microsoft Advertising account
    campaigns stay `bing`; a mention of YouTube, TV screens or CTV is not a reason to
    move. (Campaign Manager 360, Search Ads 360 and Google Ad Manager are in
    neither track — drop.) Two DV360 items reached Google Ads readers in editions 3 and 4
    before this test existed.
  - A **programmatic** item that is really about a performance account (Google Ads, the
    Microsoft Advertising account, ChatGPT Ads, Meta, TikTok, LinkedIn) — move it the
    other way under `criteria.md`, or drop it.
  - **Before moving, check the target track:** if it already holds the same story (the
    same change, from any source), drop the mis-filed copy instead of moving it. A
    change that lands in both a performance account and a DSP may legitimately sit in
    both tracks — one record each — since each team reads its own view; leave those.
  - Within programmatic, check the platform by **where the change lands**
    (criteria-programmatic.md → product test): a CTV story that names one of our DSPs
    belongs to that DSP; Prime Video belongs to Amazon DSP and YouTube bought in DV360 to
    DV360; `ctv` is only for a CTV publisher's own ad-product change that is bought
    through several DSPs. Fix the `platform` if it is wrong.

### 2a. Empty-edition guard (IMPORTANT — stop if nothing qualifies)
If **no items** survive selection + filtering (i.e. there are zero un-presented,
relevant updates across **all three teams**), **STOP the whole run**: do **not** build a
deck, do **not** post anything to Slack, do **not** commit. This edition simply does
not happen. (Optional: leave a one-line note in the run log.) Only continue to step 3
when there is at least one qualifying update.

### 3. Balance & per-platform cap, then the hero subtitles
The digest serves three audiences — **Search**, **Social** and **Programmatic** (table
above) — so no single platform may dominate.
- Group by platform in the table's order — **Search block, then Social, then
  Programmatic**: Google Ads → Microsoft Advertising → ChatGPT Ads → Meta → TikTok →
  LinkedIn → The Trade Desk → DV360 → Amazon DSP → Microsoft Programmatic → CTV (omit
  empty platforms). Emit the `platforms` array in the deck data in exactly this order.
- Within each platform, order **high → medium → low** impact.
- **No overall cap.** **Per-platform cap = 6** items: take the top 6 by impact.
- **`high`-impact items are always included** — if a platform has more than 6 highs, raise
  its cap to fit them, up to an absolute **max of 8** per platform. (So a platform ships
  6 normally, up to 8 only when highs demand it.)
- Items beyond the cap stay `presented: false` and carry to a future edition (subject to
  the 28-day expiry in step 1). Caps live in Open config.
- **There is no TL;DR block** (removed 2026-10). The summary job is done by the **hero
  subtitles** — the line under "The Bi-Weekly Brief", which follows the news filter:
  - `subtitles.all` — **one sentence leading with the single top story of each team that
    has items** (two or three clauses joined naturally, max ~45 words). Example:
    *"Google Ads auto-applies promotions scraped from advertiser sites, Meta freezes new
    US political ads, and The Trade Desk opens Netflix to first- and third-party data
    targeting."*
  - `subtitles.search` / `subtitles.social` / `subtitles.programmatic` — **one sentence
    with that team's top 2–3 changes** (max ~35 words), only for teams that have items.
  - Lead with the change itself (what changed, in plain words); no preamble, no dates or
    version numbers — the cards carry the detail. "Top" = highest impact, ties broken by
    recency — the same items the Slack highlights pick in step 6.

### 3a. Translate the content (EN → RU / ES / SR)
The deck ships in four languages behind a header language pill. There is **no runtime
translation API** — you pre-translate here, at build time, and the result is baked into
the deck file.

- **Translate exactly three things**, and nothing else:
  1. every **hero subtitle** (each key of `subtitles`),
  2. every item's **`title`**, 3. every item's **`summary`**.
  Everything else on the page is fixed English chrome that lives in the template —
  brand, filters, the hero headline itself, team headings, dates, counters, platform
  names, category/impact chips, card meta, Other editions, sign-off, footer. **Never
  translate those, and never touch the template to do it.**
- **Languages:** `ru` (Cyrillic), `es`, `sr` — **Serbian in LATIN script**, not Cyrillic.
  English is the base and stays in the plain `subtitles` / `title` / `summary` fields.
- **Length rule:** the limits in step 3 are defined **in English**. A translation must
  carry the **same meaning at the same length** — same one-sentence subtitle, same 2–3
  sentence summary. Don't pad, don't compress, and don't apply a per-language character
  cap; match the English.
- **Leave in English inside translated text:** product and feature names (Performance
  Max, AI Max, Demand Gen, Audience Ads, oCPC, Kokai, Programmatic Guaranteed, line
  items), UI paths (`Tools > Content suitability`), company and publication names.
  Translate the surrounding prose around them.

#### How to translate (this is the part that goes wrong)

**Translate the meaning for a PPC specialist, never the English word order.** The output
has to read like it was written in that language by someone who runs ad accounts. A
sentence a reader can only decode by mentally reconstructing the English original has
failed, even if every word in it is "correct".

- **Read the whole item first** (title + summary together), then write the translation from the *fact*, not from the sentence. You are
  re-stating what changed, not substituting words.
- **Expand English's compressed constructions.** English stacks nouns and drops words
  that other languages need. Restore them:
  - *"goes video-only"* is not *"станет только видео"* — video **what**? Write
    *"останется только для видеообъявлений"*.
  - *"on by default"* → *"будет включена по умолчанию"*, not *"включится по умолчанию"*
    where the subject is a setting someone else flips.
  - Noun stacks like *"view-through conversion optimization"* become a normal phrase:
    *"оптимизация по post-view конверсиям"*.
- **Use the term the platform's own localized UI uses** where one exists (Google Ads RU
  says *объекты* for assets, *широкое соответствие* for broad match, *назначение ставок*
  for bidding). Where the industry keeps the English term in daily speech (post-view,
  oCPC, Performance Max, brand safety), keep it — a forced native coinage is worse than
  the borrowed word everyone actually says.
- **Never invent a translation for a UI control's name.** Name it descriptively in the
  target language and keep the exact English control name where the reader would go
  looking for it (e.g. *"…в разделе Tools > Content suitability > Excluded content
  terms"*).
- **Read it back cold.** If a phrase would make a specialist stop and re-read, or if it
  only parses because you know the English, rewrite it. Prefer two clear clauses over
  one clever compressed one.
- The same applies to **Spanish and Serbian** — no *"pasa a ser solo vídeo"* /
  *"postaje samo video"* either; say *"queda limitada a los anuncios de vídeo"* /
  *"ostaje samo za video oglase"*.
- Write the translations into the deck data as:
  - `"subtitles_i18n": { "ru":{ "all":"…", "search":"…", "social":"…", "programmatic":"…" }, "es":{…}, "sr":{…} }`
    — **the same keys** as `subtitles`.
  - on each item: `"i18n": { "ru":{"title":"…","summary":"…"}, "es":{…}, "sr":{…} }`.
- A missing language, item or field falls back to English on the page, so a partial
  translation degrades gracefully — but ship all three languages for every item and
  every subtitle.

### 4. Build the deck
- Copy `style/deck-template.html` → `decks/deck-YYYY-MM-DD.html` (today's date).
- Replace the `<script id="deck-data">` JSON block with the real data (schema is
  documented inside the template). Fill `period_label`, `generated`, `edition_no`,
  `author` ("Ivan Potekhin" — the footer reads "Made for MPO Team with 💜 by …"),
  `subtitles` + `subtitles_i18n`, `archive`, and `platforms[]` — each platform as
  `{ "name": "<deck name from the table>", "team": "search|social|programmatic", "items": [ … ] }`.
  Each item carries `title`, `url` (the record's stored `url`, as is), `source`, `published`,
  `category`, `impact`, `summary` and `i18n`. `edition_no` = this run's number (see
  editions registry below). Do **not** write `tldr` / `tldr_i18n` / `subtitle`.
- **`period_label` = the digest window, NOT the min/max of the items' `published`
  dates.** Compute it as: **start** = the previous edition's date + 1 day (from
  `editions.json`); for the **first** edition, `generated − 14 days`. **end** =
  `generated`. Format `"Mon D – Mon D, YYYY"` (e.g. `Jul 20 – Aug 3, 2026`). This keeps
  the label a clean ~2-week span independent of individual item dates.
- **Editions strip:** read `data/editions.json` (the registry) and pass its entries as
  `archive` in the deck data — each as
  `{ "no": <n>, "label": "<period_label>", "url": "deck-YYYY-MM-DD.html" }`. Use the
  **bare filename** for `url` (decks are siblings in `decks/`, so `deck-….html`
  resolves correctly; do NOT prefix `decks/`). If the registry is empty, omit `archive`.
  - This array is a **fallback**. At view time the deck re-reads `data/editions.json`
    itself (same origin) and lists every *other* edition, newest first — which is what
    lets an already-published deck link **forward** to editions released after it. The
    baked array only takes over when that read can't happen (opened from disk, offline).
  - **You therefore never rewrite a published deck to add a newer edition to its strip.**
    Appending to `editions.json` in step 7 is enough; every existing deck picks it up on
    its next view.
- Sanity check: open/parse the file; ensure valid JSON in the data block; that every
  platform has a `team` matching the table and a deck name from it; that `subtitles` has
  `all` plus a key for every team present (and none for an absent team); that
  `subtitles_i18n` has `ru` / `es` / `sr` with the same keys; and that every item carries
  `i18n` for all three languages.

### 5. Publish the deck (GitHub Pages)
- Commit the deck to `main` (via the PR flow); the Pages URL is
  `https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html`.
- Requires Pages enabled once (Settings → Pages, serve `main` / root — see
  `routines/SETUP.md`). Give Pages a moment to build before posting the link.
- **Verify the URL returns 2xx before step 6** — the Slack messages are the one thing a
  reader can't undo, so never announce a link you haven't loaded. If the fetch is refused
  by the environment's egress policy rather than by Pages (`connect_rejected` on
  `ipotekhin.github.io`), that is a config gap, not a broken deck: still post, but say in
  the step-8 note that the link went out unverified and why.
- This is the **only** delivery format. Do not use Slack Canvas.

### 6. Post to Slack — one message per team (Search / Social / Programmatic)
The digest is announced to **three teams**, each in its own channel, with a message
adapted to that team and a link that opens the deck **already filtered to their view**.
The deck itself is a single file with all platforms; the links only differ by a `?team=`
param.

- **Split the included (capped) items into the three teams** by the table above.
- **For each team that has at least one included item**, compose a message by following
  **`style/slack-summary.md`** exactly (greeting → digest intro with the date range →
  highlights intro → **one highlight line per platform** → digest summary → link). The
  highlights and the summary count use **only that team's platforms**. Rotate the wording
  per that file — including its greeting rule (**the Programmatic message never names the
  team**: common greetings only) — and use **standard-markdown** Slack formatting —
  **`**bold**`** (double asterisks; a single `*word*` renders as *italic* in this
  connector) and `:emoji:`.
- **One highlight per platform (not a list of updates).** For each platform in the team
  that has items, pick its **single hottest** item (highest impact; break ties by
  recency) and write one line for it. Don't list several updates from the same platform —
  the deck holds the full detail; the message is a teaser of the top change per platform.
  Platform names in the message are the deck names from the table.
- **Deep-linked deck URL:** append `?team=search`, `?team=social` or `?team=programmatic`
  to the deck URL so the link opens straight to that team's filtered view (readers can
  switch to *All news* in the header). E.g.
  `https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html?team=programmatic`.
- **Four language links, not one.** The CTA line carries **four markdown links embedded
  next to flag emoji** — English plus the three translations, all pointing at this team's
  filtered view:
  `:us: [EN](URL?team=T), :ru: [RU](URL?team=T&lang=ru), :es: [ES](URL?team=T&lang=es) and :flag-rs: [SR](URL?team=T&lang=sr)`
  English takes **no** `lang` param (it is the deck's default). Labels are language codes
  (`EN` / `RU` / `ES` / `SR`), matching the deck's language pill; only the Serbian flag
  shortcode is country-based (`:flag-rs:`). Never paste a bare URL — the link text is the
  language code.
- **Send** each team's message via `slack_send_message` to that team's channel from Open
  config (`search_channel_id` / `social_channel_id` / `programmatic_channel_id`). Posts
  from Ivan's own Slack identity.
- **Empty team = skip:** if a team has **no** included items this edition, **do not post to
  that team's channel** — its digest simply doesn't happen this time. (If *all* teams are
  empty, the run already stopped at step 2a.)
- **Verify each send** (tool returned ok / message link). If a team's send fails, leave
  **that team's** items un-presented for the next run; a success for another team still
  counts. Only mark presented (step 7) for items whose team message was confirmed.

### 7. Mark presented (only after success)
- For every item **whose team message was confirmed delivered** in step 6, set
  `presented: true` in `data/updates.json`. Items in a team whose send failed — or a team
  skipped for being empty — stay `presented:false` for the next run. (Items marked
  `expired: true` in step 1 keep that flag; `track` / `platform` corrections from step 2
  stay as written.)
- **Append this edition to `data/editions.json`** (once at least one team's message was
  delivered) so the next run lists it under Other editions: `{ "no": <n>, "date":
  "YYYY-MM-DD", "period_label": "<period_label>", "url": "deck-YYYY-MM-DD.html" }`
  (`no` = previous max + 1).
- Commit `data/updates.json` (presented + expired flags, any step-2 corrections) +
  `data/editions.json` + `decks/deck-YYYY-MM-DD.html` with a message like
  `present: deck YYYY-MM-DD, M items -> Slack`.
- Push to your `claude/*` branch, then open a pull request into `main` and merge it
  (Pages serves `main`, so the deck link resolves after merge).
- **Branch cleanup is GitHub's job:** the repo's "Automatically delete head branches"
  setting removes the branch when a PR merges. Don't run `git push --delete` — the
  session's git proxy refuses it (HTTP 403). When the run merges twice (deck first,
  then the presented flags), the branch is gone after the first merge: pushing the
  second commit to the same name recreates it, and the second PR opens from it as usual.

### 8. Status note to Ivan (internal — always to Ivan's DM)
Send a **separate short** `slack_send_message` to **Ivan's DM (`channel_id =
U065VBRHYV7`)** — this is an internal ops note and always goes to Ivan regardless of
where the digest itself is sent. Send it on **every run that got past step 2a**, even
when something below failed — that is when Ivan needs it most. (A stop at step 0 or 2a is
silent, as before.)

**Problems first.** If anything went wrong, open the note with one `:warning:` line per
problem, saying what failed (with the error) and what Ivan has to do:
- a team's Slack message failed to send → that team's items stay un-presented and go out
  next edition; nothing to do unless it repeats;
- the run stopped before posting (deck failed its sanity check, Pages never served it) →
  no team got a message; say where it stopped;
- a PR did not merge into `main` → say which one and link it. If it is the
  presented-flags PR and messages already went out, Ivan must merge it by hand, or the
  next run presents the same items again;
- the deck link went out unverified (step 5).

Then summarize the backlog pressure so Ivan can decide whether to raise a platform's cap
later — grouped by team (Search / Social / Programmatic):
- **Included:** total shipped this edition, per team.
- **Deferred** (still `presented:false`, hit the per-platform cap): count **by platform**.
- **Expired this run** (aged past 28 days, retired): count **by platform**.
- **Moved between tracks** in step 2, if any: one line each.
If nothing failed and nothing was deferred or expired, one line ("no backlog — everything
relevant shipped") is enough. Keep it compact; this note is for Ivan only, not the teams.
If the DM itself fails, say so in the run's final report; don't retry in a loop.

---

## Open config (single source of truth — edit here, routines pick it up)
- **Slack delivery targets (three teams):** the digest goes to the team channels —
  `search_channel_id = C04PJUZMN91` (**#paid-search-team**),
  `social_channel_id = C04AU6G17GT` (**#paid-social-team**) and
  `programmatic_channel_id = C065RGQMUKF` (**#media_performance**). All three are
  **private** channels, so Ivan (whose identity the connector posts under) must be a
  member of each. Change an ID here to redirect a team later (nothing else changes).
  Team→platform mapping: the table at the top of this file.
- **Deep-linked digest URL:** each team message links to the *same* deck with
  `?team=search` / `?team=social` / `?team=programmatic` appended, so it opens on that
  team's filtered view; the reader can switch to *All news* in the header (step 6).
- **Empty team:** a team with no items this edition gets **no post** (step 6). If all
  teams are empty, no edition at all (step 2a).
- **Delivery mechanism:** GitHub Pages only (no Slack Canvas).
- **Deck format:** the "Zine" HTML template in `style/` (Google Fonts + local sticker
  PNGs + GTM). PPTX not used.
- **Author credit (footer):** "Ivan Potekhin" (`author` in deck data); the template
  renders it as "Made for MPO Team with 💜 by Ivan Potekhin".
- **Empty edition:** if nothing qualifies, no deck and no Slack post (step 2a).
- **Cadence gate:** min **12 days** between editions (step 0). Set the trigger to fire
  every Wednesday (the time is set in the routine UI); the gate makes the actual digest
  land bi-weekly.
  Change the 12-day threshold here to adjust cadence.
- **Per-platform cap:** **6** items (up to **8** to fit all `high`-impact); **no overall
  cap** (step 3). Same for every team. Raise a platform's cap here if its backlog note
  shows steady pressure.
- **Backlog expiry:** un-presented items older than **28 days** (`first_seen`) are retired
  (`expired:true`, step 1). Raise this if too much is aging out.
- **Status note:** step 8 always DMs Ivan (`U065VBRHYV7`) — problems first (failed
  send, unmerged PR, a stop before posting), then a short deferred/expired summary by
  platform — on every run that got past step 2a.

---

## Guardrails
- **`presented` is set ONLY after a confirmed Slack post**, per team. A failed post must
  leave that team's items un-presented so nothing is silently lost.
- **Idempotent:** re-running before a successful post just rebuilds; re-running after
  finds nothing new to present.
- Never invent updates — the deck contains only what's in `updates.json`.
- The only record edits this routine makes are `presented`, `expired`, and step-2
  corrections of `category` / `impact` / `track` / `platform`.
- Keep the channel message short; depth lives in the deck.
