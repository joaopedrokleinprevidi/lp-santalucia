---
name: video-to-website
description: Use when turning one or more videos (MP4/MOV, filmed or AI-generated in Google Flow/Veo/Runway) into a scroll-driven landing page where scroll position controls playback frame by frame. Covers frame extraction, canvas rendering, choreography and the React/Vite port.
argument-hint: [caminho-do-video] [nome-da-secao]
---

# Video to Scroll-Driven Website

| | |
|---|---|
| **ENTRADA** | as linhas `canvas frames` de `design/video-plan.md` (capítulo, duração de origem, alvo); o clipe correspondente em `design/renders/NN-secao.mp4` ou `assets-source/*.mp4`; `design/creative-direction.json` (`budget.mediaDesktopMB` / `mediaMobileMB`) e `design/design-system.json` (cor e tipo) |
| **SAÍDA** | `public/frames/<capitulo>/frame_%04d.webp` nos dois sets (1920 e 1280), a contagem de frames em `src/generated/media.ts`, o componente de canvas e o último frame congelado como poster de `prefers-reduced-motion` |
| **ANTES** | quem chama é `video-decisao`, uma linha de plano por vez, e só para `canvas frames`. `landing-motion-expert` (Fase 10c) roteia primeiro para `video-decisao`, nunca direto para cá — a escolha da técnica é irreversível depois da extração |
| **DEPOIS** | `gsap-scrolltrigger-expert` liga o progresso do capítulo ao índice do frame; `audit-performance` (Fase 11c) pesa `public/frames/` contra o orçamento de mídia |

Fora do trilho linear: esta skill não tem uma fase própria. Ela é a implementação de uma linha
`canvas frames` do plano de vídeo — sem essa linha, não roda.

Scroll becomes the timeline. The user is not watching a video — they are *moving* it.

This skill covers the **canvas frame-sequence** technique: the video is decoded once into
still frames, and scroll picks which frame to paint. This is how Apple does product pages.
It is the only approach that is frame-accurate on iOS Safari.

> **Choosing the technique** — read [decision.md](decision.md) before extracting anything.
> Frame sequences are expensive (2–8 MB per section). Not every video deserves one.
> A looping `<video>` behind a chapter is often the correct, cheaper answer.

## Input

- One or more video files, and the section each one belongs to.
- The design JSON, if `brand-dna-extractor` already ran — colors and type come from there.
- Whether the footage is **product/hero motion** (frames) or **ambience** (loop).

