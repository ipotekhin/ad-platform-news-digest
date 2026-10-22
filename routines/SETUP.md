# Setup — putting the routines into operation

One-time steps to go from "repo scaffolded" to "routines running on schedule."

---

## 1. Allow the source domains in the run environment (required)

A routine fetches live pages. The environment it runs in (**Ad-platform-news-digest**)
**must** permit outbound HTTPS to every host behind the sources in `sources.yaml` and
`sources-programmatic.yaml`, or fetches fail with `CONNECT tunnel failed, response 403`
(policy denial):

```
# performance (sources.yaml)
blog.google
support.google.com
searchengineland.com
www.seroundtable.com
about.ads.microsoft.com
blogs.bing.com
www.adexchanger.com
about.fb.com
www.socialmediatoday.com
ads.tiktok.com
www.linkedin.com
ppc.land
ppcnewsfeed.com
www.searchenginejournal.com
# programmatic (sources-programmatic.yaml) — plus blog.google, support.google.com,
# about.ads.microsoft.com, www.adexchanger.com and ppc.land from above
www.thetradedesk.com
developers.google.com
digiday.com
www.mediapost.com
www.adweek.com
# Routine 2
ipotekhin.github.io        # verifies the published deck URL
```

Configure this on the environment used by the routines (environment menu → Edit →
Network access → Allowed domains). If you use "trusted"/open network access for that
environment, that covers it too. When a source is added later, add its host here and in
the environment; if it is missing, the collect routine marks the source `blocked` and DMs
Ivan.

## 2. Enable GitHub Pages (chosen delivery)

Repo → **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main`,
folder `/ (root)` → Save. Deck links then resolve to:

```
https://ipotekhin.github.io/ad-platform-news-digest/decks/deck-YYYY-MM-DD.html
```

GitHub Pages is the **only** delivery format — no Slack Canvas.

## 3. Confirm the Slack connector + channel

- Make sure the **Slack connector is attached to all three routines** (not just a
  desktop session) — Routine 2 posts the digest, the two collect routines DM Ivan when a
  source breaks or recovers. Messages post from Ivan's own Slack identity.
- Destinations live in each routine doc's **Open config**, never in a trigger prompt:
  the team channels in `routine-2-presentation.md`, Ivan's DM in all three.

---

## 4. Create the three routines (cloud, scheduled)

Use **cloud** routines (desktop scheduled tasks only run while the app is open). Point
each routine's environment at the network-enabled one from step 1. Paste these as the
trigger prompts — the logic lives in the repo docs, so the prompts stay short.

Schedule (set manually in the routine UI; it can change any time without touching the
logic): **both collect routines once a week, a few hours apart** (e.g. Tuesday `0 7 * * 2`
for performance and `0 9 * * 2` or later for programmatic — both append to
`data/updates.json`, so their runs must not overlap), **Routine 2 = every Wednesday
`0 7 * * 3`** — its step-0 cadence gate then makes the digest land bi-weekly (standard
cron can't express "every 14 days"; the gate does it exactly and self-corrects).

Routine names in the UI: **Ad-platform-news-digest — Routine 1 — Collect — Performance**,
**Ad-platform-news-digest — Routine 1 — Collect — Programmatic**,
**Ad-platform-news-digest — Routine 2 — Presentation**.

### Routine 1 — Collect — Performance  ·  schedule: **weekly**
```
You are Routine 1 — Collect — Performance, for the ad-platform-news-digest repo.
Read routines/routine-1-collect-performance.md and follow it exactly. That file is the
source of truth for this run — its scope, rules, outputs and stop conditions are kept
current, and if anything in this prompt ever disagrees with it, the file wins. Work on a
claude/-prefixed branch off main and land the result through a pull request into main,
as the file specifies. Stay inside this routine's job — collect the performance track
only; do not do the programmatic collection or Routine 2's work.
```

### Routine 1 — Collect — Programmatic  ·  schedule: **weekly, a few hours after the performance run**
```
You are Routine 1 — Collect — Programmatic, for the ad-platform-news-digest repo.
Read routines/routine-1-collect-programmatic.md and follow it exactly. That file is the
source of truth for this run — its scope, rules, outputs and stop conditions are kept
current, and if anything in this prompt ever disagrees with it, the file wins. Work on a
claude/-prefixed branch off main and land the result through a pull request into main,
as the file specifies. Stay inside this routine's job — collect the programmatic track
only; do not do the performance collection or Routine 2's work.
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

1. **Manually run each collect routine once.** Then inspect `data/updates.json` — check items are
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
1. branches from `main` to a short-lived `claude/*` branch (routines can only push to
   `claude/`-prefixed branches; the platform usually assigns the name),
2. commits + pushes its changes there,
3. opens a pull request into `main` and merges it,
4. GitHub deletes the branch as soon as the PR merges — merged branches are never kept.

That last step **requires Settings → General → Pull Requests → "Automatically delete
head branches" = ON.** Routines and chat sessions cannot delete branches themselves (the
session git proxy refuses ref deletion with HTTP 403), so without this setting every run
leaves a branch behind. `backup*` branches are Ivan's manual snapshots — never removed.

This keeps `data/` and `decks/` current on `main` so the published deck and state stay
in sync. (Set the repo's **default branch to `main`** and point **Settings → Pages** at
`main` / root — Pages was initially built from the scaffold branch.)
