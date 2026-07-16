# Bundles

## JavaScript

How it's bundled: single bundle style for AP's own code, not route or component splitting. The Coverage tab shows a file literally named All.min.ff2ba6899876ee78a1e76c66d6a06a13.gz.js, a combined, hashed, gzipped bundle. The Sources tree confirms this too, apnews.com's own footprint is almost nothing (just the index page and manifest.json), everything else loads from third-party domains.

Is this the right decision: for AP's own code, a single bundle is a defensible, simple choice, but it's completely overshadowed by the real story here: the JS payload is dominated by third-party ad tech and engagement tools, not AP's own code. The Lighthouse treemap shows a 5.7 MiB total JS payload split across dozens of vendors: Google reCAPTCHA (373.1 KiB), Primis video ads (346.9 KiB plus another 337.4 KiB plus 251.6 KiB in separate chunks), Google's IMA video ad SDK (259.0 KiB x2 plus 151.4 KiB), Google Publisher Tag (195.1 KiB), Prebid/PubFig header bidding (193.9 + 192.7 KiB), Viafoura (184.6 KiB just for its main script), GTM (177.6 KiB), Confiant ad-fraud protection (168.1 KiB), a second GA4 property (147.9 KiB), a quiz widget from Riverdrop (138.5 KiB), OneTrust (135.1 KiB), JW Player's ad bidding module (116.5 KiB), Bounce Exchange (116.1 KiB), and Amazon's ad tech (90.8 KiB). AP's own bundle isn't even large enough to place in the top rows of the treemap.

Is there unused JS: yes, confirmed with real Coverage tab numbers. Some specifics:
* All.min.ff2ba6899876ee78a1e76c66d6a06a13.gz.js (AP's own bundle): 455,855 bytes total, 376,668 unused, 82.6 percent
* gstatic.com/recaptcha/releases/.../recaptcha__en.js: 891,388 bytes total, 589,216 unused, 66.1 percent
* ssl.p.jwpcdn.com provider.hlsjs.js: 588,024 bytes total, 417,626 unused, 71 percent
* ssl.p.jwpcdn.com bidding.js: 473,362 bytes total, 367,243 unused, 77.6 percent
* cdn.viafoura.net vf-v2.js: 679,044 bytes total, 334,519 unused, 49.3 percent
* cdn.viafoura.net chunks/da.e386745...js: 145,043 bytes total, 133,605 unused, 92.1 percent
* cdn.onesignal.com OneSignalSDK.page.es6.js: 140,682 bytes total, 100,089 unused, 71.1 percent
* apcdp.apnews.com/script.js (AP's own): 141,197 bytes total, 74,985 unused, 53.1 percent

Source maps: not checked directly against a Lighthouse audit this time (that check was done for the personal project, not repeated here), but no .js.map files appeared anywhere in the Network or Sources tabs during this capture, so nothing suggests they're being shipped for the combined bundle either.

## CSS

How it's bundled: single bundle, same pattern as the JS. The Coverage and Network tabs both show a file named All.min.70c2eda7933b1b601140d901727ba769.gz.css, one combined stylesheet for the whole site, alongside separate smaller stylesheets pulled in by Viafoura's widget.

Is this the right decision: not really, based on what the Coverage tab shows. A single combined CSS bundle only pays off if most pages actually use most of the rules in it, and that's clearly not happening here.

Is there unused CSS: yes, and it's severe.
* All.min...gz.css (the main combined bundle): 801,002 bytes total, 705,570 unused, 88.1 percent
* fonts.googleapis.com Roboto/Merriweather CSS: 23,373 bytes total, 23,373 unused, 100 percent, entirely wasted
* fonts.googleapis.com Poppins CSS: 3,591 bytes total, 3,591 unused, 100 percent, also entirely wasted
* cdn.viafoura.net 186.4146c4e5889380923a03.css: 94,262 bytes total, 91,579 unused, 97.2 percent
* cdn.viafoura.net 0.e386745fa4c1fae7ff6e.css: 79,440 bytes total, 79,345 unused, 99.9 percent
* cdn.viafoura.net 56.f82e03984f6d77c06501.css: 9,699 bytes total, 9,094 unused, 93.8 percent

## Images

Are images shipped in multiple sizes/formats: partially. Product/article images route through a resizing proxy at dims.apnews.com (URLs pass the original assets.apnews.com image through a "90/?url=..." wrapper), and width parameters are used (poster.jpg?width=720). But the format side is inconsistent, several images through that same proxy come back as plain jpeg rather than webp, so modern format delivery isn't applied uniformly.

Is this the right decision: the resizing part is good practice, the inconsistent format delivery (some webp, some plain jpeg through the same pipeline) is not, that's a real gap worth closing.

Is the full resolution file exposed: the proxy pattern (dims.apnews.com/...?url=https://assets.apnews.com/...) suggests the underlying assets.apnews.com host serves the original files directly, and the resizing/format logic is layered on top via the query parameter wrapper rather than baked into assets.apnews.com itself, which is the same general pattern as Corsair's Cloudinary setup, an unwrapped assets.apnews.com URL is likely to return the original.

## Third party resources

Confirmed real, from the Network tab, Sources tree, and a direct console query listing all 45 unique hostnames the page contacts:

* Advertising and video ad tech: Google Publisher Tag/DoubleClick, Prebid and PubFig header bidding, Google's IMA video ad SDK, Primis and Sekindo, Connatix, NTV, Amazon's apstag.js, Confiant (ad-fraud protection), JW Player's own bidding module
* Analytics and tag management: Google Tag Manager, two separate Google Analytics 4 properties, Parse.ly
* Audience/identity/personalization: Viafoura (comments and engagement), Zephr (subscription/paywall), Kameleoon (A/B testing), Permutive, BlueConic, Bounce Exchange, Dianomi
* Consent, accessibility, and notifications: OneTrust, Usablenet (accessibility overlay), OneSignal (push notifications), Google reCAPTCHA
* Interactive content: a quiz widget from Riverdrop
* Infrastructure: Statically and githack.com (third-party CDN/proxy services used for some assets)

How are they loaded: based on the timelines across the captures, the large majority load up front, in the first several seconds, not deferred or lazy. GTM and the ad-tech stack in particular start immediately.

How much is each impacting load: the treemap gives real transfer size per script (see the JavaScript section above), and it's clear the video ad stack (Primis, IMA SDK, GPT, prebid) collectively accounts for more of the payload than any single other category, including AP's own code.

Do any seem unnecessary: hard to call any of these truly unnecessary given this is an ad-supported publisher, advertising and engagement tools are the business model. But having two separate GA4 properties, and running both Prebid/PubFig and JW Player's own separate bidding module at the same time, looks like redundant tooling worth a second look. The 100 percent unused Google Fonts CSS (both the Roboto/Merriweather request and the separate Poppins request) is also a clear, no-tradeoffs waste.

## Bundle related corrective findings

1. The main CSS bundle (All.min...gz.css, 801 KB) is 88.1 percent unused, a single combined stylesheet strategy that isn't paying off. See findings.md finding 12.
2. The video ad tech stack (Primis, Google's IMA SDK, GPT, Prebid/PubFig, JW Player's bidding module) makes up a bigger share of the 5.7 MiB JS payload than AP's own code. See findings.md finding 13.
3. Two separate Google Fonts CSS requests (Roboto/Merriweather and Poppins) are both 100 percent unused. See findings.md finding 14.
4. Viafoura's CSS is close to entirely unused across three separate files (97.2 percent, 99.9 percent, and 93.8 percent unused). See findings.md finding 15.
