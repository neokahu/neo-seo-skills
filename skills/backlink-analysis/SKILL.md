---
name: backlink-analysis
version: "1.0.0"
description: 'Deep backlink profile analysis with distributions by authority score, website type, link type, anchor classification, mention context, guest post type, and website topic. Use when asked to "analyze backlinks", "backlink profile", "link audit", "who links to", "backlink distribution", "link building intel", or "competitor backlink analysis". Requires Ahrefs MCP. Uses Firecrawl for page crawling and Tavily as fallback.'
---

# Backlink Analysis

Deep backlink profile analysis that goes beyond standard metrics to classify every dimension of a link profile.

## When to Use

- Analyzing a competitor's backlink profile for strategic intelligence
- Auditing your own backlink health and distribution
- Finding replicable link building patterns from competitors
- Identifying toxic link signals
- Planning link building strategy based on real data

## Required Tools

| Tool | Purpose | Required? |
|------|---------|-----------|
| **Ahrefs MCP** | Backlink data, referring domains, anchors, DR scores | Yes |
| **Firecrawl MCP** | Crawl linking pages for context classification | Recommended |
| **Tavily MCP** | Web research fallback | Optional |

## Instructions

When a user requests backlink analysis for a domain:

### Phase 1: Data Collection (Ahrefs MCP)

Run these queries in parallel where possible. Use `mode=subdomains` for domain-level analysis. Use `doc` tool first if unsure about column names.

**1.1 Core Metrics**
- `site-explorer-domain-rating` — DR score and Ahrefs rank
- `site-explorer-metrics` — organic keywords, traffic, traffic value, paid metrics
- `site-explorer-backlinks-stats` — live/all-time backlinks, live/all-time referring domains

**1.2 Backlink Samples (pull 3-4 slices for diverse coverage)**

Use `site-explorer-all-backlinks` with these select columns:
```
url_from, domain_rating_source, url_to, anchor, is_dofollow, is_content, is_image, is_text, is_ugc, is_sponsored, is_nofollow, page_category_source, title, root_name_source, traffic, traffic_domain
```

Pull separate samples ordered/filtered differently:
- Sorted by `domain_rating_source:desc` with `is_dofollow=true` (high-authority dofollow)
- Sorted by `traffic:desc` (highest-traffic linking pages)
- Filtered by `is_content=true` + `is_dofollow=true` + `domain_rating_source<=80` (mid-tier editorial links)
- Filtered by `is_content=true` + `is_dofollow=true` + `domain_rating_source<=50` (lower-tier content links)
- Filtered by `is_image=true` (image backlinks)

**1.3 Anchor Text Distribution**

Use `site-explorer-anchors` with columns: `anchor, links_to_target, refdomains, dofollow_links`. Order by `refdomains:desc`, limit 50+.

**1.4 Referring Domains by DR Tier**

Use `site-explorer-referring-domains` with columns: `domain, domain_rating, links_to_target, dofollow_links, traffic_domain`.

Pull separate queries filtered by DR range to build the full distribution:
- DR 95-99 (mega platforms)
- DR 80-94 (high authority)
- DR 60-79 (mid-high authority)
- DR 40-59 (mid authority)
- DR 20-39 (low-mid authority)
- DR 0-19 (low authority)

**1.5 Pages Attracting Most Backlinks**

Use `site-explorer-pages-by-backlinks` with columns: `url_to, refdomains_target, dofollow_to_target, links_to_target`. Order by `refdomains_target:desc`.

**1.6 Broken Backlinks**

Use `site-explorer-broken-backlinks` with columns: `url_from, domain_rating_source, url_to, anchor, is_dofollow, http_code`. Limit 100. Used for broken/reclaimed classification in Phase 3.

### Phase 2: Page Crawling for Classification (Firecrawl/Tavily)

Select 10-15 representative linking pages from the backlink samples — pick diverse examples across DR tiers, website types, and content types. Use a background agent to crawl them in parallel.

For each page, classify:
1. **Website type**: blog, forum/community, educational (.edu), government (.gov), social platform, directory/listing, news/media, SaaS/tool, wiki/reference, personal site, ecommerce, CDN/infrastructure, nonprofit
2. **Link context**: tool recommendation, incidental mention, resource page listing, user-generated content, how-to guide mention, comparison/review, editorial/news mention, guest post, platform auto-link, image embed, sidebar/footer widget
3. **Website topic**: tech, marketing, education, business, food/hospitality, nonprofit, real estate, finance, healthcare, design, government, general/personal
4. **Link status**: live or dead (404/DNS failure/expired)

### Phase 3: Analysis & Classification

Using the collected data, compute the following distributions. Use sampled data + domain-level patterns to estimate percentages across the full profile.

**3.1 Total Backlink Overview**
- Live backlinks, all-time backlinks, live/all-time referring domains
- Link decay rate: `(all_time - live) / all_time * 100`
- Dofollow vs nofollow vs UGC vs sponsored ratio (from samples)

**3.2 Distribution by Authority Score (DR)**

