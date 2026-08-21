# Padrões de motion de componente

Código para a stack real do projeto: React 19.2, TypeScript 5.9, GSAP 3.15, Lenis 1.3,
Tailwind 4.

Desde a 3.13 todos os antigos plugins de clube são gratuitos e vêm no pacote `gsap` —
`node_modules/gsap/` já traz `Flip.js`, `DrawSVGPlugin.js`, `MorphSVGPlugin.js`, `SplitText.js`.
Eles existem, mas nenhum está registrado: `src/lib/gsap.ts` só registra `ScrollTrigger`. Usar
qualquer outro exige `gsap.registerPlugin()` **naquele mesmo arquivo** — nunca em outro, ou o
plugin entra duas vezes no bundle. Nenhum dos padrões abaixo precisa de plugin.

Cada padrão traz o custo e o limite. Nenhum deles é obrigatório: consulte a tabela
"motion que informa vs motion que decora" no [SKILL.md](SKILL.md) antes de adicionar.

---

## 1. Card com tilt e lift

Rotação máxima de 5°. Acima de 8° o card lê como brinquedo e o texto dentro dele fica ilegível
nas bordas. Os eventos de ponteiro são coalescidos em um `requestAnimationFrame` — um mouse de
alta taxa emite vários `pointermove` por frame e só o último importa.

```ts
// src/hooks/useTilt.ts
import { useCallback, type RefCallback } from 'react'

import { gsap } from '../lib/gsap'

interface TiltOptions {
  /** Rotação máxima em graus. 4–5 em card de conteúdo, 6–8 só em card de imagem pura. */
  max?: number
  /** Distância da câmera em px. Menor = perspectiva mais dramática. */
  perspective?: number
  /** Elevação em px no hover. */
  lift?: number
}

export function useTilt({ max = 5, perspective = 900, lift = 4 }: TiltOptions = {}): RefCallback<HTMLElement> {
  return useCallback(
    (node: HTMLElement | null) => {
      if (!node) return
      if (!window.matchMedia('(hover: hover) and (pointer: fine)').matches) return
      if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return

      gsap.set(node, { transformPerspective: perspective })

      const rotX = gsap.quickTo(node, 'rotationX', { duration: 0.5, ease: 'power3.out' })
      const rotY = gsap.quickTo(node, 'rotationY', { duration: 0.5, ease: 'power3.out' })
      const rise = gsap.quickTo(node, 'y', { duration: 0.5, ease: 'power3.out' })

      let box: DOMRect | null = null
      let frame = 0
      let latest: PointerEvent | null = null

      const apply = () => {
        frame = 0
        const event = latest
        latest = null
        if (!event) return
        box ??= node.getBoundingClientRect()
        // -0.5 a +0.5 a partir do centro do card.
        const px = (event.clientX - box.left) / box.width - 0.5
        const py = (event.clientY - box.top) / box.height - 0.5
        rotY(px * max * 2)
        rotX(-py * max * 2)
      }

      const onEnter = () => {
        box = node.getBoundingClientRect()
        rise(-lift)
      }
      const onMove = (event: PointerEvent) => {
        latest = event
        if (!frame) frame = requestAnimationFrame(apply)
      }
      const settle = () => {
        if (frame) cancelAnimationFrame(frame)
        frame = 0
        latest = null
        box = null
        rotX(0)
        rotY(0)
        rise(0)
      }

      node.addEventListener('pointerenter', onEnter)
      node.addEventListener('pointermove', onMove)
      node.addEventListener('pointerleave', settle)
      node.addEventListener('pointercancel', settle)

      return () => {
        node.removeEventListener('pointerenter', onEnter)
        node.removeEventListener('pointermove', onMove)
        node.removeEventListener('pointerleave', settle)
        node.removeEventListener('pointercancel', settle)
        if (frame) cancelAnimationFrame(frame)
        gsap.killTweensOf(node)
        gsap.set(node, { clearProps: 'transform,transformPerspective' })
      }
    },
    [max, perspective, lift],
  )
}
```

