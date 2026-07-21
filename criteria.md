# Relevance Criteria

> **Human-edited file.** Ivan (and future editors) tune the rules here. Routine 2
> reads this file at run time and filters the collected updates against it before
> building the deck. Keep it in plain language — the LLM applies judgment, not regex.

---

## Include an update if it is one of

- **New ad feature, format, or placement** — including betas / limited releases.
- **Targeting, bidding, or budget/automation change** — audiences, Smart Bidding,
  PMax, Advantage+, budget controls, automation defaults.
- **Policy, compliance, or eligibility change** — ad policies, verification,
  restricted categories, account requirements.
- **Deprecation / sunset** of a feature, setting, report, or metric.
- **Measurement, tracking, or attribution change** — tags, conversions, consent
  mode, enhanced conversions, GA4/CAPI, attribution models.
- **Notable API or platform change** relevant to campaign management — Google Ads
  API, Marketing API, reporting/UI changes that affect day-to-day work.

## Exclude

- Generic brand/marketing "inspiration" posts, case studies, customer stories.
- Event announcements, awards, hiring, corporate PR with **no product change**.
- Consumer-product news unrelated to advertising (e.g. a new consumer app feature
  with no ad surface).
- Pure thought-leadership / trend commentary with no concrete platform change.

When in doubt, **lean include** at collection time (Routine 1) and let this filter
plus impact scoring (Routine 2) do the trimming. It is cheaper to drop a collected
item than to miss one entirely.

---

## Impact scoring

| Impact   | Meaning                                                        |
|----------|----------------------------------------------------------------|
| `high`   | Affects live campaigns / requires action soon.                 |
| `medium` | Worth knowing, no immediate action.                            |
| `low`    | Minor / informational.                                         |

The deck (Routine 2) leads with `high`, then `medium`; `low` items are grouped or
dropped if the deck is getting long (see `style/`).

---

## Category tags (for `updates.json.category`)

`feature` · `beta` · `policy` · `deprecation` · `measurement` · `api` · `other`

Pick the single best fit. Use `other` sparingly.

---

## Canonical-source preference (dedup)

When the **same story** appears on both an `official` source and an `aggregator`
(Search Engine Land), keep the **official** one as the canonical record and drop the
aggregator duplicate. If only the aggregator has it, keep the aggregator version but
note that it is unconfirmed by an official post. This is exactly what dedup Layer 2
(semantic) is for — see `routines/routine-1-collect.md`.
