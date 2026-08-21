---
name: ai-visual-prompt-director
description: Use when generating AI images and animated clips for each landing page section — writes the GPT/Sora image prompts and the Google Flow/Veo motion prompts, keeping every section visually consistent with the brand. Cria prompts de imagem e animacao por secao para virar video-to-website.
argument-hint: [secao] [papel-visual]
---

# AI Visual Prompt Director

Pipeline this skill sits inside:

```
design-system.json  →  still per section (GPT)  →  clip per section (Google Flow)  →  video-to-website
```

The job is not "write a nice prompt". It is to make eight independently generated images look
like they were shot by one photographer, on one day, for one brand — and then to make each one
move in a way scroll can drive.

Read `design/design-system.json` first. Every prompt inherits from it.

## The consistency problem

Generative models have no memory between calls. Eight prompts written independently produce
eight different studios. The fix is a **style anchor**: one paragraph, written once, prepended
verbatim to every image prompt in the project. It never varies. Only the subject block after it
changes.

```
STYLE ANCHOR (identical in every prompt, never edited mid-project):
Professional editorial photography for a veterinary clinic brand. Soft diffused daylight from
camera left, gentle falloff, no hard shadows. Shallow depth of field at f/2.0. Warm neutral
palette built on cream #F7F4EF with deep violet #4E2A96 and amber #FCB400 appearing only as
real objects in frame — uniforms, equipment, signage. Clean uncluttered composition, generous
negative space. Shot on 50mm. Natural skin and fur tones, no color grading, no filter look.
```

Three rules for the anchor:

1. **Describe light, lens and palette** — those three carry consistency further than any
   subject wording.
2. **Bind brand color to physical objects.** "Violet uniforms and equipment" survives
   generation. "Violet color scheme" produces a violet wash over everything and looks fake.
3. **Freeze it.** If the anchor changes at section 5, sections 1–4 no longer match. Regenerate
   all or change none.

## Step 1 — Assign a visual role per section

Not every section needs a generated image. Decide the role first:

| Role | Use generated image? | Why |
|---|---|---|
| Hero | Yes — this is the one to over-invest in | Sets the anchor everything else matches |
| Proof / credential | **No** — use the real photo | A generated storefront is a lie about a real place |
| Team / people | **No** unless faces are unrecognizable | Fabricating identifiable staff misrepresents the business |
| Service / process | Yes | Generic enough to generate, specific enough to matter |
| Atmosphere / transition | Yes | Texture and mood, low risk |
| Stats / data | No | Type and motion carry it; an image competes with the numbers |
| Testimonial | No | Real names deserve real context, or none |
| CTA | Optional | Often stronger as flat brand color |

**Hard line:** never generate an image that asserts a fact about the real business — the actual
building, actual staff, actual equipment they may not own. Use the client's real photos for
anything that makes a claim. Generated imagery is for illustrative and atmospheric roles only.
For this workspace that means the storefront and team photos in `assets/` are used as-is.

## Step 2 — Write the image prompt

Structure, in this order. The order matters — models weight early tokens more heavily.

```
[STYLE ANCHOR verbatim]

SUBJECT: <one sentence, concrete, present tense>
COMPOSITION: <where the subject sits, where the empty space is>
DETAIL: <two or three specifics that make it real>
FRAMING: <shot size and angle>
EXCLUDE: text, letters, logos, watermarks, signage copy, extra limbs, distorted paws
```

### Composition is a layout instruction, not a taste

The image has to hold DOM copy on top of it. Say where the copy goes:

| Section layout | Composition line |
|---|---|
| Copy left | "Subject occupies the right third; left two-thirds is soft out-of-focus background" |
| Copy right | mirror of the above |
| Copy below | "Subject in the upper half, lower third is clean uninterrupted surface" |
| Full-bleed with scrim | "Subject centered, even tonal field, no bright hotspots near the edges" |

Get this wrong and you will be dropping a 70%-opacity scrim over a good photo to rescue
legibility, which wastes the photo.

### Never generate text

