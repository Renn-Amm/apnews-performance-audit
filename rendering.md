# Rendering, Coverage Detail, and Rendering Strategy

## Coverage

Critical CSS: not extracted. The combined All.min...css bundle (see bundles.md) loads as a single external stylesheet with no sign of an inlined critical-path subset in the document head, and it's 88.1 percent unused on this page, consistent with one shared stylesheet being sent everywhere rather than a scoped critical extract per page.
* Corrective finding: no critical CSS inlining, the browser has to fetch and parse the full combined stylesheet before it can paint, which lines up with the render-blocking requests finding already in findings.md.

Unused CSS and JS: real numbers, already documented in bundles.md. Headline repeat: the combined CSS bundle is 88.1 percent unused, two entire Google Fonts stylesheets are 100 percent unused, and Viafoura's CSS is 93 to 99.9 percent unused depending on the file. On the JS side, reCAPTCHA alone is 891 KB with two thirds unused, and the video/ad stack (Primis, Google's IMA SDK, GPT, Prebid) makes up more of the JS payload than AP's own code.

## Flame chart, page load / scroll / interaction

Real recordings, page load, scroll, and a nav click, all captured.

Page load: two frames came in at 133.2 ms and 83.2 ms early in the load, both well past the 16.7 ms budget, before settling into a smooth run of ~16.7 ms frames. Right in that window, the Summary panel shows Rendering (1,172 ms) as the single largest main-thread category over a 5.11 s range, ahead of Scripting (1,092 ms). A specific long task in this window measured 335.82 ms, and its own breakdown was 195 ms Rendering versus 117 ms Scripting, rendering work, not script execution, is the dominant cost here. The 1st/3rd party breakdown for this same range also shows 2,445.3 ms of "[unattributed]" main thread time, more than any named script, which usually points to layout/style recalculation work that isn't cleanly attributed to one source.
* Corrective finding: page load jank here is primarily a rendering/layout cost, not a scripting cost, which is a different root cause than the personal project (where JS execution dominated). That means the fix is different too, it's about reducing DOM complexity, layout thrashing, or expensive CSS, not just deferring scripts.

Scrolling: excessive and unexpected. A single scroll recording shows five separate frames at 349.7 ms, 532.8 ms, 532.8 ms, 366.3 ms, and 233.1 ms, all captured while scrolling through the article feed. That's not occasional jank, that's most of the scroll being well outside a usable frame budget, up to 32 times longer than 16.7 ms.
* Corrective finding: scrolling is janky throughout, not just at isolated moments, confirmed by five separate multi-hundred-millisecond frames in a single short recording.

Interaction (clicking a nav link into a section page): worse still. Frame durations of 283.1 ms, 566.2 ms, 732.8 ms, 549.5 ms, and 749.4 ms were recorded during this single navigation, the longest being 45 times the frame budget. One long task in this window (113.37 ms) was scripting-dominated (109 ms of 113 ms), meaning route/page transition JS is a real contributor here, on top of the rendering cost seen elsewhere.
* Corrective finding: navigating between sections causes near-second-long stalls multiple times in a row, this is excessive for what should be a lightweight in-app navigation.

## Layers and animations

Real data from the Layers panel and a Rendering-tab recording.

* Corrective finding: hovering a single top nav item (to open its dropdown mega-menu) triggers a paint flash across almost the entire viewport, not just the dropdown itself. That's a paint-driven interaction where a composition-only approach (transform/opacity on just the menu panel) would be expected instead, and it's a real, visible cost every time a user explores the nav.
* Corrective finding: JW Player creates a separate compositor layer pair (a `video.jw-video` layer plus a `.jw-preview` layer) for multiple video embeds on a single article page, at least three separate pairs were visible in one Layers panel session, including ones presumably below the fold. There's no indication these are being deferred until the video is actually scrolled into view.
* UsableNet's accessibility overlay widget creates its own small set of layers (`div#usntA40Toggle`, `a#usntA40Link`, `div#usntA40Txt`, `i#usntA40Icon`), each a separate compositor layer for what is a small, fixed-position accessibility button. Not necessarily wrong, but worth knowing this third-party widget adds to the layer count on every page.
* The page's scrollable root shows a "Slow scroll regions" / non-fast-scrollable area, the same pattern as the personal project, meaning a touch or wheel event handler attached broadly enough that the browser can't confidently fast-path scrolling. This is a real, direct contributor to the scroll jank documented above.

## Rendering strategy

Confirmed: fetching an AP News article without running JavaScript still returns full, readable article text, this is server-rendered content, not a client-side-only shell (consistent with AP's known Arc XP publishing platform, a traditional server-rendered CMS rather than a client-hydrated SPA).

Corrective finding 1: server-rendering gets real article content to the browser quickly (FCP is a reasonably fast 1.4 s on desktop, 2.0 s on mobile), but LCP lags dramatically behind (13.0 s desktop, 20.3 s mobile) because the actual largest visible element, usually a hero image or embedded video player, waits behind the ad/video-ad infrastructure documented in bundles.md. The rendering strategy for content itself is sound, the problem is everything competing with it for the main thread and network before the page actually looks finished.

Corrective finding 2: video embeds appear to initialize eagerly rather than being deferred based on viewport visibility. Multiple JW Player video/preview layer pairs exist on a single article page load, which means the video player JS and its associated ad-bidding module (see bundles.md) are likely being set up for videos the user hasn't scrolled to yet, rather than lazy-loading the player only when a video actually enters the viewport.
