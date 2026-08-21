# Receitas de motion

Implementações prontas para os tokens do [SKILL.md](SKILL.md). Todas importam de
`src/lib/gsap` (`gsap`, `ScrollTrigger`, `q`, `seal` — ajuste a profundidade do caminho
relativo ao arquivo) e rodam dentro de um `gsap.matchMedia('(prefers-reduced-motion:
no-preference)')`.

## 0. Esqueleto de um capítulo

A timeline autorada em story units. `useChapterTimeline` já faz tudo isto — use-o. A forma
explícita existe para entender o que ele faz e para depurar quando não faz:

```tsx
useLayoutEffect(() => {
  const scope = ref.current
  if (!scope) return

  // Escopo: todo seletor em string aqui dentro resolve só dentro deste capítulo.
  const mm = gsap.matchMedia(scope)

  mm.add('(prefers-reduced-motion: no-preference)', () => {
    gsap.set('[data-lead]', { autoAlpha: 0, y: 24 })

    const tl = gsap.timeline({
      // Sem isto o ease padrão do GSAP (power1.out) entra em cima do scrub.
      defaults: { ease: 'none' },
      scrollTrigger: {
        trigger: scope,
        // O stage é sticky: progresso 0 é quando ele gruda, 1 quando solta.
        start: 'top top',
        end: 'bottom bottom',
        scrub: true,
        invalidateOnRefresh: true,
      },
    })

    // seal() fixa a duração total em exatamente 1. Sem ele o scrub normaliza
    // pelo último tween — 0.16 + 0.035 = 0.195 viraria progresso 1, e todas as
    // posições em story unit passariam a significar outra coisa.
    seal(tl).to('[data-lead]', { autoAlpha: 1, y: 0, duration: 0.035 }, 0.16)

    return () => {
      /* cleanup extra — listeners, observers, triggers criados à mão */
    }
  })

  return () => mm.revert()
}, [ref])
```

Seletor em string dentro do `mm.add` já resolve só neste capítulo — use `q(scope, sel)` de
`src/lib/gsap.ts` apenas quando precisar do array em si (medir, checar `.length`, iterar).

`matchMedia` **é** um `gsap.context()` com condição: não empilhe os dois. Sob `reduce` o bloco
nunca roda, nenhuma propriedade é escrita no DOM e o markup fica no estado natural, visível —
melhor que animar com duração zero, que ainda deixa inline styles para trás.

### Ordem de medição

`start`/`end` são pixels calculados no momento da criação do trigger. Chame
`ScrollTrigger.refresh()` depois de qualquer coisa que mude a altura do documento: fontes
carregando, canvas montando, imagem sem `aspect-ratio` reservado.

Antes de medir line boxes espere as fontes, mas com teto. `typographyReady()` em
`src/lib/gsap.ts` faz `Promise.race` entre `document.fonts.ready` e 1600ms: em cache frio as
faces podem demorar segundos, e bloquear nelas deixa o visitante vendo um filme mudo. Quebras de
linha ligeiramente diferentes são uma falha muito menor que copy que nunca chega.

Segurar um elemento na tela é `position: sticky`, não `pin: true` — a comparação completa está
na tabela de `gsap-scrolltrigger-expert`.

## 1. Reveal de copy que toca uma vez

O scroll decide *quando* a linha chega; a animação decide *a que velocidade*. Enquanto o
reveal estava preso à posição de scroll, o ritmo dele era o ritmo do dedo: um flick de trackpad
nascia e enterrava uma frase dentro do mesmo gesto, e parar no meio congelava a frase com
buracos.

Já implementado em `src/hooks/useChapterReveals.ts`. Use-o. A forma mínima, para quando não há
capítulo:

```tsx
useLayoutEffect(() => {
  const scope = ref.current
  if (!scope) return
  const mm = gsap.matchMedia(scope)

  mm.add('(prefers-reduced-motion: no-preference)', () => {
    const items = q(scope, '[data-reveal]')
    gsap.set(items, { autoAlpha: 0, y: 26 })

    const trigger = ScrollTrigger.create({
      trigger: scope,
      // `top 80%` é o default de reveal de copy do gsap-scrolltrigger-expert, que é dono
      // dos pontos de disparo. Um valor próprio aqui faz esta seção acender fora de fase
      // com todas as outras da página.
      start: 'top 80%',
      // Sem toggleActions de reverse: uma frase não se desescreve sob quem lê.
      onEnter: () =>
        gsap.to(items, { autoAlpha: 1, y: 0, duration: 0.9, ease: 'expo.out', stagger: 0.08 }),
    })

    return () => trigger.kill()
  })

  return () => mm.revert()
}, [ref])
```

