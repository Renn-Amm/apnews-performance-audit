# Mobile

All PSI numbers labeled mobile in baseline.md and findings.md are the mobile scan, run under Lighthouse's Moto G Power emulation with slow 4G throttling, matching the throttling used in class. A desktop PSI scan has now been run for real comparison, same URL, July 15 2026.

## Desktop vs mobile, real comparison

Field data (real users, desktop): Core Web Vitals Assessment Failed. LCP 3.6 s, INP 164 ms, CLS 0.02, FCP 2.3 s, TTFB 0.2 s.
Field data (real users, mobile): Core Web Vitals Assessment Failed. LCP 2.9 s, INP 177 ms, CLS 0.06, FCP 2.0 s, TTFB 0.2 s.

Lab data (desktop, custom throttling): Performance 31, FCP 1.4 s, LCP 13.0 s, TBT 2,570 ms, CLS 0, Speed Index 9.0 s.
Lab data (mobile, Moto G Power, slow 4G): Performance 28, FCP 6.5 s, LCP 20.3 s, TBT 1,900 ms, CLS 0.001, Speed Index 15.6 s.

This is an important difference from the personal project. On Corsair, desktop field data passed and desktop lab numbers were meaningfully better than mobile. Here, desktop field data still fails the Core Web Vitals assessment, and desktop TBT (2,570 ms) is actually worse than mobile TBT (1,900 ms). That's a real signal that this page's problems are not primarily about mobile constraints, the third-party and ad-tech stack documented in findings.md and bundles.md is heavy enough to hurt a full-power desktop browser too.

## Mobile specific findings

1. The gap between real world and worst case mobile is still large, even though the underlying problem isn't mobile-specific. Field LCP is 2.9 s but lab LCP under slow 4G throttling on a mid range phone is 20.3 s.
   * Metric(s): LCP
   * Cause: real users in the field sample skew toward faster devices and connections than the slow 4G plus low end device combination PSI tests in the lab, so the lab number is a worst case a real chunk of mobile users will actually hit, on top of the underlying third-party weight that hurts every device.
   * Solution: corrective. Test and set a performance budget specifically for the slow network and low end device case, on top of whatever gets done about the third-party stack generally.

2. Visual completeness is dramatically worse on mobile specifically. Speed Index is 15.6 s in the mobile lab test versus 9.0 s on desktop, a bigger gap than LCP shows.
   * Metric(s): Speed Index
   * Cause: the same root causes documented in findings.md (heavy ad tech, unused CSS/JS, 45 third-party domains), but a slower CPU and throttled network amplify the visual-completeness impact more than they amplify LCP specifically.
   * Solution: corrective. Re test Speed Index specifically under the Moto G Power / slow 4G profile after any fixes to the third-party stack, since it's the most mobile-sensitive metric in this set.

Both findings above are corrective.
