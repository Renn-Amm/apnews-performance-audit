# Prioritization System

This is a different system from the personal project one. The personal project used IRCE, an additive score (Severity plus Frequency, times Confidence, divided by Effort). For this project I'm using a multiplicative system instead, Reach times Pain times Ease, or RPE.

## How it works

For each corrective finding, rate three things, each on a 1 to 5 scale.

Reach. How much of the traffic actually hits this. 1 is a narrow edge case, 5 is essentially every visitor.

Pain. How bad it is for the user when they hit it. 1 is barely noticeable, 5 is the page feels broken or unusable.

Ease. How easy the fix is. 1 is a big restructuring project, 5 is a small config or markup change.

Score = Reach times Pain times Ease. Higher score means fix it sooner. Because this is multiplicative instead of divided by effort, a cheap fix (high Ease) can jump ahead of a more severe problem that's expensive to fix, which is a different dynamic than the personal project's system.

## Applied to current corrective findings

| Finding | Reach | Pain | Ease | Score |
|---|---|---|---|---|
| Does not use HTTPS, 1 insecure request | 5 | 3 | 4 | 60 |
| Too many preconnect origins slowing connection setup | 5 | 2 | 5 | 50 |
| LCP delayed by image delivery and render blocking | 5 | 5 | 2 | 50 |
| Main CSS bundle 88.1 percent unused | 5 | 3 | 3 | 45 |
| Heavy JS blocking interactivity (TBT) | 5 | 4 | 2 | 40 |
| Inefficient caching hurting repeat visits | 3 | 3 | 4 | 36 |
| Video ad stack outweighs AP's own code | 4 | 4 | 2 | 32 |
| Viafoura's CSS almost entirely unused | 4 | 2 | 4 | 32 |
| Mobile lab LCP far worse than field LCP under throttling | 3 | 5 | 2 | 30 |
| Legacy JavaScript shipped unnecessarily | 5 | 1 | 5 | 25 |
| Two Google Fonts CSS requests, 100 percent unused | 5 | 1 | 5 | 25 |
| Accessibility gaps (alt text, contrast, unnamed controls) | 2 | 4 | 3 | 24 |
| Duplicated JavaScript across bundles | 5 | 1 | 4 | 20 |
| Uses third-party cookies, 49 found | 5 | 2 | 2 | 20 |
| Speed Index slow specifically under mobile throttling | 3 | 3 | 2 | 18 |
| AP's own first-party footprint is nearly nonexistent | 5 | 3 | 1 | 15 |

## Resulting priority order

1. Fix the HTTPS/insecure request issue, highest score and a real security-adjacent bug, not just a performance nice-to-have
2. Trim the preconnect list to only critical origins (tied with LCP fix below)
3. Fix LCP delivery (preload, remove render blocking resources ahead of it)
4. Split the main CSS bundle so pages stop loading 88 percent unused CSS
5. Cut and defer JS to fix TBT
6. Fix cache headers for static assets
7. Reconsider whether the full video ad stack needs to load immediately on every page (tied with Viafoura's CSS below)
8. Scope Viafoura's CSS loading instead of shipping the full set everywhere
9. Re test LCP specifically under mobile throttling once the above are fixed
10. Set a modern browserslist target to drop legacy JS (tied with the Google Fonts cleanup below)
11. Remove the two fully unused Google Fonts CSS requests
12. Fix accessibility gaps
13. De duplicate shared JS across bundles (tied with the cookie audit below)
14. Audit third-party vendors for unnecessary cookie usage
15. Re check Speed Index under mobile throttling after everything else
16. Treat the thin first-party footprint as a standing architectural constraint, not a one-time fix

The HTTPS fix jumping to the top makes sense here, unlike most of the performance findings, this one is a binary correctness bug with an easy fix, not a matter of degree.
