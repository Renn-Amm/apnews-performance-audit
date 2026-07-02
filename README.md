# Course Project Performance Audit Report — AP News

## Website

**Name:** AP News (The Associated Press)
**URL:** https://apnews.com/

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
