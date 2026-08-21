---
name: "brand-dna-extractor"
description: "Use when you have a brand's real assets (logo, social posts, storefront photos, printed material) and need to extract a design system from them — colors, typography, shapes, motifs, tone — into a design JSON that drives the landing page. Extrai identidade visual de assets reais e gera design tokens."
argument-hint: "[pasta-de-assets] [nome-da-marca]"
allowed-tools: Read, Glob, Grep, Write, Bash(node *), Bash(npx *)
---

# Brand DNA Extractor — Fase 2

| | |
|---|---|
| **ENTRADA** | `design/inventario.json` (os vereditos por arquivo de `estudo-assets`: `usable`, `maxRenderWidth`, `reference`), os arquivos aprovados dentro de `assets-source/`, e `design/briefing.json` (`meta.niche`, `business.name`, `services`) |
| **SAÍDA** | `design/design-system.json` — `brand`, `voice`, `color` (hex medidos, rampa, contraste), `typography.observed` com `chosen: null`, `shape`, `motifs`, `facts`, `inferences`, `unverified` |
| **ANTES** | `estudo-assets` (Fase 1) já abriu cada arquivo e decidiu quais servem. Esta skill não re-inventaria: começa nos pixels dos arquivos aprovados |
| **DEPOIS** | `auditoria-dados` (Fase 3) cruza `facts` e `unverified` daqui com o briefing e monta `design/lacunas.md` |

Real brands arrive as a folder of JPEGs, not a style guide. This skill turns that folder into
`design/design-system.json` — the single source of truth every downstream specialist reads.

The output is not a mood board. It is tokens: hex values, font stacks, radii, shape rules,
motif inventory. If a value cannot be measured from the assets or stated by the client, it does
not go in the JSON as fact — it goes in `inferences` with the reasoning attached.

## Why this exists

Without it, every section gets designed from a fresh guess and the page drifts: three purples,
two yellows, a border radius that changes per component. The client sees their brand "almost"
rendered, which reads worse than a page that never tried.

## Step 1 — Inventory the assets

**If `design/inventario.json` exists, read it first and do not re-inventory.** `estudo-assets`
already opened every file, measured it, and recorded `dimensions`, `usable`, `maxRenderWidth`,
`shows`, `rights` and the reason behind every discard. Trust those verdicts: a second, divergent
inventory is how a project ends up with two answers about the same logo. Your job here starts at
the pixels — colour, type class, shape, motif — on the files that inventory marked usable, plus
any file marked `reference: true`.

The boundary in one line: `estudo-assets` asks "is this true and may we publish it?";
this skill asks "what is the brand made of?".

Only when there is no `inventario.json` (this skill invoked on its own, outside the pipeline):

```bash
ls -la <ASSETS_DIR>
```

Classify each file, because each type yields different evidence:

| Type | Yields |
|---|---|
| Logo (flat color background) | The brand's true primary — the most trustworthy color sample on disk |
| Social posts | Typography, headline patterns, motifs, layout habits, the client's own copy voice |
| Storefront / interior photos | Environmental palette, materials, real-world lighting, signage |
| Team / service photos | Wardrobe colors, props, what the camera actually sees |
| Printed material | Secondary palette, iconography |

Read every image. Do not sample colors from a file you have not looked at — a photo of a purple
wall at sunset will hand you an orange that is not in the brand.

## Step 2 — Measure the palette

Never eyeball hex values. Sample them. Two passes:

**Pass A — dominant colors** tells you what the brand's surfaces are made of:

```js
const { data, info } = await sharp(file).resize(120, 120, { fit: 'inside' })
  .raw().toBuffer({ resolveWithObject: true })
// quantize each channel to 12-level buckets, count, take the top 6
```

**Pass B — chromatic isolation** tells you what the brand's *colors* are. Dominant sampling on
a post with a cream background returns cream. Filter to saturated pixels first, then group by
hue range:

```js
// keep only s >= 45%, 12% <= l <= 88%, then bucket by hue
if (h >= 250 && h <= 295) purple.set(key, count + 1)   // brand hue window
```

Run pass B across the logo and social posts only. Photos contribute environment, not identity.

Record for every color: hex, HSL, which asset it came from, and its share of chromatic pixels.
The winner by count in each hue window is the brand color. Neighbours within ~6% lightness are
the same color under different compression — collapse them.

### Build the ramp

