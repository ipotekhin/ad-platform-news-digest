# CLAUDE.md — operational context for this repo

This repo automates an **ad-platform news digest**: two Claude routines + this repo as
memory. Read [`README.md`](README.md) for the full design. This file is the quick
operational reference for a routine session.

## What you might be asked to do
- **Run Routine 1 (Collect):** follow [`routines/routine-1-collect.md`](routines/routine-1-collect.md) exactly.
- **Run Routine 2 (Presentation):** follow [`routines/routine-2-presentation.md`](routines/routine-2-presentation.md) exactly.
- **Edit relevance rules:** they live in [`criteria.md`](criteria.md) (human-owned).
- **Re-skin the deck:** CSS vars in [`style/deck-template.html`](style/deck-template.html).

## Hard rules
- `presented` is set **only after** a confirmed Slack post (Routine 2). Never before.
- Routine 1 **never** builds a deck; Routine 2 **never** collects.
- Dedup: Layer 1 = canonical-URL exact match (`sha1(canonical_url)[:12]`); Layer 2 =
  LLM semantic match, preferring official sources over aggregators.
- Only advance a source's `last_collected` in `state.json` if you actually read it.
- The deck (`style/deck-template.html`) loads Google Fonts (Playfair Display, Courier
  Prime) from the Google Fonts CDN and reads sticker PNGs from `assets/stickers/` —
  these are the only external/local requests it makes. Don't add anything beyond that
  (no analytics, no other third-party JS/CSS).

## Data files (don't hand-corrupt)
- `data/updates.json` — array of update records (see README schema).
- `data/state.json` — `{ "<source_key>": { "last_collected": "YYYY-MM-DD" } }`.

## Environment gotchas
- **Egress:** source domains must be allowed by the run environment's network policy,
  or fetches fail with `403 CONNECT`. See README → Network requirement.
- **Slack:** MCP has no file upload — post a summary + link, never a file attachment.
- **Branches:** `main` is the mainline (GitHub Pages serves it). Routines can only push
  to `claude/`-prefixed branches, so each run branches from `main`, pushes to a
  `claude/*` branch, then merges into `main` via pull request.
- Use **cloud** routines for scheduling (desktop tasks only run while the app is open).