O par de teclado não é o tilt — é o lift estático, no CSS do card:

```css
@layer components {
  .card {
    transition:
      box-shadow var(--dur-panel) var(--ease-out-quart),
      border-color var(--dur-panel) var(--ease-out-quart);
  }
  .card:focus-within {
    box-shadow: var(--shadow-lift);
    border-color: color-mix(in srgb, var(--color-ink) 25%, transparent);
  }
  /* Se camadas internas precisam flutuar acima da superfície. */
  .card--layered { transform-style: preserve-3d; }
  .card--layered > .card__media { transform: translateZ(24px); }
}
```

Custo: o tilt mantém o card em uma camada de composição enquanto o ponteiro estiver sobre ele.
Isso é aceitável para um card por vez. Nunca aplique `useTilt` a uma grade inteira de uma vez.

---

## 2. Menu com stagger, com saída de verdade

O problema do `PageChrome` hoje: `hidden={!menuOpen}` tira o sheet do render no mesmo frame em
que a saída deveria começar, então a saída nunca aparece. A correção é uma timeline pausada,
construída uma vez, tocada para frente ao abrir e revertida ao fechar — a saída roda a
`timeScale(1.6)` porque sair do caminho não precisa ser notado.

```tsx
import { useEffect, useRef } from 'react'

import { gsap } from '../../lib/gsap'

interface ChapterSheetProps {
  id: string
  open: boolean
  /** Chamado quando a animação de saída termina — não quando `open` vira false. */
  onClosed: () => void
}

export function ChapterSheet({ id, open, onClosed }: ChapterSheetProps) {
  const sheetRef = useRef<HTMLDivElement>(null)
  const timeline = useRef<gsap.core.Timeline | null>(null)
  /* Ref, não state: só serve para não reverter uma timeline que nunca tocou.
     Como state, entraria nas deps do efeito abaixo e dispararia um `play()` extra. */
  const everOpened = useRef(false)
  /* O callback muda de identidade a cada render do pai; a timeline lê a versão
     atual em vez de o efeito re-rodar por causa dela. */
  const closedRef = useRef(onClosed)
  useEffect(() => {
    closedRef.current = onClosed
  }, [onClosed])

  /* Construída uma única vez. gsap.matchMedia reverte sozinha se a preferência mudar. */
  useEffect(() => {
    const sheet = sheetRef.current
    if (!sheet) return

    /* Escopada no sheet: `'[data-menu-item]'` resolve dentro dele e nunca alcança
       outro menu da página. É a mesma convenção de `useChapterTimeline.ts`. */
    const mm = gsap.matchMedia(sheet)

    mm.add(
      {
        motion: '(prefers-reduced-motion: no-preference)',
        still: '(prefers-reduced-motion: reduce)',
      },
      (context) => {
        const { still } = context.conditions as { motion: boolean; still: boolean }

        const tl = gsap.timeline({ paused: true })
        tl.fromTo(
          sheet,
          { autoAlpha: 0 },
          { autoAlpha: 1, duration: still ? 0.15 : 0.3, ease: 'none' },
        )

        if (!still) {
          // autoAlpha, não opacity: no zero o item sai do hit-testing e da ordem de tabulação.
          tl.fromTo(
            '[data-menu-item]',
            { autoAlpha: 0, y: 18 },
            // 0.3 é o teto de entrada de sheet na tabela do SKILL.md (250–350ms). Com
            // stagger de 0.06 e seis itens, o último chega em 0.66s — já no limite.
            { autoAlpha: 1, y: 0, duration: 0.3, stagger: 0.06, ease: 'power4.out' },
            0.06,
          )
        }

        timeline.current = tl
        return () => {
          timeline.current = null
        }
      },
    )

    return () => mm.revert()
  }, [])

  useEffect(() => {
    const tl = timeline.current
    if (!tl) return
    if (open) {
      everOpened.current = true
      tl.timeScale(1).play()
      return
    }
    if (!everOpened.current) return
    // Saída 1,6× mais rápida que a entrada: 300ms viram ~190ms.
    tl.eventCallback('onReverseComplete', () => closedRef.current())
    tl.timeScale(1.6).reverse()
  }, [open])

  return (
    <div
      id={id}
      ref={sheetRef}
      role="dialog"
      aria-modal="true"
      aria-label="Capítulos"
      /* `inert` retira do fluxo de foco e da árvore de acessibilidade sem tirar do
         render — a saída ainda pode animar. `autoAlpha` do GSAP escreve o
         visibility: hidden quando a opacidade chega a 0. */
      inert={!open}
      className="bg-ink invisible fixed inset-0 z-50 flex flex-col lg:hidden"
    >
      {/* itens marcados com data-menu-item */}
    </div>
  )
}
```

