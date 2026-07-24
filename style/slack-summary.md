# Slack message format (Routine 2 output)

> Routine 2 sends this as a single message via `slack_send_message` to the target in
> `routines/routine-2-presentation.md` → Open config (currently **Ivan's DM**,
> `U065VBRHYV7`), from Ivan's own Slack identity. The deck is **linked**, never
> attached. Use Slack formatting: `*bold*` and `:emoji:` shortcodes.
>
> **If the edition is empty (no qualifying updates), send nothing at all.**

## Structure

```
[GREETING]

[DIGEST INTRO — includes the date range]

[HIGHLIGHTS INTRO]

[EMOJI] *[PLATFORM]* — [KEY UPDATE]
[EMOJI] *[PLATFORM]* — [KEY UPDATE]
[EMOJI] *[PLATFORM]* — [KEY UPDATE]
[EMOJI] *[PLATFORM]* — [KEY UPDATE]   ← drop this line if only 3 highlights

[DIGEST SUMMARY — total update count + platforms]

[LINK EMOJI] [LINK CTA]: <DECK_URL>
```

Rules:
- Pick the **top 3–4** items by impact (high first) for the highlight lines.
- `[DIGEST SUMMARY]` counts **all** updates in the deck, not just the highlighted ones.
- Date range: short month names, no year — e.g. `Jul 6–22`.
- Embed the deck URL directly (no tracking params).
- **Rotate** the wording each edition (options below) so it never reads canned.

## Wording options (rotate per edition)

**[GREETING]**
- `Hi team! Hope your week is going well 🤗`
- `Hey team! Hope you're having a great day 🤗`
- `Hi everyone! Hope you're having a productive but not-too-busy week 😄`
- `Hey everyone! Hope your week is treating you well ☀️`
- `Happy [Weekday], team! Hope you're having a good one ✨`

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

## Worked example (edition Jul 6–22, 2026)

```
Hey hey, team! Hope your week is going well 🤗

It's time for another round of *Ad Platform Updates*, covering *Jul 6–22*.

A few updates worth keeping on your radar:

:mag: *Google Ads* — Local Services Ads now run inside Performance Max
:dart: *Google Ads* — Google drops the $50K spend gate on Lead Form assets
:bar_chart: *Google Ads* — Bulk-link multiple accounts to one GA4 property
:briefcase: *LinkedIn* — AI creative tools come to Campaign Manager

We've included *11 updates* in total across Google Ads and LinkedIn.

:point_right: The full overview is available here: <DECK_URL>
```
