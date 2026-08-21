# Receitas de scroll

React 19 + GSAP 3.15 + Lenis 1.3 + Tailwind 4. Toda receita roda dentro de
`gsap.matchMedia(scope)` e é desmontada com `mm.revert()` — sem isso, o Strict Mode do React 19
monta o efeito duas vezes e cada ScrollTrigger existe em dobro.

Os helpers `gsap`, `ScrollTrigger`, `q`, `seal`, `words`, `lineGroups` e `typographyReady` vêm
de `src/lib/gsap.ts` — o único módulo que importa `gsap/ScrollTrigger`. `useChapterTimeline` e
`useChapterReveals` já implementam as receitas 1 e 3 para capítulos; as versões abaixo são a
forma genérica, para quando o alvo não é um capítulo.

Os `data-*` das receitas não colidem entre si de propósito: `[data-bg]`/`[data-mid]`/
`[data-front]` são camadas de parallax dentro de uma seção, `[data-page-bg]` é a cor que uma
seção entrega ao documento. Se duas receitas dividirem o mesmo atributo na mesma página, uma
vai animar o alvo da outra.

---

## 1. Reveal escalonado

**Quando usar.** Copy entrando numa seção: eyebrow → título → parágrafo → CTA. É a receita mais
usada da página e a única que nunca leva scrub.

**Por que sem scrub.** Preso ao scroll, o ritmo da frase vira o ritmo do gesto: um flick de
trackpad nasce e enterra a sentença dentro do mesmo movimento, e parar no meio congela a copy
com buracos. Aqui o scroll só dá o start; a linha sobe no relógio dela, sempre na mesma
velocidade legível, e fica.

```tsx
// src/hooks/useReveal.ts
import { useLayoutEffect, type RefObject } from 'react'
import { gsap, q } from '../lib/gsap'

// Os três números são da linguagem de motion do landing-motion-expert — esta receita os
// consome, não os redefine. Trocar um deles aqui desalinha esta seção das outras seis.
const DURATION = 0.9 // token `slow`
const STAGGER = 0.12 // faixa de 2–3 itens; 4–8 itens caem em 0.06–0.08
const TRAVEL = 24 // px. Bloco de copy anda 22–26; acima disso a frase sai de onde vai ser lida.

export function useReveal(ref: RefObject<HTMLElement | null>, selector = '[data-reveal]'): void {
  useLayoutEffect(() => {
    const scope = ref.current
    if (!scope) return

    const mm = gsap.matchMedia(scope)

    mm.add('(prefers-reduced-motion: no-preference)', () => {
      const items = q(scope, selector)
      if (items.length === 0) return

      // Estado inicial explícito, escrito antes do primeiro paint. Um .from()
      // sem isto grava o estado atual como destino e quebra na reconstrução.
      gsap.set(items, { autoAlpha: 0, y: TRAVEL })

      gsap.to(items, {
        autoAlpha: 1,
        y: 0,
        duration: DURATION,
        ease: 'expo.out',
        stagger: STAGGER,
        scrollTrigger: {
          trigger: scope,
          start: 'top 80%',
          // Uma via só: rolar de volta não pode desescrever uma frase, e quem
          // chega por âncora não pode encontrar copy que nunca foi acionada.
          once: true,
        },
      })
    })

    return () => mm.revert()
  }, [ref, selector])
}
```

