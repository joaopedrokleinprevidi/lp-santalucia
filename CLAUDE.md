# Motion First Landing Workflow

This repository is built around specialized skills.

Every task MUST use the appropriate specialist.

Never solve everything yourself.

Never think like a single frontend developer.

Think like an elite product team.

---

# Mandatory Workflow

Every implementation MUST follow this order. The numbering is the one in
`landing-page-factory/SKILL.md`, which is the source of truth — if this list ever
disagrees with it, this list is the one that is wrong.

```
0  briefing-cliente → 1 estudo-assets → 2 brand-dna-extractor → 3 auditoria-dados
   → 4 niche-research (if there is a site, an Instagram or a local competitor)
5  creative-direction-expert (pass 1)
6a estrutura-secoes → 6b copy-conversao
7  creative-direction-expert (pass 2)
8a prompt-imagem (sections that earn a generated still)
8b prompt-animacao (stills that earn a clip)
9  the dev generates the media in ChatGPT and Google Flow
10 product-design-expert → frontend-design → landing-motion-expert, which routes:
     scroll drives it            → gsap-scrolltrigger-expert
     there is video              → video-decisao → video-encode | video-to-website
     button|card|menu|form|modal → motion-ui-expert
11 audit-responsivo (11a) → audit-acessibilidade (11b) → audit-performance (11c)
12 publicar-lp
```

Never skip this order.

Every specialist has a single responsibility.

To run the whole chain end to end — from a folder of client assets to a live URL —
invoke `landing-page-factory`. It orchestrates every specialist below, with a
verification gate between phases.

---

# Skill Roster

Twenty-four skills, one layer each. A skill that decides in two layers is two skills.

## Entry — phases 0 to 4

| Skill | Owns | Produces |
|---|---|---|
| `briefing-cliente` | Credential preflight, and the three lists of what to ask the client: what blocks, what hurts, what enriches | `design/briefing.json` + `assets-source/` |
| `estudo-assets` | Every image opened and read one by one, with two verdicts per file | `design/inventario.json` |
| `brand-dna-extractor` | Measuring the brand from its real assets, colour sampled pixel by pixel | `design/design-system.json` |
| `auditoria-dados` | What is still missing or contradictory, in one consolidated round | `design/lacunas.md` |
| `niche-research` | The client's own channels, 3–5 local competitors, the banned industry phrases | `design/pesquisa.md` + `scripts/check-banned-copy.mjs` |

## Direction — phases 5 to 7

| Skill | Owns | Produces |
|---|---|---|
| `creative-direction-expert` | Experience Score, Creative Budget, WOW moments, the 20-40-60-80-100 curve | `design/creative-direction.json` |
| `estrutura-secoes` | Section order, archetype, pacing, `share` converted to scroll per section | `design/estrutura.md`, `src/data/story.ts` |
| `copy-conversao` | Headline, eyebrow, body, button label, CTA strategy, FAQ | `design/landing-blueprint.md`, `src/data/site.ts` |

## Media — phases 8a and 8b

| Skill | Owns | Produces |
|---|---|---|
| `prompt-imagem` | The Style Anchor, and which sections earn a generated still at all | `design/image-prompts.md` |
| `prompt-animacao` | One motion prompt per approved still, and the MB budget per technique | `design/motion-prompts.md` |

## Design — phase 10

| Skill | Owns | Produces |
|---|---|---|
| `product-design-expert` | Type scale, spacing, grid, hierarchy, palette, contrast, tokens | The `@theme` block in `src/styles/index.css` |
| `frontend-design` | Aesthetic character, type pairing, texture, colour proportion | The page's point of view, on top of those tokens |

## Motion — phase 10

| Skill | Owns | Produces |
|---|---|---|
| `landing-motion-expert` | The motion language, and routing between the three motion specialists | Duration, easing and stagger tokens |
| `gsap-scrolltrigger-expert` | Scroll implementation: pin, scrub, parallax, reveal, horizontal | The scroll hooks in `src/hooks/` |
| `motion-ui-expert` | Component motion and every interaction state | Buttons, cards, menus, forms, modals |
| `video-decisao` | Whether the clip stays, what it says, which technique carries it | `design/video-plan.md` |
| `video-encode` | ffmpeg renditions, poster frame and the `<video>` element | `public/media/` + `src/generated/media.ts` |
| `video-to-website` | Canvas frame sequences driven by scroll, frame-accurate on iOS | `public/frames/` + the canvas component |

