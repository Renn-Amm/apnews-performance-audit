# Findings

Findings below are split into corrective (something is actually wrong) and good (something is already working well). Findings 1 through 7 come from the original PageSpeed Insights scan. Findings 8 through 16 come from real Chrome DevTools Network, Coverage, and treemap data, plus a Desktop PSI scan, see networking.md and bundles.md for the full detail behind each one. Each finding is meant to be independently observable, not a restatement of the same root cause under a different metric name.

## Corrective

1. Visible page load is delayed to an extreme degree, in the lab test the main content doesn't finish painting until past 20 seconds, which on a real slow connection reads as a broken or blank page.
   * Metric(s): LCP
   * Cause: improve image delivery flags about 3,162 KiB of possible savings on images, and render blocking requests plus heavy main thread JS work push back when the browser even starts loading the LCP image.
   * Solution: compress and serve the hero image in a modern format at the right size, add fetchpriority="high" and preload it, and clear render blocking resources out of its path.

2. Right after the page shows up, taps on buttons and links don't respond for a long stretch, the page looks ready but isn't.
   * Metric(s): Total Blocking Time (1,900 ms mobile, 2,570 ms desktop)
   * Cause: JS execution time is 5.8 s and main thread work totals 12.1 s on mobile, and this is now confirmed to be worse, not better, on desktop. The treemap shows the video ad stack and Viafoura alone account for well over 1 MiB combined.
   * Solution: defer or lazy load non critical third party scripts so they run after the page is interactive, and remove or trim the unused JS.

3. Assets don't get meaningfully faster on a second visit the way they should.
   * Metric(s): FCP, LCP, repeat visit load time
   * Cause: use efficient cache lifetimes flags about 1,354 KiB of assets without proper long term cache headers on mobile, and 2,545 KiB on desktop, plus about 650 ms lost to render blocking requests on first paint.
   * Solution: set long Cache Control and max age headers with versioned filenames on static assets, and inline critical CSS while deferring the rest.

4. Screen reader and keyboard users can't tell what several images or buttons are, and some text is hard to read.
   * Metric(s): Accessibility score (75 mobile, 76 desktop)
   * Cause: flagged issues include images missing alt attributes, buttons and links with no accessible name, and text/background color combinations that don't meet contrast requirements. Confirmed consistent across both mobile and desktop scans.
   * Solution: add descriptive alt text to every image, aria labels to icon only buttons and links, and adjust the color palette to meet WCAG AA contrast.

5. Too many separate origins are being connected to up front, which slows down connection setup for everything else on the page.
   * Metric(s): overall Performance score, contributes to LCP and TBT
   * Cause: more than 4 preconnect connections were flagged as a warning on both mobile and desktop scans. A direct console query of every hostname the page contacts found 45 distinct third-party domains.
   * Solution: audit the preconnect list and only keep it for the two or three origins that are actually critical to first paint, drop the rest.

6. Some JavaScript being shipped is unnecessary for the browsers that make up the vast majority of AP's traffic.
   * Metric(s): overall page weight, contributes to Speed Index
   * Cause: legacy JavaScript is flagged on both scans (23 KiB mobile, 35 KiB desktop), meaning the build is sending older syntax, polyfills, or transforms aimed at very old browsers.
   * Solution: set a modern browserslist target in the build config so the bundler stops outputting legacy fallback code for evergreen browsers.

7. The same JavaScript logic appears to be shipped more than once.
   * Metric(s): overall page weight
   * Cause: duplicated JavaScript is flagged on both scans, which usually happens when multiple bundles each include their own copy of the same shared library or utility instead of sharing one copy.
   * Solution: de duplicate shared dependencies through the bundler's shared chunk or vendor splitting configuration.

