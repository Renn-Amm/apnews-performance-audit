# Bundles

Real data from the Lighthouse treemap and Coverage tab on apnews.com.

## JavaScript

How it's bundled: this isn't a single app bundle the way the personal project's Next.js app is, it's dominated by a very large stack of independently loaded third party scripts. The Lighthouse treemap shows a total JS payload of 5.7 MiB, more than double the personal project's 2.8 MiB, and it's overwhelmingly ad tech and video infrastructure:

* gstatic.com/recaptcha (Google reCAPTCHA): 373.1 KiB, 6 percent
* live.primis.tech (video ad platform), three separate files: prebidVid, liveVideo.php/vpaidManager, and an HLS.js copy, adding up to roughly 936 KiB combined
* imasdk.googleapis.com (Google's IMA video ad SDK), loaded twice, 259.0 KiB each, 518 KiB combined for the same library
* s.ntv.io, loaded twice, 258.9 KiB each
* securepubads.g.doubleclick.net (Google Publisher Tag / programmatic ads): 195.1 KiB
* a.pub.network (prebid header bidding): 193.9 KiB plus a separate pubfig.engine.js at 192.7 KiB
* cdn.viafoura.net (comments): 184.6 KiB
* googletagmanager.com/gtm.js (GTM-KT7RHVG): 177.6 KiB
* cdn.confiant-integrations.net (ad quality/security monitoring): 168.1 KiB
* googletagmanager.com/gtag/js?id=G-CW1LSOSXPK: 147.9 KiB, a third distinct Google Analytics property beyond what showed up in the Network tab
* client.riverdrop.com (quiz widget): 138.5 KiB
* c.amazon-adsystem.com/apstag.js (Amazon's ad marketplace): 90.8 KiB
* assets.bouncexchange.com (exit-intent/conversion marketing): appears twice, 116.1 KiB and 41.4 KiB
* www.dianomi.com (native content recommendation/advertising)
* cds.connatix.com (another video ad platform)

Is this the right decision: there's no single "bundle" decision to evaluate the way there is for a first party app, this is a publisher site where the JS footprint is mostly an accumulation of ad tech and engagement vendors layered on over time. The real question isn't whether the bundling strategy is right, it's whether all of these vendors are still needed, several appear to overlap (Google's own ad stack, Amazon's, and a header bidding layer all doing similar jobs; two separate video ad SDKs, Primis and Connatix, both present).

Is there unused JS: yes, confirmed with the Coverage tab. Aggregate details:
* recaptcha_en.js: 891,388 bytes total, 589,216 unused, 66.1 percent
* ssl.p.jwpcdn.com/provider.hlsjs.js: 588,024 total, 417,626 unused, 71 percent
* assets.apnews.com bundle: 455,855 total, 376,668 unused, 82.6 percent
* jwpcdn bidding.js: 473,362 total, 367,243 unused, 77.6 percent
* cdn.viafoura.net/vf-v2.js: 679,044 total, 334,519 unused, 49.3 percent
* apcdp.apnews.com/script.js (AP's own): 141,197 total, 74,985 unused, 53.1 percent
* hj2bd2l1dv.kameleoon.io/engine.js: 157,667 total, 65,260 unused, 41.4 percent

Corrective finding: unlike the personal project where the single biggest unused-code offender was the app's own bundle, here it's almost entirely third party. reCAPTCHA alone ships 891 KB and two thirds of it goes unused on a page that (for most visitors) never triggers a CAPTCHA challenge.

Bundle analysis: the app's own first party code (apcdp.apnews.com/script.js) is a small fraction of the total, about 141 KB out of 5.7 MiB. The real story is that this page is carrying at least four overlapping categories of third party tooling: video ad delivery (Primis, Connatix, Google IMA), display/header bidding ads (GPT, Prebid, Amazon), audience/engagement (Viafoura, Kameleoon, Riverdrop, BounceExchange, Dianomi), and infrastructure (GTM, OneSignal, Zephr, OneTrust, reCAPTCHA).

Source maps: not checked directly on this pass, no .js.map files appeared in any of the captured Network views, consistent with the personal project's pattern of not shipping maps publicly.

## CSS

How it's bundled: a combined, minified stylesheet (All.min.[hash].gz.css) plus a handful of Google Fonts stylesheets (Roboto, Merriweather, Poppins) and a date-stamped custom file (ap-custom-02-06-2026.min.css).

Is this the right decision: the combined approach is reasonable for a content site like this, but the Coverage data below suggests it isn't scoped well.

Is there unused CSS: yes, and severely in places.
* The combined All.min.[hash].gz.css: 801,002 bytes total, 705,570 unused, 88.1 percent
* Google Fonts CSS for Roboto/Merriweather: 23,373 bytes total, 23,373 unused, 100 percent
* Google Fonts CSS for Poppins: 3,591 bytes total, 3,591 unused, 100 percent
* cdn.viafoura.net's CSS: 79,440 total, 79,345 unused, 99.9 percent, and another Viafoura CSS file at 94,262 total, 91,579 unused, 97.2 percent

Corrective finding: two entire Google Fonts stylesheets load and go 100 percent unused, and Viafoura's CSS is effectively all unused too. That's real, identifiable dead weight on every page load.

## Images

Are images shipped in multiple sizes/formats: partially. There's a real resizing proxy in front of assets.apnews.com (URLs go through what looks like an image resizing service returning webp), but the Img-filtered Network tab also shows plenty of plain jpeg still being served, including the video poster image. So it's a mix, not fully modernized the way the personal project's image pipeline is.

Is this the right decision: the resizing proxy itself is good practice, but the inconsistency (some webp, some plain jpeg) means part of the image weight isn't getting the benefit.

Is the full resolution file exposed: not confirmed on this pass, would need the same test as the personal project, request one of the resizing proxy URLs without its size/format parameters.

## Third-party resources

Full real list, combining the Network tab, Coverage tab, and a console script that listed every unique hostname that made a request:

* Google: reCAPTCHA, Tag Manager (GTM-KT7RHVG), two separate gtag/GA4 properties, Fonts, IMA video ad SDK, Publisher Tag/DoubleClick, Amazon's ad system (apstag.js)
* Video ads: Primis (live.primis.tech), Connatix (cd.connatix.com), JW Player plus its entitlements service (DRM/paywall for video) and bidding module
* Prebid/header bidding: a.pub.network (prebid.js, pubfig.engine.js)
* Ad quality monitoring: Confiant
* Audience/CDP/personalization: Viafoura (comments), Kameleoon (A/B testing), Permutive (first-party audience data), BlueConic (customer data platform), Parse.ly (publisher analytics), Sailthru (ak.sail-horizon.com, email/marketing)
* Identity/subscription: Zephr
* Consent: OneTrust
* Notifications: OneSignal
* Interactive content: a quiz widget from client.riverdrop.com, a native content recommendation widget from Dianomi
* Accessibility: a40.usablenet.com, an accessibility overlay/remediation service
* A couple of domains that stood out as worth a second look: rawcdn.githack.com (a raw GitHub content CDN, unusual to see in a production ad/analytics stack) and html-load.cc plus a couple of similarly obfuscated-looking domains, consistent with ad tech vendors that intentionally use unbranded domains

How are they loaded: based on the timeline, most fire up front rather than being deferred, GTM in particular pulls in a large share of the rest.

How much is each impacting load: the treemap gives real transfer size per script (see JavaScript section above). reCAPTCHA and the Primis/IMA video ad stack are the two biggest individual contributors.

Do any seem unnecessary or overlapping: yes, more clearly than on the personal project. There are at least two competing video ad SDKs (Primis and Connatix) and multiple overlapping ad marketplaces (Google's stack, Amazon's, and Prebid's header bidding layer all running at once). Whether all of these are still active/needed is a real question for whoever manages ad operations, from the outside it looks like some of this could be consolidated.

## Bundle related corrective findings

1. reCAPTCHA ships 891 KB with 66 percent unused on a page that doesn't typically challenge visitors with a CAPTCHA. See findings.md.
2. Two entire Google Fonts CSS files (Roboto/Merriweather and Poppins) load and are 100 percent unused. See findings.md.
3. The video ad and header bidding stack is duplicated across vendors (Primis and Connatix both providing video ad delivery, Google/Amazon/Prebid all running header bidding), inflating the JS payload well beyond what a single, consolidated setup would need.
