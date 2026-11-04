# SDG Home Concepts Migration Memory

This file is the repository-level operating plan for agents working on the
WordPress/Elementor-to-static-code migration of `https://sdghomeconcepts.com/`.
Read it before crawling, importing content, changing routes, generating a
sitemap, or launching the coded site.

## Objective

Rebuild the site as statically rendered HTML, CSS, and JavaScript while keeping
the legitimate public SEO output equivalent. The implementation may use a
build-time templating/static-generation tool, but production pages must deliver
complete, crawlable HTML and must not depend on client-side JavaScript for their
main content, links, metadata, or headings.

Changing the codebase is allowed. Changing legitimate URLs or SEO signals is
not allowed unless the change is documented and approved in the migration map.

## Known State (Observed 2026-11-04)

- The repository initially contained only `README.md`.
- The live Yoast page sitemap contained 63 normal-looking pages.
- The live post sitemap contained 163 posts, many with unrelated Office
  download, vulnerability, debug, or random-string slugs. Treat these as
  suspected injected spam, not legitimate content.
- The site uses trailing-slash URLs.
- Existing routes include inconsistent naming. For example, Dallas uses
  `/custom-homes-dallas/`, while other cities use
  `/custom-home-builder-{city}/`. Never regenerate routes from a naming formula;
  retain an explicit route map.
- `/aubrey-general-contractor-2/` exists and needs Search Console/analytics/
  backlink review before deciding whether it is legitimate, duplicate, or spam.
- The live page source includes a phone-click tracking webhook and a credential-
  like token. Do not copy the exposed value. The owner should rotate it and the
  replacement must be provided through environment/hosting configuration.
- The current visual system prominently uses `#633B13`, `#FFF5E2`, `#0D1218`,
  white, Poppins, and Lora. Visual recreation is separate from SEO parity.

These observations are a baseline, not permanent truth. Re-crawl the live site
and record the crawl date before implementation or launch.

## Non-Negotiable Migration Rules

1. Preserve each legitimate URL exactly, including lowercase spelling and
   trailing-slash behavior.
2. A legitimate unchanged URL must return `200`; it must not redirect merely
   because the implementation changed.
3. Preserve or deliberately approve changes to title, meta description,
   canonical, robots directives, H1, body copy, headings, image alt text,
   structured data, breadcrumbs, and internal links.
4. Every canonical must use the production HTTPS hostname, never staging.
5. Do not copy the current sitemap blindly. The new sitemap contains only
   approved, canonical, indexable URLs returning `200`.
6. Suspected injected/spam URLs must not be redirected to the homepage. Once
   confirmed, remove them from all sitemaps and return `410 Gone` or a real
   `404 Not Found`.
7. When a legitimate URL must change, use a direct one-to-one server-side
   `301` or `308` to its closest equivalent. Avoid redirect chains.
8. Preserve existing image URLs where practical. If an important indexed or
   linked image moves, add a direct permanent redirect.
9. Production HTML must contain the primary content and internal links before
   JavaScript executes.
10. Never launch with a staging `noindex`, staging canonical, robots block, or
    authentication requirement still active.
11. Do not commit DataForSEO, Search Console, analytics, webhook, hosting, or
    other credentials. Use environment variables and provide `.env.example`
    with names only.
12. Do not change page copy, information architecture, and platform in the same
    migration unless each SEO/content change is separately reviewed. Prefer a
    parity migration first and optimization after rankings stabilize.

## Sources of Truth and Priority

No single crawl is a complete URL inventory. Merge these sources:

1. Google Search Console pages, queries, links, index status, Security Issues,
   and Manual Actions.
2. GA4 organic landing pages and conversions; call/form conversion records.
3. DataForSEO ranked pages/keywords and backlink destination pages.
4. A link crawl of the live site.
5. All XML sitemaps and `robots.txt` declarations.
6. Server logs, historic redirects, and known campaign/profile URLs when
   available.