Para headline em máscara por linha, meça com `lineGroups()` **depois** de `typographyReady()` —
as quebras medidas antes das faces reais pertencem à fonte de fallback.

## 2. Quando GSAP é exagero: IntersectionObserver + CSS

Um fade-in de uma seção não precisa de ScrollTrigger. Um trigger registra callbacks no ticker e
recalcula `start`/`end` em todo refresh; um observer custa uma entrada na lista do compositor.

```css
[data-io] {
  opacity: 0;
  transform: translate3d(0, 26px, 0);
  transition:
    opacity 0.9s var(--ease-out-expo),
    transform 0.9s var(--ease-out-expo);
}
[data-io].is-in {
  opacity: 1;
  transform: none;
}
@media (prefers-reduced-motion: reduce) {
  [data-io] { opacity: 1; transform: none; transition: none; }
}
```

```tsx
useEffect(() => {
  const scope = ref.current
  if (!scope) return

  const io = new IntersectionObserver(
    (entries) => {
      for (const entry of entries) {
        if (!entry.isIntersecting) continue
        entry.target.classList.add('is-in')
        io.unobserve(entry.target) // uma vez só; nada de re-observar
      }
    },
    { rootMargin: '0px 0px -20% 0px' },
  )

  // Escopado ao ref, nunca `document.querySelectorAll`: uma busca global pega os
  // `[data-io]` de outros componentes e dois observers passam a brigar pelo mesmo nó.
  for (const node of q(scope, '[data-io]')) io.observe(node)

  return () => io.disconnect()
}, [ref])
```

O estado inicial mora no CSS, então o reduced motion é resolvido pela media query e nenhum JS
precisa saber disso.

## 3. Hover e pointer follow com `quickTo`

```tsx
const moveX = gsap.quickTo(node, 'x', { duration: 0.45, ease: 'power3.out' })
const moveY = gsap.quickTo(node, 'y', { duration: 0.45, ease: 'power3.out' })
```

`quickTo` reaproveita uma única tween e só reescreve o alvo. `gsap.to()` a cada `pointermove`
cria dezenas de tweens por segundo, cada uma tentando sobrescrever a anterior — o elemento
gagueja e o GC trabalha à toa.

Guardas obrigatórias antes de instalar o listener (ver `src/hooks/useMagnetic.ts`):

```ts
if (!window.matchMedia('(pointer: fine)').matches) return   // não há hover no toque
if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return
```

Cap de deslocamento: 5px. `gsap.utils.clamp(-5, 5)`.

## 4. Counter com separador de milhar

```tsx
const el = ref.current!
const target = Number(el.dataset.value)
const counter = { value: 0 }

gsap.to(counter, {
  value: target,
  // 1.2s é o teto do token `cinematic`. Um counter mais longo que isso passa a
  // ser espera, e o número já está no DOM — não há suspense a construir.
  duration: 1.2,
  // Exceção declarada à tabela de easing: `expo.out` chega em ~200ms e depois
  // rasteja nas últimas casas decimais, o que parece travamento. `power1.out`
  // é quase linear e o número sobe em ritmo legível até o fim.
  ease: 'power1.out',
  onUpdate: () => {
    el.textContent = Math.round(counter.value).toLocaleString('pt-BR')
  },
  scrollTrigger: { trigger: el, start: 'top 75%', once: true },
})
```

Anime um número cru e formate no `onUpdate`. Animar `textContent` direto imprime `70000` em vez
de `70.000`, e qualquer `snap` briga com o separador.

O valor final precisa estar no DOM como texto no HTML inicial: sob reduced motion o counter
nunca roda, e um número que só existe contando não existe para quem desligou o movimento.

## 5. Marquee

Duas implementações, escolhas diferentes:

**Loop ambiente** — CSS puro, sem custo de JS. Já existe como `.marquee-track` (42s linear).
Duplique o nó de texto para o rabo nunca revelar vazio, e marque o wrapper `aria-hidden="true"`:
um leitor de tela lendo a mesma frase duas vezes é ruído.

**Ligado ao scroll** — quando o movimento precisa reagir ao gesto:

```ts
gsap.to(track, {
  xPercent: -25,
  ease: 'none', // scrub já é a curva
  scrollTrigger: { trigger: section, start: 'top bottom', end: 'bottom top', scrub: true },
})
```

Nunca os dois no mesmo elemento: a animação CSS e a tween do GSAP escrevem a mesma matriz de
transform e a última a rodar no frame vence, o que aparece como tremor.

## 6. Parallax de mídia