`inert` como booleano é React 19 — no 18 era preciso escrever `inert=""`. O `id` volta porque o
botão que abre o sheet aponta para ele com `aria-controls`.

Continua valendo tudo o que o `PageChrome` já faz e não deve ser perdido: `stop()` do Lenis
enquanto o sheet está aberto, `Escape` para fechar, `Tab` circulando entre o primeiro e o
último focável, e o foco devolvido ao botão que abriu.

---

## 3. Botão em carregamento

O giro precisa de uma animação própria. `animate-spin` + `[animation-duration:800ms]` é frágil:
`animate-spin` escreve a shorthand `animation`, que reseta `animation-duration`, e quem vence
depende da ordem em que o Tailwind emite as duas classes. `--animate-*` é namespace de tema, então
a animação inteira vira um token e o problema some:

```css
@theme {
  --animate-spinner: spinner 800ms linear infinite;

  /* Dentro do @theme, não fora: é assim que o Tailwind 4 amarra o keyframe ao
     utilitário `animate-spinner` e o emite junto com ele. */
  @keyframes spinner {
    to { rotate: 360deg; }
  }
}
```

```tsx
import { useEffect, useState } from 'react'

interface SubmitProps {
  pending: boolean
  children: React.ReactNode
  onClick?: () => void
}

/** Envio que resolve antes disso não merece spinner: o piscar cansa mais que a espera. */
const SPINNER_DELAY = 200

export function Submit({ pending, children, onClick }: SubmitProps) {
  const [showSpinner, setShowSpinner] = useState(false)

  useEffect(() => {
    if (!pending) {
      setShowSpinner(false)
      return
    }
    const t = window.setTimeout(() => setShowSpinner(true), SPINNER_DELAY)
    return () => window.clearTimeout(t)
  }, [pending])

  return (
    <>
      <button
        type="submit"
        onClick={onClick}
        disabled={pending}
        aria-busy={pending}
        className="relative inline-flex min-h-14 items-center justify-center rounded-pill bg-ink px-8 text-white
                   transition-[background-color,box-shadow,transform] duration-(--dur-hover) ease-out-quart
                   hover:shadow-lift hover:-translate-y-px
                   focus-visible:shadow-lift focus-visible:-translate-y-px
                   active:scale-[0.98] active:duration-(--dur-press)
                   disabled:cursor-not-allowed disabled:translate-y-0 disabled:opacity-45
                   motion-reduce:transition-none motion-reduce:active:scale-100"
      >
        {/* O label sai de vista mas continua ocupando a caixa: a largura nunca muda.
            Amarrado ao spinner, não a `pending`: um envio de 50ms apagaria e
            reacenderia o texto — o mesmo piscar que o atraso existe para evitar. */}
        <span className={`transition-opacity duration-150 ${showSpinner ? 'opacity-35' : 'opacity-100'}`}>
          {children}
        </span>
        {showSpinner ? (
          <span
            aria-hidden="true"
            data-motion-safe="spin"
            /* `inset-0 m-auto` centra sem depender de como o navegador resolve a
               posição estática de um absoluto dentro de um flex container. */
            className="animate-spinner absolute inset-0 m-auto size-4 rounded-full
                       border-2 border-white/30 border-t-white"
          />
        ) : null}
      </button>
      {/*
         Fora do botão, de propósito. Um `sr-only` lá dentro entraria no nome
         acessível — o leitor anunciaria "Enviar Enviando" — e uma live region
         cujo texto é parte do nome do controle não é anunciada de forma
         confiável. O anúncio é texto; o giro é invisível para o leitor de tela.
       */}
      <span role="status" className="sr-only">
        {pending ? 'Enviando' : ''}
      </span>
    </>
  )
}
```

