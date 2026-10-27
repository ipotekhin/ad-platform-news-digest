# Relevance Criteria — Programmatic track

> **Human-edited file.** Ivan (and future editors) tune the rules here. Both routines
> that touch the programmatic track read it: **Routine 1 — Collect — Programmatic**
> applies it *loosely* (lean include), **Routine 2** applies it *strictly* to every item
> with `"track": "programmatic"`. The performance track has its own file, `criteria.md`.
> Keep it in plain language — the LLM applies judgment, not regex.

---

## Scope — what this track is about

**Only updates that change how a programmatic trader works — in the DSPs we actually buy
in, or in CTV bought through them.** Same spirit as the performance digest: if a change
would make a programmatic specialist open the DSP, set up a deal, change targeting,
bidding, a report or a measurement setup — it's in. If it wouldn't, it's out.

The test for every candidate:
> "Does this change something I can see or do in **our programmatic stack** — the DSP
> seats we run (campaign / line-item setup, inventory and deals, targeting and audiences,
> bidding and optimisation, creative formats, reporting and measurement, brand safety,
> policy, the API behind them), or in **CTV buying through those DSPs**?"

If **no** → drop it, however big the headline.

**Product test — "our platforms" is an exact list.** We run **five** programmatic
platforms and only these:

| Platform (`platform` value) | What counts |
|---|---|
| **The Trade Desk** (`ttd`) | Kokai and its releases (e.g. Zuma), Koa / AI agents, OpenPath, UID2 / EUID, TTD data marketplace, retail and CTV inventory in TTD, TTD reporting and measurement |
| **Display & Video 360** (`dv360`) | DV360 line items and insertion orders, YouTube & partners / Demand Gen inside DV360 — **including YouTube on TV screens** — deals and Marketplace, audiences, reporting, brand controls, the DV360 API when it exposes a new capability |
| **Amazon DSP** (`amazon_dsp`) | Amazon DSP buying, **Prime Video ads** and other Amazon CTV (Fire TV, Twitch) bought programmatically, Amazon audiences and signals in the DSP, Amazon Marketing Cloud as used for DSP campaigns |
| **Microsoft programmatic** (`microsoft_programmatic`) | Microsoft's programmatic stack (the former Xandr): Microsoft Invest (DSP), Microsoft Curate deals, programmatic CTV / Netflix deals via Microsoft, open-exchange display, native and DOOH bought on CPM |
| **CTV** (`ctv`) | A CTV publisher's own ad-product change that a buyer can use through several DSPs — Netflix, Disney+, Roku, Paramount+, Max, Peacock, Tubi, smart-TV home screens (LG, Samsung, Vizio), etc.: new formats (e.g. pause ads), programmatic availability, measurement, targeting. **Not** Prime Video (→ Amazon DSP) or YouTube (→ DV360): a publisher owned by one of our DSPs' companies goes under that DSP |

**Judge by where the change lands, not by who announced it.** A partner announcing
something that becomes usable *inside* one of our DSPs is in: "Gracenote's show-level
CTV data is now available in The Trade Desk" → `ttd`. A story that names a DSP gets that
DSP's platform, not `ctv`: "LinkedIn B2B audiences now available on CTV via DV360" →
`dv360`. Prime Video always goes to `amazon_dsp` and YouTube to `dv360`. Use `ctv` only
when the change is a publisher's own and not tied to one of our DSPs — e.g. "Netflix
opens a new measurement option to all programmatic buyers", "Disney+ makes pause ads
buyable programmatically". Expect the CTV section to be empty in many editions.

### Microsoft: performance vs programmatic

Microsoft Advertising sells both, and the two tracks split it:

| Performance track (`bing`, `criteria.md`) | Programmatic track (`microsoft_programmatic`, this file) |
|---|---|
| Everything set up in the **Microsoft Advertising** account (formerly Bing Ads): Search, Shopping, Performance Max, Audience / Display / Video campaigns in that UI, UET, Copilot ads, Google Ads import | Microsoft's **programmatic stack** (Xandr heritage), bought separately on a CPM basis across an open network: Microsoft Invest, Microsoft Curate, programmatic CTV including Netflix, open-exchange display / native / DOOH |

**Microsoft Monetize** (the publisher ad server / SSP side) is in neither track.

## Include if it is one of

- **New platform feature / capability** in one of our DSPs — incl. betas, limited
  releases and named releases (Kokai Zuma, DV360 quarterly updates).
- **Inventory and deals** — new inventory sources, deal types (PMP, PG, curated deals),
  CTV / audio / DOOH / retail-media inventory becoming buyable in our DSPs.
- **Targeting / audience / identity** — new segments and data sources inside the DSP,
  contextual and show-level targeting, UID2 / EUID, first-party data connections.
- **Bidding / optimisation / automation** — bid strategies, pacing, AI agents and
  assistants (Koa, agentic buying features) that change how a campaign is run.