If the section list is missing, read `src/data/site.ts` (or the project's content source)
rather than inventing sections.

## Non-Negotiables

These are measured, not opinions. Every number here came from a shipped page that felt wrong
until it hit the threshold.

| # | Rule | Threshold |
|---|------|-----------|
| 1 | Smooth scroll via Lenis | `duration: 1.2`, native scroll feels like a document, not an experience |
| 2 | Animation variety | 4+ entrance types, never the same one twice in a row |
| 3 | Staggered reveals | label → heading → body → CTA, `stagger: 0.10–0.15` |
| 4 | No glass cards | text sits on the background; hierarchy comes from size/weight/color |
| 5 | Direction variety | consecutive sections enter from different axes |
| 6 | Stats overlay | dark scrim `0.88–0.92`, counters animate from 0 |
| 7 | Oversized marquee | at least one text element at `10–15vw` sliding on scroll |
| 8 | Counters | every number counts up, never static |
| 9 | Typography | hero `clamp(3.5rem, 12vw, 11rem)`, sections `3rem+`, labels `0.7rem/0.15em` |
| 10 | CTA persists | `data-persist="true"` — the final section never animates back out |
| 11 | Scroll budget | hero ≥20% of range; ≥800vh total for 6 sections (≥550vh mobile) |
| 12 | Side-aligned copy | text lives in the outer 40%; center is the product. Exception: stats |
| 13 | Hero handoff | hero is standalone 100vh; canvas enters via `clip-path: circle()` |
| 14 | Frame speed | `FRAME_SPEED` 1.8–2.2 — motion completes by ~55% of scroll |

Rule 12 has one more exception: when the frame content is a *face* (a vet, a client, a pet),
center it and put copy in a bottom-left third. Faces read as subjects, not products.

**Escopo desta tabela: a página de canvas frame sequence.** Rules 2, 3, 5 and 9 restate numbers
another skill owns, calibrated for copy sitting over a full-screen canvas in motion. Numa landing
de capítulos, a variedade e a contagem de entradas são de `creative-direction-expert`, o stagger
e as distâncias são de `landing-motion-expert`, e a escala tipográfica é de
`product-design-expert` — os valores deles vencem, e o hero em `clamp(3.5rem, 12vw, 11rem)` da
rule 9 é headline curta em inglês com grotesca, não uma copy em português num serif de display.

## Accessibility (blocking — the page ships without this only over my objection)

The source skill this was adapted from omitted accessibility entirely. It is not optional here.

- Under `prefers-reduced-motion`, **never** build the canvas. Render the poster frame
  (`frame_0001.webp` upscaled, or the video's own poster) and let sections reveal with a
  plain opacity transition. No pinning, no scrubbing, no marquee movement.
- Every canvas gets `role="img"` and an `aria-label` describing what the footage shows.
- Copy must be real DOM text in reading order. Never bake a sentence into a frame.
- Keyboard: the page must be fully readable with `Tab` + `Space`/`PageDown`. Lenis keeps
  native keyboard scrolling — do not intercept keys.
- Contrast: `#666` is the lightest acceptable body text on cream. Headings use the brand ink.

```js
const reduced = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
if (reduced) { renderPosterFallback(); return; }  // before any frame is fetched
```

## Workflow

### Step 1 — Probe the source

```bash
npx ffprobe -v error -select_streams v:0 \
  -show_entries stream=width,height,duration,r_frame_rate,nb_frames \
  -of csv=p=0 "<VIDEO>"
```

`ffmpeg-static` / `ffprobe-static` are already devDependencies in this project — use those
binaries via `node scripts/…`, or `npx`. Never install ffmpeg globally, and never hardcode a
machine-specific binary path into a script.

Decide the frame budget from duration:

| Source duration | Extract at | Target frames |
|---|---|---|
| < 10s | native fps, capped | 150–300 |
| 10–30s | 10–15 fps | 200–300 |
| 30s+ | 5–10 fps | 240–300 |

Below 150 frames the motion strobes. Above 300 the payload stops being worth it.

### Step 2 — Extract

```bash
mkdir -p public/frames/<section>
npx ffmpeg -i "<VIDEO>" \
  -vf "fps=<FPS>,scale=<WIDTH>:-1:flags=lanczos" \
  -c:v libwebp -quality 78 \
  "public/frames/<section>/frame_%04d.webp"
```

Cap width at 1920 desktop. Emit a second 1280px set for mobile when the section is above the
fold — a phone loading 300×1920 frames will be killed by the OS.

Verify: `ls public/frames/<section> | wc -l` and total weight `du -sh`. If the set exceeds
**8 MB**, drop fps before dropping quality — fewer good frames beat more mushy ones.

### Step 3 — Preload in two phases

First 10 frames block the loader; the rest stream in behind it. Hide the loader only when
frame 10 is decoded, not when all are — the user can start scrolling while the tail loads.

```js
async function preload(count, onProgress) {
  const load = (i) => new Promise((res) => {
    const img = new Image();
    img.onload = img.onerror = () => { frames[i] = img; onProgress(); res(); };
    img.src = `/frames/${section}/frame_${String(i + 1).padStart(4, "0")}.webp`;
  });
  await Promise.all(Array.from({ length: 10 }, (_, i) => load(i)));   // phase 1 — blocking
  hideLoader();
  Array.from({ length: count - 10 }, (_, i) => load(i + 10));          // phase 2 — background
}
```

### Step 4 — Render: padded cover

```js
const IMAGE_SCALE = 0.85;              // 0.82–0.90. Pure cover clips into the header.
function drawFrame(i) {
  const img = frames[i];
  if (!img) return;
  const { width: cw, height: ch } = canvas;
  const scale = Math.max(cw / img.naturalWidth, ch / img.naturalHeight) * IMAGE_SCALE;
  const dw = img.naturalWidth * scale, dh = img.naturalHeight * scale;
  ctx.fillStyle = bgColor;             // sampled from frame corners every ~20 frames
  ctx.fillRect(0, 0, cw, ch);
  ctx.drawImage(img, (cw - dw) / 2, (ch - dh) / 2, dw, dh);
}
```

Fill with the sampled corner color **before** drawing so the padded border disappears into the
page. Scale the backing store by `devicePixelRatio` or the frames render soft on retina.

### Step 5 — Bind frames to scroll

```js
const FRAME_SPEED = 2.0;
ScrollTrigger.create({
  trigger: scrollContainer, start: "top top", end: "bottom bottom", scrub: true,
  onUpdate: (self) => {
    const i = Math.min(Math.floor(Math.min(self.progress * FRAME_SPEED, 1) * COUNT), COUNT - 1);
    if (i !== current) { current = i; requestAnimationFrame(() => drawFrame(current)); }
  },
});
```

Guard on `i !== current` — repainting an unchanged frame is the most common cause of a page
that profiles at 40fps for no visible reason.

### Step 6 — Choreograph the sections

Each section declares its own entrance. Full timeline code and the animation table live in
[choreography.md](choreography.md). The rule that matters: **read the previous section's
`data-animation` before choosing the next one.** Repeating an entrance is the single fastest
way to make an expensive page feel cheap.

### Step 7 — Verify

Serve over HTTP (`npm run dev`) — `file://` cannot fetch frames.

- [ ] Every section uses a different entrance than the one before it
- [ ] Counters reach their target and stop
- [ ] Marquee slides; CTA persists at the end
- [ ] Reduced-motion pass: no canvas built, poster visible, all copy readable
- [ ] Throttled to 4× CPU: scroll still tracks
- [ ] Mobile at 375px: sections stack, text is centered with a scrim, ≤550vh

## React / Vite port

Both projects in this workspace are React 19 + Vite, not vanilla. The technique is identical;
the lifecycle is not. Use `useFrameSequence` — see [react-port.md](react-port.md) for the hook,
which handles `gsap.context()` cleanup, `ResizeObserver`, and the reduced-motion bail-out.

Do **not** load GSAP/Lenis from a CDN in these projects — they are npm dependencies, and a CDN
copy will register a second, conflicting ScrollTrigger instance.

## AI-generated footage (Google Flow / Veo / Runway)

This is the intended pipeline here: `brand-dna-extractor` → `prompt-imagem` → GPT still per
section → `prompt-animacao` → Flow animates it → this skill turns the clip into scroll.

Flow output needs different handling than filmed footage:

- Clips come out **short (4–8s) and loop-hostile**. Extract at native fps; you will land near
  150 frames, which is the floor — do not decimate further.
- Generated video drifts: the last frame rarely matches the first. Never `loop` a scrubbed
  sequence; let it rest on the final frame.
- Flow renders 720p/1080p. **Do not upscale** — extract at the native width and let the padded
  cover handle the gap. Upscaled AI frames look plastic.
- Check for morphing artifacts around hands, paws and text before committing 300 frames.
  If a section morphs badly, regenerate the still rather than accepting the clip.
- Generated footage has no real motion blur. Keep `FRAME_SPEED` at the low end (1.8) — fast
  scrubbing over blur-free frames strobes.

## Anti-patterns

- **Cycling cards inside one pinned section** — each card gets too little scroll. Give every
  feature its own 8–10% range and its own entrance.
- **Pure cover (`scale 1.0`)** — clips into the header. **Pure contain** — leaves a border that
  never matches the page. Use the padded cover.
- **`FRAME_SPEED` < 1.8** — motion finishes after the copy has already left; feels sluggish.
- **Hero under 20% of scroll** — the first impression gets no room.
- **Frame sequence for ambience** — a looping `<video>` costs 300 KB and reads the same.
- **Baking copy into frames** — unreadable, untranslatable, invisible to search.
- **Scroll-jacking the page length** — 800vh is a budget, not a target. If a section has
  nothing to reveal, it does not get scroll.

## Troubleshooting

| Symptom | Cause |
|---|---|
| Frames never load | Served over `file://`, or the path is missing the leading `/` |
| Choppy scrub | Repainting unchanged frames — add the `i !== current` guard |
| White flash on entry | Loader hidden before phase-1 frames decoded |
| Soft/blurry canvas | `devicePixelRatio` not applied to the backing store |
| Scroll fights itself | GSAP loaded twice (npm + CDN), or Lenis not wired to `ScrollTrigger.update` |
| Killed tab on iOS | Frame set over ~8 MB, or full-width frames served to mobile |
| Counters freeze mid-count | `data-value` missing, or `snap` disagrees with `data-decimals` |