O `data-motion-safe="spin"` é o que mantém o giro sob `prefers-reduced-motion`: o bloco global
de `src/styles/index.css` zera `animation-duration` e força `animation-iteration-count: 1` em
`*`, o que pararia o spinner no primeiro frame. O opt-in está no [SKILL.md](SKILL.md) — 1200ms
por volta em vez de 800ms, mais lento mas nunca parado. É a única prova de que um processo
indeterminado ainda está vivo.

---

## 4. Skeleton

Regra de forma: o skeleton tem a altura exata do conteúdo que vai substituir. Se não tiver, a
troca causa salto de layout — que é precisamente o que o skeleton existia para evitar.

```css
@layer components {
  .skeleton {
    position: relative;
    overflow: hidden;
    border-radius: 6px;
    background-color: color-mix(in srgb, var(--color-line) 70%, transparent);
  }

  /* Um pseudo-elemento transladado é composto. Animar `background-position`
     diretamente repinta a caixa inteira a cada frame — aceitável em blocos
     pequenos, caro numa tela cheia de skeletons. */
  .skeleton::after {
    content: '';
    position: absolute;
    inset: 0;
    background-image: linear-gradient(90deg, transparent, rgb(255 255 255 / 0.6), transparent);
    transform: translateX(-100%);
    animation: skeleton 1400ms linear infinite;
    /* Sem `will-change`: uma animação de `transform` já roda no compositor, e uma
       tela de skeletons são dezenas de camadas permanentes de GPU sem ganho. */
  }

  @keyframes skeleton {
    to { transform: translateX(100%); }
  }
}

@media (prefers-reduced-motion: reduce) {
  /* O bloco cinza continua comunicando "carregando"; o brilho não acrescenta nada. */
  .skeleton::after { animation: none; opacity: 0; }
}
```

```tsx
export function CardSkeleton() {
  return (
    <div aria-hidden="true" className="rounded-card border border-line p-6">
      <div className="skeleton h-40 w-full rounded-card" />
      <div className="skeleton mt-5 h-4 w-2/3" />
      <div className="skeleton mt-3 h-3 w-full" />
      <div className="skeleton mt-2 h-3 w-4/5" />
    </div>
  )
}
```

O container que troca skeleton por conteúdo leva `aria-busy="true"` enquanto carrega. Os
skeletons em si são `aria-hidden`: repetir "carregando" quatro vezes não ajuda ninguém.

---

## 5. Toast

`@starting-style` dá a entrada sem estado de montagem em React. Se o alvo incluir navegadores
sem suporte, o fallback é um `useState` que vira `true` dentro de um `requestAnimationFrame`
depois da montagem — o efeito é o mesmo, o código é maior.

```css
@layer components {
  .toast {
    transition:
      opacity var(--dur-toast) var(--ease-out-quint),
      transform var(--dur-toast) var(--ease-out-quint);
  }

  @starting-style {
    .toast { opacity: 0; transform: translateY(12px); }
  }

  .toast[data-leaving='true'] {
    opacity: 0;
    transform: translateY(6px);
    transition-duration: 150ms; /* saída mais curta que a entrada */
  }
}

@media (prefers-reduced-motion: reduce) {
  .toast,
  .toast[data-leaving='true'] { transform: none; }
}
```

