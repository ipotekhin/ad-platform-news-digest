# Setup — putting the routines into operation

One-time steps to go from "repo scaffolded" to "routines running on schedule."

---

## 1. Allow the source domains in the run environment (required)

A routine fetches live pages. The environment it runs in **must** permit outbound
HTTPS to the 6 domains behind the 9 validated sources, or fetches fail with
`403 CONNECT` (policy denial):

```
blog.google
support.google.com
searchengineland.com
about.fb.com
socialbee.com
blogs.bing.com
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

### Routine 1 — Collect  ·  suggested: **Wed + Fri, ~08:00**
```
You are Routine 1 (Collect) for the ad-platform-news-digest repo.
Read routines/routine-1-collect.md and follow it exactly. Branch from main to a
claude/collect-run branch, commit the updated data/updates.json and
data/state.json (and any sources.yaml validation edits), push, then open a pull
request into main and merge it. Do not build any deck.
```

### Routine 2 — Presentation  ·  suggested: **every 2 weeks, Fri ~16:00**
```
You are Routine 2 (Presentation) for the ad-platform-news-digest repo.
Read routines/routine-2-presentation.md and follow it exactly. Branch from main
to a claude/present-run branch. Build the deck and publish it to GitHub Pages.
For test runs, deliver the summary as a Slack draft or a DM to Ivan for review;
in live mode, post it to Slack channel #<CHANNEL>. Only after a delivery Ivan
accepts, set presented:true on the included items, commit, push, open a pull
request into main and merge it.
```

> **Cadence note:** if the scheduler can't express "every 2 weeks," run Routine 2
> **weekly on Friday** instead — it only presents items still marked
> `presented:false`, so an extra run with nothing new is a harmless no-op. Just expect
> a post whenever ≥1 new item has accumulated.

## 5. First run = smoke test

1. **Manually run Routine 1 once.** Then inspect `data/updates.json` — check items are
   real (last-2-weeks window), deduped, ad-account-relevant, with a clear title and a
   1–3 sentence summary written from the article body.
2. **Manually run Routine 2 in test mode.** It builds the deck and, instead of posting
   to the channel, either creates a **draft** or sends the summary to **Ivan's DM** for
   review (see `routine-2-presentation.md` step 6). Confirm the deck renders and the
   Pages link resolves. Items are marked `presented` only after a delivery Ivan accepts.
3. Once both look right, flip Routine 2 to live (post to `#<CHANNEL>`) and leave them on
   the schedule above.

## 6. Branch / merge workflow (main is the mainline)

`main` is the canonical branch and what GitHub Pages serves. Each routine run:
1. branches from `main` to a short-lived `claude/collect-run` or `claude/present-run`
   branch (routines can only push to `claude/`-prefixed branches),
2. commits + pushes its changes there,
3. opens a pull request into `main` and merges it.

This keeps `data/` and `decks/` current on `main` so the published deck and state stay
in sync. (Set the repo's **default branch to `main`** and point **Settings → Pages** at
`main` / root — Pages was initially built from the scaffold branch.)