Models render text as plausible-looking garbage, and Portuguese diacritics make it worse. All
copy is DOM text — that is also what makes it selectable, translatable and searchable.
`EXCLUDE: text, letters, logos, watermarks` goes in every prompt.

### Aspect ratio by role

| Role | Ratio | Note |
|---|---|---|
| Hero, full-bleed | 16:9 | Generate wider than needed; crop in the pipeline, never upscale |
| Section split | 4:3 or 1:1 | Matches a half-width column without awkward cropping |
| Portrait / mobile-first | 9:16 | Only when the section is mobile-led |
| Texture / transition | 21:9 | Reads as a band, not a picture |

Generate at the highest resolution the tool offers. `prepare-assets.mjs` will emit the AVIF and
WebP renditions; feed it the largest original you have.

## Step 3 — Write the motion prompt (Google Flow / Veo)

The still is the first frame. The clip has to be *scrubable*, which is a stricter requirement
than "looks good".

```
[same STYLE ANCHOR, one line]

MOTION: <one continuous movement, one direction, no cuts>
CAMERA: <static | slow push in | slow pull back | gentle pan left-to-right>
SUBJECT ACTION: <small, natural, completes within the clip>
PACE: slow, deliberate, 4–6 seconds
EXCLUDE: cuts, scene changes, camera shake, zoom snaps, text overlays, speed ramps
```

### What survives frame extraction

| Works | Fails |
|---|---|
| Slow camera push / pull | Any cut — a scrub across a cut reads as a glitch |
| Single-axis pan | Handheld shake — scrubbing amplifies it into jitter |
| Fur, steam, fabric, water settling | Fast limb motion — morphs badly frame to frame |
| Light shifting across a surface | Anything with rendered text |
| A head turning once, slowly | Loops — generated ends rarely meet their starts |
| Depth-of-field rack | Crowds, many moving subjects |

### Rules that come from the extraction stage

- **4–6 seconds.** At native fps that lands near the 150-frame floor. Longer clips force
  decimation and the motion starts to strobe.
- **One movement.** Scroll maps linearly to frame index. Two movements in one clip means
  scrolling produces a stutter at the seam.
- **Never request a loop.** The sequence rests on its final frame — see `video-to-website`.
- **Keep the subject inside the safe area.** The canvas renders at `IMAGE_SCALE` 0.85, so the
  outer ~7% of each edge is padding. Anything important out there gets cut.
- **Ask for slow.** Generated footage has no true motion blur; fast motion strobes under scrub.

## Step 4 — Emit the prompt files

Two files in `design/`, one row per section, so they can be pasted into the tools in order:

- `design/image-prompts.md` — the anchor once at the top, then one block per section
- `design/motion-prompts.md` — the same section order, each referencing its still

Each block carries: section name, visual role, aspect ratio, the full prompt, and a
one-line acceptance test ("what would make me regenerate this").

## Step 5 — Review what came back

Reject and regenerate rather than accepting something you will have to fix in CSS.

- [ ] Does it sit beside the hero image without looking like a different brand?
- [ ] Is the negative space where the composition line asked for it?
- [ ] Any invented text, letterform or logo anywhere in frame? → regenerate
- [ ] Paws, hands, ears, eyes anatomically sane at 100%? → the usual failure point
- [ ] Brand color present as an object, not as a wash?
- [ ] For clips: any cut, shake or speed change? → unusable for scrub, regenerate
- [ ] Does the last frame of the clip work as a still? It will be on screen the longest.

## Anti-patterns

- **Rewriting the anchor per section** — this is the single cause of a page that looks
  assembled from stock. One anchor, frozen.
- **"Cinematic, 8k, hyperrealistic, award-winning"** — quality-word soup. These push toward a
  generic render look. Describe light, lens and material instead.
- **Generating the real storefront or real staff** — misrepresents the business. Real photos only.
- **Prompting for the brand color as a color scheme** — produces a monochrome wash. Bind it
  to objects.
- **Requesting text in the image** — always garbage, never necessary, breaks translation and SEO.
- **Fast or complex motion** — morphs during generation and strobes during scrub.
- **Accepting a near-miss** — a section that is 80% on-brand drags the whole page down more
  than a section with no image at all.
