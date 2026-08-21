# Motion First Landing Workflow

This repository is built around specialized skills.

Every task MUST use the appropriate specialist.

Never solve everything yourself.

Never think like a single frontend developer.

Think like an elite product team.

---

# Mandatory Workflow

Every implementation MUST follow this order.

Brand DNA (if client assets exist)

↓

Creative Direction

↓

Landing Storytelling

↓

AI Visual Prompts (if sections need generated imagery)

↓

Product Design

↓

Frontend Design (aesthetic character)

↓

Landing Motion

↓

GSAP ScrollTrigger (if scroll exists)

↓

Scroll Video Director (if video exists)

↓

Video to Website (if scroll must control playback frame by frame)

↓

Motion UI (component interactions)

↓

Responsive & Accessibility

Never skip this order.

Every specialist has a single responsibility.

To run the whole chain end to end — from a folder of client assets to a live URL —
invoke `landing-page-factory`. It orchestrates every specialist below, with a
verification gate between phases.

---

# Skill Roster

| Skill | Owns | Produces |
|---|---|---|
| `landing-page-factory` | Orchestration of the entire pipeline | The live site |
| `brand-dna-extractor` | Measuring a brand from its real assets | `design/design-system.json` |
| `creative-direction-expert` | Experience Score, Creative Budget, WOW moments | `design/creative-direction.json` |
| `landing-storytelling-director` | Section order, narrative, copy, CTA strategy | `design/landing-blueprint.md` |
| `ai-visual-prompt-director` | Image and animation prompts per section | `design/image-prompts.md`, `design/motion-prompts.md` |
| `product-design-expert` | Type scale, spacing, grid, tokens, hierarchy | The design system in code |
| `frontend-design` | Aesthetic character, type pairing, texture | The page's point of view |
| `landing-motion-expert` | Motion language and routing between specialists | Duration/easing/stagger tokens |
| `gsap-scrolltrigger-expert` | Scroll implementation: pin, scrub, parallax, reveal | Chapter timelines |
| `scroll-video-director` | The `<video>` element and video direction | Encoded renditions, poster fallbacks |
| `video-to-website` | Canvas frame sequences driven by scroll | Extracted frames + the scrubber |
| `motion-ui-expert` | Component motion and every interaction state | Buttons, cards, menus, forms |
| `responsive-e-acessibility` | The final audit — this gate blocks the merge | Pass or fail, item by item |
| `skill-builder` | Creating and auditing the skills themselves | Skill files |

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

## landing-storytelling-director

Responsible for

- section order
- storytelling
- user journey
- pacing
- CTA strategy
- conversion flow

This specialist decides WHAT users experience.

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

## scroll-video-director

Responsible for

- scroll synchronized videos
- currentTime mapping
- video storytelling
- cinematic sections

Whenever MP4 assets exist,

always evaluate whether they should become scroll-driven experiences.

Default answer should be YES.

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

## responsive-e-acessibility

Responsible for

- mobile
- tablet
- keyboard
- screen readers
- reduced motion
- accessibility
- adaptive layouts

No implementation is complete until approved by this specialist.

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

## ai-visual-prompt-director

Responsible for

- the Style Anchor that keeps every generated image on the same brand
- one image prompt per section
- one motion prompt per section, written for scroll scrubbing
- deciding which sections earn a generated image at all

Never generates the real storefront or the real team.

Those are real photographs of a real business. Fabricating them asserts something false.

---

## video-to-website

Responsible for

- canvas frame sequences driven by scroll
- frame extraction and the two-phase preloader
- padded-cover rendering and the scroll binding
- section choreography with entrance variety

Owns frame accuracy.

Delegates the plain `<video>` element back to scroll-video-director.

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

## landing-page-factory

Responsible for

- running the entire pipeline end to end
- verifying credentials before any work begins
- the gate between each phase
- publishing to GitHub and Vercel

Does not do the work. Coordinates the specialists who do.

---

## skill-builder

Responsible for

- creating, rewriting and auditing the skills themselves
- deciding which skill owns a piece of knowledge
- keeping every SKILL.md under 500 lines

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