```ts
gsap.fromTo(
  mediaEl,
  { yPercent: -8 },
  {
    yPercent: 8,
    ease: 'none',
    scrollTrigger: { trigger: section, start: 'top bottom', end: 'bottom top', scrub: true },
  },
)
```

`yPercent`, não `y` em px: em px o mesmo número some no desktop e estoura no mobile. A mídia
precisa ser 16% mais alta que o container (`height: 116%`) ou a borda aparece nos extremos.

## 7. Troca de cor de fundo entre seções

```ts
// GSAP interpola cores parseadas; `var(--x)` não é uma cor que ele saiba ler, e
// uma tween para essa string salta para o valor final em vez de percorrê-lo.
// Resolva o token uma vez, fora do trigger.
const token = (name: string) =>
  getComputedStyle(document.documentElement).getPropertyValue(name).trim()

const ink = token('--color-ink')
const canvas = token('--color-canvas')

ScrollTrigger.create({
  trigger: section,
  start: 'top 50%',
  end: 'bottom 50%',
  onToggle: ({ isActive }) => {
    gsap.to(document.body, {
      backgroundColor: isActive ? ink : canvas,
      duration: 0.6,
      ease: 'power2.out',
      overwrite: true,
    })
  },
})
```

`overwrite: true` é o que importa: rolar rápido por três seções dispara três tweens sobre a
mesma propriedade, e sem overwrite elas se somam e a cor pisca no caminho.

`background-color` é paint, não composite — aceitável no `body` porque acontece uma vez por
seção, inaceitável dentro de um scrub.

---

## Invariantes de setup

O setup é escrito uma vez, em `src/lib/gsap.ts` e
`src/components/providers/SmoothScrollProvider.tsx`, e é de `gsap-scrolltrigger-expert`. Estas
oito invariantes são o que se **confere** antes de aprovar qualquer motion. Cada uma quebra de um
jeito específico, e o sintoma é como se descobre qual falhou:

| Invariante | Sintoma quando falta |
|---|---|
| `gsap.registerPlugin(ScrollTrigger)` só em `src/lib/gsap.ts` | Com HMR do Vite cada save soma uma instância; a página fica progressivamente mais lenta em dev e volta ao normal depois de um reload |
| `ScrollTrigger.config({ ignoreMobileResize: true })` | Barra de URL retrátil muda a viewport no meio do scroll e todo capítulo re-mede e salta |
| `lenis.on('scroll', () => ScrollTrigger.update())` | O ScrollTrigger lê a posição um frame atrás: o texto arrasta visivelmente atrás do fundo |
| `gsap.ticker.add((t) => lenis.raf(t * 1000))` | O ticker conta em segundos e o Lenis espera ms; sem o `× 1000` o scroll nunca assenta |
| `gsap.ticker.lagSmoothing(0)` | Um frame longo é "recuperado" ajustando o delta, e numa timeline scrubbed isso é um salto de posição |
| Lenis nunca instanciado sob `prefers-reduced-motion: reduce` | Suavizar o scroll de quem pediu menos movimento é movimento não solicitado |
| Todo efeito dentro de `gsap.matchMedia(scope)`, com `mm.revert()` no cleanup | Strict Mode do React 19 monta duas vezes: sem `revert()` sobram triggers apontando para nós fora do DOM, e a animação roda com o dobro da velocidade — só em dev |
| `useLayoutEffect`, nunca `useEffect`, para o `gsap.set` que esconde a copy | O navegador pinta o texto visível e só depois ele some: flash em toda montagem |

Duas dependências, não três: o pacote é `lenis` — `@studio-freight/lenis` foi renomeado e está
obsoleto. GSAP e Lenis vêm do `package.json`, nunca de CDN: uma segunda cópia registra um segundo
ScrollTrigger e os dois disputam o mesmo scroller.

## Custo por propriedade

