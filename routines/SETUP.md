# Setup — putting the routines into operation

One-time steps to go from "repo scaffolded" to "routines running on schedule."

---

## 1. Allow the source domains in the run environment (required)

A routine fetches live pages. The environment it runs in **must** permit outbound
HTTPS to the 8 domains behind the 13 validated sources, or fetches fail with
`403 CONNECT` (policy denial):

```
blog.google
support.google.com
searchengineland.com
about.fb.com
www.socialmediatoday.com
blogs.bing.com
ppc.land
www.seroundtable.com
```

Configure this on the environment used by the routines. If you use "trusted"/open
network access for that environment, that covers it too.

## 2. Enable GitHub Pages (chosen delivery)

Repo → **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main`,
folder `/ (root)` → Save. Deck links then resolve to:

```
https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html
```

GitHub Pages is the **only** delivery format — no Slack Canvas.

## 3. Confirm the Slack connector + channel

- Make sure the **Slack connector is enabled in the routine's environment** (not just a
  desktop session). Messages post from Ivan's own Slack identity.
- Decide the target channel for the digest and drop its name into Routine 2's trigger
  prompt below (replace `#<CHANNEL>`). For **test runs**, no channel is needed — Routine
  2 posts a draft or a DM to Ivan for review first (see step 5).

---

## 4. Create the two routines (cloud, scheduled)

Use **cloud** routines (desktop scheduled tasks only run while the app is open). Point
each routine's environment at the network-enabled one from step 1. Paste these as the
trigger prompts — the logic lives in the repo docs, so the prompts stay short.

Schedule (set manually in the routine UI; it can change any time without touching the
logic): **Routine 1 = once a week** (e.g. Monday `0 7 * * 1`), **Routine 2 = every
Wednesday `0 7 * * 3`** — its step-0 cadence gate then makes the digest land bi-weekly
(standard cron can't express "every 14 days"; the gate does it exactly and self-corrects).

### Routine 1 — Collect  ·  schedule: **weekly**
```
You are Routine 1 (Collect) for the ad-platform-news-digest repo.
Read routines/routine-1-collect.md and follow it exactly. That file is the source of
truth for this run — its scope, rules, outputs and stop conditions are kept current, and
if anything in this prompt ever disagrees with it, the file wins. Work on a
claude/-prefixed branch off main and land the result through a pull request into main,
as the file specifies. Stay inside this routine's job — do not do Routine 2's work.
```

### Routine 2 — Presentation  ·  schedule: **every Wednesday** `0 7 * * 3` (gate makes it bi-weekly)
```
You are Routine 2 (Presentation) for the ad-platform-news-digest repo.
Read routines/routine-2-presentation.md and follow it exactly. That file is the source of
truth for this run — its scope, rules, outputs, delivery targets and stop conditions are
kept current, and if anything in this prompt ever disagrees with it, the file wins. Check
its stop conditions first and do nothing at all if they say stop. Work on a
claude/-prefixed branch off main and land the result through a pull request into main,
as the file specifies. Stay inside this routine's job — do not do Routine 1's work.
```

> **These prompts are deliberately contentless, and must stay that way.** The stored copy
> in the routine UI is not version-controlled and nobody re-reads it, so any specific
> written into it rots silently the next time the repo changes — the Routine 2 prompt
> still named Ivan's DM as the destination months after the two-channel split. A prompt
> holds only what cannot expire: which routine it is, which file to obey, that the file
> outranks the prompt, and the branch/PR mechanic the platform forces. Everything
> else — destinations, thresholds, caps, languages, outputs, steps — lives in
> `routines/*.md`, where it is edited and reviewed. Paste these once; after that, tuning
> the system means editing the repo, never the routine UI.

## 5. First run = smoke test

1. **Manually run Routine 1 once.** Then inspect `data/updates.json` — check items are
   real (last-2-weeks window), deduped, ad-account-relevant, with a clear title and a
   1–3 sentence summary written from the article body.
2. **Manually run Routine 2 once.** It builds one deck and sends **up to two Slack
   messages — one per team** (Search → **#paid-search-team**, Social → **#paid-social-team**;
   IDs in `routine-2` Open config). Both are **private** channels, so Ivan must be a member
   of each. Each message links the deck with `?team=search|social` so it opens on that
   team's filtered view. Confirm the deck renders, the Pages links resolve on the right
   filter, and both messages look right. A team with no items sends no message. The step-8
   backlog note goes to Ivan's DM (`U065VBRHYV7`). Items are marked `presented` per team,
   only after that team's delivery succeeds.
3. Once both look right, leave them on the weekly / bi-weekly schedule. To redirect a team
   later, change `search_channel_id` / `social_channel_id` in `routine-2`
   Open config — no other edits needed.

## 6. Branch / merge workflow (main is the mainline)

`main` is the canonical branch and what GitHub Pages serves. Each routine run:
1. branches from `main` to a short-lived `claude/collect-run` or `claude/present-run`
   branch (routines can only push to `claude/`-prefixed branches),
2. commits + pushes its changes there,
3. opens a pull request into `main` and merges it.

This keeps `data/` and `decks/` current on `main` so the published deck and state stay
in sync. (Set the repo's **default branch to `main`** and point **Settings → Pages** at
`main` / root — Pages was initially built from the scaffold branch.)