```tsx
import { useCallback, useEffect, useRef, useState } from 'react'

const DWELL = 4000
const DWELL_WITH_ACTION = 5500
const EXIT = 150
/** Depois de o ponteiro sair, o toast não recomeça a contagem cheia. */
const RESUME = 2000

interface ToastProps {
  id: string
  tone?: 'info' | 'success' | 'error'
  children: React.ReactNode
  action?: React.ReactNode
  onDismiss: (id: string) => void
}

export function Toast({ id, tone = 'info', children, action, onDismiss }: ToastProps) {
  const [leaving, setLeaving] = useState(false)
  const timer = useRef(0)

  const schedule = useCallback((ms: number) => {
    window.clearTimeout(timer.current)
    timer.current = window.setTimeout(() => setLeaving(true), ms)
  }, [])

  /*
     Booleano, não o próprio `action`. `action` é ReactNode: se for JSX, o pai
     cria um objeto novo a cada render, a dependência muda sempre, o efeito
     re-roda, o timer reinicia — e o toast nunca some.
   */
  const hasAction = Boolean(action)

  useEffect(() => {
    schedule(hasAction ? DWELL_WITH_ACTION : DWELL)
    return () => window.clearTimeout(timer.current)
  }, [hasAction, schedule])

  useEffect(() => {
    if (!leaving) return
    const t = window.setTimeout(() => onDismiss(id), EXIT)
    return () => window.clearTimeout(t)
  }, [leaving, id, onDismiss])

  return (
    <div
      className="toast bg-ink shadow-lift flex items-center gap-4 rounded-card px-5 py-4 text-white"
      data-leaving={leaving}
      /* Erro interrompe; o resto espera a próxima pausa do leitor de tela. */
      role={tone === 'error' ? 'alert' : 'status'}
      aria-live={tone === 'error' ? 'assertive' : 'polite'}
      /* O timer para enquanto alguém está lendo — com o ponteiro ou com o teclado. */
      onPointerEnter={() => window.clearTimeout(timer.current)}
      onPointerLeave={() => schedule(RESUME)}
      onFocusCapture={() => window.clearTimeout(timer.current)}
      onBlurCapture={() => schedule(RESUME)}
    >
      <span className="text-[0.9375rem]">{children}</span>
      {action}
    </div>
  )
}
```

Limites: no máximo 3 toasts visíveis; o quarto descarta o mais antigo. Toast nunca recebe foco
programático — quem precisa de foco é diálogo, não notificação.

Uma live region só é anunciada se já estiver no DOM quando o texto muda. Um toast que **monta**
com o texto dentro chega tarde demais para vários leitores de tela. Renderize o container
`role="status"` vazio no layout desde o início e injete o texto nele; a caixa visual pode montar
e desmontar à vontade.

---

## 6. Hover com clip-path

`clip-path` só interpola entre a mesma função com o mesmo número de argumentos: `inset()` →
`inset()` anima, `circle()` → `inset()` corta seco. E é operação de pintura, não de composição:
mantenha a área pequena — um botão, um selo, uma miniatura. Numa seção inteira, use
`transform: scaleY()` em um pseudo-elemento com `transform-origin: bottom`.

