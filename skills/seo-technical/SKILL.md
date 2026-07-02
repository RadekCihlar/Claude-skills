---
name: seo-technical
description: Use when building or auditing public-facing pages — titles/meta, canonicals, structured data, sitemaps, hreflang/i18n, Core Web Vitals, and crawlability. Technical SEO only, not content marketing.
---

# Technical SEO

Search engines reward pages that are fast, unambiguous, and machine-readable. Every rule here is verifiable — no cargo cult.

## One URL per thing

- **One canonical host + scheme**, everything else 301s (www→apex, http→https, trailing-slash policy — pick once). Duplicate hosts split ranking signal.
- `rel=canonical` on every indexable page, absolute URL, pointing at itself (or the preferred variant for parametrized/duplicate views).
- Redirects: permanent moves = 301, chains ≤1 hop, no redirect loops. Removed content: 410 if gone forever, 301 to the closest replacement if one exists — not soft-404 (200 with "not found" text).

## Head hygiene (per page, not per site)

- **Title**: unique, ~50-60 chars, most specific words first ("Radar Brno — počasí teď" not "MyApp | Pages | Radar | Brno"). It's the search-result headline — write it as one.
- **Meta description**: unique, ~150 chars, states what the page answers; it's the CTR pitch, not a keyword sack.
- **Open Graph/Twitter**: `og:title`, `og:description`, `og:image` (1200×630, absolute URL) — social embeds are a distribution channel.
- `<html lang>` correct; one `<h1>`; heading hierarchy without skips.

## Structured data

JSON-LD matching what the page visibly shows (mismatch risks manual action): the specific type that fits — Article, Product, FAQPage, BreadcrumbList, LocalBusiness, Event. Validate with the Rich Results test. CSP with nonces → JSON-LD `<script>` needs the nonce too.

## Crawlability

- **Content in the HTML response.** Client-only rendering of primary content is a gamble; SSR/SSG anything that should rank. Verify: `curl` the page — is the content in the source?
- `robots.txt` blocks only what must be blocked (never CSS/JS); `noindex` via meta/header for thin pages (search results, filters, infinite parameter spaces) — robots.txt-blocked pages can't see their own noindex.
- **XML sitemap**: canonical URLs only (no redirects, no noindexed), real `lastmod`, split at 50k, referenced from robots.txt.
- Internal linking: every page reachable by crawlable `<a href>` links, descriptive anchor text; orphan pages don't rank. Facets/filters that explode URL space → canonical to the base or noindex.

## Bilingual/i18n (hreflang)

`hreflang` annotations must be **reciprocal** (cs page points to en AND en points back), each page lists itself + all alternates + `x-default`. Language versions = separate canonical URLs; never auto-redirect by Accept-Language on the canonical (Googlebot crawls from the US).

## Core Web Vitals

Budgets: **LCP < 2.5s** (preload the hero/LCP image, no lazy-loading it, server response fast), **CLS < 0.1** (width/height on every image/embed, no layout-shifting late banners, `font-display: swap` with fallback metrics), **INP < 200ms** (break up long main-thread tasks, defer non-critical JS). Measure field data (CrUX/Search Console), not just Lighthouse lab runs.

## Verify before "done"

`curl -I` canonical + redirect behavior · content present in raw HTML · titles/descriptions unique across templates · Rich Results test green · sitemap fresh + canonical-only · hreflang reciprocal · CWV within budget on a real page.