## Gates — phase 11

| Skill | Owns | Produces |
|---|---|---|
| `audit-responsivo` | Gate 11a — mobile, tablet, viewport, motion density per breakpoint | `design/laudo-responsivo.md` |
| `audit-acessibilidade` | Gate 11b — keyboard, screen reader, contrast, reduced motion | `design/laudo-acessibilidade.md` |
| `audit-performance` | Gate 11c — page weight and Core Web Vitals on the production build | `design/laudo-performance.md` |

## Publication — phase 12

| Skill | Owns | Produces |
|---|---|---|
| `publicar-lp` | Secret preflight, the `lp-<slug>` repo, Vercel deploy, cache headers, DNS | The production URL, delivered to the client |

## Meta — outside the pipeline

| Skill | Owns | Produces |
|---|---|---|
| `landing-page-factory` | The canonical numbering of the 13 phases and the gate between them | No file of its own — the executed sequence |
| `skill-builder` | Creating, auditing and splitting the skills themselves | `.claude/skills/<name>/SKILL.md` |

One source of truth: `design/design-system.json`. Downstream skills reference it —
they never copy its values. A corrected hex propagates instead of being pasted in
five places.

---

# Video to Website

Whenever a section's motion is the story beat, scroll owns the timeline.

Three techniques, three costs. Choose before extracting anything —
see `video-to-website/decision.md`.

Looping `<video>`

↓

300 KB – 2 MB. Ambience behind copy. The default.

Canvas frame sequence

↓

2 – 8 MB. Frame-accurate everywhere, including iOS Safari. The hero technique.

`currentTime` scrubbing

↓

Legacy. iOS Safari does not seek compressed video frame-accurately.

One page gets ONE frame sequence. Two if the page is long and they are far apart.
Three and the page becomes a download.

The AI pipeline for this repository:

design-system.json

↓

one still per section, generated in GPT

↓

each still animated in Google Flow into a 4–6s clip

↓

frames extracted, canvas bound to scroll

Generated clips must be a single continuous movement — no cuts, no shake, no loop.
A cut reads as a glitch the moment scroll passes over it.

---

# Experience Score

Before writing a single line of code, define the Experience Score.

This score determines the expected quality of the final experience.

★☆☆☆☆

Corporate website.

Minimal interactions.

---

★★☆☆☆

Professional landing.

Simple animations.

Good UX.

---

★★★☆☆

Premium SaaS.

Strong hierarchy.

Custom interactions.

Reusable animations.

---

★★★★☆

Premium Product Experience.

GSAP.

Pinned storytelling.

Layered motion.

Creative interactions.

Memorable sections.

(Default)

---

★★★★★

Flagship experience.

Award-level quality.

Scroll storytelling.

Custom motion language.

Video synchronized with scroll.

Immersive interactions.

Unforgettable experience.

---

Before implementation define

Experience Score

Motion Complexity

Creative Budget

Primary WOW Moment

Performance Budget

Accessibility Strategy

Only then begin implementation.

---

# Creative Budget

Every landing receives a Creative Budget.

Animation is expensive.

Attention is limited.

Spend creativity intentionally.

Never spend everything in the Hero.

Users should continuously discover new experiences while scrolling.

Preferred progression

20%

↓

40%

↓

60%

↓

80%

↓

100%

The experience should continuously evolve.

---

Every landing should contain

1 Major WOW Moment

-

2 Medium WOW Moments

-

Several Small Delights

Examples

Major

• Scroll-controlled video

• Pinned storytelling

• Massive product reveal

• Canvas animation

Medium

• Card stacking

• Horizontal gallery

• Device rotation

• Creative CTA

Small

• Hover

• Counters

• Icon motion

• Buttons

• Micro interactions

---

# Specialists

## creative-direction-expert

Responsible for

- creative direction
- wow moments
- rhythm
- creative opportunities
- motion planning
- emotional pacing

This specialist decides WHY users will remember the experience.

Never implement code.

---

## estrutura-secoes

Responsible for

- section order and the narrative archetype
- the user journey and the order in which objections fall
- emotional pacing, `share` per section
- the `scrollMobile / scroll` ratio

This specialist decides WHAT users experience, and in what order.

Writes no copy.

---

## copy-conversao

Responsible for

- headline, eyebrow, body, button label, FAQ
- measured character ceilings per slot
- CTA strategy and how many CTAs the page gets
- conversion flow

This specialist decides WHICH words the page says, inside the structure it is handed.