Search Console and analytics are authoritative for actual Google performance
and conversions. DataForSEO supplies external rankings, backlinks, crawling,
and estimates; do not treat estimated traffic as actual traffic.

## Credential Handling

Expected local/CI environment variable names:

```text
DATAFORSEO_LOGIN
DATAFORSEO_PASSWORD
```

Never print credential values in logs, reports, fixtures, screenshots, or tool
output. Before making paid API calls, validate the target, location, language,
page limit, and endpoint. Use a small test request first. Cap crawl pages and
result limits to control cost.

## Crawl Plan

### Phase 1: Baseline the Live WordPress Site

Record the date/time, crawler configuration, hostname, user agent, rendering
mode, and crawl limits. Save raw API responses outside version control if they
contain sensitive data; commit sanitized reports only when useful.

Use DataForSEO OnPage API to crawl the production hostname. Start with raw HTML
and resource loading disabled or minimized. Enable JavaScript/browser rendering
only when testing rendered-content parity or Elementor-loaded resources because
those options add cost.

Collect at minimum for every URL:

- requested and final URL
- HTTP status and redirect chain
- content type
- indexability and robots directives/headers
- title and meta description
- canonical
- H1 and ordered H2/H3 headings
- main visible text or normalized content hash and word count
- internal inlinks, outlinks, anchor text, and crawl depth
- structured data types and validation issues
- images, source URLs, dimensions where available, and alt text
- Open Graph/Twitter metadata
- response time and broken resources
- sitemap membership

Also retain the raw HTML for important landing pages so metadata and schema can
be independently verified.

Supplement the link crawl with:

- all sitemap URLs
- URLs discovered through Search Console and GA4
- DataForSEO ranked/relevant pages
- DataForSEO backlink destination pages
- known redirects and server-log URLs

This is how orphan URLs are found. Do not assume that a URL absent from the
navigation or sitemap has no SEO value.

### Phase 2: Pull DataForSEO Search and Link Data

Use DataForSEO Labs for the US/English market relevant to the business to pull:

- ranked keywords for the domain and important URLs
- ranking URL, position, search volume, intent, and estimated traffic
- relevant pages
- historical ranking data where useful
- local/service SERP checks for priority Frisco/DFW queries

Use DataForSEO Backlinks for:

- domain pages/domain pages summary
- backlinks and referring domains by destination URL
- anchor text
- live/lost status
- broken backlink destinations

Do not exhaustively request every optional field by default. Fetch enough data
to identify which URLs and assets carry ranking or link equity.

Use DataForSEO Lighthouse only for representative priority pages, not every URL,
unless the budget explicitly allows it. Include homepage, service hubs, the top
ranking city/service pages, contact, and at least one page from each template.

### Phase 3: Build the Master URL Inventory

Create a machine-readable inventory (CSV and/or JSON) with one row per normalized
URL and these fields where available:

```text
url
source_discovery
current_status
final_url
indexable
title
meta_description
canonical
h1
word_count
content_hash
internal_inlinks
crawl_depth
gsc_clicks
gsc_impressions
gsc_average_position
ga4_organic_sessions
conversions
ranking_keyword_count
best_ranking_position
referring_domains
backlinks
classification
migration_action
redirect_target
priority
notes
```

Allowed classifications:

- `preserve`: legitimate page stays at the exact URL
- `redirect`: legitimate old URL has an approved one-to-one replacement
- `remove-410`: confirmed injected/spam or intentionally removed content with no
  equivalent
- `remove-404`: removed/unknown URL for which `404` is appropriate
- `investigate`: evidence is insufficient or conflicting

Never auto-classify solely from a suspicious-looking slug. Cross-check Search
Console, analytics, backlinks, content, and business relevance. Do not generate
redirects for `investigate` rows.

Priority should reflect actual value:

- `critical`: traffic, conversions, valuable backlinks, or high rankings
- `high`: legitimate service/location hub or strong internal-link role
- `normal`: legitimate supporting page
- `cleanup`: confirmed spam/removal target

