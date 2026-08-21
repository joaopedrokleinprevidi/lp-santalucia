# Which video technique?

Three techniques, very different costs. Pick before extracting anything.

| | Looping `<video>` | Scrubbed `currentTime` | Canvas frame sequence |
|---|---|---|---|
| Weight | 300 KB–2 MB | 1–4 MB | 2–8 MB |
| Frame accuracy | n/a | poor on Safari/iOS | exact, every browser |
| Scroll-controlled | no | yes, with stutter | yes |
| Decode cost | low | high (seeking) | none after load |
| Good for | ambience, texture, b-roll | long clips, desktop-only | hero, product, transformation |

## Decide

**Looping `<video>`** — the default. Use when the footage sets a mood and the user is meant to
read over it: a clinic interior, hands working, a pet resting. The reference LP in this
workspace (`ChapterFilm.tsx`) does exactly this, with IntersectionObserver gating.

**Canvas frames** — use when the footage *is* the story beat and scroll should own it: a
transformation, a reveal, a product rotating, a 24h day-to-night pass. One or two per page.
Never every section.

**`currentTime` scrubbing** — legacy. Only when the clip is too long to frame-extract (>30s of
essential motion) and the page is desktop-only. iOS Safari does not seek compressed video
frame-accurately, so it visibly stutters. Prefer frames; if you must scrub, throttle updates
and accept the jitter.

## The honest test

> If this section were a still image with a good caption, would anything be lost?

No → use a still. Yes, but only atmosphere → looping video. Yes, the *change over time* is the
point → canvas frames.

## Budget

One page gets **one** frame sequence, or two if the page is long and they are far apart.
Three or more and the page becomes a download, not an experience. Everything else is loops
and stills.

Frame-sequence weight for a full page should stay under **12 MB** on desktop and **5 MB** on
mobile, counted across all sequences. Past that, cut a section rather than the quality of the
one that remains.