Never changes the section order.

---

## product-design-expert

Responsible for

- layout
- spacing
- hierarchy
- typography
- premium UI
- composition

Never redesign without consulting this specialist.

---

## landing-motion-expert

Responsible for

- overall motion language
- animation consistency
- section rhythm
- motion orchestration

This specialist decides WHICH motion specialist should be used.

---

## gsap-scrolltrigger-expert

Responsible for

- scroll animations
- timelines
- parallax
- reveal
- pinning
- scrub
- cinematic scrolling

This specialist decides HOW scrolling behaves.

---

## video-decisao

Responsible for

- whether the clip stays or becomes a photograph
- the narrative role: place, process, transformation, atmosphere
- the technique per section: looping video, canvas frames, legacy `currentTime` scrub
- the iOS Safari fallback

Whenever MP4 assets exist,

always evaluate whether they should become scroll-driven experiences.

Default answer should be YES.

Transcodes no byte and writes no component.

---

## video-encode

Responsible for

- the ffmpeg transcode, CRF, faststart and the poster frame
- desktop and mobile renditions, and the weight per second
- the `<video>` element: muted, playsInline, IntersectionObserver gating
- the reduced-motion fallback

Owns the plain `<video>`. Takes direction from video-decisao, never re-decides it.

---

## motion-ui-expert

Responsible for

- buttons
- cards
- menus
- page transitions
- hover
- loading
- micro interactions

This specialist owns component motion.

---

## audit-responsivo

Responsible for

- mobile, tablet and viewport behaviour
- 375px with no horizontal scroll, 44px touch targets, 320px reflow
- motion density per breakpoint: pin that becomes sticky, parallax with reduced travel
- adaptive layouts

Gate 11a. Runs first, because every layout fix moves font size and element position.

---

## audit-acessibilidade

Responsible for

- keyboard, focus order, focus-visible, skip link, focus traps
- screen readers, semantics, heading hierarchy, landmarks, useful alt text
- contrast at 4.5:1 and 3:1, and the scrim over the brightest video frame
- reduced motion

Gate 11b. Runs on the approved responsive report, because contrast and focus
order only exist at runtime.

---

## audit-performance

Responsible for

- page weight in MB against the Experience Score budget
- LCP 2.5s, CLS 0.1, INP 200ms on slow 4G
- the critical path, lazy loading, frame sequence cost
- fonts, `og:image`, AVIF/WebP

Gate 11c. Runs on the production build, because only it has the real numbers.

No implementation is complete until all three gates pass with no BLOQUEIO.

---

## briefing-cliente

Responsible for

- the credential preflight: node, git, gh, vercel, image generator
- the three lists of what to ask the client: what blocks, what hurts, what enriches
- the briefing template, written for someone who is not a designer
- one consolidated round of questions

Phase 0. Nothing starts before it.

Asks once. The third phone call to the client is the one that stops being answered.

---

## estudo-assets

Responsible for

- opening and looking at every single image, one at a time
- classifying by the fact the file carries, never by its filename
- two independent verdicts per file: usable on the page and at what width, usable as a reference
- triaging the assets that carry nothing

Phase 1. A filename is a claim, not evidence.

---

## brand-dna-extractor

Responsible for

- measuring the brand from its real assets
- color sampled pixel by pixel, never estimated
- typographic class identification and substitution
- shape, motif and photo-treatment inventory
- separating verified facts from inferences

Runs first whenever the client provided assets.

Never invent a contact detail. If it is not legible in an asset, it is unverified.

---

## auditoria-dados

Responsible for

- what is still missing once the briefing, the inventory and the design system exist
- the blocks / important / optional split, each with the exact question in plain Portuguese
- the default assumption behind every question that is not asked
- conflicts between two sources that both came from the client

Phase 3. One of the three stops that wait for the dev.

A conflict is never decided alone. Two of the client's own sources contradicting
each other needs the client.

---

## niche-research

Responsible for

- the client's site, Instagram and Google reviews
- 3 to 5 local competitors, and the gap between them
- the industry phrases that are banned from the copy
- the objections, ordered by what each one costs

Phase 4. Runs before a single word of copy is written.

Uses the literal words of the reviews, never the words the industry uses about itself.

---

## prompt-imagem

Responsible for

- the Style Anchor that keeps every generated image on the same brand
- one image prompt per section, and deciding which sections earn one at all
- the three gates: no lettering, no recognizable face, concrete subject
- which real files get attached as reference