Bucket all referring domains into DR tiers. Show:
- Estimated count and % per tier
- Notable domains per tier
- ASCII bar chart visualization
- Analysis of where the real SEO value concentrates

**3.3 Distribution by Website Type**

Classify referring domains into categories. For each category show:
- Estimated % and domain count
- Key examples
- Whether the pattern is replicable

Categories: blogging platforms (hosted), independent blogs, forums/communities, educational (.edu), tech/SaaS platforms, social platforms, news/media, wiki/reference, directories/listings, ecommerce/business, government/nonprofit, CDN/infrastructure, spam/PBN

**3.4 Distribution by Link Type**

Classify from backlink sample attributes:
- **Text links** (is_text=true, anchor not empty)
- **Naked URL links** (anchor matches URL pattern)
- **Image links** (is_image=true)
- **Empty anchor / widget** (anchor="" and is_text=true, or not is_content)
- **Frame/embed** (if detectable)

**3.5 Distribution by Anchor Text**

Classify all anchors into categories:
- **Brand name**: contains the domain/company name
- **Naked URL**: full URL as anchor
- **Page title (auto-generated)**: contains " | " or matches title tag patterns
- **Generic keyword**: short keyword phrases like "QR code", "click here"
- **Contextual keyword**: descriptive phrases with target keywords
- **Spam/irrelevant**: unrelated commercial terms, foreign-language spam
- **Empty/image**: no text anchor (image links)

**3.6 Distribution by Mention Context**

Classify how the target is mentioned on linking pages:
- **Tool recommendation (resource page)**: "use this tool" on resource lists
- **Incidental mention**: passing reference in unrelated content
- **Platform/widget auto-link**: CMS-generated, sidebar, footer
- **User-generated content**: forum posts, Q&A, comments
- **How-to guide / tutorial**: step-by-step guides referencing the tool
- **Comparison / review**: listicles and comparison articles
- **Editorial / news mention**: journalistic coverage
- **Guest post / contributed content**: content placed on third-party sites
- **Internal ecosystem**: cross-linking from parent/sister properties
- **Image embed**: pages displaying generated content linking back

**3.7 Distribution by Guest Post Type** (subset of guest post links)

If guest posts are detected, sub-classify:
- **Full promotional**: dedicated article about the product
- **Incidental mention in contributed content**: natural mention in broader topic
- **Comparison / roundup inclusion**: featured in "best of" guest posts
- **Sponsored/paid placement**: identifiable as paid content

**3.8 Distribution by Website Topic/Category**

Use `page_category_source` from Ahrefs data plus manual classification:
- Technology/Software, Marketing/Advertising, Education/Academic, General/Personal, Business/Industrial, Food/Hospitality, Nonprofit/Charity, News/Media, Design/Creative, Real Estate, Finance, Healthcare, Government/Public Sector

**3.9 Distribution by Acquisition Method**

Classify each backlink by how it was acquired — the primary strategic lens for replicability. Determine acquisition method from a combination of Ahrefs data fields (is_content, is_ugc, is_sponsored, anchor patterns, page_category_source) and crawled page context from Phase 2.

Categories:
- **Profile link**: Link from a user profile page on a platform (forum profile, SaaS account, social bio, author bio on blogging platforms). Signals: URL pattern contains `/profile/`, `/user/`, `/members/`; page title contains "profile" or username; low word count on linking page.
- **Guest post**: Content contributed to a third-party site with author byline. Signals: author bio box present, "guest post" / "contributor" / "guest author" labels, byline doesn't match site owner.
- **Editorial/contextual**: Naturally placed by a content creator referencing the target as a source, tool, or example. Signals: is_content=true, link embedded mid-paragraph in substantial content, no guest post indicators.
- **Directory/listing**: Submitted to a business or tool directory. Signals: page is a structured listing with multiple similar entries, URL contains `/listing/`, `/tool/`, `/app/`; sites like Capterra, G2, Product Hunt, AlternativeTo.
- **Resource page**: Listed on a curated "best tools", "useful resources", or "recommended" page. Signals: page title contains "best", "top", "resources", "tools"; bulleted/numbered list of links with brief descriptions.
- **Forum/community**: Link posted in a discussion thread, Q&A answer, or community post. Signals: is_ugc=true, forum URL patterns (`/thread/`, `/topic/`, `/discussion/`, `/answers/`), sites like Reddit, Quora, Stack Overflow.
- **Blog comment**: Link left in a comment section. Signals: URL fragment contains `#comment`, page context shows comment thread, typically nofollow.
- **Social bookmark/curation**: Submitted to content curation platforms. Signals: domains like scoop.it, mix.com, flipboard.com, pinterest.com; page is a "scoop" or "pin" or "collection".
- **Press/PR**: From press releases, news coverage, or journalist mentions. Signals: news/media domain, article format with dateline, press release distribution sites (PRWeb, BusinessWire, PR Newswire).
- **Affiliate/partner**: From business relationship pages — integration partners, affiliate reviews, co-marketing. Signals: page contains affiliate disclaimers, URL has tracking parameters, partner/integration directory pages.
- **Competitor comparison**: From "vs", "alternatives to", or comparison articles. Signals: title contains "vs", "alternative", "compared to", "competitor"; structured comparison table present.
- **Educational/institutional (.edu/.gov)**: From academic or government domains. Signals: root domain ends in .edu or .gov; often resource page or course material context.
- **Sponsored/paid**: Identifiable paid placements. Signals: is_sponsored=true, "sponsored" label on page, disclosure statements, advertorial format.
- **Aggregator/scraper**: Auto-generated by bots or RSS scrapers. Signals: duplicate content from other sites, auto-generated page titles, domain is a known scraper/mirror site, thin or no original content.
- **Image/embed**: Link from an embedded image, infographic, badge, or widget. Signals: is_image=true, anchor is empty or image filename, page embeds a visual asset from target domain.
- **Broken/reclaimed**: Previously dead links that were reclaimed or redirected. Signals: identified via `site-explorer-broken-backlinks` data, 301 redirects in place.