```css
@layer components {
  .fill-up {
    position: relative;
    isolation: isolate;
    /* A confirmação do hover é a cor do texto, e ela respeita o teto de 250ms.
       O varrer atrás dela pode ser mais lento — ninguém está esperando por ele. */
    transition: color var(--dur-hover) var(--ease-out-quart);
  }

  .fill-up::before {
    content: '';
    position: absolute;
    inset: 0;
    z-index: -1;
    border-radius: inherit;
    background-color: var(--color-rose);
    clip-path: inset(100% 0 0 0);
    transition: clip-path 260ms var(--ease-out-quint);
  }

  @media (hover: hover) and (pointer: fine) {
    .fill-up:hover { color: var(--color-canvas); }
    .fill-up:hover::before { clip-path: inset(0 0 0 0); }
  }

  /* O par de teclado é obrigatório e idêntico. */
  .fill-up:focus-visible { color: var(--color-canvas); }
  .fill-up:focus-visible::before { clip-path: inset(0 0 0 0); }
}

@media (prefers-reduced-motion: reduce) {
  /* O preenchimento vira troca de cor: a informação permanece, o varrer some. */
  .fill-up::before { clip-path: inset(0 0 0 0); opacity: 0; transition: opacity 150ms; }
  .fill-up:is(:hover, :focus-visible)::before { opacity: 1; }
}
```

---

## 7. Sublinhado que cresce

Uma única propriedade animada (`background-size`). A posição troca sem transição: no hover ela
vai para a esquerda e o traço cresce a partir dali; ao sair volta para a direita e o traço
recolhe naquele sentido. É o detalhe que separa um sublinhado que entra e sai do que apenas
pisca.

```css
@layer components {
  .link-underline {
    padding-bottom: 2px;
    background-image: linear-gradient(currentColor, currentColor);
    background-repeat: no-repeat;
    background-size: 0% 1px;
    background-position: 100% 100%;
    transition: background-size 220ms var(--ease-out-quart);
  }

  @media (hover: hover) and (pointer: fine) {
    .link-underline:hover { background-size: 100% 1px; background-position: 0 100%; }
  }

  .link-underline:focus-visible { background-size: 100% 1px; background-position: 0 100%; }
}

@media (prefers-reduced-motion: reduce) {
  /* Links dentro de texto corrido precisam do sublinhado desde o repouso. */
  .link-underline { background-size: 100% 1px; background-position: 0 100%; }
}
```

Se o link estiver dentro de um parágrafo, ele já deve nascer sublinhado — o sublinhado que só
aparece no hover é aceitável em navegação, não em texto corrido, onde ele é o único sinal de
que a palavra é clicável.

---

## 8. Accordion

`height: auto` não é um número: sem valor inicial e final numéricos o navegador não interpola e
o painel salta. `grid-template-rows: 0fr → 1fr` interpola, e o navegador resolve a altura real
do conteúdo a cada frame.

```css
@layer components {
  .accordion__panel {
    display: grid;
    grid-template-rows: 0fr;
    transition: grid-template-rows var(--dur-panel) var(--ease-out-quint);
  }

  .accordion__panel > * { overflow: hidden; }

  .accordion__panel[data-open='true'] { grid-template-rows: 1fr; }
}
```

Custo: é layout a cada frame, não composição. Aceitável para um painel de cada vez, caro para
uma lista inteira abrindo junto. O gatilho é um `<button>` com `aria-expanded` e
`aria-controls` — nunca uma `<div>` com `onClick`.

---

## 9. Cursor customizado

Só faz sentido quando a página tem superfícies grandes de mídia. Nunca aplique `cursor: none`
globalmente: isso remove o cursor do sistema junto com o tamanho e o contraste que o usuário
configurou na acessibilidade do SO, e deixa campos de texto sem I-beam.

