# Slack message format (Routine 2 output)

> Routine 2 sends **up to two messages per edition — one per team** — via
> `slack_send_message`, from Ivan's own Slack identity. The deck is **linked**, never
> attached.
>
> **Formatting — this connector uses standard markdown, not Slack mrkdwn.** Use
> **`**double asterisks**`** for bold; a single `*word*` renders as *italic* (that's the
> bug the first run hit — platform names came out italic). Emoji via `:shortcode:`.
>
> - **Search** team (Google Ads + Microsoft/Bing + ChatGPT Ads) → `search_channel_id` = **#paid-search-team**.
> - **Social** team (Meta + TikTok + LinkedIn) → `social_channel_id` = **#paid-social-team**.
> - Targets (channel IDs) live in `routines/routine-2-presentation.md` → Open config. The
>   internal backlog/system note (step 8) still goes to **Ivan's DM `U065VBRHYV7`**, not the
>   team channels.
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

[EMOJI] **[PLATFORM]** — [SINGLE TOP UPDATE]      ← one line per platform in this team
[EMOJI] **[PLATFORM]** — [SINGLE TOP UPDATE]

[DIGEST SUMMARY — this team's update count + platforms]

[LINK EMOJI] [LINK CTA]: :us: [EN](<DECK_URL>?team=<T>), :ru: [RU](<DECK_URL>?team=<T>&lang=ru), :es: [ES](<DECK_URL>?team=<T>&lang=es) and :flag-rs: [SR](<DECK_URL>?team=<T>&lang=sr)
```

Rules:
- **Per team:** highlights and the summary use **only this team's platforms** (Search =
  Google Ads + Microsoft/Bing + ChatGPT Ads · Social = Meta + TikTok + LinkedIn).
- **One highlight line per platform** — the platform's **single hottest** item (highest
  impact; break ties by recency). Do **not** list several updates from the same platform;
  the deck holds the full detail, the message is a teaser of the top change per platform.
  So the number of highlight lines = the number of this team's platforms with items
  (e.g. Search with Google + Microsoft → 2 lines; a team with one platform → 1 line).
- `[DIGEST SUMMARY]` counts **this team's** updates in the deck, not the other team's.
- Date range: short month names, no year — e.g. `Jul 20 – Aug 3`.
- **Deep-linked deck URL:** append `?team=search` or `?team=social` to the deck URL so the
  link opens on that team's filtered view — e.g.
  `…/decks/deck-YYYY-MM-DD.html?team=social`. No tracking params, ever.
- **Four language links on one line.** The deck ships in English, Russian, Spanish and
  Serbian, so the CTA carries **four links, embedded in the text next to a flag** — not a
  bare URL:
  `:us: [EN](…?team=T), :ru: [RU](…?team=T&lang=ru), :es: [ES](…?team=T&lang=es) and :flag-rs: [SR](…?team=T&lang=sr)`
  - `T` = this message's own team (`search` / `social`), the same on all four links.
  - **English carries no `lang` param** — it is the deck's default.
  - Labels are **language** codes — `EN`, `RU`, `ES`, `SR` — matching the deck's own
    language pill and the `lang=` param. (The Serbian flag shortcode is `:flag-rs:`
    because that is the country code; the label next to it stays `SR`.)
  - Markdown links (`[EN](url)`) — this connector renders standard markdown, so the URL
    itself never appears in the message text.
- **Rotate** the wording each edition (options below) so it never reads canned, and keep
  the Search and Social messages worded differently from each other.

## Wording options (rotate per edition)

**[GREETING]** — pick one at **random** each time (keep it varied). The pool has **both**
common greetings and **team-named** ones; for each team, the team-named variant swaps in
its own name (`Search team` / `Social team`). Rotate freely across both kinds — just make
sure the team-named options below stay available for each team so it's not always generic.

_Common_ (either team):
- `Hi team! Hope your week is going well 🤗`
- `Hey everyone! Hope your week is treating you well ☀️`
- `Hi everyone! Hope you're having a productive but not-too-busy week 😄`
- `Happy [Weekday], team! Hope you're having a good one ✨`

_Team-named_ (use the message's own team — Search or Social):
- `Hi Search team! Hope your week is going well 🤗`  ·  `Hi Social team! Hope your week is going well 🤗`
- `Hey Search team! Hope you're having a great day 🤗`  ·  `Hey Social team! Hope you're having a great day 🤗`
- `Happy [Weekday], Search team! Hope you're having a good one ✨`  ·  `Happy [Weekday], Social team! …`

**[DIGEST INTRO]** (must contain the date range)
- `It's time for another round of **Ad Platform Updates**, covering **[DATE RANGE]**.`
- `We're back with another round of **Ad Platform Updates**, covering **[DATE RANGE]**.`
- `The latest **Ad Platform Updates** are here, covering **[DATE RANGE]**.`
- `A fresh batch of **Ad Platform Updates** has landed, covering **[DATE RANGE]**.`

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
- `We've included **[N] updates** in total across [PLATFORM NAMES].`
- `This edition includes **[N] updates** across [PLATFORM NAMES].`
- `In total, we've collected **[N] updates** from [PLATFORM NAMES].`
- `[PLATFORM NAMES]` = e.g. `Google Ads and Microsoft`, or `across the major ad platforms` if many.

**[LINK EMOJI]** `:point_right:` / `:link:`

**[LINK CTA]**
- `The full overview is available here`
- `You can find the full digest here`
- `Check out the full digest here`
- `Take a look at the full edition here`
- `Read the full digest here`

## Worked examples (edition Jul 20 – Aug 3, 2026) — one per team, one line per platform

**Search channel** (Google Ads + Microsoft/Bing + ChatGPT Ads), link `?team=search`:

```
Happy Monday, Search team! Hope you're having a good one ✨

It's time for another round of **Ad Platform Updates**, covering **Jul 20 – Aug 3**.

A few updates worth keeping on your radar:

:eyes: **Google Ads** — Local Services Ads fold into Performance Max as pay-per-lead from August
:loudspeaker: **Microsoft** — Predictive Matching is retired and folded into AI Max search-term matching

This edition includes **8 updates** across Google Ads and Microsoft.

:point_right: The full overview is available here: :us: [EN](<DECK_URL>?team=search), :ru: [RU](<DECK_URL>?team=search&lang=ru), :es: [ES](<DECK_URL>?team=search&lang=es) and :flag-rs: [SR](<DECK_URL>?team=search&lang=sr)
```

**Social channel** (Meta + TikTok + LinkedIn), link `?team=social`:

```
Hey everyone! Hope your week is treating you well ☀️

A fresh batch of **Ad Platform Updates** has landed, covering **Jul 20 – Aug 3**.

Here's what stood out this time:

:gear: **Meta** — Marketing API v26.0 drops the Instagram Explore Feed ad placement
:dart: **TikTok** — Q3 preview adds TopView regional exclusions and TopReach Max Reach
:briefcase: **LinkedIn** — Draft with AI, Brand Kit and Ad Variants land in Campaign Manager

This edition includes **5 updates** from Meta, TikTok and LinkedIn.

:link: You can find the full digest here: :us: [EN](<DECK_URL>?team=social), :ru: [RU](<DECK_URL>?team=social&lang=ru), :es: [ES](<DECK_URL>?team=social&lang=es) and :flag-rs: [SR](<DECK_URL>?team=social&lang=sr)
```

(If a team has no items this edition, that team's message is simply not sent. One line
per platform means a team with a single platform sends a single highlight line.)
