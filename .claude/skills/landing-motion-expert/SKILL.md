---
name: landing-motion-expert
description: Use when choosing which motion specialist owns a task, when defining or auditing a project's motion language (duration, easing, stagger, travel distance, entrance variety), when setting the per-section motion budget, or when authoring a chapter timeline in story units. Roteia entre gsap-scrolltrigger-expert, scroll-video-director, video-to-website e motion-ui-expert. Também para diagnosticar jank, layout thrash, ScrollTrigger duplicado, animação inconsistente entre seções e movimento que não sobrevive a prefers-reduced-motion.
argument-hint: [secao-ou-componente]
---

# Landing Motion Expert

Esta skill não implementa a animação. Ela faz duas coisas:

1. **Roteia** — dado o sintoma na sua frente, diz qual especialista assume.
2. **Define a linguagem** — durações, curvas, stagger, distâncias e orçamento. Todo especialista
   que ela chama usa estes tokens. É isso que faz seis seções feitas em momentos diferentes
   parecerem a mesma página.

A ordem dos especialistas e o Creative Budget estão no [CLAUDE.md](../../../CLAUDE.md). Esta
skill não os repete.

## Entrada

- A seção ou componente em questão, e se ela já tem markup.
- Os vídeos disponíveis (`src/generated/media.ts`) — a rota muda se existir MP4.
- O que já foi animado nas seções vizinhas. Sem isso não dá para escolher a entrada: a regra
  mais violada do projeto é duas seções consecutivas entrando do mesmo jeito.

Se não souber o que as vizinhas fazem, leia os componentes em `src/components/chapters/` antes
de escolher. Não invente.

---

## Tabela de roteamento

