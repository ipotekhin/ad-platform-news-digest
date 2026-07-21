# Relevance Criteria

> **Human-edited file.** Ivan (and future editors) tune the rules here. Both routines
> read this file at run time — Routine 1 applies it *loosely* (lean include), Routine 2
> applies it *strictly*. Keep it in plain language — the LLM applies judgment, not regex.

---

## Scope — what this digest is about

**Only updates that change how a paid-advertising account / ads manager works.**
If a change would make a PPC/SEM specialist open the ads manager, adjust a campaign,
a report, or a tracking setup — it's in. If it wouldn't, it's out.

The test for every candidate:
> "Does this change something I can see or do inside the ad account (campaign setup,
> targeting, bidding, budgets, creative/format, reporting, tracking, policy/eligibility,
> or the ads API)?"

If **no** → drop it, however big the headline.

## Include if it is one of

- **New ad feature / capability** in the ads manager — incl. betas & limited releases.
- **New or changed ad format / placement / creative option** (e.g. new video format,
  new asset type, new inventory/placement).
- **Targeting / audience change** — new audience types, signals, segments, exclusions.
- **Bidding / budget / automation change** — bid strategies, budget controls, automation
  defaults, PMax / Advantage+ / Demand Gen behavior.
- **Reporting / measurement / attribution change** — new reports, columns, metrics,
  dashboards; conversion tracking, tags, consent mode, attribution models.
- **Algorithm / delivery / auction change** that affects performance or optimization.
- **Policy / compliance / eligibility change** that gates what ads can run or how.
- **Deprecation / sunset** of an ad feature, setting, report, or metric.
- **Ads API / platform change** relevant to campaign management or reporting.

## Exclude (even if it's from an ad platform)

- Consumer-product / app news with **no ad surface** (e.g. a new messaging feature).
- Corporate PR: earnings, hiring, awards, events, partnerships, brand campaigns.
- Case studies, customer success stories, generic "best practices" / inspiration.
- Pure thought-leadership / trend commentary with no concrete product change.
- Safety/policy news that doesn't touch advertiser-facing rules or eligibility.

When in doubt at **collection** time (Routine 1), lean include; the strict filter and
impact scoring at **presentation** time (Routine 2) do the trimming.

---

## Per-platform guidance (what to grab, what to ignore)

The official newsrooms (esp. Meta) publish a lot of non-ads news. Use this to focus.

| Platform | GRAB (ads-manager relevant) | IGNORE |
|---|---|---|
| **Google Ads** | Performance Max, Demand Gen, Search/Shopping/PMax features, bidding & budget controls, audience signals, asset/format changes, Google Ads API, new reports/columns, conversion & consent tracking, policy/eligibility. | Consumer Search/Gemini features with no ad surface, Workspace, hardware, corporate PR. |
| **Meta** | Advantage+ (shopping/app/leads), Ads Manager features, new placements/formats (Reels, Stories), audience & Advantage+ audience, attribution/CAPI/Events Manager, catalog/commerce ads, ad policy. | Consumer app features (Threads/IG/WhatsApp product news), VR/AI research, policy for users (not advertisers), corporate/earnings PR. |
| **TikTok** | TikTok Ads Manager features, Spark Ads / new ad formats, Smart+ / automated campaigns, targeting & optimization, Events API / attribution, commerce/Shop ads, ad policy. | Creator/consumer features, entertainment/culture posts, corporate PR. |
| **LinkedIn** | Campaign Manager features, new ad formats (Thought Leader, Conversation, Document ads), audience/targeting, bidding, LinkedIn Insight Tag / conversion tracking, reporting, Marketing API. | Consumer feed features, hiring/talent products, corporate PR, general LinkedIn tips. |
| **Microsoft / Bing** | Microsoft Advertising features, Performance Max, new formats/extensions, Copilot/AI ads, audience & bidding, UET / conversion tracking, reporting, Ads API, import from Google Ads, policy. | Bing consumer search/Copilot news with no ad surface, Windows/Edge, corporate PR. |

---

## Impact scoring

| Impact   | Meaning                                                        |
|----------|----------------------------------------------------------------|
| `high`   | Affects live campaigns / requires action soon.                 |
| `medium` | Worth knowing, no immediate action.                            |
| `low`    | Minor / informational.                                         |

The deck (Routine 2) leads with `high`, then `medium`; `low` items are grouped or
dropped if the deck is getting long (see `style/`).

## Category tags (for `updates.json.category`)

`feature` · `beta` · `policy` · `deprecation` · `measurement` · `api` · `other`

Pick the single best fit. Use `other` sparingly.

## Canonical-source preference (dedup)

When the **same story** appears on both an `official` source and an `aggregator`
(Search Engine Land / SocialBee), keep the **official** one as the canonical record and
drop the aggregator duplicate. If only the aggregator has it, keep the aggregator
version but note it is unconfirmed by an official post. This is what dedup Layer 2
(semantic) is for — see `routines/routine-1-collect.md`.
