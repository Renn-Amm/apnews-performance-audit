# AP News Performance Review — Summary for Decision Makers

## The short version

apnews.com scores 28 out of 100 for mobile performance on Google's standard testing tool, and 31 out of 100 on desktop. That second number is the one worth paying attention to, on most sites, mobile is the harder problem and desktop is fine. Here, both are bad, and desktop is actually slightly worse on one key measure. That tells us the core problem isn't about phones, it's about the sheer amount of advertising and tracking software running on every single page, on every device.

There is also a direct security finding: the site does not use a secure connection consistently, at least one request loads over plain, unencrypted HTTP. And nearly 50 separate third-party cookies get set on a single page view.

None of the biggest problems here are AP's own article-writing platform. The page's actual content system is fast and working as intended. The weight comes almost entirely from advertising and audience-tracking tools layered on over time.

## What this costs the business

A page that takes many seconds to become usable is a page readers abandon, particularly for a news outlet where readers often arrive from a single link or search result and have no loyalty to wait around. The problems documented here show up on every device type, which is a wider exposure than a typical "our mobile site is slow" finding.

There's also a direct, measurable cost buried in the technical detail: at least two separate video-advertising platforms and three separate ad-marketplace integrations appear to be running on the same pages at the same time. These are typically alternatives to each other, not complementary. That pattern usually means either duplicate ad spend, or an older vendor that was never fully removed when a newer one was brought in, either way, it's worth a direct look from whoever manages ad revenue and vendor contracts, because there is a real chance this is costing money without adding value.

The security finding (the unencrypted request) and the cookie count are both the kind of thing that show up in browser warnings to visitors and in any outside security or privacy review of the site.

## How you can check this yourself

1. Go to pagespeed.web.dev, paste in `https://apnews.com/`, click Analyze, check both the Mobile and Desktop tabs. You'll see the same 28 and 31 scores we did.
2. In the same report, scroll to the "Best Practices" score, it will show the HTTPS finding and the cookie count directly, in plain language, no technical background needed.
3. Open the site in Chrome, open the three-dot menu, More Tools, Developer Tools, click the Network tab, reload the page, and look at how many separate outside domains show up in the request list. You don't need to understand what each one does to see that the list is long.

## What's already working

This is not a site with a broken foundation, and it's worth being clear about that before anything else.

* Once a page finishes loading, it responds quickly to taps and clicks, on both mobile and desktop. The problem here is entirely about getting the page ready, not about how it behaves afterward.
* The page doesn't jump around while loading, which is a common, visible problem on ad-heavy sites and a real risk given how many separate tools are running here. It isn't happening.
* The article content itself loads directly and quickly, readers get real text almost immediately. The delay comes from everything layered on top of that content, not the content system itself.

## What it costs to fix, honestly

**Cheap and fast:**
* The unencrypted request. Once identified, this is typically a small fix.
* Two entirely unused font files that load on every single page for no visible benefit. Simple to remove once confirmed.

**Requires a decision from whoever manages advertising and vendor relationships, not engineering alone, and is the single biggest opportunity on this list:**
* Two competing video-advertising platforms and three overlapping ad-marketplace tools appear to be running simultaneously. Reviewing and consolidating this is likely to produce the largest single improvement in this entire report, and it is fundamentally a business and vendor-management decision, not something engineering can quietly resolve alone.
* The high cookie count and broad third-party footprint both trace back to the same review: which vendors are actually still needed.

**Moderate engineering work:**
* The page's shared stylesheet loads almost entirely unused content on every page, splitting it up so each page only loads what it needs is a real but contained project.
* Video players on article pages appear to load fully even for videos a reader hasn't scrolled to yet, deferring that until a reader actually reaches the video is a reasonable, moderate-effort fix.

## Priority order

1. Fix the unencrypted (non-HTTPS) request. This is a direct, easily explained finding and should not wait on anything else.
2. Review the advertising and audience-tracking vendor stack for overlap and redundancy. This is the highest-value item on the list by a wide margin, and it needs a decision-maker's involvement, not just a developer's.
3. Reduce the third-party cookie count as part of the same vendor review above.
4. Address the page's shared, mostly-unused stylesheet and the two dead font files.
5. Defer video player setup on article pages until a reader actually scrolls to the video.

## Bottom line

The core of this site, the part that actually delivers news content to readers, is in good shape. Nearly everything wrong here comes from what's been added around that content over time: advertising platforms, tracking tools, and engagement widgets, several of which appear to duplicate each other. The single highest-value action available is a vendor-level review of that stack, and that decision sits with whoever manages advertising and analytics relationships, not with engineering alone. The security finding should be treated as a separate, immediate fix regardless of the timeline on everything else.
