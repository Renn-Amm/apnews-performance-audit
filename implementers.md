# AP News Performance Audit — For the person fixing this

Site: apnews.com
Tooling used: Google PageSpeed Insights (Lighthouse 13.4.0), Chrome DevTools Network, Coverage, Performance, and Layers panels.
Measurements taken between July 1 and July 16, 2026. This is a live news site with constantly changing homepage content and a heavy load of third-party advertising, so exact numbers will shift day to day. The mechanisms and file names below are what to check for, not a promise the millisecond counts will match exactly on a re-test.

Two throttling profiles were used throughout:
* Mobile: PageSpeed Insights default, emulated Moto G Power, slow 4G network.
* Desktop: PageSpeed Insights default, custom desktop throttling, no device emulation.

One thing worth knowing before anything else: on this site, unlike a typical single-page app, desktop is not meaningfully better than mobile. Desktop Total Blocking Time (2,570ms) is actually worse than mobile (1,900ms). That's a real signal that the core problem here is the volume of third-party code running on the page, not a mobile-specific constraint.

## How to reproduce the baseline

1. Go to pagespeed.web.dev, enter `https://apnews.com/`, run both the Mobile and Desktop tabs.
2. Open the same URL in Chrome, DevTools, Network tab, check "Disable cache," hard reload, record the summary line.
3. Uncheck "Disable cache," reload again, record the same numbers for the warm-load comparison.
4. Open the Coverage tab, record, reload, for the unused JS/CSS numbers.
5. Open the Lighthouse report's "View Treemap" link for the JS payload breakdown by script.
6. Record a Performance trace across a page load, a scroll through an article feed, and a click navigating into a section page.
7. Open the Layers panel for the compositor layer findings.

## Loading (LCP)

**Mechanism:** Lighthouse flags roughly 2,802 KiB of possible savings on image delivery, plus render-blocking requests, plus heavy main-thread JS work, all pushing back when the browser even starts loading the largest visible element. This is not one script, it's the combined weight of everything described under Third-party below competing for the same critical path.

**Reproduce:** run the Mobile PageSpeed scan, check the "Improve image delivery" and "Render-blocking requests" insights.

**Fix:** compress and correctly size the hero/lead image for each article template, add `fetchpriority="high"` and a preload tag for it, and reduce what's render-blocking ahead of it (see Critical CSS below, that's a direct contributor).

**Effort:** structural. Unlike a typical single-app fix, the render-blocking cost here comes from a large stack of independently-managed third-party scripts (see Third-party section), so fixing LCP fully requires changes across multiple vendor integrations, not one template.

**Verify:** re-run the Mobile PageSpeed scan, LCP should move down from ~20 seconds meaningfully, though given how much of the delay is third-party, full resolution likely requires the third-party changes below too, not just image/template work alone.

## Interactivity (TBT)

**Mechanism:** the Lighthouse treemap shows a 5.7 MiB total JS payload, more than double the personal project's 2.8 MiB, and it's overwhelmingly video-advertising and header-bidding infrastructure: Primis (a video ad platform, ~936 KiB across three files), Google's IMA video ad SDK (loaded twice, 518 KiB combined for the same library), Google Publisher Tag (195.1 KiB), Prebid header bidding (193.9 KiB plus a separate 192.7 KiB engine file), and Google reCAPTCHA (373.1 KiB). AP's own first-party code (`apcdp.apnews.com/script.js`) is about 141 KB, a small fraction of the total.

**Reproduce:** open the Lighthouse treemap, sort by size. Cross-check against the Coverage tab, the same files show high unused percentages (reCAPTCHA is 66.1 percent unused, the JW Player bidding module is 77.6 percent unused).

**Fix:** this is not a first-party code fix, AP's own bundle is already small. The fix is a review of the ad-tech and engagement vendor stack: specifically, Primis and Connatix both appear to be running as separate, competing video ad platforms on the same page, and Google's ad stack, Amazon's, and Prebid's header bidding layer all appear to be running simultaneously. Consolidating overlapping vendors would cut a meaningful share of this weight directly.

**Effort:** structural. This requires a decision from whoever manages ad operations and vendor contracts, not an engineering change alone.

**Verify:** re-run the treemap after any vendor consolidation, total JS payload should drop, and Total Blocking Time (currently 1,900ms mobile, 2,570ms desktop) should drop correspondingly.

## Caching

**Mechanism:** checked response headers directly on three separate scripts.
* AP's own first-party script (`apcdp.apnews.com/script.js`): `Cache-Control: public, no-cache="Set-Cookie", max-age=600`, only 10 minutes.
* OneSignal's SDK (a third party): `Cache-Control: public, max-age=259200`, 3 days, served from a CDN edge.
* Viafoura's entry script (a third party): `Cache-Control: public, max-age=600, s-max-age=60`, only 60 seconds at the CDN edge.

