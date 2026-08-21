# Receitas de motion

Implementações prontas para os tokens do [SKILL.md](SKILL.md). Todas importam de
`src/lib/gsap` (`gsap`, `ScrollTrigger`, `q`, `seal` — ajuste a profundidade do caminho
relativo ao arquivo) e rodam dentro de um `gsap.matchMedia('(prefers-reduced-motion:
no-preference)')`.

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
