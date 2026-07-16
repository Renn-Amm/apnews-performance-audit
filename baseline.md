# Baseline

## Core Web Vitals (mobile)

* Largest Contentful Paint (LCP): 2.9 s
* Interaction to Next Paint (INP): 177 ms
* Cumulative Layout Shift (CLS): 0.06

## PageSpeed Insights at / (mobile)

* Performance: 28
* Accessibility: 75
* Best Practices: 77
* SEO: 85
* First Contentful Paint: 6.5 s
* Largest Contentful Paint: 20.3 s
* Total Blocking Time: 1,900 ms
* Cumulative Layout Shift: 0.001
* Speed Index: 15.6 s

Mobile scan was run under Lighthouse's Moto G Power emulation with slow 4G throttling, which matches the throttling used in class.

## PageSpeed Insights at /, Desktop

* Performance: 31
* Accessibility: 76
* Best Practices: 35
* SEO: 85
* Agentic Browsing: 1/2
* First Contentful Paint: 1.4 s
* Largest Contentful Paint: 13.0 s
* Total Blocking Time: 2,570 ms
* Cumulative Layout Shift: 0
* Speed Index: 9.0 s
* Field data (real users): Core Web Vitals Assessment Failed, LCP 3.6 s, INP 164 ms, CLS 0.02, FCP 2.3 s, TTFB 0.2 s

Unlike the personal project, desktop does not pass here, real desktop users are also having a degraded experience. See mobile.md for the full comparison.

## Networking

Real data in place, see networking.md. Short version: hard load is 185 requests and 8.4 MB transferred out of 16.0 MB uncompressed (about 47.5 percent compression), a soft refresh drops that to 300 KB thanks to caching (about 96.4 percent less), and a direct console query found 45 distinct third-party hostnames contacted by the page.

## Bundles

Real Coverage tab and Lighthouse treemap data in place, see bundles.md. Headline numbers: both JS and CSS use a single combined bundle strategy (files literally named All.min...js and All.min...css), the CSS bundle is 88.1 percent unused, and the 5.7 MiB JS payload is dominated by video advertising tech (Primis, Google's IMA SDK, Google Publisher Tag, Prebid/PubFig), not AP's own code. AP's own first-party footprint (outside that one combined bundle) is essentially just the index document and a manifest.json.

## Known bugs, not just performance issues

* "Does not use HTTPS, 1 insecure request found" (Lighthouse Best Practices, desktop scan)
* "Uses third-party cookies, 49 cookies found" (Lighthouse Best Practices, desktop scan)
* "Document doesn't have a valid hreflang" (Lighthouse SEO, desktop scan)