AP's own script and one of the third-party scripts are both cached for a shorter time than is typical for versioned static JS.

**Reproduce:** Network tab, click each of these requests, check the Cache-Control response header.

**Fix:** for AP's own script, if it's safe to cache longer (check whether it's genuinely dynamic per request or just conservatively configured), extend the max-age and use a versioned filename if it isn't already. For Viafoura, this is a third-party-controlled header, flag it to Viafoura's account team, there may be a longer-cache option in their integration docs.

**Effort:** local for AP's own script if it's just a config change. Not directly fixable in-house for the Viafoura header, that requires a vendor conversation.

**Verify:** re-check the same headers after any change.

## CSS

**Mechanism:** the site loads a single combined, minified stylesheet (`All.min.[hash].gz.css`) plus separate Google Fonts requests and a date-stamped custom file. The Coverage tab shows the combined stylesheet is 801,002 bytes with 705,570 unused, 88.1 percent. Two Google Fonts requests (Roboto/Merriweather at 23,373 bytes, Poppins at 3,591 bytes) are both 100 percent unused, meaning they load in full and apply to nothing on this page. Viafoura's CSS (comments widget) is similarly close to fully unused across its files (97.2 percent, 99.9 percent, 93.8 percent depending on the file).

**Reproduce:** Coverage tab, filter by `.css`, sort by Unused Bytes.

**Fix:** for the combined stylesheet, this is a site-wide bundling decision, splitting it by page template would need to happen at the CMS/build level, this is the same category of fix as the personal project's CSS finding but larger in scope given how much bigger the unused percentage is. For the two Google Fonts requests, confirm which font family is actually rendered on this page and remove the requests for the ones that aren't. For Viafoura, same as the caching note above, this is a vendor-controlled asset.

**Effort:** structural for the combined stylesheet split. Local for removing the two dead Google Fonts requests once confirmed unused. Not directly fixable in-house for Viafoura's CSS.

**Verify:** Coverage tab after changes, the combined stylesheet's unused percentage should drop, and the two Google Fonts requests should either disappear or show near-zero unused bytes.

## Critical CSS

**Mechanism:** no critical-path CSS is inlined in the document head, the combined stylesheet loads as a single external request with nothing extracted for above-the-fold content.

**Reproduce:** View Source, check the `<head>` for an inlined `<style>` block, there isn't one.

**Fix:** add a critical CSS extraction step to whatever builds the page templates, so above-the-fold styles are inlined and the rest loads async.

**Effort:** structural, this is a build/templating pipeline change.

**Verify:** View Source again, confirm an inlined style block exists, re-run the render-blocking-requests audit to confirm estimated savings drop.

## Images

**Mechanism:** there's a real image-resizing proxy in front of `assets.apnews.com` that returns WebP for some images, but the Network tab also shows plain JPEG still being served for others, including the main video poster image. It's a partial rollout, not a consistent pipeline.

**Reproduce:** filter Network by Img, check the Type column, you'll see a mix of `webp` and `jpeg` for what should be equivalent content.

**Fix:** find whichever templates or components are still requesting the unproxied/JPEG version and route them through the same resizing proxy already used elsewhere on the page.

**Effort:** local-to-moderate, depends on how many separate templates reference images directly versus through the proxy.

**Verify:** re-check the Img-filtered Network tab, the JPEG requests for content that has a WebP equivalent should disappear.

Whether a full-resolution original is exposed by stripping the proxy's URL parameters wasn't confirmed on this pass, worth a direct check before assuming either way.

## Third-party resources

This is the center of gravity for nearly every finding above, so it gets its own detailed breakdown. Full list confirmed via Network tab, Coverage tab, and a console query of every unique hostname the page contacted:

* **Google:** reCAPTCHA, Tag Manager (GTM-KT7RHVG), two separate GA4 properties plus a third distinct one found in the treemap, Fonts, IMA video ad SDK, Publisher Tag/DoubleClick, Amazon's ad system script
* **Video ads:** Primis, Connatix, JW Player (video player) plus its entitlements service and bidding module
* **Header bidding:** a.pub.network (Prebid, PubFig)
* **Ad quality monitoring:** Confiant
* **Audience/personalization:** Viafoura (comments), Kameleoon (A/B testing), Permutive, BlueConic, Parse.ly, Sailthru
* **Identity/subscription:** Zephr
* **Consent:** OneTrust
* **Notifications:** OneSignal
* **Interactive widgets:** a quiz widget from client.riverdrop.com, a content recommendation widget from Dianomi
* **Accessibility overlay:** a40.usablenet.com (UsableNet)
* Two domains worth a second look: `rawcdn.githack.com` (a raw GitHub content CDN, an unusual choice for production ad/analytics infrastructure) and a couple of similarly unbranded-looking domains

**How they load:** based on the recorded timeline, the large majority fire up front rather than being deferred, GTM in particular pulls in a large share of the rest as soon as it loads.

**Overlap identified:** Primis and Connatix both provide video ad delivery on the same page. Google's own ad stack, Amazon's ad marketplace script, and Prebid's header bidding layer all appear to run at once, these three are typically alternative approaches to the same job (selling ad inventory), not complementary ones.

**Fix:** this needs a review by whoever manages ad operations and vendor relationships: confirm whether both video ad platforms are still needed, and whether all three ad-marketplace integrations are actually required simultaneously or whether one has been superseded by another without the old one being removed.

**Effort:** structural, this is a vendor management decision, not a code change any one engineer can make unilaterally.

**Verify:** after any vendor changes, re-run the treemap, total JS payload should drop substantially given how much of it is currently duplicated ad infrastructure.

## Known bugs, not just performance issues

**Not using HTTPS consistently.** Lighthouse's desktop Best Practices audit states directly: "Does not use HTTPS, 1 insecure request found." This is also why Best Practices scored only 35, the lowest category score anywhere in this audit.
* **Reproduce:** the Lighthouse audit detail will name the specific insecure request.
* **Fix:** update that request to HTTPS, or remove it if it comes from a vendor that doesn't support secure delivery.
* **Effort:** local once the specific request is identified.
* **Verify:** re-run the audit, it should clear.

**49 third-party cookies set on a single page load.** Confirmed directly by the same audit category.
* **Fix:** audit which of the third-party vendors listed above actually require cookie-based tracking versus a cookieless alternative, and reduce vendor count where possible (this overlaps directly with the ad-stack consolidation above).
* **Effort:** structural, tied to the same vendor review as the ad-stack overlap.

## Mobile-specific

**Mechanism:** field LCP is 2.9 seconds, mobile lab LCP under slow 4G is 20.3 seconds. As with the personal project, this gap reflects the difference between typical real-world devices/connections and a deliberately worst-case test condition. What's different here: desktop lab LCP (13.0 seconds) is also bad, so this isn't a mobile-only problem the way it might first appear, mobile throttling makes an already-heavy page significantly worse, it doesn't create the problem from nothing.

**Reproduce:** compare Mobile and Desktop PageSpeed lab data side by side, both show a failing LCP, at different severities.

**Fix:** everything under Interactivity and Third-party above. There's no mobile-only fix separate from reducing the underlying third-party weight.

**Effort:** same as the underlying fixes.

**Verify:** re-run the Mobile PageSpeed lab scan after third-party changes land, and compare against the Desktop scan to see whether the gap between them narrows (it should, if the fix is actually about shared third-party weight and not something mobile-specific).

Speed Index shows the same amplification pattern, 15.6 seconds mobile lab versus 9.0 seconds desktop lab, a larger relative gap than LCP shows, meaning visual completeness is hit even harder than the single-element LCP metric by the mobile constraints.

## Page load, scroll, and interaction jank

**Mechanism:** a Performance recording of page load shows two early frames at 133.2ms and 83.2ms, both past the 16.7ms budget. In the same recording window, the Summary tab shows Rendering (1,172ms) as the single largest main-thread category, ahead of Scripting (1,092ms), over a 5.11-second range. A specific 335.82ms long task in this window broke down to 195ms Rendering versus 117ms Scripting, meaning the dominant cost on load here is layout/rendering work, not script execution. This is a different root cause than the personal project, where scripting time dominated. The 1st/3rd-party breakdown for the same window also shows 2,445.3ms of unattributed main-thread time, more than any single named script, typically a sign of layout/style recalculation that isn't cleanly attributed to one source.

Scrolling is worse: a single recording caught five separate frames at 349.7ms, 532.8ms, 532.8ms, 366.3ms, and 233.1ms, all during ordinary scrolling through the article feed, up to 32 times the frame budget.

Section navigation (clicking a nav link) is worse still: frames at 283.1ms, 566.2ms, 732.8ms, 549.5ms, and 749.4ms during a single click-through, the longest 45 times the frame budget. One long task in this window (113.37ms) was scripting-dominated (109ms of 113ms), meaning page-transition JS is a real contributor here specifically, distinct from the rendering-dominated cost seen on load and scroll.

**Reproduce:** Performance tab, record across a page load, a scroll, and a nav click separately. Check the Frames track for long bars, click into Main underneath at that timestamp, check Summary and Bottom-Up for what's consuming the time.

**Fix:** for the load-time rendering cost, this points toward DOM complexity or expensive CSS/layout recalculation rather than JS specifically, profile the unattributed time with the Bottom-Up view to find what's triggering it. For scroll and navigation jank, check what's attached to scroll/navigation events across the page, given the volume of third-party widgets (each potentially listening for scroll or route changes), this is a reasonable place to start isolating the cause.

**Effort:** structural. This spans multiple root causes (rendering cost, third-party listeners, page-transition script), not a single fix.

**Verify:** re-record the same three scenarios, frame durations should move toward the 16.7ms budget.

## Compositor layers

**Mechanism:** three things found in the Layers panel and a Rendering-tab recording:
1. Hovering a single top nav item to open its dropdown menu triggers a paint flash across almost the entire viewport, not just the dropdown itself, confirmed visually with Paint Flashing enabled.
2. JW Player creates a separate compositor layer pair (a video layer plus a preview layer) for each video embed, at least three separate pairs were found in one Layers session on a single article page, including videos not currently in view. There's no sign these are deferred until a video actually scrolls into view.
3. The page's scrollable root shows a "Slow scroll regions" / non-fast-scrollable area, the same pattern found on the personal project, meaning a touch or wheel handler attached broadly enough that the browser can't confidently fast-path scroll input.

A smaller, lower-priority note: UsableNet's accessibility overlay widget creates its own small set of compositor layers for a fixed-position button, not wrong, just worth knowing it adds to the layer count on every page.

**Reproduce:** Rendering tab, enable Paint Flashing, hover the nav. Layers panel, look for `video.jw-video` / `.jw-preview` pairs, check how many exist relative to how many videos are actually visible.

**Fix:** for the nav paint flash, scope whatever triggers the dropdown's repaint to the dropdown element itself rather than a broader ancestor. For JW Player, check its lazy-load configuration, most video players support deferring player initialization until the embed is near the viewport, this should cut the layer count and associated JS setup cost for videos a user hasn't scrolled to. For the scroll handler, same as the personal project, find what's attached broadly and scope it or mark it passive.

**Effort:** local for the nav paint flash. Moderate for JW Player lazy-loading, likely a configuration change rather than custom code, but needs testing across article templates. Moderate for the scroll handler.

**Verify:** re-check Paint Flashing on nav hover, it should be scoped to the dropdown only. Re-check the Layers panel on a page with multiple videos, only in-view or near-view videos should have active layer pairs.

## Rendering strategy

**Mechanism:** confirmed by fetching an article URL with JavaScript disabled, full readable article text still comes back, this is server-rendered content (consistent with AP's known Arc XP publishing platform), not a client-hydrated shell. First Contentful Paint is reasonably fast as a result (1.4s desktop, 2.0s mobile). But LCP lags far behind (13.0s desktop, 20.3s mobile) because the actual largest element, typically a hero image or video player, waits behind the third-party ad/video infrastructure documented above.

**Fix:** no change needed to the rendering strategy itself, server rendering the article content is the right call and is working. The gap between FCP and LCP is entirely explained by what's documented under Interactivity and Third-party, fixing those closes this gap, there's no separate rendering-strategy fix to make.

**Effort:** tracked through the related findings above, not a separate item.

Separately, video embeds initializing eagerly (see Compositor layers above) is itself a rendering-strategy-adjacent decision worth revisiting, whether every video on a page needs its player and ad-bidding module set up immediately versus only once scrolled near.

## What's already fine, don't spend time here

* Interactivity itself, once something loads, is solid. INP field data is 177ms mobile, 164ms desktop, both good ratings. The problem across this whole audit is getting the page ready, not how it responds once it is.
* Layout stability is good across every measurement taken (CLS between 0 and 0.06 depending on device and field/lab). With 45 distinct third-party domains loading onto this page, that's a real accomplishment, layout shift is an easy thing to get wrong with that many independent widgets.
* Caching within a single browsing session works well, a soft refresh drops transferred data from 8.4 MB to 300 KB, over 96 percent less. The caching problem documented above (AP's own script, Viafoura's asset) is about cache duration for specific files, not whether caching works at all.

## What wasn't assessed

* Whether a full-resolution image is exposed by stripping the resizing proxy's URL parameters, flagged but not directly tested.
* Whether apnews.com uses a service worker or has offline behavior, this wasn't part of the covered scope for this project and no evidence either way was collected.
* A full security header review beyond the specific HTTPS finding above, Best Practices scored 35 overall and likely has more findings than the two documented here, a dedicated security pass would surface more.
* Only the homepage, a couple of section/hub pages, and one article page were tested in depth. Given how much of what's documented here is third-party and potentially varies by page or article type (video-heavy versus text-only, for example), a broader page sample would likely surface additional detail.