| Sintoma / necessidade | Especialista | Por quê |
|---|---|---|
| Pin, scrub, parallax, progresso de seção, horizontal scroll | `gsap-scrolltrigger-expert` | O problema é mapear posição de scroll para progresso de timeline. É medição de pixels, não de estética. |
| Existe MP4 no projeto | `scroll-video-director` primeiro — ver [rota de vídeo](#rota-de-vídeo) | Três técnicas com custos de 300 KB a 8 MB. Escolher errado é irreversível depois da extração. |
| Hover, botão, card, modal, menu, form, loading, transição de página | `motion-ui-expert` | Motion de componente responde a input direto do usuário. A escala de tempo é 0.12–0.4s, outra da do scroll. |
| "Que seção vem antes de qual", CTA, jornada, pacing narrativo | `landing-storytelling-director` | Ordem de seção é conversão, não movimento. Animar a seção errada não conserta a ordem. |
| "Qual é o WOW moment", onde gastar o Creative Budget | `creative-direction-expert` | Decide *o que* merece motion caro antes de alguém escrever timeline. |
| Espaçamento, tipografia, hierarquia, composição | `product-design-expert` | Se a hierarquia estática está errada, animar só faz o erro se mover. |
| Mobile, teclado, screen reader, reduced motion, breakpoints | `responsive-e-acessibility` | Bloqueia o merge. Nenhuma implementação fecha sem esse passe. |
| Durações/curvas inconsistentes entre seções; "parece template" | **esta skill** — [linguagem de motion](#linguagem-de-motion) | É um problema de tokens, não de implementação. Um especialista chamado agora só espalharia a inconsistência. |
| Jank, fps abaixo de 55, layout thrash | **esta skill** — [tabela de custo](#tabela-de-custo-por-propriedade), depois `gsap-scrolltrigger-expert` | Quase sempre é uma propriedade de layout sendo animada. Isso se resolve trocando a propriedade, não a timeline. |
| ScrollTrigger duplicado, animação só quebra em dev | **esta skill** — [invariantes de setup](#setup--invariantes) | É Strict Mode sem `revert()`. Nenhum especialista de motion vai olhar o ciclo de vida do React. |
| A animação não responde a "por que ela existe?" | ninguém — remova | Motion ROI, no CLAUDE.md. |

**Um especialista por vez, na ordem do CLAUDE.md.** Chamar `gsap-scrolltrigger-expert` antes de
`landing-storytelling-director` gasta uma timeline inteira numa seção que vai ser cortada.

### Rota de vídeo

`scroll-video-director` é a porta de entrada de qualquer coisa com vídeo: decide se o clipe
fica, o que ele diz, e é dono do elemento `<video>` — loop vs once, poster, gating por
IntersectionObserver, renditions desktop/mobile, encode com ffmpeg, fallback de reduced motion.
Ele delega para `video-to-website` quando a precisão de frame é o ponto.

Leia [decision.md](../video-to-website/decision.md) antes de extrair qualquer frame.

| Papel do vídeo na seção | Técnica | Quem assume |
|---|---|---|
| Ambiência atrás de copy que se lê por cima | `<video muted loop playsInline>` com IntersectionObserver | `scroll-video-director`; a implementação de referência já existe em `src/components/media/ChapterFilm.tsx` |
| A mudança ao longo do tempo **é** a batida da história | canvas frame sequence | `video-to-website` |
| Clipe >30s de movimento essencial, página desktop-only | scrub de `currentTime` | `scroll-video-director` — e aceite o stutter no iOS Safari, que não busca frame a frame em vídeo comprimido |

O teste honesto: *se esta seção fosse uma foto com uma boa legenda, o que se perderia?* Nada →
foto. Só o clima → loop. A transformação → frames.

Uma página recebe **uma** frame sequence. Duas se forem distantes. Três e a página vira download.

---

## Linguagem de motion

Tokens do projeto. Um número fora desta tabela precisa de justificativa escrita no código.

### Durações

| Token | Valor | Onde |
|---|---|---|
| `instant` | 0.12s | Estado que o dedo já causou: press, toggle, checkbox |
| `fast` | 0.2s | Hover, focus ring, cor, tooltip, `.skip-link` |
| `base` | 0.6s | Entrada de um elemento único, troca de conteúdo, drawer |
| `slow` | 0.9s | Bloco de copy assentando, item de stagger |
| `cinematic` | 1.2s | Linha de headline saindo da máscara, clip-path, cortina |
| `ambient` | 20s+ | Marquee, aurora — loop infinito, `ease: 'none'` |

Nada entre **1.3s e 19s**. Acima de 1.2s a animação deixou de ser resposta e ainda não é
ambiente: é o usuário esperando o site liberar a leitura. O projeto já respeita isso —
`LINE_DURATION = 1` em `useChapterReveals.ts`, `marquee 42s` no CSS.

**Em timeline scrubbed, duração não é tempo — é fração de scroll.** Os capítulos são autorados
em *story units* de 0 a 1 (`seal()` em `src/lib/gsap.ts`), independentes da altura em pixels:

| Story unit | Em `<Chapter scroll={4.8}>`, viewport 900px | Serve para |
|---|---|---|
| 0.01 | ~43 px de scroll | Um corte. É o mínimo — abaixo disso vira glitch, não transição |
| 0.035 | ~151 px | Um elemento entrando (`data-eyebrow`, `data-lead`) |
| 0.08 | ~346 px | Um beat que precisa ser lido antes de sair |
| 0.10 | ~432 px | Um princípio segurando o quadro sozinho (`CYCLE_BEAT`) |

Converta antes de escrever: `pixels = unit × scroll × altura_do_viewport`. O `scroll` do
`<Chapter>` é *scroll além da primeira tela* — `.chapter` mede `(scroll + 1) × 100svh` e a
distância de progresso 0→1 é exatamente `scroll × 100svh`.

Faça a conta duas vezes. Cada capítulo declara `scroll` e `scrollMobile`, e o segundo é ~30%
menor (`ChapterCare`: 4.8 / 3.4). O mesmo 0.08 que dá 346px num desktop de 900px dá 190px num
celular de 700px. **Um beat de texto com menos de 300px de scroll é ilegível em trackpad** — um
flick passa por cima dele inteiro. Se a conta do mobile não fecha, aumente `scrollMobile` ou
corte um beat; não encolha a unidade.

### Curvas de easing

| Nome | cubic-bezier | GSAP | Quando |
|---|---|---|---|
| `ease-out-expo` | `cubic-bezier(0.16, 1, 0.3, 1)` | `expo.out` | Entrada de copy e reveals. Chega quase instantâneo e assenta. Já é token em `src/styles/index.css` |
| `ease-out-quint` | `cubic-bezier(0.22, 1, 0.36, 1)` | `power4.out` | UI que precisa parecer rápida sem ser abrupta: menu, drawer, painel. Token no mesmo bloco `@theme` |
| `ease-out-quart` | `cubic-bezier(0.25, 1, 0.5, 1)` | `power3.out` | Padrão de deslocamento de elemento: hover, magnetic, card lift |
| `ease-out-cubic` | `cubic-bezier(0.33, 1, 0.68, 1)` | `power2.out` | Scale-up e fades onde expo parece exagerado |
| `ease-in-out-quint` | `cubic-bezier(0.83, 0, 0.17, 1)` | `power4.inOut` | Só para movimento que sai **e** volta: cortina, transição de página, clip fechando |
| `ease-out-back` | `cubic-bezier(0.34, 1.56, 0.64, 1)` | `back.out(1.7)` | Confirmação, toggle que "prende". Máximo um por página |
| `linear` | — | `none` | Scrubbed e loops, sempre |

As duas primeiras já existem como variáveis CSS. As de GSAP são as equivalentes aproximadas —
use o nome GSAP em JS e a variável em CSS, nunca um bezier literal solto no meio do código.

Três regras com causa:

- **Nunca `ease-in` numa entrada.** O elemento começa parado e só acelera no fim; o olho lê os
  primeiros 200ms como travamento, não como movimento.
- **Nunca `ease-in-out` em copy.** A rampa lenta nos dois lados faz a frase parecer que
  engasgou no meio. `in-out` é para algo que vai e volta.
- **Nunca easing sobre `scrub`.** A curva já é o dedo do usuário. Empilhar `expo.out` em cima
  do scroll faz a página parecer atrasada em relação ao gesto. Por isso
  `useChapterTimeline.ts` fixa `defaults: { ease: 'none' }` e `scrub: true` sem número.

### Stagger

| Itens | Stagger | Efeito |
|---|---|---|
| 2–3 | 0.10–0.14s | Cada item é um evento separado |
| 4–8 | 0.06–0.08s | Onda legível |
| 9–20 | 0.03–0.04s | Uma coisa só varrendo |
| >20 | `stagger: { amount: 0.6 }` | O total para de crescer com a contagem |

`stagger × contagem ≤ 0.9s`. Doze itens a 0.12s são 1.44s até o último aparecer — o usuário já
rolou para fora antes de ver o fim da onda.

Sempre em ordem de leitura: eyebrow → título → lead → itens → CTA. Nunca `from: 'random'` em
texto — o olho tenta ler na ordem em que as coisas surgem, e aleatório sabota isso.

### Distâncias de deslocamento

| Elemento | Distância | Por quê |
|---|---|---|
| Linha de headline em máscara | `yPercent: 110` | A máscara tem a altura da linha. Em px, a mesma regra quebra quando o `clamp()` da tipografia muda de tamanho |
| Bloco de copy, lead, parágrafo | 22–26px de curso | O suficiente para o olho registrar a chegada sem tirar a frase do lugar onde ela vai ser lida. O sinal é escolha da seção — o modo `block` de `useChapterReveals` desce de `y: -26`; a magnitude é que não negocia |
| Eyebrow / label | `x: -16px` | Entra lateralmente para não competir com o título que sobe ao mesmo tempo |
| Card, item de feature | `y: 40–60px` | Elementos grandes precisam de curso proporcional ou parecem estáticos |
| Item dentro de um stagger | `y: 24px` | O coletivo já carrega o movimento; 60px × 8 itens vira bagunça |
| Beat saindo de quadro | `y: -22 a -30px` + `autoAlpha: 0` | Sai para cima, contra a direção do scroll: lê como "isso já passou" |
| Hover lift | `y: -2 a -4px`, `scale` ≤ 1.02 | Acima disso o cursor descola do elemento e o hover pisca |
| Magnetic | cap de 5px | `useMagnetic.ts`. A mão sente, ninguém percebe conscientemente |
| Parallax de mídia | 8–15% da altura do container | Em px fixos o efeito some no desktop e estoura no mobile |
| Stage / seção inteira | nunca translada | Use `autoAlpha` ou `clip-path`. Transladar a stage move junto tudo que está pinado dentro dela |

Regra que gera as outras: **deslocamento ≈ 0.4–0.8× a altura da linha do elemento.** Um h1 de
5.5rem subindo 22px parece que não se mexeu. Um label de 11px subindo 60px parece que caiu do
teto.

### Opacidade

Sempre `autoAlpha`, nunca `opacity`. `autoAlpha` também escreve `visibility: hidden` no zero,
o que tira o elemento da árvore de acessibilidade e do hit-testing. Com `opacity: 0` puro, um
elemento invisível continua tabulável e continua capturando cliques por cima do que está
visível — o bug mais chato de rastrear numa página com beats sobrepostos.

---

## Orçamento de motion por seção

**No máximo 2 coisas em movimento simultâneo dentro de um viewport.** Uma delas costuma ser o
fundo, o que deixa exatamente uma vaga para o texto.

| O que está se movendo | Custa |
|---|---|
| Um stagger de N itens | 1 |
| Parallax de mídia de fundo | 1 |
| Counter contando | 1 |
| Clip-path / mask reveal | 1 |
| Vídeo em loop, marquee, aurora | 0 — movimento constante, o olho o descarta em ~2s |
| Hover, magnetic, cursor | 0 — só existe sob o ponteiro |

Dois "ambientes" ao mesmo tempo (marquee + aurora + loop) voltam a contar: o que era textura
vira concorrência.

**Sobreposição entre beats:** máximo `0.03` story units — 3% do capítulo, ~130px num
`scroll={4.8}` a 900px. É o que `ChapterCare` usa (`at + CYCLE_BEAT - 0.03`). O beat que sai
começa a sair enquanto o próximo entra, mas só isso. Overlap maior e as duas frases estão
legíveis ao mesmo tempo — nenhuma é lida.

**Variedade:** duas seções consecutivas nunca usam a mesma entrada. Leia a anterior antes de
escolher, e pegue outra linha desta tabela:

| Entrada | Estado inicial | Duração | Ease | Serve para |
|---|---|---|---|---|
| `mask-lines` | `yPercent: 110`, `filter: blur(5px)`, por linha | 1.0, stagger 0.12 (`LINE_DURATION`) | `expo.out` | Headline de capítulo |
| `settle` | `y: ±22–26`, `autoAlpha: 0` | 0.9–1.1 (modo `block`) | `expo.out` | Lead, parágrafo, citação |
| `side-label` | `x: -16`, `autoAlpha: 0` | 0.6 | `expo.out` | Eyebrow, label, contador de seção |
| `stagger-rise` | `y: 24`, `autoAlpha: 0` | 0.6 + stagger da tabela acima | `power3.out` | Grade de cards, lista de features |
| `clip-wipe` | `clipPath: inset(100% 0 0 0)` | 1.2 | `power4.out` | Mídia, faixa, cortina que só abre |
| `scale-settle` | `scale: 0.94`, `autoAlpha: 0` | 0.6 | `power2.out` | Retrato, card único, selo |
| `counter` | valor final já no DOM como texto | 1.2 | `power1.out` | Número, preço, métrica |

Não use a tabela de [choreography.md](../video-to-website/choreography.md) aqui: aquela serve
uma página de canvas frame sequence, os cursos dela (`y: 50–80`) são o dobro destes e ela e esta
skill não podem valer ao mesmo tempo na mesma página.

**Repetição:** a mesma animação usada 3+ vezes na página deixa de ser motion e vira estilo. Isso
é bom em hover e ruim em reveal — um reveal repetido é o que faz uma página cara parecer
template.

---

## Setup — invariantes

O setup de GSAP + Lenis é escrito uma vez e é de `gsap-scrolltrigger-expert` — a tabela linha a
linha está em [a skill dele](../gsap-scrolltrigger-expert/SKILL.md#setup--a-ordem-importa) e a
implementação viva em `src/lib/gsap.ts` e
`src/components/providers/SmoothScrollProvider.tsx`. **Não reescreva nenhum dos dois.**

O que esta skill faz é conferir estas oito invariantes antes de aprovar qualquer motion. Cada
uma quebra de um jeito específico, e o sintoma é como você descobre qual falhou:

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
obsoleto. GSAP e Lenis vêm do `package.json`, nunca de CDN: uma segunda cópia registra um
segundo ScrollTrigger e os dois disputam o mesmo scroller.

`matchMedia` **é** um `gsap.context()` com condição — não empilhe os dois. Sob `reduce` o bloco
nunca roda, nenhuma propriedade é escrita no DOM e o markup fica no estado natural, visível.
Isso é melhor que animar com duração zero, que ainda deixa inline styles para trás.

### O esqueleto de um capítulo

Esta é a parte que **é** desta skill: a timeline autorada em story units. Use
`useChapterTimeline`, que já faz tudo isto. A forma explícita, para entender o que ele faz:

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

### Ordem de medição

`start`/`end` são pixels calculados no momento da criação do trigger. Chame
`ScrollTrigger.refresh()` depois de qualquer coisa que mude a altura do documento: fontes
carregando, canvas montando, imagem sem `aspect-ratio` reservado.

Antes de medir line boxes, espere as fontes — mas com teto. `typographyReady()` em
`src/lib/gsap.ts` faz `Promise.race` entre `document.fonts.ready` e 1600ms: em cache frio as
faces podem demorar segundos, e bloquear nelas deixa o visitante vendo um filme mudo. Quebras
de linha ligeiramente diferentes são uma falha muito menor que copy que nunca chega.

Segurar um elemento na tela é `position: sticky`, não `pin: true` — a comparação completa está
na tabela de `gsap-scrolltrigger-expert`.

---

## Tabela de custo por propriedade

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

### Anti-patterns

- **`setTimeout` / `setInterval` para animar** — não sincroniza com o refresh do display: em
  120Hz dispara cedo, e em aba de fundo o browser throttla para 1 tick por segundo. Use a
  posição na timeline ou `gsap.delayedCall`.
- **`will-change: transform` espalhado no CSS** — cada elemento vira uma camada de composição
  permanente. No mobile isso estoura a memória de GPU e o Safari começa a descartar camadas,
  o que aparece como flash branco. O projeto declara `will-change: transform` em exatamente um
  seletor — `.word__inner`, as palavras que realmente viajam dentro de máscara, e cujo número é
  limitado pelas headlines da página. O GSAP já promove sozinho o que está animando.
- **`filter: blur(0px)` deixado no fim de um reveal** — um blur assentado ainda mantém o texto
  em camada própria e ainda consome VRAM. `useChapterReveals.ts` faz
  `gsap.set(lines.flat(), { clearProps: 'filter' })` no `onComplete` por isso.
- **`gsap.to()` dentro de um `onUpdate` de ScrollTrigger** — cria uma tween nova por frame, e
  cada uma tenta sobrescrever a anterior. Use `gsap.quickTo` (como `useMagnetic.ts`) ou escreva
  a propriedade direto.
- **Um ScrollTrigger por item numa lista de 40** — cada um recalcula `start`/`end` em todo
  refresh e todo resize. Um trigger no container, com `stagger`.
- **`scrub: 1` numérico sobre Lenis** — o Lenis já é o amortecedor. Um número em cima soma dois
  lags e a página parece atrasada em relação ao dedo. `scrub: true`.
- **Reveal que reverte no scroll para cima** — desescreve uma frase debaixo de quem está lendo,
  e faz um link de âncora cair no meio de copy que nunca foi encenada. Reveals tocam uma vez
  (`onEnter`, sem `toggleActions` de reverse); quem encena a batida é a timeline scrubbed.
- **Informação que só existe no movimento** — um número que só aparece contando não existe para
  quem usa reduced motion. O valor final mora no DOM como texto; o counter só o anima.

---

## Checklist de verificação

```bash
npm run typecheck && npm run lint
npm run dev
```

Com a página aberta em `http://localhost:5173`:

- [ ] **DevTools > Rendering > Frame Rendering Stats** — rolar um capítulo inteiro sem cair
      abaixo de 55 fps.
- [ ] **Performance > CPU 4× slowdown** — o scrub ainda acompanha o gesto.
- [ ] **Rendering > Paint flashing** — durante o scroll só a mídia repinta. Texto piscando verde
      significa que alguma propriedade de paint está sendo animada.
- [ ] **Rendering > Layer borders** — contar camadas. Mais de ~30 no mobile é `will-change`
      espalhado.
- [ ] **Rendering > Emulate `prefers-reduced-motion: reduce`** e recarregar — toda a copy
      visível, capítulos empilhados, nenhum `style="transform…"` escrito pelo GSAP no Elements.
- [ ] Navegar de ida e volta e conferir no console que a contagem de triggers é estável.
      Dobrar entre montagens = `revert()` faltando.
- [ ] `Tab` do topo ao rodapé: a ordem de foco segue a ordem de leitura e nenhum foco cai em
      elemento invisível.
- [ ] Nenhuma duração no código entre 1.3s e 19s, ou há a justificativa escrita ao lado.
- [ ] Duas seções consecutivas não usam a mesma entrada.
- [ ] Nenhum viewport tem mais de 2 elementos em movimento simultâneo.
- [ ] Story units convertidos com `scrollMobile`, não só com `scroll`: a 360×640 nenhum beat de
      texto fica abaixo de 300px.
- [ ] Toda timeline de capítulo passou por `seal()` — o último beat termina junto com o capítulo,
      não antes.

```js
// Exponha temporariamente em src/lib/gsap.ts: Object.assign(window, { ScrollTrigger })
ScrollTrigger.getAll().length                      // estável entre navegações
document.documentElement.scrollWidth - innerWidth  // deve ser 0 em 375px
getComputedStyle(document.body).getPropertyValue('--ease-out-expo')
```

---

Receitas prontas (reveal mascarado, hover com `quickTo`, counter com locale, marquee, parallax,
troca de cor de fundo) e a tabela de sintoma → causa estão em [patterns.md](patterns.md).
