# Sources log

History of the sources in [`sources.yaml`](../sources.yaml): when a source was added,
broke, recovered or changed how it has to be read. `sources.yaml` holds only the current
state; this file holds how it got there. Newest first.

**Routine 1 appends here only when something changes** — a source's `validation` status
changes, its URL or reading method changes, or a source is added or removed. A run in
which nothing changed writes nothing here ("still blocked" is not an entry).

Entry format:

```
## YYYY-MM-DD
- `<key>`: <old status> → <new status> — <what was observed, in one line>.
```

---

## 2026-10-21
Source list rebuilt from Ivan's updated table; every entry point re-validated via curl
from the run environment.
- **Added, all `readable`:** `seroundtable_google_ads`, `microsoft_ads_blog`,
  `seroundtable_microsoft_ads`, `adexchanger_chatgpt_ads`, `socialmediatoday_facebook`,
  `socialmediatoday_instagram`, `tiktok_ads_blog`, `socialmediatoday_tiktok`,
  `linkedin_ads_blog`, `ppcland_pulse`, `ppcnewsfeed`, `sej_ppc`, `sej_social`.
  `about.ads.microsoft.com`, `www.adexchanger.com`, `ppcnewsfeed.com` and
  `www.searchenginejournal.com` were added to the environment's egress allowlist first.
  The TikTok and LinkedIn official blogs, dropped as unreadable in July, now serve their
  article lists server-side.
- **URL choices that differ from the table:** SER Microsoft uses `/category/adcenter`
  (SER's Microsoft Advertising category) instead of `/category/bing` (the same ads posts
  among organic SERP tests); PPC News Feed uses `/ppc-news/` because the homepage does
  not paginate; the Google Ads & Commerce blog now points straight at its RSS feed.
- **Removed:** `ppcland_search` and `ppcland_social` (`/tag/search/`, `/tag/social/` —
  frozen since August, see below), replaced by the single `ppcland_pulse` (scope: all).
- `searchengineland_*` (all 5): still `blocked` (Cloudflare challenge). Kept in the list
  in case the site recovers; one request per run, no workarounds.
- `bing_blogs`: readable → readable (note corrected) — post URLs are now `/search/YYYY/M/<Slug>/` with a numeric month, not `/search/<Month-YYYY>/<Slug>`; newest post June 2026.
- `sources.yaml` reduced to current state; its dated header notes moved here. Routine 1
  now records status changes here and DMs Ivan in Russian when a status changes.

## 2026-10-20
- PPC Land restructured. `/tag/news/` and `/tag/news/page/N/` return HTTP 404. The tag
  set was rebuilt around five section tags: `pulse` (news — what `/tag/news/` used to
  carry), `decoded` (glossary entries, ~40% of all cards, never news), `gridlock` (word
  games), `intermission` (entertainment), `landline` (telecom/privacy roundups).
  `/tag/search/` and `/tag/social/` still frozen (8th week). That run read PPC Land by
  paging back through the homepage (`/page/N/`, ~55 pages for 16 days, keeping only
  `pulse` cards). `/rss/` holds only ~2 days; `sitemap-posts.xml` dates are lastmod.

## 2026-09-22
- `searchengineland_google`, `searchengineland_meta`, `searchengineland_tiktok`,
  `searchengineland_microsoft`, `searchengineland_chatgpt`: readable → **blocked**.
  Every searchengineland.com URL (library pages, articles, `/feed`, `/sitemap_index.xml`)
  returns HTTP 403 with `cf-mitigated: challenge` — Cloudflare's interactive bot
  challenge — under a Chrome UA, curl's UA, no UA and a Googlebot UA alike. Headless
  Chromium loads the interstitial, but the challenge script host
  `challenges.cloudflare.com` is denied by the egress policy, so it cannot be solved.
  Re-checked unchanged on 2026-09-29, 10-06, 10-13 and 10-20. Ivan confirmed the site
  does not open for him in a browser either (2026-10-21). This cost Meta and TikTok
  their main source until the 2026-10-21 additions.
- `google_ads_commerce_blog`: the HTML index exposed only 3 article links;
  `/products/ads-commerce/rss/` returned the full dated list and also carries ads posts
  filed under other blog.google paths. Switched to the feed.

## 2026-09-16
- `ppcland_search` carried two Display & Video 360 items filed as `google_ads`
  (editions 3 and 4). Product test added to criteria.md; the source note told to drop
  DV360 / CM360 / SA360 / Ad Manager.

## 2026-09-01 → 2026-10-13
- `ppcland_search` / `ppcland_social`: `/tag/search/` and `/tag/social/` froze (newest
  cards Aug 19 / Aug 17) while the site kept publishing daily. Not a fetch problem
  (HTTP 200; cache-busting and `Cache-Control: no-cache` changed nothing); per-platform
  tags such as `/tag/google-ads/` are 404. Runs paged back through `/tag/news/page/N/`
  (~12 cards per page, ~2 weeks ≈ 15–27 pages) and applied the search / social scope.
  On 2026-09-01 that pagination fell into 2016 archive posts after page 14; from
  2026-09-08 it ran unbroken. On 2026-10-13 the per-page count read 11 once the
  `/signin/` card was discarded.

## 2026-08-17
- Added `seroundtable_chatgpt_ads` and `searchengineland_chatgpt` (new platform:
  ChatGPT Ads, Search team). SEL `/library/ppc` was evaluated and not tracked — its
  ChatGPT items duplicated the two sources above, plus evergreen "what is PPC" content.
  `ppcland_search` confirmed to carry ChatGPT Ads news.

## 2026-08-04
- Removed SocialBee (monthly roundup): its `roundup` cadence let month-old LinkedIn
  items past the 2-week window; Social Media Today covers the same announcements as a
  dated feed. No `roundup` source remains.
- searchengineland.com origin outage: index pages and ~14 articles read, then a fixed
  subset of article URLs returned Cloudflare 504 / timeouts on every attempt (4 retry
  rounds over ~40 min, varied UA, HTTP/1.1, cache-busting), and later the whole domain
  went 504. Origin-side, not anti-bot. Unreadable items were skipped and flagged rather
  than summarised from headlines. archive.org is not a fallback (denied by egress).

## 2026-08-03
- Added `ppcland_search` and `ppcland_social` (PPC Land tag pages; mixed platforms).
- searchengineland.com returned HTTP 504 for most requests in a parallel burst (8 at
  once → 17/20 failed); every 504 succeeded on retry at a concurrency of 3 or lower.
- `google_ads_announcements`: newest entry still May 20, 2026 (GML 2026) — updates
  rarely. `bing_blogs`: newest post June 2026.

## 2026-07-24
- `socialmediatoday_linkedin`: readable once the egress host was corrected to
  `www.socialmediatoday.com` (the apex 301s to www).

## 2026-07-21
- First validation: every source fetched via in-session curl (HTTP 200 once egress was
  open; the managed WebFetch tool stays blocked by the sites' anti-bot). Sources that
  could not be read — Meta for Business news, the TikTok blog, the LinkedIn Ads blog
  and the Google Search blog — were dropped.