Show distribution as table with estimated count, percentage, dofollow ratio per type, replicability rating (easy/medium/hard), and 1-2 example domains per type.

**3.10 Toxic Link Assessment**

Flag specific toxic signals with severity ratings:
- Spam anchor text clusters
- Excessive empty-anchor volume
- Dark web / suspicious domain links
- Auto-generated blog spam
- Single-domain excessive link counts
- Link decay rate interpretation

Assign a profile health score (1-10).

### Phase 4: Report Generation

Structure the output as a single markdown file with these sections:

```
1. Executive Summary (3-4 sentences)
2. Total Backlink Overview (table)
3. Distribution by Authority Score (table + chart + analysis)
4. Distribution by Website Type (table + analysis)
5. Distribution by Link Type (table)
6. Distribution by Anchor Text (table + classification)
7. Distribution by Mention Context (table + analysis)
8. Distribution by Guest Post Type (table, if applicable)
9. Distribution by Acquisition Method (table + replicability ratings)
10. Distribution by Website Topic (table + analysis)
11. Toxic Link Assessment (table + health score)
12. Replicable Link Building Patterns (strategy table)
13. Action Plan (tiered: immediate / short-term / long-term)
14. Key Takeaways (numbered list)
15. Appendix: Crawled Page Classification (table from Phase 2)
```

### Phase 5: Visualization Dashboard

After the markdown report is complete, build a single-file HTML dashboard that visualizes the key findings. Use Chart.js (loaded from CDN) for all charts.

The dashboard should include:

**Summary cards row:** Total genuine links, unique domains, dofollow %, profile health score

**Charts:**
- Donut chart: DR tier distribution (with link counts and percentages)
- Horizontal bar chart: Website type distribution
- Donut chart: Anchor text classification
- Donut chart: Mention context distribution
- Horizontal bar chart: Acquisition method distribution (with replicability color coding)
- Horizontal bar chart: Link type breakdown (text/image/frame)

**Tables:**
- Top DR 40+ domains with DR score, link type (DF/NF), and context
- Top referring domains by link count
- Toxic link signals with severity ratings

**Design guidelines:**
- Clean, modern dashboard layout with CSS grid
- Dark header with domain name and key stats
- White cards with subtle shadows for each section
- Responsive — works on desktop and tablet
- Color palette: use consistent colors across all charts (blue primary, grays for secondary)
- Save as `[domain]-backlink-dashboard.html` alongside the markdown report

**Important:** Only visualize the genuine/filtered link data, not the raw inflated numbers. The dashboard should tell the same story as the report — show the real competitive surface.

### Ahrefs MCP Column Reference

Common column names that differ from intuitive naming:
- Use `sum_traffic` not `traffic` for keyword traffic
- Use `best_position` not `position` for rankings
- Use `best_position_url` not `url` for ranking URLs
- Use `links_to_target` not `backlinks` in referring-domains
- Use `refdomains` not `referring_domains` in anchors
- Use `url_to` not `url` in pages-by-backlinks
- Use `refdomains_target` not `referring_domains` in pages-by-backlinks
- Use `is_dofollow` not `dofollow` in all-backlinks
- Where filter uses JSON syntax: `{"field":"name","is":["operator","value"]}`
- Boolean filters: `{"field":"is_dofollow","is":["eq",true]}`
- Range filters: `{"and":[{"field":"domain_rating","is":["gte",60]},{"field":"domain_rating","is":["lte",79]}]}`
- Always use `doc` tool first if a column error occurs

## Validation Checkpoints

### Before Writing Report
- [ ] Backlinks sampled across 3+ DR tiers (not just top-DR)
- [ ] Anchor text distribution covers 20+ unique anchors
- [ ] At least 10 linking pages crawled for context classification
- [ ] All 9 distribution types computed
- [ ] Toxic signals identified with severity ratings
- [ ] Replicable patterns identified with effort/impact assessment

### Output Quality
- [ ] Every percentage cites sample size or estimation method
- [ ] Data source noted for each metric (Ahrefs API, Firecrawl crawl, estimated)
- [ ] Dead backlinks flagged in crawl appendix
- [ ] Action plan items are specific and actionable
- [ ] Key takeaways directly inform link building strategy
