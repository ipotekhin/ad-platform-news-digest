# Slack message format (Routine 2 output)

> Routine 2 sends **up to two messages per edition — one per team** — via
> `slack_send_message`, from Ivan's own Slack identity. The deck is **linked**, never
> attached. Use Slack formatting: `*bold*` and `:emoji:` shortcodes.
>
> - **Search** team (Google Ads + Microsoft/Bing) → `search_channel_id`.
> - **Social** team (Meta + TikTok + LinkedIn) → `social_channel_id`.
> - Targets live in `routines/routine-2-presentation.md` → Open config. **Currently both
>   = Ivan's DM `U065VBRHYV7` for testing.**
>
> Each message covers **only its own team's platforms** (highlights + summary count), and
> its link opens the deck **pre-filtered to that team** via a `?team=` param (see the link
> line below). Build each message from the same structure/wording options below, but keep
> the two messages independently randomized so they don't read identically.
>
> **Empty team → no message for that team.** If the whole edition is empty, send nothing.

## Structure

```
[GREETING]

[DIGEST INTRO — includes the date range]

[HIGHLIGHTS INTRO]

[EMOJI] *[PLATFORM]* — [KEY UPDATE]
[EMOJI] *[PLATFORM]* — [KEY UPDATE]
[EMOJI] *[PLATFORM]* — [KEY UPDATE]
[EMOJI] *[PLATFORM]* — [KEY UPDATE]   ← drop this line if only 3 highlights

[DIGEST SUMMARY — this team's update count + platforms]

[LINK EMOJI] [LINK CTA]: <DECK_URL>?team=<search|social>
```

Rules:
- **Per team:** highlights and the summary use **only this team's platforms** (Search =
  Google Ads + Microsoft/Bing · Social = Meta + TikTok + LinkedIn).
- Pick the **top 3–4** items by impact (high first) for the highlight lines — from this
  team only (so a team with few items may have just 1–2 highlight lines).
- `[DIGEST SUMMARY]` counts **this team's** updates in the deck, not the other team's.
- Date range: short month names, no year — e.g. `Jul 6–22`.
- **Deep-linked deck URL:** append `?team=search` or `?team=social` to the deck URL so the
  link opens on that team's filtered view — e.g.
  `…/decks/deck-YYYY-MM-DD.html?team=social`. (The `?team=` param is the only param — no
  tracking params.)
- **Rotate** the wording each edition (options below) so it never reads canned, and keep
  the Search and Social messages worded differently from each other.

## Wording options (rotate per edition)

**[GREETING]** — **always address the message's own team by name** (`Search team` /
`Social team`), so both messages read as tailored, never generic. Both messages must use
the team-named form; only vary the rest of the wording between them.
- `Hi Search team! Hope your week is going well 🤗`  ·  `Hi Social team! Hope your week is going well 🤗`
- `Hey Search team! Hope you're having a great day 🤗`  ·  `Hey Social team! Hope you're having a great day 🤗`
- `Happy [Weekday], Search team! Hope you're having a good one ✨`  ·  `Happy [Weekday], Social team! …`
- `Hey Search team! Hope your week is treating you well ☀️`  ·  `Hey Social team! Hope your week is treating you well ☀️`

**[DIGEST INTRO]** (must contain the date range)
- `It's time for another round of *Ad Platform Updates*, covering *[DATE RANGE]*.`
- `We're back with another round of *Ad Platform Updates*, covering *[DATE RANGE]*.`
- `The latest *Ad Platform Updates* are here, covering *[DATE RANGE]*.`
- `A fresh batch of *Ad Platform Updates* has landed, covering *[DATE RANGE]*.`

**[HIGHLIGHTS INTRO]**
- `A few updates worth keeping on your radar:`
- `Here's what stood out this time:`
- `Here are the main highlights from this edition:`
- `A few key updates to keep on your radar:`
- `Here's a quick look at the top news:`

**[EMOJI] by update type** (Slack shortcodes)
- New product / feature: `:rocket:` / `:sparkles:`
- Big change: `:eyes:` / `:loudspeaker:`
- AI feature: `:bulb:`
- Reporting / analytics: `:bar_chart:` / `:chart_with_upwards_trend:`
- Targeting / optimization: `:target:` / `:dart:`
- Technical / setup: `:gear:`
- Research / discovery: `:mag:`
- B2B / LinkedIn: `:briefcase:`

**[DIGEST SUMMARY]**
- `We've included *[N] updates* in total across [PLATFORM NAMES].`
- `This edition includes *[N] updates* across [PLATFORM NAMES].`
- `In total, we've collected *[N] updates* from [PLATFORM NAMES].`
- `[PLATFORM NAMES]` = e.g. `Google Ads and LinkedIn`, or `across the major ad platforms` if many.

**[LINK EMOJI]** `:point_right:` / `:link:`

**[LINK CTA]**
- `The full overview is available here`
- `You can find the full digest here`
- `Check out the full digest here`
- `Take a look at the full edition here`
- `Read the full digest here`

## Worked examples (edition Jul 6–22, 2026) — one per team

**Search channel** (Google Ads + Microsoft/Bing), link `?team=search`:

```
Hey Search team! Hope your week is going well 🤗

It's time for another round of *Ad Platform Updates*, covering *Jul 6–22*.

A few updates worth keeping on your radar:

:mag: *Google Ads* — Local Services Ads now run inside Performance Max
:dart: *Google Ads* — Google drops the $50K spend gate on Lead Form assets
:bar_chart: *Google Ads* — Bulk-link multiple accounts to one GA4 property
:gear: *Microsoft* — UET consent-mode signals expand in the UI

This edition includes *10 updates* across Google Ads and Microsoft.

:point_right: The full overview is available here: <DECK_URL>?team=search
```

**Social channel** (Meta + TikTok + LinkedIn), link `?team=social`:

```
Hey Social team! Hope you're having a great day 🤗

A fresh batch of *Ad Platform Updates* has landed, covering *Jul 6–22*.

Here's what stood out this time:

:briefcase: *LinkedIn* — AI creative tools come to Campaign Manager

This edition includes *1 update* from LinkedIn.

:link: You can find the full digest here: <DECK_URL>?team=social
```

(If a team has no items this edition, that team's message is simply not sent.)
