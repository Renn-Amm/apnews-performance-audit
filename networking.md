# Networking

Real data pulled from DevTools screenshots (Network tab, apnews.com, No throttling).

## Hard load (Disable cache checked)

* Total requests: 185
* Total transferred: 8.4 MB
* Total resource size (uncompressed): 16.0 MB
* Compression reduction: (16.0 MB - 8.4 MB) / 16.0 MB, about 47.5 percent
* Finish time: 10.43 s, DOMContentLoaded: 1.43 s, Load: 3.24 s

## Soft refresh (Disable cache unchecked)

* Total requests: 167
* Total transferred: 300 KB
* Total resource size: 14.5 MB
* Reduction versus hard load: 8.4 MB down to 300 KB, about 96.4 percent less data over the wire
* Load: 2.57 s

## Breakdown by type (soft refresh, filtered one at a time)

* JS: 66 of 188 requests, 74.8 KB transferred, 5,842 KB resource size, about 40 percent of the 14.5 MB total
* CSS: 7 of 192 requests, 566 KB transferred (from cache mostly), 1,102 KB resource size, about 8 percent of total
* Images: 64 of 196 requests, 803 KB transferred, 6,996 KB resource size, about 48 percent of total

Note the compression reduction on a hard load (47.5 percent) is noticeably worse than Corsair's (79 percent) for the same kind of test, worth flagging on its own.

## Compression and caching, checked on individual files

* apcdp.apnews.com/script.js: Cache-Control public, no-cache="Set-Cookie", max-age=600 (10 minutes), gzip encoding, 41,667 bytes
* cdn.onesignal.com OneSignalSDK.page.js: served through Cloudflare (Cf-Cache-Status: HIT), Cache-Control public, max-age=259200 (3 days), br (Brotli) encoding
* cdn.viafoura.net entry/index.js: Cache-Control public, max-age=600, s-max-age=60 (the CDN edge only holds it for 1 minute), br encoding

Cache durations here are short and inconsistent across vendors, some content-owned scripts cache for only 10 minutes, and Viafoura's CDN edge cache is set to just 60 seconds.

## Third party and console evidence found along the way

A console command listing every unique hostname contacted by the page returned 45 distinct hostnames, confirming a very large, mostly third-party footprint. Named vendors identified from the requests and Sources tree:

* Google Tag Manager (GTM-KT7RHVG) and a separate Google Analytics 4 property (G-CW1LS0SXPK)
* Google reCAPTCHA
* Google Publisher Tag / DoubleClick (securepubads.g.doubleclick.net) and Prebid header bidding (a.pub.network, PubFig)
* Google IMA SDK (video ads), Primis and Sekindo (video ad platforms), Connatix and NTV (more video ad platforms)
* Amazon Advertising (apstag.js)
* JW Player, including its own ad-bidding module (bidding.js)
* Viafoura, a commenting/audience engagement platform (multiple subdomains: cdn, api, i, livecomments, notifications, tyrion)
* OneTrust (cookie consent) and OneSignal (push notifications)
* Zephr, a subscription/paywall and identity platform
* Kameleoon, an A/B testing tool
* Parse.ly, a publisher content analytics platform
* Permutive and BlueConic, both audience data platforms
* Usablenet, an accessibility overlay service
* Dianomi, a native content/advertising platform
* A quiz widget from Riverdrop
* Statically and githack.com, both third-party CDN/proxy services

Two real bugs, not just performance issues: the Best Practices audit flags "Does not use HTTPS, 1 insecure request found" (a real mixed-content problem), and "Uses third-party cookies, 49 cookies found."

## Networking findings

See findings.md, corrective findings 8 through 11 are built from this data.
