# Reduced motion — implementação

A **matriz** — o que cada tipo de animação vira sob `prefers-reduced-motion: reduce` — está em
[SKILL.md](SKILL.md#a5--reduced-motion), porque é critério de auditoria. Esta página é o código
que faz cada linha dela acontecer. React 19 + GSAP 3.15 + Lenis 1.3 + Tailwind 4.

## As três camadas, e o que cada uma cobre

| Camada | Cobre | Não cobre |
|---|---|---|
| `gsap.matchMedia` | tudo que a timeline escreveria | nada — é a camada correta |
| CSS `@media (prefers-reduced-motion: reduce)` | layout, `animation`, `transition` | estilo inline que o GSAP já escreveu |
| `useReducedMotion()` no React | montagem condicional (`<video>`, canvas) | o que já montou antes do primeiro efeito |

A ordem importa: **decida antes de construir**. Reverter depois deixa estados presos.

## 1. Reveals e timelines — o guard

Este é o mecanismo. As outras camadas são rede.

```ts
import { gsap, q } from '../lib/gsap' // `q(scope, sel)` devolve HTMLElement[] dentro do escopo

const scope = ref.current
if (!scope) return

const mm = gsap.matchMedia(scope)

mm.add('(prefers-reduced-motion: no-preference)', () => {
  // Só aqui dentro é permitido esconder qualquer coisa.
  const pillars = q(scope, '[data-pillar]')
  gsap.set(pillars, { autoAlpha: 0, y: 24 })

  const tl = gsap.timeline({ scrollTrigger: { trigger: scope, scrub: true } })
  tl.to(pillars, { autoAlpha: 1, y: 0, stagger: 0.05 })
})

return () => mm.revert()
```

Passe **elementos**, nunca texto de seletor, para `set` e `to`. Sete capítulos coexistem no mesmo
documento: um `'[data-pillar]'` solto pega os pilares dos sete, e o capítulo que ninguém está lendo
é revelado junto com o que está. `q(scope, …)` resolve dentro da seção e só dela.

`mm.revert()` no cleanup não é opcional: o Strict Mode do React 19 monta duas vezes, e sem ele
todo ScrollTrigger é registrado em dobro — a página passa a rolar com dois donos.

**`autoAlpha`, nunca `opacity`.** `autoAlpha` escreve `visibility: hidden` quando a opacidade
chega a 0, e isso tira o elemento da ordem de foco. `opacity: 0` deixa um controle plenamente
focável e plenamente invisível — é o achado A2 mais comum desta página.

## 2. Stage sticky → seção em fluxo

Nenhum JS. O capítulo perde a altura extra e a stage volta ao fluxo.

```css
@media (prefers-reduced-motion: reduce) {
  .chapter { height: auto !important; }

  .chapter__stage {
    position: relative;
    height: auto;
    min-height: 100svh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    /* Nada é mais transladado para dentro do quadro, então nada pode ser cortado. */
    overflow: visible;
  }

  .beat {
    position: relative;
    inset: auto;
    opacity: 1 !important;
    transform: none !important;
    filter: none !important;
    padding-block: clamp(2.5rem, 6vw, 5rem);
  }
}
```

Os `!important` existem porque estilo inline do GSAP só perde para eles. São rede de segurança
contra um `gsap.set` que escapou do guard — não são o mecanismo, e não escalam: a próxima timeline
pode animar uma propriedade que não está nesta lista.

## 3. Trilho horizontal → cards que descem

Sem o movimento lateral, os cards a partir do segundo ficam fora da tela **para sempre**,
inclusive para o teclado.

```css
@media (prefers-reduced-motion: reduce) {
  [data-rail] {
    flex-wrap: wrap;
    justify-content: center;
    padding-inline: 0 !important;
  }
  /* O contêiner recortava o que saía pela direita; agora nada sai.
     Repare no espaço: o trilho não é filho direto de `.chapter` — mora dentro de um
     beat, dentro da stage. `.chapter:has(> [data-rail])` (sem o espaço) não casaria
     com nada, e os cards continuariam recortados. */
  .chapter :has(> [data-rail]) { overflow: visible; }

  /* Um progresso que não pode avançar é ruído. */
  [data-progress-row] { display: none; }
}
```

## 4. Vídeo de fundo — não montar

O elemento `<video>` pertence a `video-encode`. Aqui só o que a auditoria verifica:

- `document.querySelectorAll('video').length === 0` com reduced motion ligado. Pausar não conta:
  reverter não desfaz um download que já aconteceu.
- O poster está no DOM com um `alt` que descreve **o clipe**, não o quadro. Sob reduced motion o
  `posterAlt` deixa de ser legenda de espera e vira a única descrição da cena que existe — ver
  [alt-e-scrim.md](alt-e-scrim.md#alt-text).

## 5. Canvas de frame sequence — sair antes do primeiro fetch

```ts
if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
  renderPosterFallback()
  return               // antes de qualquer new Image()
}
```

300 frames baixados e nunca pintados são 2–8 MB cobrados de alguém que pediu menos movimento.

## 6. Lenis — não instanciar

A configuração canônica do Lenis é de `gsap-scrolltrigger-expert`; aqui só o guard:

```tsx
useEffect(() => {
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return
  // …só então instancie o Lenis e ligue-o ao ticker do GSAP.
}, [])
```

Sem instância, o contexto de `SmoothScrollProvider` cai no `NATIVE_FALLBACK` e `jumpTo` vira
`window.scrollTo`. Com `scroll-behavior: auto` no `html`, a navegação por âncora vira corte
instantâneo — exatamente o comportamento desejado.

**Como verificar:** `document.documentElement.classList.contains('lenis')` devolve `false` — a
instância marca o elemento raiz com essa classe, então a ausência dela prova que nenhuma foi
criada.

## 7. Micro-interações — o que sobrevive

```tsx
// Perde a escala, mantém a cor. A resposta ao toque continua existindo.
'active:scale-[0.98] motion-reduce:active:scale-100'

// Ícone que avança no hover: o deslocamento é decoração, a cor é o estado.
'group-hover:translate-x-1 motion-reduce:transition-none motion-reduce:group-hover:translate-x-0'
```

`motion-reduce:` é a variante nativa do Tailwind 4 para `(prefers-reduced-motion: reduce)`.
Prefira-a a um `@media` manual: ela fica ao lado da classe que corrige, não a 300 linhas dali.

## 8. Loops perpétuos e a varredura final

```css
@media (prefers-reduced-motion: reduce) {
  .marquee-track { animation: none; }

  *, *::before, *::after {
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
  }
}
```

Movimento perpétuo é o gatilho vestibular clássico, não preferência estética. Um marquee parado
precisa continuar legível: se o texto só faz sentido em movimento, o componente está errado, não o
media query.

A varredura é a última linha de defesa, depois de tudo. Zera também foco e press — aceitável
quando o estado final é idêntico ao animado. **Deixa de ser aceitável** quando a transição *é* a
informação: um acordeão que só se entende abrindo, um passo a passo que só se entende avançando.
Nesses casos, reconstrua a transição acima desta regra com 150–200ms e sem deslocamento, e marque
o sobrevivente com `[data-motion-safe='spin']` ou `[data-motion-safe='fade']`.

## Como testar de verdade

1. Emule `prefers-reduced-motion: reduce` no DevTools (Rendering → Emulate CSS media feature).
2. **Recarregue.** Os guards rodam na montagem; alternar sem recarregar não testa nada.
3. Role a página inteira até o rodapé.
4. Rode o script do passo A5 — o array precisa voltar vazio.
5. Confirme `document.querySelectorAll('video').length === 0`.
6. Confirme que a altura do documento caiu: `document.body.scrollHeight` deve ficar próximo da
   soma dos conteúdos, não dos 8–20 viewports do modo animado.