A brand hands you 2–3 colors. A landing page needs ~24. Derive, don't invent — and derive with
the **one** ramp this project uses: the eleven OKLCH steps of
[product-design-expert](../product-design-expert/tokens.md#rampa-de-cor-a-partir-de-uma-cor-de-marca),
which holds lightness per step and modulates chroma. That skill owns the ramp; this one owns the
input hex. A second ramp invented here produces two palettes for the same brand.

- **Where the brand color lands**: measure its lightness and slot it into the nearest step —
  don't decree it is the 500. A brand at L≈0.74 filed as 500 leaves two near-identical steps.
- **Ink**: the darkest brand shade, not `#000`. Pure black next to a warm cream reads as a hole.
- **Surface**: the off-white measured from the posts, not `#fff`. Real brands rarely sit on pure white.
- **Verify contrast at build time**, not at review time. Body text needs 4.5:1, large text 3:1.
  If the brand's primary fails on its own surface, say so and provide the passing shade —
  do not silently swap the brand color.

## Step 3 — Read the typography

You cannot extract a font file from a JPEG. You can identify the *class* and match it.

Look at the headline letterforms in the social posts and answer:

- Serif or sans? If sans: geometric (circular `o`, single-storey `a`) / grotesque / humanist?
- Weight used for headlines — is the brand shouting (800+) or speaking (500–600)?
- Is there a second face, and is it doing a different job (script for warmth, condensed for labels)?
- Case habits: all-caps labels? Sentence-case headlines? Mixed-weight headlines where one word
  changes color?

**Stop at the class.** Which family actually ships is
[frontend-design](../frontend-design/type-pairs.md)'s call — it owns the catalogue, the pairing
matrix and the Portuguese verification procedure. This skill hands it the observed class plus any
constraint the brand imposes (a mandatory licensed face, an all-caps habit), and leaves `chosen`
empty for that skill to fill:

```json
"typography": {
  "observed": { "headline": "geometric sans, 800 weight, tight tracking", "source": "post-banho-e-tosa.jpg" },
  "constraints": { "mandatory": null, "caseHabit": "all-caps labels, sentence-case headlines" },
  "chosen": null
}
```

Do not name a family here just because one comes to mind — a class recorded honestly gives
`frontend-design` five valid candidates; a family guessed here gets copied forward as if it had
been decided.

## Step 4 — Catalogue shape and motif

This is what separates a page that *is* the brand from a page that merely uses its colors.

For each recurring visual element, record what it is, where it appears, and what it means:

- **Corner treatment** — radius in the posts, whether it is uniform or one-corner
- **Container shapes** — pills, rounded rectangles, organic blobs, arches
- **Decorative motifs** — the small repeated marks (paw prints, hearts, bubbles, sparkles);
  note their opacity and whether they sit behind or in front of content
- **Dividers** — solid rule, dotted, dashed, or a shape
- **Badges** — how the brand marks a claim (a circle with an icon, a pill with text)
- **Photo treatment** — cutout, rounded frame, arch mask, full-bleed
- **Curve language** — do sections meet in straight lines or in a swoosh?

Motifs are reusable as page furniture: a paw print at 6% opacity is a background texture, a
heart is a section divider, an arch is a photo mask. Write down which is which.

## Step 5 — Extract the real data

The assets carry facts. Pull them and mark each one's source, because these end up on a page
where being wrong is expensive:

- Phone, address, hours, CEP, founding year
- Service list — take it verbatim from the posts; the client's own wording is the right wording
- Claims and numbers (rating, review count, years, volume)
- Any regulatory or license info visible on signage

Anything not visible in an asset or a client document goes in `unverified` with a note. Never
promote an inference to a fact. A phone number that is one digit wrong costs the client money.

## Step 6 — Write the JSON

Output to `design/design-system.json`. Structure:

```json
{
  "brand": { "name": "", "segment": "", "founded": "", "location": "" },
  "voice": { "tone": [], "persona": "", "avoid": [] },
  "color": {
    "primary": { "hex": "", "hsl": "", "source": "", "share": "" },
    "accent":  { "hex": "", "hsl": "", "source": "", "share": "" },
    "ink": "", "surface": "",
    "ramp": { "purple": {}, "amber": {}, "neutral": {} },
    "contrast": [ { "pair": "ink-on-surface", "ratio": 0, "passes": "AA" } ]
  },
  "typography": { "observed": {}, "constraints": {}, "chosen": null },
  "shape": { "radius": {}, "containers": [], "dividers": [], "photoTreatment": [] },
  "motifs": [ { "name": "", "usage": "", "opacity": "", "source": "" } ],
  "facts": { "phone": "", "address": "", "hours": "", "services": [], "claims": [] },
  "inferences": [ { "claim": "", "basis": "", "confidence": "" } ],
  "unverified": []
}
```

Keep `facts` and `inferences` structurally separate. Downstream skills are allowed to render
`facts` as page copy and are **not** allowed to render `inferences` without a human check.

## Step 7 — Sanity check

- [ ] Every hex traces to a named asset file
- [ ] Ramps are monotonic in lightness with no duplicate steps
- [ ] Body text on surface ≥ 4.5:1; headline on surface ≥ 3:1 — computed, not assumed
- [ ] `typography.observed` names a class, not a family; `chosen` is still `null`
- [ ] Every motif names the asset it came from
- [ ] No phone/address in `facts` that you did not read off an asset or client document
- [ ] The JSON parses (`node -e "JSON.parse(require('fs').readFileSync('design/design-system.json'))"`)

## Anti-patterns

- **Sampling a photo for the brand color** — ambient light shifts hue; a purple wall at dusk
  yields a color the brand does not own. Sample flat-color assets.
- **Taking the dominant color as the brand color** — on a cream post the dominant is cream.
  Isolate by saturation first.
- **Inventing a palette that "elevates" the brand** — the client will not recognize their
  business. Elevate through composition, typography and motion; keep their colors.
- **Pure `#000` / `#fff`** — reads as a hole and a glare against a measured warm palette.
- **Naming the font family here** — the class is evidence, the family is a design decision, and
  a family written into this JSON gets treated downstream as measured fact. Class only.
- **One JSON per section** — the whole point is one source of truth. One file, all sections read it.
- **Guessing contact data** — if it is not in an asset, it is `unverified`, full stop.

## Handoff

`design/design-system.json` feeds, in pipeline order (phases 5 → 6a → 6b → 8a → 10a → 10b):
`creative-direction-expert` (budget) → `estrutura-secoes` (section order) → `copy-conversao`
(copy) → `prompt-imagem` (image prompts — the palette becomes the style anchor) →
`product-design-expert` (scale, ramp, contrast) → `frontend-design` (family, texture, colour
proportion — it fills `typography.chosen`).

When the JSON changes, those six re-read it. It is not copied into their outputs — it is
referenced, so a corrected hex propagates instead of being pasted in five places.