8. The page does not use HTTPS consistently. Confirmed directly by Lighthouse's Best Practices audit on the desktop scan: "Does not use HTTPS, 1 insecure request found."
   * Metric(s): Best Practices score (35, the lowest category score in the whole project)
   * Cause: at least one resource on the page is being requested over plain HTTP instead of HTTPS.
   * Solution: find the specific insecure request (Lighthouse's audit detail will name it) and update it to HTTPS, or remove it if it's from a vendor that doesn't support secure delivery.

9. The page sets an unusually high number of third-party cookies. Confirmed directly: "Uses third-party cookies, 49 cookies found."
   * Metric(s): Best Practices score, and increasingly relevant to browser privacy restrictions that block third-party cookies outright
   * Cause: the large number of ad tech, analytics, and engagement vendors (see bundles.md) each set their own tracking cookies.
   * Solution: audit which of the 45 third-party domains actually need cookie-based tracking versus a cookieless alternative, and reduce vendor count where possible.

10. Two entire CSS files load and are never used at all. Confirmed via the Coverage tab: both the Google Fonts request for Roboto/Merriweather (23,373 bytes) and a separate request for Poppins (3,591 bytes) come back at 100 percent unused.
    * Metric(s): overall page weight, render blocking time
    * Cause: font CSS is being requested for font families that either aren't actually applied on this page or are already satisfied by a different font loading method.
    * Solution: remove the unused Google Fonts requests, or confirm which font family is actually rendered and drop the others.

11. The site's main combined CSS bundle is almost entirely unused on this page. Confirmed via Coverage tab: All.min...gz.css is 801,002 bytes total with 705,570 unused, 88.1 percent.
    * Metric(s): overall page weight, render blocking time, Reduce unused CSS (146 KiB flagged directly by PSI)
    * Cause: a single, site-wide combined CSS bundle strategy, most of the rules in it apply to pages or components other than this one.
    * Solution: split the bundle so each page or template only loads the CSS it actually needs, rather than one file for the entire site.

12. The video advertising stack is a bigger share of the total JS payload than AP's own code. Confirmed via the Lighthouse treemap: Primis, Google's IMA SDK, Google Publisher Tag, and Prebid/PubFig header bidding combined are well over 1.5 MiB of the 5.7 MiB total JS, while AP's own combined bundle is under 500 KB.
    * Metric(s): overall page weight, TBT
    * Cause: this page loads a full video-advertising pipeline (ad request, header bidding, video ad SDK) even on an article page where the video may not be the primary content.
    * Solution: confirm whether the video ad stack should load immediately on every article or only once a user actually interacts with the video player.

13. Viafoura's CSS is close to entirely unused. Confirmed via Coverage tab: three separate Viafoura stylesheet files came back at 97.2 percent, 99.9 percent, and 93.8 percent unused.
    * Metric(s): overall page weight
    * Cause: Viafoura (the comments/engagement widget) is loading its full stylesheet set regardless of which of its features are actually visible on this page.
    * Solution: ask Viafoura's integration docs whether a scoped or on-demand CSS loading option exists, instead of the full set on every page.

14. AP's own first-party code is a tiny fraction of what loads on this page. Confirmed via the Sources tree: apnews.com's own footprint is just the index document and a manifest.json, every other script and stylesheet comes from a third-party domain.
    * Metric(s): overall page weight, TBT, general architecture
    * Cause: the page is built almost entirely out of embedded third-party widgets (comments, video, ads, personalization, identity) rather than first-party components.
    * Solution: not a quick fix, but worth tracking as a standing constraint, most performance wins on this page will come from managing which third parties load and when, not from optimizing AP's own code, because there isn't much of it to optimize.

## Good

15. Interactivity itself is actually fine. INP field data comes back at 177 ms on mobile and 164 ms on desktop, both good ratings.
    * Metric(s): INP
    * Why it's good: once a real user actually taps or clicks something, the response is fast on both device types, the problem is everything that happens before that point, not the responsiveness itself.

16. Layout stays visually stable while the page loads. CLS came back at 0.06 in the mobile field, 0.001 in the mobile lab, 0.02 in the desktop field, and 0 in the desktop lab, all in the good range.
    * Metric(s): CLS
    * Why it's good: with 45 different third-party scripts and widgets loading onto this page, layout shift would be an easy thing to get wrong, and it isn't happening here.

Mobile-specific findings are in mobile.md. Bundle-specific findings are detailed further in bundles.md.