- **Creative formats** — new ad formats and placements (pause ads, interactive CTV,
  shoppable formats, home-screen ads bought through our DSPs).
- **Reporting / measurement / attribution** — new reports, metrics, lift studies,
  attribution and measurement integrations in the DSP.
- **Brand safety / suitability / transparency** — controls, verification, supply-path
  controls in the DSP.
- **Policy / compliance / eligibility** — rules that gate what can run or how.
- **Deprecation / sunset** — a feature, line-item type, metric or product being retired
  (e.g. legacy line items archived, a DSP being wound down).
- **API change** — only when it exposes a capability traders use; pure developer
  changes are out (below).
- **External change that concretely hits programmatic buying in our DSPs** — browser /
  OS / privacy / regulation changes that affect targeting, identity or measurement there.

## Exclude (even if it names one of our platforms)

- **Other DSPs, SSPs and ad-tech vendors — always out.** Yahoo DSP, StackAdapt, Viant,
  PubMatic, Magnite, Index Exchange, OpenX, verification and measurement vendors' own
  products, agencies' trading desks. The exception is the "where it lands" rule above: a
  vendor capability that becomes usable *inside* our DSP is in.
- **Developer-only API news** — new API versions, client libraries, field renames,
  quota or auth changes with no new capability for the trader.
- Case studies, customer success stories, "how brand X grew Y%" results.
- Business news: earnings, stock moves, layoffs, hires and exec moves, M&A, funding,
  partnerships with no product change.
- Lawsuits and rulings, unless the outcome changes how ads are bought in our DSPs.
- Thought leadership, research reports, trend commentary, event announcements.
- Streaming / TV business news with no ad-product change (subscribers, ratings,
  content deals, viewership studies).
- Performance-account news — it belongs to the performance track (`criteria.md`): Google
  Ads, Microsoft Advertising account campaigns, Meta, TikTok, LinkedIn, ChatGPT Ads.

When in doubt at **collection** time, lean include; the strict filter and impact scoring
at **presentation** time (Routine 2) do the trimming. **The vendor exclusion is the
exception — apply it at both stages.**

---

## Per-platform guidance (what to grab, what to ignore)

| Platform | GRAB | IGNORE |
|---|---|---|
| **The Trade Desk** | Kokai / Zuma release features, Koa and agentic tools, new data and inventory in the platform (CTV, retail, audio), OpenPath and supply-path controls, UID2 / EUID, measurement and reporting, deprecations. | Earnings, stock, layoffs, exec moves, Jeff Green interviews, case studies, research pieces ("Instant Intelligence", "Behind the research"), webinars. |
| **DV360** | Quarterly "Coming soon" updates, line-item and IO changes, YouTube & partners / Demand Gen in DV360, deals and Marketplace, audiences, brand controls, reporting, API releases that expose new capabilities. | API-only housekeeping (versions, client libraries), Search Ads 360, Campaign Manager 360, Google Ad Manager (publisher side). |
| **Amazon DSP** | DSP features, Prime Video and Amazon CTV buying, new audiences and signals, AMC for DSP, agentic / creative tools in the DSP, deprecations and transitions (e.g. Microsoft Invest customers moving to Amazon DSP). | Amazon earnings, Sponsored Products / seller news, retail operations, case studies. |
| **Microsoft programmatic** | Microsoft Invest and Curate changes, programmatic CTV / Netflix deals via Microsoft, open-exchange inventory changes. | Microsoft Advertising account campaigns (performance track), Monetize / publisher news, Microsoft corporate PR. |
| **CTV** | New CTV ad formats and placements, programmatic availability of a publisher's inventory, CTV measurement and targeting available to buyers. | Subscriber counts, ratings, content and distribution deals, streaming business analysis. |

---

## Impact scoring

| Impact   | Meaning                                                        |
|----------|----------------------------------------------------------------|
| `high`   | Affects live campaigns / requires action soon.                 |
| `medium` | Worth knowing, no immediate action.                            |
| `low`    | Minor / informational.                                         |

Same caps as the performance track (Routine 2 step 3): **max 6 items per platform**, up to
**8** to fit all `high`-impact items; overflow carries to a future edition; un-presented
items older than 28 days are retired.

## Category tags (for `updates.json.category`)

`feature` · `beta` · `policy` · `deprecation` · `measurement` · `api` · `other`

Pick the single best fit (inventory and deal changes are `feature`). Use `other`
sparingly.

## Canonical-source preference (dedup)

When the **same story** appears on both an `official` source (The Trade Desk, Google,
Microsoft) and an `aggregator` (AdExchanger, Digiday, MediaPost, Adweek, PPC Land), keep
the **official** one as the canonical record and drop the aggregator duplicate. If only
the aggregator has it, keep the aggregator version.
