# React / Vite port

Same technique, different lifecycle. Everything runs inside `gsap.context()` so a single
`revert()` tears down triggers, tweens and inline styles on unmount — React 19 Strict Mode
double-invokes effects, and a page without this will register every ScrollTrigger twice.

## The hook

```tsx
import { useEffect, useRef, useState } from 'react'
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

import { useReducedMotion } from '../hooks/useMediaQuery'

interface Options {
  /** Public path prefix, e.g. `/frames/hero`. */
  path: string
  count: number
  /** 1.8–2.2. Higher finishes the motion earlier in the scroll. */
  speed?: number
  /** 0.82–0.90. Leaves breathing room so the frame never clips the header. */
  imageScale?: number
}

export function useFrameSequence(
  scroller: React.RefObject<HTMLElement | null>,
  { path, count, speed = 2, imageScale = 0.85 }: Options,
) {
  const canvasRef = useRef<HTMLCanvasElement>(null)
  const [ready, setReady] = useState(false)
  const reducedMotion = useReducedMotion()

  useEffect(() => {
    // Reduced motion never pays for the frames — the poster carries the section.
    if (reducedMotion) return

    const canvas = canvasRef.current
    const node = scroller.current
    if (!canvas || !node) return

    const ctx2d = canvas.getContext('2d', { alpha: false })
    if (!ctx2d) return

    const frames: (HTMLImageElement | undefined)[] = new Array(count)
    let current = -1
    let bg = '#ffffff'
    let disposed = false

    const src = (i: number) =>
      `${path}/frame_${String(i + 1).padStart(4, '0')}.webp`

    const load = (i: number) =>
      new Promise<void>((resolve) => {
        const img = new Image()
        img.onload = img.onerror = () => {
          frames[i] = img
          resolve()
        }
        img.src = src(i)
      })

    /* Sample the corners so the padded border blends into the page. */
    const sampleBg = (img: HTMLImageElement) => {
      const probe = document.createElement('canvas')
      probe.width = probe.height = 8
      const p = probe.getContext('2d')
      if (!p) return
      p.drawImage(img, 0, 0, 8, 8)
      const [r, g, b] = p.getImageData(0, 0, 1, 1).data
      bg = `rgb(${r},${g},${b})`
    }

    const draw = (i: number) => {
      const img = frames[i]
      if (!img || disposed) return
      const cw = canvas.width
      const ch = canvas.height
      const scale = Math.max(cw / img.naturalWidth, ch / img.naturalHeight) * imageScale
      const dw = img.naturalWidth * scale
      const dh = img.naturalHeight * scale
      ctx2d.fillStyle = bg
      ctx2d.fillRect(0, 0, cw, ch)
      ctx2d.drawImage(img, (cw - dw) / 2, (ch - dh) / 2, dw, dh)
    }

    /* Backing store follows the device, not the CSS box, or retina renders soft. */
    const resize = () => {
      const dpr = Math.min(window.devicePixelRatio || 1, 2)
      canvas.width = canvas.clientWidth * dpr
      canvas.height = canvas.clientHeight * dpr
      if (current >= 0) draw(current)
    }

    const observer = new ResizeObserver(resize)
    observer.observe(canvas)

    const gsapCtx = gsap.context(() => {
      ScrollTrigger.create({
        trigger: node,
        start: 'top top',
        end: 'bottom bottom',
        scrub: true,
        onUpdate: (self) => {
          const accelerated = Math.min(self.progress * speed, 1)
          const i = Math.min(Math.floor(accelerated * count), count - 1)
          // Repainting an unchanged frame is the usual cause of a 40fps scroll.
          if (i === current) return
          current = i
          requestAnimationFrame(() => draw(i))
        },
      })
    }, node)

    /* Phase 1 unblocks the section; phase 2 streams in behind the user. */
    void (async () => {
      const head = Math.min(10, count)
      await Promise.all(Array.from({ length: head }, (_, i) => load(i)))
      if (disposed) return
      if (frames[0]) sampleBg(frames[0])
      resize()
      draw(0)
      setReady(true)
      ScrollTrigger.refresh()

      for (let i = head; i < count; i += 1) {
        if (disposed) return
        await load(i)
        if (i % 20 === 0 && frames[i]) sampleBg(frames[i]!)
      }
    })()

    return () => {
      disposed = true
      observer.disconnect()
      gsapCtx.revert()
      frames.length = 0
    }
  }, [count, imageScale, path, reducedMotion, scroller, speed])

  return { canvasRef, ready, reducedMotion }
}
```

## Using it

```tsx
export function ChapterTransformation() {
  const scroller = useRef<HTMLDivElement>(null)
  const { canvasRef, ready, reducedMotion } = useFrameSequence(scroller, {
    path: '/frames/transformacao',
    count: 240,
  })

  if (reducedMotion) {
    return <Picture image={media.transformacao.poster} alt="…" sizes="100vw" />
  }

  return (
    <div ref={scroller} className="relative h-[800vh]">
      <div className="sticky top-0 h-screen">
        <canvas
          ref={canvasRef}
          role="img"
          aria-label="A tosa completa, do começo ao fim"
          className="h-full w-full transition-opacity duration-500"
          style={{ opacity: ready ? 1 : 0 }}
        />
      </div>
    </div>
  )
}
```

## Project rules

- GSAP and Lenis come from `package.json`, never a CDN — a second copy registers a second
  ScrollTrigger and the two fight over the same scroller.
- Register plugins once, in `src/lib/gsap.ts`, not per component.
- Frames live in `public/frames/<section>/` so Vite serves them unhashed and the numbered
  path stays predictable. Do not import them through the bundler.
- Generate frames from `scripts/prepare-assets.mjs` alongside the existing image/video
  derivatives, using the `ffmpeg-static` binary already in devDependencies. Emit the frame
  count into `src/generated/media.ts` so the component never hardcodes it.
- `ScrollTrigger.refresh()` after phase-1 load — the canvas changes the document height.