```tsx
// src/components/ui/Cursor.tsx
import { useEffect, useRef } from 'react'

import { gsap } from '../../lib/gsap'
import { useMediaQuery, useReducedMotion } from '../../hooks/useMediaQuery'

export function Cursor() {
  const dot = useRef<HTMLDivElement>(null)
  const fine = useMediaQuery('(hover: hover) and (pointer: fine)')
  const still = useReducedMotion()

  useEffect(() => {
    const node = dot.current
    if (!node || !fine || still) return

    /* xPercent/yPercent centram o ponto sem disputar o transform com o Tailwind:
       o GSAP escreve a propriedade `transform` inteira e sobrescreveria qualquer
       classe -translate-*. */
    gsap.set(node, { xPercent: -50, yPercent: -50, opacity: 0 })

    const moveX = gsap.quickTo(node, 'x', { duration: 0.28, ease: 'power3.out' })
    const moveY = gsap.quickTo(node, 'y', { duration: 0.28, ease: 'power3.out' })
    const scaleTo = gsap.quickTo(node, 'scale', { duration: 0.25, ease: 'power2.out' })

    let visible = false
    const show = () => {
      if (visible) return
      visible = true
      gsap.to(node, { opacity: 1, duration: 0.15 })
    }
    const hide = () => {
      visible = false
      gsap.to(node, { opacity: 0, duration: 0.15 })
    }

    const onMove = (event: PointerEvent) => {
      // O primeiro movimento é o que revela o ponto. Esperar por um evento de
      // entrada deixaria o cursor invisível para sempre no caso mais comum: a
      // página carrega com o mouse já dentro da janela, e nenhuma entrada ocorre.
      show()
      moveX(event.clientX)
      moveY(event.clientY)
      const target = event.target instanceof Element ? event.target.closest('[data-cursor]') : null
      node.dataset.state = target?.getAttribute('data-cursor') ?? 'default'
      scaleTo(target ? 2.6 : 1)
    }

    // `pointerout` com `relatedTarget` nulo é o ponteiro saindo da janela.
    // `pointerleave` no `document` não serve: enter/leave não borbulham.
    const onOut = (event: PointerEvent) => {
      if (!event.relatedTarget) hide()
    }

    window.addEventListener('pointermove', onMove, { passive: true })
    window.addEventListener('pointerout', onOut)
    window.addEventListener('blur', hide)

    return () => {
      window.removeEventListener('pointermove', onMove)
      window.removeEventListener('pointerout', onOut)
      window.removeEventListener('blur', hide)
      gsap.killTweensOf(node)
    }
  }, [fine, still])

  if (!fine || still) return null

  return (
    <div
      ref={dot}
      aria-hidden="true"
      className="pointer-events-none fixed top-0 left-0 z-100 size-3 rounded-full bg-white mix-blend-difference"
    />
  )
}
```

```css
/* Escopo mínimo: só as superfícies que declaram um estado de cursor. */
[data-cursor] { cursor: none; }

/*
   Controles nativos devolvem o cursor do sistema mesmo dentro dessas superfícies.
   Valores explícitos, não `revert`: `cursor` é herdada, e `revert` num elemento
   que a folha do navegador não declara cai de volta no valor herdado — que aqui
   é `none`. O botão continuaria sem cursor.
 */
[data-cursor] :is(a[href], button, select, [role='button']) { cursor: pointer; }
[data-cursor] :is(input, textarea) { cursor: auto; }
```

O cursor é decoração pura: não substitui nenhum estado de hover. O elemento embaixo continua
precisando da sua própria mudança de cor, borda ou sombra — quem navega por teclado nunca verá
o ponto.

---

## 10. Botão magnético

O hook existente (`src/hooks/useMagnetic.ts`) mede o elemento dentro de `pointermove`.
`getBoundingClientRect()` ali força style + layout síncronos a cada evento, e um mouse de alta
taxa emite mais de um evento por frame. A versão abaixo mede na entrada, coalesce os eventos em
um `requestAnimationFrame`, adiciona o press e limpa o transform ao desmontar.

A assinatura muda de `useMagnetic(strength)` para `useMagnetic({ strength, press })`.
`Action.tsx` chama sem argumento e não quebra; toda chamada com número posicional precisa virar
objeto na mesma edição.

