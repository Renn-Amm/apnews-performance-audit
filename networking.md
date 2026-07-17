# Networking

Real data pulled from DevTools screenshots on apnews.com (No throttling).

## Hard load (Disable cache checked)

* Total requests: 185
* Total transferred: 8.4 MB
* Total resource size (uncompressed): 16.0 MB
* Compression reduction: (16.0 - 8.4) / 16.0, about 47 percent
* Finish time: 10.43 s, DOMContentLoaded: 1.43 s, Load: 3.24 s

## Soft refresh (Disable cache unchecked)

* Total requests: 167
* Total transferred: 300 KB
* Total resource size: 14.5 MB
* Reduction versus hard load: 8.4 MB down to 300 KB, over 96 percent less data over the wire

Caching within a session works about as well here as it did on the personal project, the drop from a cold load to a warm one is dramatic.

## Breakdown by type (soft refresh, filtered one at a time)

* JS: 66 requests, 74.8 KB transferred, 5,842 KB resource size
* CSS: 7 requests, 0.0 KB transferred (fully cached), 1,102 KB resource size
* Images: 64 requests, 503 KB transferred, 6,996 KB resource size, a real mix of formats, some webp through an image resizing proxy (assets.apnews.com), but plenty of plain jpeg too, not fully modernized

## Compression and caching, checked on individual files

* apcdp.apnews.com/script.js (AP's own first party analytics/data-layer script): Cache-Control public, no-cache="Set-Cookie", max-age=600, that's only 10 minutes, and it's gzip compressed.
* cdn.onesignal.com's SDK: Cache-Control public, max-age=259200 (3 days), Cf-Cache-Status HIT (served from Cloudflare's edge), Content-Encoding br. This third party asset is cached far better than AP's own script.
* cdn.viafoura.net/entry/index.js: Cache-Control public, max-age=600, s-max-age=60, so only 60 seconds at the CDN edge and 10 minutes in browser, both short for a JS library.

## Real console/network evidence found along the way

* Google Tag Manager, container GTM-KT7RHVG
* OneSignal (push notifications)
* Google reCAPTCHA (recaptcha_en.js and api.js)
* JW Player (video), with its own bidding.js module confirming programmatic video ad auctions
* Viafoura (comments/engagement platform, many subdomains: api, i, livecomments, notifications, tyrion)
* Kameleoon (A/B testing/personalization), several requests blocked in this capture by the browser's ad blocker extension
* Zephr (subscriber/paywall identity platform)
* A quiz widget from client.riverdrop.com
* Two blocked requests worth noting: a POST to na-data.kameleoon.io and a POST to i.viafoura.co, both net::ERR_BLOCKED_BY_CLIENT, likely the ad blocker extension active during capture rather than a real server issue
* A preloaded image (dims.apnews.com crop) flagged by Chrome as unused within a few seconds of load, same pattern as the personal project's font preload issue

## Networking findings

See findings.md, corrective findings added from this data cover the short cache duration on AP's own script, and the still-present preload waste.