**Números.** `start: 'top 80%'` — desta skill · `duration 0.9`, `stagger 0.12`, `y 24`,
`ease expo.out` — do
[landing-motion-expert](../landing-motion-expert/SKILL.md#linguagem-de-motion).
Ordem de leitura sempre: label → heading → body → nota → CTA.

**Custo.** Um ScrollTrigger por grupo. `autoAlpha` escreve `visibility` junto com `opacity`, o
que tira o elemento do hit-testing enquanto invisível.

---

## 2. Parallax por camada

**Quando usar.** Dar profundidade a uma mídia de fundo com uma ou duas camadas por cima. Nunca
em texto de corpo — o deslocamento relativo atrapalha a leitura.

**A regra do overscan.** A camada de fundo se desloca `2 × depth` no total. Ela precisa ser
maior que o quadro nessa mesma proporção, ou a borda aparece. `depth: 12` exige
`height: 124%` (ou `scale(1.24)`) na mídia.

```tsx
// src/hooks/useParallax.ts
import { useLayoutEffect, type RefObject } from 'react'
import { gsap, q } from '../lib/gsap'

export interface Layer {
  select: string
  /** % de deslocamento para cada lado. 12 = fundo, 6 = meio, 2 = primeiro plano. */
  depth: number
}

export function useParallax(ref: RefObject<HTMLElement | null>, layers: readonly Layer[]): void {
  useLayoutEffect(() => {
    const scope = ref.current
    if (!scope) return

    const mm = gsap.matchMedia(scope)

    mm.add(
      {
        desktop: '(min-width: 768px) and (prefers-reduced-motion: no-preference)',
        mobile: '(max-width: 767px) and (prefers-reduced-motion: no-preference)',
      },
      (context) => {
        const { desktop } = context.conditions as { desktop: boolean; mobile: boolean }
        // Metade do travel no celular: a tela é curta, o overscan custa mais
        // pixels decodificados, e o efeito lê igual.
        const factor = desktop ? 1 : 0.5

        const tl = gsap.timeline({
          defaults: { ease: 'none' },
          scrollTrigger: {
            trigger: scope,
            // Do primeiro ao último pixel visível: o parallax cobre toda a
            // travessia do elemento pela tela.
            start: 'top bottom',
            end: 'bottom top',
            scrub: true,
            invalidateOnRefresh: true,
          },
        })

        for (const layer of layers) {
          const d = layer.depth * factor
          // yPercent é relativo ao próprio elemento: nada precisa ser medido, e
          // o valor continua correto depois de qualquer resize.
          tl.fromTo(q(scope, layer.select), { yPercent: -d }, { yPercent: d }, 0)
        }
      },
    )

    return () => mm.revert()
  }, [ref, layers])
}
```

```tsx
// Constante de módulo, nunca um literal inline. O efeito depende de `layers`;
// um array novo a cada render faz `mm.revert()` e reconstrói a timeline inteira
// em todo render. É como `ChapterCare.tsx` passa `REVEALS`.
const LAYERS: readonly Layer[] = [
  { select: '[data-bg]', depth: 12 },
  { select: '[data-mid]', depth: 6 },
  { select: '[data-front]', depth: 2 },
]

useParallax(ref, LAYERS)
```

**Números.** `depth` 12 / 6 / 2 · overscan `2 × depth` · `scrub: true` · `ease: 'none'`.
Nunca passe de 120px de deslocamento absoluto.

**Custo.** Um trigger para todas as camadas. Só `transform` — zero layout. O custo real é a
mídia sobredimensionada: 124% de altura são ~24% mais pixels para decodificar e compor.

---

## 3. Seção pinada com timeline interna

**Quando usar.** 2–4 beats que precisam acontecer dentro do mesmo quadro: uma frase que troca
sobre a mesma imagem, um contador que sobe, uma composição que se monta.

### 3a. Sticky CSS — o padrão deste repositório

Nada é medido, nada é re-medido, não há `pin-spacer` para dessincronizar do Lenis, e o
reduced-motion desmonta tudo com uma linha de CSS.

```css
/* src/styles/index.css — já existe; abreviado aqui só com o que faz o pin */
.chapter {
  position: relative;
  height: calc(100svh + (var(--chapter-scroll-mobile, 2) * 100svh));
}
@media (min-width: 768px) {
  .chapter { height: calc(100svh + (var(--chapter-scroll-desktop, 3) * 100svh)); }
}
.chapter__stage {
  position: sticky;
  top: 0;
  height: 100svh;
  overflow: hidden;
  transform: translateZ(0);
}
@media (prefers-reduced-motion: reduce) {
  .chapter { height: auto !important; }
  .chapter__stage { position: relative; height: auto; overflow: visible; }
}
```

```tsx
// Autoria em "story units": 0 é o instante em que o stage gruda, 1 o instante
// em que ele solta. Independente da altura em pixels.
useChapterTimeline(ref, (tl, scope) => {
  gsap.set(q(scope, '[data-mark]'), { scaleX: 0 })

  tl.fromTo('[data-beat="1"]', { autoAlpha: 0, y: 24 }, { autoAlpha: 1, y: 0, duration: 0.08 }, 0.05)
    .to('[data-beat="1"]', { autoAlpha: 0, y: -24, duration: 0.06 }, 0.34)
    .fromTo('[data-beat="2"]', { autoAlpha: 0, y: 24 }, { autoAlpha: 1, y: 0, duration: 0.08 }, 0.40)
    .to('[data-mark]', { scaleX: 1, duration: 0.2, ease: 'power2.inOut' }, 0.42)
})
```

`seal(tl)` (aplicado dentro do hook) fixa o comprimento total em exatamente 1, para que as
posições continuem honestas mesmo quando o último tween termina antes do fim.

### 3b. `pin: true` — quando o sticky não resolve

Quando o alvo não pode virar filho direto de uma seção alta, ou o pin precisa começar longe do
topo.

```tsx
mm.add('(min-width: 768px) and (prefers-reduced-motion: no-preference)', () => {
  gsap.timeline({
    defaults: { ease: 'none' },
    scrollTrigger: {
      trigger: scope,
      // Pine um WRAPPER, nunca o elemento que carrega margem ou é item de grid:
      // o pin-spacer entra entre o pai e ele.
      pin: '[data-stage]',
      pinSpacing: true, // false só quando o que vem depois desliza por cima de propósito
      anticipatePin: 1, // mata o flicker de um frame em scroll rápido
      start: 'top top',
      end: '+=300%', // 3 beats × ~100vh
      scrub: true,
      invalidateOnRefresh: true,
    },
  })
})
```

**Números.** 60–100vh de scroll por beat legível. Máximo 2 pins na página. No celular, um beat
só ou nenhum pin.

**Custo.** Sticky: zero medição. `pin: true`: um `pin-spacer` no DOM, re-medição a cada
`refresh()`, e o layout do pai muda.

---

## 4. Scrub horizontal

**Quando usar.** Uma galeria ou um processo que é naturalmente lateral e não cabe na largura.
Este é o caso legítimo de `pin: true` — o trilho precisa ser mais largo que a tela e o
container precisa estar preso enquanto ele corre.

```tsx
// src/hooks/useHorizontalScrub.ts
import { useLayoutEffect, type RefObject } from 'react'
import { gsap } from '../lib/gsap'

/** >1 dá respiro: 1.0 é 1px vertical por 1px horizontal, 1.25 desacelera 25%. */
const PACE = 1.2

export function useHorizontalScrub(
  ref: RefObject<HTMLElement | null>,
  trackSelector = '[data-track]',
): void {
  useLayoutEffect(() => {
    const scope = ref.current
    if (!scope) return

    const mm = gsap.matchMedia(scope)

    // Só no desktop. Num celular, arrastar horizontal é o gesto nativo do
    // trilho — sequestrar o scroll vertical para movê-lo confunde o polegar.
    mm.add('(min-width: 768px) and (prefers-reduced-motion: no-preference)', () => {
      const track = scope.querySelector<HTMLElement>(trackSelector)
      if (!track) return

      const distance = () => Math.max(track.scrollWidth - window.innerWidth, 0)
      if (distance() === 0) return

      gsap.to(track, {
        // Função, não literal: recalculada a cada refresh junto com o end.
        x: () => -distance(),
        ease: 'none',
        scrollTrigger: {
          trigger: scope,
          pin: true,
          start: 'top top',
          end: () => `+=${distance() * PACE}`,
          scrub: true,
          invalidateOnRefresh: true, // sem isto o x fica congelado no valor antigo
          anticipatePin: 1,
        },
      })
    })

    return () => mm.revert()
  }, [ref, trackSelector])
}
```

No mobile, o mesmo trilho vira scroll nativo:

```css
@media (max-width: 767px) {
  [data-track] { overflow-x: auto; scroll-snap-type: x mandatory; }
  [data-track] > * { scroll-snap-align: center; }
}
```

**Números.** `end` = largura excedente × 1.0–1.4 · `scrub: true` · `ease: 'none'`. Cada card
deve ocupar 60–80vw para caber legível com um vislumbre do vizinho.

**Custo.** Pin + leitura de `scrollWidth`, que força reflow a cada `refresh()`. Mantenha um
único trilho horizontal por página.

**Alternativa sem pin.** `ChapterJourney` resolve isso deslocando o rail com `x` dentro de um
stage sticky, em passos discretos: 28% de cada intervalo viajando, 72% parado. O card que está
sendo lido fica parado — que é o defeito do scrub horizontal contínuo.

---

## 5. Card stacking

**Quando usar.** 3–5 itens comparáveis que ganham em ser vistos empilhados: planos, etapas,
depoimentos. Acima de 5, o card do fundo fica escondido tempo demais e a pilha vira ruído.

O empilhamento é CSS (`position: sticky` com offset por índice). O GSAP só recua os cards que
já foram passados.

```tsx
// Cada card carrega o próprio índice, que é o que escalona o offset do sticky:
// <li data-card style={{ '--i': i } as CSSProperties}>
mm.add('(prefers-reduced-motion: no-preference)', () => {
  const cards = q(scope, '[data-card]')
  if (cards.length < 2 || cards.length > 5) return

  cards.forEach((card, i) => {
    const next = cards[i + 1]
    if (!next) return // o último nunca recua

    gsap.to(card, {
      scale: 0.94, // 0.92–0.96. Abaixo de 0.90 a pilha parece cair
      filter: 'brightness(0.72)',
      ease: 'none',
      scrollTrigger: {
        // O gatilho é o card SEGUINTE chegando: o recuo acontece exatamente
        // enquanto ele sobe por cima, não em algum ponto arbitrário.
        trigger: next,
        start: 'top bottom',
        end: 'top top',
        scrub: true,
      },
    })
  })
})
```

```css
[data-card] { position: sticky; top: calc(6rem + var(--i) * 0.75rem); }
@media (prefers-reduced-motion: reduce) {
  [data-card] { position: relative; top: auto; transform: none !important; filter: none !important; }
}
```

**Números.** 3–5 cards · offset 12px por índice · `scale` final 0.94 · `brightness` 0.72 ·
cada card ocupa 80–100vh de scroll.

**Custo.** Um ScrollTrigger por card. Aceitável até 5 — a regra dos "um trigger por elemento"
existe para listas de dezenas, onde o custo de `refresh()` explode. `filter: brightness` força
camada própria: aceitável em 5 cards, não em 40.

---

## 6. Mask / clip reveal

**Quando usar.** Entrada de uma imagem ou vídeo que merece um momento. Uma ou duas vezes por
página — repetido, vira maneirismo.

### Versão barata (preferir): wrapper + transform

Só `transform`, nunca sai do compositor.

```tsx
mm.add('(prefers-reduced-motion: no-preference)', () => {
  const wrap = scope.querySelector('[data-mask]')
  const inner = wrap?.firstElementChild
  // A mídia é o filho do filho. Sem este guard o GSAP recebe null, loga
  // "target not found" e a máscara sobe com a imagem parada dentro dela.
  const media = inner?.firstElementChild
  if (!wrap || !inner || !media) return

  gsap.set(inner, { yPercent: 110 })
  gsap
    .timeline({ scrollTrigger: { trigger: wrap, start: 'top 75%', once: true } })
    .to(inner, { yPercent: 0, duration: 1.1, ease: 'expo.out' })
    // Contra-movimento: a mídia começa maior e assenta enquanto a máscara sobe,
    // e as duas velocidades diferentes é que dão a profundidade.
    .from(media, { scale: 1.14, duration: 1.4, ease: 'power2.out' }, 0)
})
```

```css
[data-mask] { overflow: hidden; }
```

### Versão clip-path — quando a forma importa

```tsx
gsap.fromTo(
  el,
  { clipPath: 'inset(0 0 100% 0)' },
  {
    clipPath: 'inset(0 0 0% 0)',
    duration: 1.2,
    ease: 'power4.inOut',
    scrollTrigger: { trigger: el, start: 'top 75%', once: true },
  },
)
```

| Forma | De | Para |
|---|---|---|
| Sobe de baixo | `inset(0 0 100% 0)` | `inset(0 0 0 0)` |
| Varre da esquerda | `inset(0 100% 0 0)` | `inset(0 0 0 0)` |
| Abre do centro | `circle(0% at 50% 50%)` | `circle(75% at 50% 50%)` — 70.7% é o mínimo que cobre os cantos de qualquer retângulo, em qualquer proporção |
| Divide ao meio | `polygon(50% 0, 50% 0, 50% 100%, 50% 100%)` | `polygon(0 0, 100% 0, 100% 100%, 0 100%)` |

**Números.** `start: 'top 75%'` · `duration` 1.1–1.4 · `ease power4.inOut` para clip,
`expo.out` para transform · contra-movimento `scale` 1.10–1.16.

**Custo.** A versão com wrapper é só `transform`: fica no compositor. `clip-path` não fica — o
GSAP escreve o valor inline a cada frame, na main thread, então não existe animação de
compositor aqui. Não dispara layout, mas repinta o elemento inteiro por frame: barato num quadro
pequeno, caro em tela cheia. `polygon()` com muitos pontos é o mais caro de todos. Use `clip-path`
uma vez, no herói, e o wrapper no resto.

---

## 7. Transição de cor de fundo entre seções

**Quando usar.** Virada de capítulo: claro → escuro → claro. Marca a mudança de assunto sem
gastar um efeito.

**Por que sem scrub.** O meio do caminho entre duas cores de marca costuma ser lama. Um toggle
na linha média da tela mais um tween curto dá uma transição limpa; o scrub obriga o visitante a
atravessar todos os tons intermediários.

```tsx
// Cada seção declara data-page-bg="#0e0d0c" e data-page-ink="#f6f2ec".
// Nomes próprios: [data-bg] já é a camada de fundo do parallax (receita 2).
mm.add('(prefers-reduced-motion: no-preference)', () => {
  for (const section of q(scope, '[data-page-bg]')) {
    ScrollTrigger.create({
      trigger: section,
      // Meia tela na entrada, meia tela na saída: a virada acontece quando a
      // seção realmente domina o quadro.
      start: 'top 50%',
      end: 'bottom 50%',
      onToggle: (self) => {
        if (!self.isActive) return
        gsap.to(document.documentElement, {
          // O GSAP interpola custom properties como string, e converte hex em
          // rgba antes de interpolar — cor de marca chega inteira do outro lado.
          '--page-bg': section.dataset.pageBg,
          '--page-ink': section.dataset.pageInk,
          duration: 0.6,
          ease: 'power2.out',
          overwrite: 'auto', // rolar rápido não deve empilhar tweens de cor
        })
      },
    })
  }
})
```

```css
:root { --page-bg: #f6f2ec; --page-ink: #171310; }
body { background-color: var(--page-bg); color: var(--page-ink); }
```

Sob reduced motion nada é criado, e cada seção continua com o próprio fundo declarado em CSS —
o contraste tem que fechar sem o tween.

**Números.** `start: 'top 50%'` / `end: 'bottom 50%'` · `duration 0.6` · `overwrite: 'auto'`.
Contraste mínimo de texto sobre fundo: 4.5:1 em cada extremo *e* no meio da transição.

**Custo.** Repaint de tela cheia por transição — 0.6s, aceitável. Com scrub seria um repaint de
tela cheia por tick de scroll, e é por isso que não se faz com scrub.

---

## 8. Texto por palavra

**Quando usar.** Uma headline, no máximo. Limite: **20 palavras**. Um parágrafo de corpo com 80
palavras vira 160 nós no DOM e alguns leitores de tela fragmentam a leitura.

O markup vem de `src/components/ui/Words.tsx`: cada palavra é `<span class="word"><span
class="word__inner">…</span></span>`. O texto real continua no documento — selecionável,
pesquisável, lido normalmente.

**Palavra por palavra ou linha por linha?** Linha. Um stagger palavra a palavra lê como
máquina de escrever, o que é barato sob uma serifada de display e ilegível quando outra coisa
está ditando o ritmo. `lineGroups()` agrupa as palavras nas linhas visuais que elas realmente
ocupam, e cada linha sobe inteira.

```tsx
mm.add('(prefers-reduced-motion: no-preference)', () => {
  // Esconder é síncrono: a copy não pode piscar visível antes do primeiro paint.
  // Medir linhas não é — depende das fontes reais.
  gsap.set(words(scope, '[data-title]'), { yPercent: 110, autoAlpha: 0, filter: 'blur(5px)' })

  typographyReady().then(() => {
    const lines = lineGroups(scope, '[data-title]')
    if (lines.length === 0) return

    const tl = gsap.timeline({ paused: true })
    lines.forEach((line, i) => {
      tl.to(
        line,
        { yPercent: 0, autoAlpha: 1, filter: 'blur(0px)', duration: 1, ease: 'expo.out' },
        i * 0.12,
      )
    })
    // Um blur(0px) assentado ainda mantém o texto numa camada própria.
    tl.eventCallback('onComplete', () => gsap.set(lines.flat(), { clearProps: 'filter' }))

    ScrollTrigger.create({ trigger: scope, start: 'top 75%', onEnter: () => tl.play() })
    // A altura mudou enquanto as fontes carregavam. Um refresh, aqui, uma vez.
    ScrollTrigger.refresh()
  })
})
```

```css
/* padding + margem negativa nos DOIS lados: embaixo para as descendentes (g, p, ç)
   e em cima para os acentos altos (Ã, Õ) — sem o par de cima o til some enquanto a
   linha viaja dentro da máscara, e só aparece rolando devagar. `will-change` aqui é
   a exceção da regra: são poucos nós, todos animados de fato, e sem a promoção a
   máscara serrilha. */
.word { display: inline-block; overflow: hidden; vertical-align: bottom;
        padding-block: 0.12em 0.16em; margin-block: -0.12em -0.16em; }
.word__inner { display: inline-block; will-change: transform; }
@media (prefers-reduced-motion: reduce) {
  .word, .word__inner { overflow: visible; transform: none !important; opacity: 1 !important; }
}
```

**Números.** ≤ 20 palavras · `duration 1` por linha · `stagger 0.12` · `yPercent 110` (110, não
100: descendentes precisam sair inteiros da máscara) · `blur 5px` → 0 · `ease expo.out` ·
`typographyReady()` com timeout de 1600ms.

**Por que o timeout.** Em cache frio os arquivos de fonte podem levar segundos, e bloquear neles
deixa o visitante assistindo a um filme mudo. Passado o limite, medimos com o que estiver na
tela: quebras de linha ligeiramente diferentes são uma falha muito menor do que copy que nunca
chega.

**Custo.** Dois nós DOM por palavra. Uma headline de 12 palavras = 24 nós, uma máscara com
`overflow: hidden` cada. `filter: blur` durante a subida força camada própria — por isso o
`clearProps` no fim.

---

## Esqueleto de matchMedia

O envelope de toda receita acima. Três condições, sempre as mesmas: desktop com movimento,
mobile com movimento, e nenhum branch para `reduce` — sob reduced motion nenhuma condição casa,
nada é criado e nada é escrito no DOM.

```ts
const mm = gsap.matchMedia(scope)

mm.add(
  {
    desktop: '(min-width: 768px) and (prefers-reduced-motion: no-preference)',
    mobile: '(max-width: 767px) and (prefers-reduced-motion: no-preference)',
  },
  (context) => {
    const { desktop } = context.conditions as { desktop: boolean; mobile: boolean }

    gsap.timeline({
      defaults: { ease: 'none' },
      scrollTrigger: {
        trigger: scope,
        start: 'top top',
        end: 'bottom bottom',
        scrub: true,
        invalidateOnRefresh: true,
      },
    }).fromTo('[data-bg]', { yPercent: desktop ? -12 : -6 }, { yPercent: desktop ? 12 : 6 }, 0)
  },
)

return () => mm.revert()
```

Desktop e mobile no mesmo tween, com o travel do mobile pela metade — duas chamadas separadas
duplicam o trigger e as duas versões coexistem no resize. `mm.revert()` no cleanup é o que
impede o Strict Mode do React 19 de deixar um trigger órfão por montagem.

---

## Tabela de sintoma e causa

Ler depois de ligar `markers: true` e conferir os quatro marcadores na tela.

| Sintoma | Causa |
|---|---|
| Dispara uma tela cedo ou tarde | `trigger` é o wrapper alto, não o elemento |
| Estava certo, saiu do lugar depois de carregar | altura mudou depois do refresh — fontes ou imagem sem dimensão reservada |
| Marcadores certos, nada anima | seletor não resolve dentro do `scope`, ou nenhuma condição do `matchMedia` casou |
| Pin salta um frame ao entrar | falta `anticipatePin: 1` |
| Layout quebra ao ligar o pin | `pin-spacer` entrou entre o pai e o filho — pine um wrapper |
| Elemento pinado some | ancestral com `overflow: hidden` ou `transform` |
| Scroll briga consigo mesmo | GSAP registrado duas vezes, ou Lenis sem `ScrollTrigger.update` |
| Micro-jitter constante com scrub | Lenis rodando o próprio `raf` em vez do `gsap.ticker` |
| Salto de posição depois de um travamento | falta `gsap.ticker.lagSmoothing(0)` |
| Tudo re-mede no scroll do celular | falta `ScrollTrigger.config({ ignoreMobileResize: true })` |
| Anima duas vezes / valores dobrados | falta `mm.revert()` no cleanup — Strict Mode montou duas vezes |
| Timeline com scrub parece atrasada | scrub numérico empilhado sobre o Lenis |

Perfil rápido: DevTools → Performance, 4× CPU throttle, rolar a seção. Barras roxas
(Layout/Recalculate Style) durante o scrub significam que algo fora de `transform`/`opacity`
está sendo animado.