```ts
// src/hooks/useMagnetic.ts
import { useCallback, type RefCallback } from 'react'

import { gsap } from '../lib/gsap'

interface MagneticOptions {
  /** Deslocamento máximo em px. 4–8: acima disso o controle vira brinquedo. */
  strength?: number
  /** Redução de escala no press. 0 desliga. */
  press?: number
}

/**
 * Um controle que se inclina de leve na direção do cursor.
 *
 * Ponteiro grosso e visitante com reduced-motion não recebem nada: não há hover
 * a que responder, e não há movimento a gastar. O equivalente de teclado não
 * está aqui — é o par `focus-visible` no CSS do próprio controle.
 */
export function useMagnetic({ strength = 5, press = 0.98 }: MagneticOptions = {}): RefCallback<HTMLElement> {
  return useCallback(
    (node: HTMLElement | null) => {
      if (!node) return
      if (!window.matchMedia('(hover: hover) and (pointer: fine)').matches) return
      if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return

      const moveX = gsap.quickTo(node, 'x', { duration: 0.45, ease: 'power3.out' })
      const moveY = gsap.quickTo(node, 'y', { duration: 0.45, ease: 'power3.out' })
      const scaleTo = gsap.quickTo(node, 'scale', { duration: 0.12, ease: 'power2.out' })
      const clamp = gsap.utils.clamp(-strength, strength)

      // O retângulo só muda quando a página rola ou muda de tamanho. Medi-lo por
      // evento de ponteiro custa um reflow por evento.
      let box: DOMRect | null = null
      let frame = 0
      let latest: PointerEvent | null = null

      const apply = () => {
        frame = 0
        const event = latest
        latest = null
        if (!event) return
        box ??= node.getBoundingClientRect()
        moveX(clamp((event.clientX - (box.left + box.width / 2)) * 0.22))
        moveY(clamp((event.clientY - (box.top + box.height / 2)) * 0.3))
      }

      const onEnter = () => {
        box = node.getBoundingClientRect()
      }
      const onMove = (event: PointerEvent) => {
        latest = event
        if (!frame) frame = requestAnimationFrame(apply)
      }
      const settle = () => {
        if (frame) cancelAnimationFrame(frame)
        frame = 0
        latest = null
        box = null
        moveX(0)
        moveY(0)
        scaleTo(1)
      }
      const onDown = () => press && scaleTo(press)
      const onUp = () => press && scaleTo(1)
      const invalidate = () => {
        box = null
      }

      node.addEventListener('pointerenter', onEnter)
      node.addEventListener('pointermove', onMove)
      node.addEventListener('pointerleave', settle)
      node.addEventListener('pointercancel', settle)
      node.addEventListener('pointerdown', onDown)
      node.addEventListener('pointerup', onUp)
      node.addEventListener('blur', settle)
      window.addEventListener('scroll', invalidate, { passive: true })
      window.addEventListener('resize', invalidate)

      // React 19 chama este retorno quando o elemento se desprende.
      return () => {
        node.removeEventListener('pointerenter', onEnter)
        node.removeEventListener('pointermove', onMove)
        node.removeEventListener('pointerleave', settle)
        node.removeEventListener('pointercancel', settle)
        node.removeEventListener('pointerdown', onDown)
        node.removeEventListener('pointerup', onUp)
        node.removeEventListener('blur', settle)
        window.removeEventListener('scroll', invalidate)
        window.removeEventListener('resize', invalidate)
        if (frame) cancelAnimationFrame(frame)
        gsap.killTweensOf(node)
        gsap.set(node, { clearProps: 'transform' })
      }
    },
    [strength, press],
  )
}
```

O `press` fica no hook e não numa classe `active:scale-[0.98]` por uma razão mecânica: o GSAP
escreve a propriedade `transform` inteira, inline. Enquanto o magnético estiver ativo, qualquer
`scale`, `translate` ou `rotate` vindo do Tailwind é sobrescrito e simplesmente não acontece.
Um controle tem um dono de `transform` — aqui é o GSAP.