### Phase 4: Crawl and Compare Staging

Staging must remain unavailable to search engines through authentication and/or
`noindex`, but the authorized crawler must be able to access it. Never weaken
staging protection just to make crawling easier.

Crawl staging using the same extraction configuration as production. Map the
staging hostname to production before comparing paths. For every `preserve` URL,
compare:

- path and trailing slash
- intended production status (`200`)
- title and description
- production canonical
- intended production robots/indexability
- H1 and ordered headings
- normalized main content and word count
- structured data
- internal links, anchors, and crawl depth
- images and alt text
- Open Graph/Twitter metadata

Known staging-only differences such as hostname, authentication, and temporary
`noindex` must be explicit exceptions. They are not allowed in production.

Generate a human-readable parity report with `pass`, `review`, and `fail` states.
Fail the migration gate for missing pages, incorrect status, wrong canonical,
unexpected `noindex`, missing H1/main content, broken navigation, or unmapped
high-value URLs.

### Phase 5: Pre-Launch Gates

Launch is blocked until all of the following pass:

- Every `critical` and `high` URL has been manually reviewed.
- Every `preserve` URL exists on staging and is expected to return production
  `200`.
- Redirect rules are one-to-one and tested without chains or loops.
- Confirmed spam URLs are absent from the sitemap and have planned `410`/`404`
  handling.
- Sitemap URLs are canonical, indexable, and expected to return `200`.
- No production page references a staging hostname.
- Production `robots.txt` allows legitimate crawling and references the clean
  sitemap.
- Metadata, headings, main copy, schema, internal links, and image alt text meet
  the approved parity threshold.
- Forms, phone links, call tracking, GTM/GA4, consent behavior, and conversions
  have been tested without exposing secrets.
- Mobile layouts, keyboard access, reduced motion, and representative Lighthouse
  checks pass the agreed thresholds.
- Backups and rollback steps exist.

## Launch and Post-Launch Crawl

Immediately after DNS/hosting cutover, run a list crawl of every approved URL and
redirect target. Verify live HTTP responses rather than trusting deployment logs.

Check:

- all `preserve` URLs return `200`
- all approved redirects reach the final destination in one hop
- all cleanup URLs return the planned `404`/`410`
- no `noindex`, staging canonical, staging hostname, or auth challenge remains
- assets, fonts, CSS, JS, images, forms, and tracking load correctly
- HTTP to HTTPS and hostname normalization behave consistently
- the clean sitemap is reachable and submitted in Search Console

Monitor Search Console daily for the first two weeks, then weekly for at least
six additional weeks. Track page-level clicks, impressions, average position,
index status, crawl errors, security issues, legitimate 404s, conversion events,
and spam URL removal. Compare pages against the same pre-migration baseline and
investigate page-specific sustained drops.

## Expected Repository Outputs

When crawl automation is implemented, prefer this organization:

```text
scripts/seo/
  crawl-production.js
  crawl-staging.js
  fetch-rankings.js
  fetch-backlinks.js
  compare-crawls.js
  generate-report.js

reports/seo/
  url-inventory.csv
  seo-parity-report.html
  spam-url-review.csv
  backlinks-at-risk.csv
  ranking-pages.csv
```

Generated reports containing credentials, full API responses, sensitive
analytics, or customer data must be ignored by Git. Prefer deterministic scripts
and sanitized fixtures over manually edited reports.

## Agent Handoff Requirements

Any agent completing crawl or migration work must report:

1. crawl date and exact target
2. tools/endpoints and rendering mode used
3. number of discovered, crawled, failed, indexable, redirected, and removed URLs
4. API cost-affecting options and limits
5. unresolved `investigate` URLs
6. parity failures and approved exceptions
7. files created or changed
8. commands/tests used to verify the result

Do not describe the migration as safe merely because the URLs look unchanged.
Safety must be demonstrated by the inventory, parity comparison, HTTP checks,
and post-launch monitoring.