Never generates the real storefront or the real team.

Those are real photographs of a real business. Fabricating them asserts something false.

---

## prompt-animacao

Responsible for

- one motion prompt per approved still, written for scroll scrubbing
- clip length, one continuous move, no cut, no shake, no loop seam
- which playback technique each section lands on
- the MB table and the ffmpeg extraction command

The still is the first frame. A clip with a cut in the middle is useless here.

---

## video-to-website

Responsible for

- canvas frame sequences driven by scroll
- frame extraction and the two-phase preloader
- padded-cover rendering and the scroll binding
- section choreography with entrance variety

Owns frame accuracy.

Delegates the plain `<video>` element to video-encode, and the decision of which
technique a section gets to video-decisao.

---

## frontend-design

Responsible for

- the aesthetic decision that precedes code
- type pairing with full Portuguese diacritic support
- texture, grain, atmosphere and depth
- avoiding generic AI aesthetics

Character, not system. The system belongs to product-design-expert.

Inside a fixed brand palette, differentiate through composition, typography and
motion — never by inventing colors the brand does not own.

---

## publicar-lp

Responsible for

- the secret and gitignore preflight before anything is pushed
- the `lp-<slug>` repository on GitHub, private until the sweep passes
- the Vercel deploy, immutable cache headers and the domain DNS
- the post-deploy checks and the message handed to the client

Phase 12. Runs only after all three gates pass with no BLOQUEIO.

Publishes nothing while an item that blocks is still open in `design/lacunas.md`.

---

## landing-page-factory

Responsible for

- running the entire pipeline end to end
- the canonical numbering of the 13 phases — every other skill derives its
  contract from it
- the gate between each phase

Does not do the work. Coordinates the 23 specialists who do.

---

## skill-builder

Responsible for

- creating, rewriting, auditing and splitting the skills themselves
- deciding which skill owns a piece of knowledge, and which owns a number
- keeping every SKILL.md between 150 and 300 lines, and every description at 450
  characters or fewer

Use it whenever a skill contradicts another one.

---

# Creative Direction

The implementation MUST NOT simply reproduce the Design JSON.

It must elevate it.

Whenever the specification leaves room for interpretation,

create premium experiences.

Think

Apple

↓

Linear

↓

Stripe

↓

Nothing

↓

Framer

↓

Vercel

Never create generic landing pages.

---

# Motion Philosophy

Motion tells the story.

Never animate because something exists.

Animate because attention must move.

Every section should have its own rhythm.

Every transition should have intention.

Every animation should reinforce hierarchy.

Never animate for decoration.

---

# Motion ROI

Every animation must answer

Why does this exist?

Accepted answers

- Guide attention
- Improve storytelling
- Improve hierarchy
- Explain something
- Increase perceived quality
- Create delight

If the answer is

"It looks cool."

Remove it.

---

# Surprise Principle

Always search for opportunities to delight users.

Examples

• Hero expands while scrolling

• Video controlled by scroll

• Cards stacking

• Horizontal storytelling

• Mask reveals

• Text reveal

• Layered parallax

• Sticky storytelling

• Dynamic background transitions

• Progressive image reveals

• Product exploding into pieces

• Device mockups rotating

• Infinite marquee

• Mouse reactive elements

• Subtle cursor interactions

Do NOT add these randomly.

Only when they improve storytelling.

---

# Video Rule

Whenever one or more videos are available,

evaluate

• Should this become a scroll-driven timeline?

• Can this replace multiple static images?

• Can scroll become the playback controller?

• Can desktop and mobile use different edits?

Default answer should be YES unless there is a strong reason not to.

---

# Component Rule

If three similar blocks exist,

create a reusable component.

If a pattern repeats,

abstract it.

If abstraction hurts readability,

do not abstract.

---

# Performance Budget

Always optimize for

60 FPS

Minimal Layout Shift

GPU animations

Small bundles

Lazy loading

Intersection Observer

Code splitting

Responsive assets

Never sacrifice performance for flashy effects.

---

# Accessibility

Every interaction must remain usable

without animations.

Respect

prefers-reduced-motion

Never lock information behind motion.

---

# Final Validation

Before considering the task complete,

ask yourself

Does this feel like Apple?

↓

If not, improve.

Does this feel like Linear?

↓

If not, improve.

Would this surprise someone?

↓

If not, improve.

Does motion improve storytelling?

↓

If not, remove it.

The goal is not to create animations.

The goal is to create memorable product experiences.
