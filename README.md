# Course Project Performance Audit Report — AP News

## Website

**Name:** AP News (The Associated Press)
**URL:** https://apnews.com/

It also covers a wide mix of content and features:
- Static content — long-form articles, photo captions, bylines
- Dynamic content — homepage top-story carousel, topic/hub pages that update as new stories publish
- Images — heavy use of AP wire photography throughout, often large hero images per article
- Video — embedded AP video clips on many articles
- Data fetching — live search, topic hub feeds, an interactive weekly rankings page (AP Top 25 poll)
- Forms — newsletter sign-up
- Third parties/ads — ad networks, trackers, and analytics scripts loaded on nearly every page

## PageSpeed Insights baseline (homepage, mobile)

Scanned `https://apnews.com/` on July 1, 2026.

**Scores**
- Performance: 28/100 (red)
- Accessibility: 75/100
- Best Practices: 77/100
- SEO: 85/100

**Core Web Vitals (field data, 28-day real users)**
- LCP: 2.9s — needs improvement
- INP: 177ms — good
- CLS: 0.06 — good
- FCP: 2.0s — needs improvement
- TTFB: 0.2s — good
- Overall assessment: Failed

**Lab data (simulated mobile, slow 4G)**
- FCP: 6.5s
- LCP: 20.3s
- Total Blocking Time: 1,900ms
- Speed Index: 15.6s
- CLS: 0.001

**Biggest flagged issues**
- Render-blocking requests — ~650ms possible savings
- Inefficient cache lifetimes — ~1,354 KiB possible savings
- Unoptimized image delivery — ~3,162 KiB possible savings
- Unused JavaScript — ~2,184 KiB possible savings
- Unused CSS — ~137 KiB possible savings
- Total page payload: 10,510 KiB
- More than 4 preconnect connections found (warning)
- Accessibility gaps: missing alt text on images, buttons/links without accessible names, insufficient color contrast, touch targets too small

## Pages the audit will focus on

1. **Homepage** — https://apnews.com/
   Already the page with the failing PSI score above, so it anchors the audit. Loads a constantly-updating top-story carousel, ad slots, and a large number of images above the fold.

2. **U.S. Supreme Court hub** — https://apnews.com/hub/us-supreme-court
   A topic/tag page that pulls in a live feed of tagged articles, good for checking how AP's hub template handles dynamic list rendering and image-heavy thumbnails at scale.

3. **Climate and Environment hub** — https://apnews.com/hub/climate-and-environment
   Same hub template as above but on a topic that leans heavily on embedded photo essays and video, useful for comparing media weight across hub pages.

4. **AP Top 25 College Football Poll hub** — https://apnews.com/hub/ap-top-25-college-football-poll
   Combines an interactive, regularly-updated rankings table with article content, a good page for testing data-fetching and interactive component performance rather than just static article rendering.

5. **Newsletter sign-up** — https://apnews.com/newsletters
   The closest thing to an account/form flow on the main site (email capture), worth checking for form handling and any related script bloat.

6. **Search results page** — https://apnews.com/search?q=trump
   Query-based page that fetches results live, useful for testing how a dynamic, user-triggered request performs compared to pre-rendered pages.

*Note: apnews.com blocks direct crawling/fetching from a lot of automated tools, so I confirmed pages 1–4 are real, live URLs, but pages 5 and 6 are built from AP's known URL patterns and I wasn't able to double-check them are still live — worth clicking through yourself before submitting.*
