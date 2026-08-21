# Section choreography

> **Escopo: só a página de canvas frame sequence desta skill.** Os cursos abaixo (`y: 50`,
> `y: 60`, `x: -80`) são o dobro dos que valem numa landing de capítulos, porque aqui a copy
> entra por cima de um canvas de tela cheia que já está se movendo — 26px de curso somem contra
> o frame. Numa página normal a dona da linguagem de motion é
> [landing-motion-expert](../landing-motion-expert/SKILL.md#linguagem-de-motion), e as duas
> tabelas não podem valer ao mesmo tempo na mesma página.

## Animation table

| Type | From | Duration | Ease |
|---|---|---|---|
| `fade-up` | `y: 50, opacity: 0` | 0.9 | `power3.out` |
| `slide-left` | `x: -80, opacity: 0` | 0.9 | `power3.out` |
| `slide-right` | `x: 80, opacity: 0` | 0.9 | `power3.out` |
| `scale-up` | `scale: 0.85, opacity: 0` | 1.0 | `power2.out` |
| `rotate-in` | `y: 40, rotation: 3, opacity: 0` | 0.9 | `power3.out` |
| `stagger-up` | `y: 60, opacity: 0` | 0.8 | `power3.out` |
| `clip-reveal` | `clipPath: inset(100% 0 0 0)` | 1.2 | `power4.inOut` |

Stagger is 0.10–0.15s between children, always in reading order:
label → heading → body → note → CTA.

## Section markup

```html
<section class="scroll-section section-content align-left"
         data-enter="22" data-leave="38" data-animation="slide-left">
  <div class="section-inner">
    <span class="section-label">002 / Estrutura</span>
    <h2 class="section-heading">Exames no local, resultado em minutos</h2>
    <p class="section-body">…</p>
  </div>
</section>
```

`data-enter` / `data-leave` are percentages of total scroll progress. Sections are positioned
absolutely at the midpoint of their range with `translateY(-50%)`.

## Building the timeline

```js
function setupSection(section) {
  const { animation: type, persist, enter, leave } = section.dataset;
  const children = section.querySelectorAll(
    ".section-label, .section-heading, .section-body, .section-note, .cta-button, .stat"
  );
  const from = {
    "fade-up":     { y: 50, opacity: 0, duration: 0.9, ease: "power3.out" },
    "slide-left":  { x: -80, opacity: 0, duration: 0.9, ease: "power3.out" },
    "slide-right": { x: 80, opacity: 0, duration: 0.9, ease: "power3.out" },
    "scale-up":    { scale: 0.85, opacity: 0, duration: 1.0, ease: "power2.out" },
    "rotate-in":   { y: 40, rotation: 3, opacity: 0, duration: 0.9, ease: "power3.out" },
    "stagger-up":  { y: 60, opacity: 0, duration: 0.8, ease: "power3.out" },
    "clip-reveal": { clipPath: "inset(100% 0 0 0)", opacity: 0, duration: 1.2, ease: "power4.inOut" },
  }[type];

  const tl = gsap.timeline({ paused: true })
    .from(children, { ...from, stagger: type === "slide-left" || type === "slide-right" ? 0.14 : 0.12 });

  const inRange = (p) => p >= enter / 100 && p <= leave / 100;
  let played = false;
  return (p) => {
    if (inRange(p) && !played) { tl.play(); played = true; }
    else if (!inRange(p) && played && persist !== "true") { tl.reverse(); played = false; }
  };
}
```

Drive every section's updater from **one** ScrollTrigger `onUpdate`, not one trigger each.
Dozens of triggers on the same scroller each recalculating start/end on resize is a real cost.

## Counters

```js
gsap.from(el, {
  textContent: 0, duration: 2, ease: "power1.out",
  snap: { textContent: decimals === 0 ? 1 : 0.01 },
  scrollTrigger: { trigger: el.closest(".scroll-section"), start: "top 70%",
                   toggleActions: "play none none reverse" },
});
```

Format the display separately from the tween value when the number needs a locale separator
(`70.000`, not `70000`) — tween a raw number, print through `toLocaleString("pt-BR")` in an
`onUpdate`.

## Marquee

```js
gsap.to(el.querySelector(".marquee-text"), {
  xPercent: parseFloat(el.dataset.scrollSpeed) || -25,
  ease: "none",
  scrollTrigger: { trigger: scrollContainer, start: "top top", end: "bottom bottom", scrub: true },
});
```

Duplicate the text node so the tail never reveals empty space, and set the wrapper
`aria-hidden="true"` — a marquee is decoration, and a screen reader reading a word twice is noise.

## Dark overlay for stats

```js
const FADE = 0.04;
// opacity ramps 0 → 0.9 across FADE before `enter`, holds, ramps back after `leave`
const opacity =
  p >= enter - FADE && p <= enter ? (p - (enter - FADE)) / FADE * 0.9 :
  p > enter && p < leave           ? 0.9 :
  p >= leave && p <= leave + FADE  ? 0.9 * (1 - (p - leave) / FADE) : 0;
```

## Hero handoff

```js
heroSection.style.opacity = Math.max(0, 1 - p * 15);          // hero clears fast
const wipe = Math.min(1, Math.max(0, (p - 0.01) / 0.06));
canvasWrap.style.clipPath = `circle(${wipe * 75}% at 50% 50%)`;
```

Other wipes when a circle is wrong for the brand:

| Shape | From | To |
|---|---|---|
| Left wipe | `inset(0 100% 0 0)` | `inset(0 0 0 0)` |
| Bottom wipe | `inset(100% 0 0 0)` | `inset(0 0 0 0)` |
| Split | `polygon(50% 0, 50% 0, 50% 100%, 50% 100%)` | `polygon(0 0, 100% 0, 100% 100%, 0 100%)` |

A circle wipe centered on a face reads as a spotlight and is worth using when the hero frame is
a person. On a product, prefer the split.