| Propriedade | Fase | Custo | Causa |
|---|---|---|---|
| `transform` (`x`, `y`, `scale`, `rotation`) | composite | GPU | O elemento já tem textura própria; mudar a matriz não remede nem repinta nada |
| `opacity` / `autoAlpha` | composite | GPU | Só o alpha de blend da camada muda |
| `filter: blur()` | composite | médio | Roda na GPU, mas refiltra a textura inteira a cada frame. Acima de 8px em elemento grande derruba o fps |
| `clip-path: inset() / circle()` | composite | médio | Formas simples são baratas; `polygon()` com muitos pontos volta para o paint |
| `color`, `background-color` | paint | médio | Não remede, mas repinta a área toda em toda frame |
| `box-shadow` | paint | alto | Repinta um retângulo maior que o próprio elemento. Use uma camada duplicada com `opacity` variando |
| `width`, `height` | **layout** | alto | Muda o tamanho da caixa, então o navegador recalcula a posição de tudo que vem depois no fluxo. Use `scaleX`/`scaleY` |
| `margin`, `padding` | **layout** | alto | Mesmo reflow; `padding` ainda repinta o fundo |
| `top`, `left`, `right`, `bottom` | **layout** | alto | Dispara reflow do containing block. Use `x`/`y` |
| `font-size`, `letter-spacing` | **layout** | muito alto | Relayout de todas as line boxes. Com máscaras por palavra (`.word`), o custo é por palavra |
| `border-width` | **layout** | alto | Muda a caixa. Anime `border-color`, ou use `box-shadow: inset` |
| `scrollTop` via JS | layout + conflito | proibido | O Lenis é dono do scroll; escrever `scrollTop` gera um evento que ele interpreta como input do usuário e o scroll briga consigo mesmo |

Teste rápido: **se a propriedade pode mudar onde outro elemento fica, ela é layout.**

## Anti-patterns de implementação

Os seis mais comuns estão no [SKILL.md](SKILL.md#anti-patterns). Estes completam a lista:

- **`filter: blur(0px)` deixado no fim de um reveal** — um blur assentado ainda mantém o texto em
  camada própria e ainda consome VRAM. `useChapterReveals.ts` faz
  `gsap.set(lines.flat(), { clearProps: 'filter' })` no `onComplete` por isso.
- **Animação CSS e tween do GSAP no mesmo elemento** — as duas escrevem a mesma matriz de
  transform e a última a rodar no frame vence, o que aparece como tremor.
- **`y` em px num parallax** — o mesmo número some no desktop e estoura no mobile. `yPercent`.
- **Tween para `var(--token)`** — o GSAP não parseia a string como cor e salta para o valor final.
  Resolva com `getComputedStyle().getPropertyValue()` fora do trigger.
- **Timeline de capítulo sem `seal()`** — o scrub normaliza pela duração do último tween, e toda
  posição em story unit passa a significar outra coisa.
- **`document.querySelectorAll` num efeito de capítulo** — a busca global pega os nós de outros
  componentes e dois observers passam a brigar pelo mesmo elemento. Escope no `ref`.

---

## Sintoma → causa

| Sintoma | Causa provável | Onde olhar |
|---|---|---|
| Animação roda com o dobro da velocidade, só em dev | Strict Mode montou duas vezes e a primeira não fez `revert()` | `mm.revert()` no cleanup do `useLayoutEffect` |
| Texto "arrasta" atrás do fundo durante o scroll | ScrollTrigger não está sendo atualizado pelo Lenis | `lenis.on('scroll', () => ScrollTrigger.update())` |
| Scroll nunca assenta, fica travado no topo | `lenis.raf(time)` recebendo segundos em vez de ms | `lenis.raf(time * 1000)` |
| Salto visível depois de uma pausa/aba de fundo | Lag smoothing padrão descartando tempo | `gsap.ticker.lagSmoothing(0)` |
| Trigger dispara no lugar errado depois que as fontes carregam | Altura do documento mudou depois da criação do trigger | `ScrollTrigger.refresh()` após `document.fonts.ready` |
| Quebras de linha do reveal não batem com o texto final | Line boxes medidas com a fonte de fallback | `typographyReady()` antes de `lineGroups()` |
| Flash do texto visível na montagem | `useEffect` em vez de `useLayoutEffect` no `gsap.set` que esconde | O `set` precisa rodar antes do paint |
| Cliques caindo em elemento invisível | `opacity: 0` em vez de `autoAlpha: 0` | `autoAlpha` também escreve `visibility: hidden` |
| fps cai em uma seção específica | Propriedade de layout na timeline, ou blur grande | Rendering > Paint flashing |
| Flash branco no Safari mobile | Camadas de composição demais (`will-change` espalhado) | Rendering > Layer borders |
| Página parece atrasada em relação ao dedo | `scrub: 1` numérico somado ao amortecimento do Lenis | `scrub: true` |
| Cor de fundo troca de golpe em vez de transicionar | Tween para `var(--token)`, que o GSAP não parseia como cor | Resolva com `getComputedStyle().getPropertyValue()` antes |
| Beat termina muito antes do fim do capítulo, ou some | Timeline sem `seal()`: o scrub normalizou pelo último tween | `seal(tl)` antes de posicionar os beats |
| Pin quebra o layout / cria espaço fantasma | `pin-spacer` injetado no DOM | Troque por `position: sticky` |
| Duas seções parecem a mesma | Mesma entrada repetida | Leia a seção anterior antes de escolher |
