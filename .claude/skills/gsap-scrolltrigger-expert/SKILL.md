---
name: gsap-scrolltrigger-expert
description: Use when building scroll behaviour with GSAP ScrollTrigger — scrubbed timeline, parallax, pinned section, reveal on scroll, horizontal scroll, card stacking, mask/clip reveal, background colour handoff, word-by-word text — or wiring Lenis smooth scroll to the GSAP ticker. Também para trigger que dispara no lugar errado, pin que quebra layout ou scroll travado. Keywords: scrub, pinSpacing, sticky, marcadores, markers, refresh, matchMedia.
argument-hint: [secao-ou-efeito]
---

# GSAP ScrollTrigger

| | |
|---|---|
| **ENTRADA** | `design/creative-direction.json` (`scroll` e `scrollMobile` por capítulo, `points`, `entrance`); os tokens de duração e easing que `landing-motion-expert` gravou em `src/styles/index.css`; os componentes em `src/components/chapters/`; `src/lib/gsap.ts` e `src/components/providers/SmoothScrollProvider.tsx` |
| **SAÍDA** | os hooks de scroll em `src/hooks/` e o `<Chapter>` de cada capítulo com trigger, `start`, `end` e `scrub` escritos — mais o bloco "Chapter mechanics" de `src/styles/index.css` quando o pin é CSS |
| **ANTES** | quem chama é `landing-motion-expert` (Fase 10c), toda vez que o scroll controla algo: pin, scrub, parallax, progresso de capítulo, trilho horizontal. `video-decisao` também chama, para o fallback `mode="scrub"` |
| **DEPOIS** | `audit-responsivo` (Fase 11a) mede `scrollMobile` e pin no celular; `audit-acessibilidade` (Fase 11b) verifica que nenhuma condição do `matchMedia` casa sob `prefers-reduced-motion` |

Fora do trilho linear: esta skill não tem uma fase própria e não roda sozinha. Ela é a
implementação para a qual a Fase 10c roteia.

Scroll é uma linha do tempo com uma cabeça de leitura que o visitante move. Esta skill decide
**como** o scroll se comporta: onde um efeito começa, quanto scroll ele consome, e o que ele
custa em milissegundos.

O que ela não decide: quais seções existem (`estrutura-secoes`), o que merece um
efeito (`creative-direction-expert`), como um botão reage ao mouse (`motion-ui-expert`), nem
quais durações, curvas, distâncias e stagger a página inteira usa — a linguagem de motion é do
`landing-motion-expert`. Os números daqui são só os de scroll: onde disparar, quanto scroll
consumir, quando pinar. O portão final de mobile é `audit-responsivo`; o de teclado e leitor de
tela, `audit-acessibilidade`.

**Esta skill é dona do setup** — registro do GSAP, instanciação do Lenis, wiring do ticker e
ciclo de vida em React. Quem audita se ele está correto é o `landing-motion-expert`, pela tabela
de invariantes dele; quem escreve o código é esta.

## Modelo mental

ScrollTrigger não anima nada. Ele mede duas posições — uma no elemento, uma na viewport — e
converte a distância entre elas em um `progress` de 0 a 1. Tudo o mais é consequência.

Duas formas de segurar um elemento na tela, e a escolha errada custa horas:

| | `position: sticky` (CSS) | `pin: true` (ScrollTrigger) |
|---|---|---|
| Quem mede | ninguém — o navegador | ScrollTrigger, a cada `refresh()` |
| Efeito no DOM | nenhum | insere um `pin-spacer` em volta do elemento |
| Resize / URL bar mobile | grátis | re-medição, risco de salto |
| Reduced motion | `height: auto` no CSS desmonta tudo | precisa matar o trigger |
| Quando usar | o elemento é filho direto de uma seção alta que você controla | o elemento não pode virar filho direto, ou o pin precisa começar longe do topo |

**Neste repositório o pin é CSS.** `.chapter` é alto, `.chapter__stage` é `sticky; top: 0;
height: 100svh`, e o ScrollTrigger só lê progresso com `start: 'top top'` / `end: 'bottom
bottom'`. Ver `src/styles/index.css` (bloco "Chapter mechanics") e `src/components/ui/Chapter.tsx`.
Use `pin: true` apenas quando o sticky não resolve — a receita de scrub horizontal é o caso.

## Setup — a ordem importa

O registro do GSAP, a instanciação do Lenis e o ciclo de vida em React são desta skill, e já
estão implementados em `src/lib/gsap.ts` e `src/components/providers/SmoothScrollProvider.tsx`.
Não reescreva nenhum dos dois — estenda. As quatro linhas que existem por causa do ScrollTrigger,
e o que quebra sem cada uma:

| Linha | O que quebra sem ela |
|---|---|
| `ScrollTrigger.config({ ignoreMobileResize: true })` | a barra de URL retrátil muda a viewport no meio do scroll, todo capítulo re-mede e salta durante um flick |
| `lenis.on('scroll', () => ScrollTrigger.update())` | o ScrollTrigger lê a posição um frame atrás da que o Lenis acabou de escrever — num scrub, isso é exatamente a sensação de borracha |
| `gsap.ticker.add((t) => lenis.raf(t * 1000))` | dois loops de rAF no mesmo frame sem ordem garantida: em metade dos frames o progresso vem da posição anterior. É o micro-jitter |
| `gsap.ticker.lagSmoothing(0)` | o ticker "recupera" um frame longo ajustando o delta, e num scrub isso injeta um salto de posição em vez de uma pausa honesta |

O ticker do GSAP conta em segundos e `lenis.raf` espera milissegundos — daí o `* 1000`. Desmonte
na ordem inversa (`ticker.remove` → `lenis.off` → `lenis.destroy`): o Strict Mode do React 19
monta duas vezes e sem isso sobra um ticker e um listener órfãos por montagem.

`ScrollTrigger.scrollerProxy()` só é necessário quando o Lenis usa `wrapper`/`content`
customizado. Com o default (a janela) não use — é uma camada a mais para dessincronizar.

### Todo efeito dentro de um context

`gsap.matchMedia(scope)` **já é** um `gsap.context()`. Não empilhe os dois. Um `mm.revert()`
no cleanup mata triggers, tweens e estilos inline escritos por dentro dele.

`useLayoutEffect`, não `useEffect`: o estado inicial (`gsap.set` com `autoAlpha: 0`) precisa
ser escrito antes do primeiro paint, senão a copy pisca visível e depois some. O hook completo,
com o `scope` do `ref` e o `mm.revert()` no retorno, está em
[recipes.md](recipes.md#esqueleto-de-matchmedia).

## START / END — a tabela

`start: "A B"`. **A é um ponto no elemento. B é um ponto na viewport.** Uma porcentagem em A
é da altura do elemento; em B é da altura da viewport, medida do topo. Confundir os dois é a
fonte número um de trigger que dispara no lugar errado.

`top 80%` não significa "80% do elemento". Significa: o topo do elemento chegou à linha que
está a 80% da altura da tela — ou seja, 20% acima da base. O elemento acabou de entrar.

| `start` | O que se vê no instante do disparo | Usar para |
|---|---|---|
| `top bottom` | topo do elemento encosta na base da tela — primeiro pixel visível | começar parallax, pré-carregar |
| `top 90%` | elemento entrou uns 10% da tela | camadas de fundo |
| `top 80%` | elemento entrou 20% da tela | **reveal de copy — o default** |
| `top center` (= `top 50%`) | topo do elemento na metade da tela | bloco grande, imagem cheia |
| `top top` | topo do elemento no topo da tela | início de pin / stage sticky |
| `center center` | elemento centrado na tela | pin de peça única mais baixa que a tela |
| `bottom bottom` | base do elemento na base da tela | fim natural de uma seção alta |
| `bottom top` | elemento saiu inteiro por cima | limpar, matar, soltar |
| `top+=200 top` | 200px depois do topo do elemento | atrasar o início sem mudar o trigger |

Para `end`:

| `end` | Significa |
|---|---|
| `bottom bottom` | acaba quando a seção acaba — par natural de `top top` |
| `+=600` | 600px de scroll depois do start |
| `+=100%` | uma altura de tela de scroll depois do start |
| `() => '+=' + (track.scrollWidth - innerWidth)` | valor medido, reavaliado a cada refresh |

Valores literais são congelados em px no refresh. **Valor que depende de layout tem que ser
função** — e o trigger precisa de `invalidateOnRefresh: true` para os tweens recalcularem junto.

## SCRUB

| Valor | Sensação | Quando |
|---|---|---|
| `true` | 1:1, sem atraso — o dedo é a cabeça de leitura | **default neste repo.** Lenis já é o amortecedor |
| `0.5` | persegue o scroll, alcança em ~0.5s | scroll nativo sem Lenis, movimento leve |
| `1` | peso perceptível, inércia | scroll nativo, camadas grandes de fundo |
| `1.5–2` | arrasta visivelmente | só atmosfera de fundo. Em texto lê como travamento |
| ausente | a animação tem clock próprio; o scroll só dá o start | **reveal de copy, contadores, entradas** |

Regra deste projeto: com Lenis ativo, `scrub: true`. Um scrub numérico sobre o Lenis empilha
dois amortecedores e o resultado não é suavidade, é lag — está escrito em `useChapterTimeline.ts`.

Dentro de timeline com scrub, `defaults: { ease: 'none' }`. O scroll já é a curva; um ease em
cima cria uma segunda aceleração e a seção parece dessincronizada do gesto. Ease só em beats
curtos onde o desalinhamento é intencional (o rail de `ChapterJourney` usa `power2.inOut` em
28% de cada passo, e os outros 72% são scroll morto de propósito — o card parado, sendo lido).

## PIN

**Quanto scroll.** O pin consome exatamente `end - start`. Cada beat legível pede **60–100vh**.
Abaixo de 50vh o conteúdo passa antes de ser lido; acima de 150vh vira scroll morto e o
visitante acha que a página travou. Três beats = ~300vh de seção. Este repo declara isso por
capítulo em `<Chapter scroll={6} scrollMobile={4.2}>` — sempre menos no celular.

**pinSpacing.** `true` (default) insere um espaçador da altura do pin, e o conteúdo seguinte
continua no lugar certo. `false` faz a próxima seção deslizar por cima do pin — só use quando
o que vem depois está posicionado por cima de propósito (um overlay), nunca em fluxo normal.

**Como não quebrar o layout.**

- O elemento pinado é envolvido num `pin-spacer`. Qualquer CSS que dependa do pai direto —
  `:first-child`, `gap` do grid pai, um `sticky` acima — deixa de valer. **Pine um wrapper,
  nunca o elemento que carrega margens ou é item de grid.**
- Nunca pine dentro de um ancestral com `overflow: hidden` — `position: fixed` não escapa e o
  elemento simplesmente some.
- Um ancestral com `transform` (inclusive `translateZ(0)`) vira o containing block de `fixed`
  e o pin fica preso dentro dele.
- `anticipatePin: 1` quando aparece um flicker de um frame na entrada em scroll rápido.

**Quando NÃO pinar.**

- Conteúdo pinado mais alto que a viewport: vira armadilha de scroll, e no celular não há como
  chegar ao que ficou embaixo.
- O efeito é só "segurar o fundo": `position: sticky` faz isso sem medir nada.
- Mais de 2 pins na página. Cada pin é uma pausa cognitiva; três seguidas e o visitante para.
- Abaixo de 768px, a menos que a seção tenha um único beat.

## ScrollTrigger.refresh()

ScrollTrigger congela start e end em pixels no momento do refresh. Ele refaz isso sozinho nos
eventos de `autoRefreshEvents` — por padrão `visibilitychange`, `DOMContentLoaded`, `load` e
`resize`, que já cobre a rotação do aparelho. **Não** refaz quando a altura do documento muda
por outro motivo:

| Mudou | Chamar refresh depois de |
|---|---|
| Fontes web carregaram e mudaram as quebras de linha | `document.fonts.ready` (com timeout — ver `typographyReady()`) |
| Imagem sem `width`/`height` reservados terminou de carregar | `onLoad` da última imagem acima do trigger |
| Conteúdo assíncrono montou (frames, lista, dados) | o `setState` que insere o conteúdo |
| Accordion/detalhe abriu ou fechou | o fim da transição de altura |
| Rota trocada em SPA | depois do commit da nova rota |

Uma chamada, depois do layout estabilizar. `refresh()` recalcula **todos** os triggers da
página — chamá-lo dentro de um `onUpdate`, ou uma vez por elemento de uma lista, é o caminho
mais rápido para uma página que engasga ao carregar.

`invalidateOnRefresh: true` é obrigatório sempre que um tween usa valor em função
(`x: () => -track.scrollWidth`). Sem ele o tween mantém o valor antigo e o refresh corrige
só os pontos de disparo, não o destino.

## matchMedia como padrão

Três condições, sempre as mesmas, sempre juntas. Reduced motion não é uma seção no rodapé —
é uma condição que simplesmente não constrói a animação. O esqueleto completo — objeto de
condições nomeadas, leitura de `context.conditions`, travel de desktop e mobile no mesmo tween,
`mm.revert()` no cleanup — está em
[recipes.md](recipes.md#esqueleto-de-matchmedia).

Não existe branch `reduced`. Nenhuma condição casa, nada é criado, nada é escrito no DOM — e
por isso **o markup tem que ser legível no estado natural**. Toda animação de entrada usa
`gsap.set()` inicial ou `.from()` *dentro* do guard. Nenhuma opacidade zero no CSS.

O CSS fecha o resto: sob `prefers-reduced-motion: reduce`, `.chapter { height: auto }` e
`.chapter__stage { position: relative }` transformam os stages pinados em seções empilhadas
normais. Ver o bloco final de `src/styles/index.css`.

Desktop e mobile não são a mesma animação mais fraca. Mobile: menos deslocamento (metade do
travel), menos beats por capítulo, zero pin de várias telas, e `scrollMobile` entre 0,68 e 0,75
× `scroll` — faixa de
[estrutura-secoes](../estrutura-secoes/SKILL.md#convertendo-share-em-scroll), que é dona
desse número; aqui ele é consumido, não redefinido.

## Receitas

Código completo, com números e custo, em [recipes.md](recipes.md).

| Receita | Quando usar | Scrub | Custo |
|---|---|---|---|
| Reveal escalonado | copy entrando numa seção | não | 1 trigger por grupo |
| Parallax por camada | profundidade em mídia de fundo | `true` | 1 trigger, só transform |
| Seção pinada com timeline interna | 2–4 beats dentro de um quadro | `true` | sticky: zero. `pin: true`: spacer + re-medição |
| Scrub horizontal | galeria ou processo lateral | `true` | pin + leitura de `scrollWidth` (reflow no refresh) |
| Card stacking | 3–5 itens comparáveis | `true` | 1 trigger por card — só até 5 |
| Mask / clip reveal | entrada de imagem ou vídeo | não | wrapper + `yPercent` é só transform; `clip-path` não faz layout, mas repinta o elemento a cada frame |
| Cor de fundo entre seções | virada de capítulo | não (toggle) | repaint de tela cheia por transição |
| Texto por palavra | headline, ≤ 20 palavras | não | 2 nós DOM por palavra |

## Depuração

`markers: true` no trigger. Aparecem quatro: `start`/`end` do **elemento** e `scroller-start`/
`scroller-end` na **viewport**. O disparo acontece quando os dois `start` se cruzam. Ler nessa
ordem economiza a maior parte do tempo:

1. Os marcadores do scroller estão onde você espera na tela? Se não, o segundo valor da string
   está errado.
2. Os marcadores do elemento estão no elemento certo? Se estão numa seção inteira quando você
   queria um parágrafo, o `trigger` é o wrapper.
3. Os marcadores se deslocam depois que uma fonte ou imagem carrega? Falta `refresh()`.

Os 12 sintomas mais frequentes com a causa de cada um, e como tirar o perfil no Performance com
4× de throttle, estão em [recipes.md](recipes.md#tabela-de-sintoma-e-causa).

## Anti-patterns

- **Um ScrollTrigger por item numa lista de 40** — cada trigger guarda start/end e é
  recalculado em *todo* `refresh()`, e cada recálculo lê `getBoundingClientRect`, forçando
  reflow. Quarenta itens = quarenta reflows a cada resize e a cada fonte que carrega. Um
  trigger no container + `stagger` faz o mesmo com um.
- **Escrever no DOM dentro de `onUpdate` sem guard** — `onUpdate` roda a cada tick de scroll,
  até 120 vezes por segundo em tela de alta taxa. Sem comparar com o valor anterior, você
  repinta sem mudança. `if (i === current) return`.
- **Criar tween dentro de `onUpdate`** — cada chamada instancia um tween novo competindo com o
  anterior pelo mesmo alvo. Crie a timeline uma vez e mova `progress`.
- **`ease` nos tweens de uma timeline com scrub** — o scroll já é a curva; a segunda aceleração
  faz a seção parecer descolada do gesto.
- **Animar `width`, `height`, `top`, `left`, `margin`, `padding`** — cada frame dispara layout
  na main thread para toda a árvore. `transform` e `opacity` ficam no compositor. Diferença
  medível: uma barra roxa por frame no Performance vs. nenhuma.
- **`filter: blur()` que fica** — mesmo um `blur(0px)` assentado mantém o elemento numa camada
  própria e ocupando VRAM. Limpe com `gsap.set(el, { clearProps: 'filter' })` no `onComplete`,
  como faz `useChapterReveals`.
- **`will-change` espalhado no CSS** — cada nó vira camada de composição permanente. Passando de
  ~30 camadas no celular o navegador começa a descartá-las, e o descarte aparece como flash
  branco. O GSAP promove sozinho o que está animando. A única exceção do repo é `.word__inner`
  em `src/styles/index.css`: são poucos nós, todos de fato animados, e a máscara depende da
  promoção para não serrilhar. O custo de cada propriedade está tabelado no
  `landing-motion-expert`.
- **Reveal de copy com scrub reversível** — rolar de volta desescreve a frase sob o leitor, e
  quem chega por âncora no meio da página encontra texto que nunca foi acionado. Copy: trigger
  dispara, animação tem clock próprio, `once: true`.
- **`.from()` num grupo sem `gsap.set()` inicial explícito** — `from` grava o estado atual como
  destino no momento da criação. Se a timeline for reconstruída depois de já ter rodado, o
  destino é o estado errado e o grupo fica meio animado.
- **`ScrollTrigger.refresh()` por elemento** — cada chamada re-mede *todos* os triggers da
  página. Vinte imagens chamando no `onLoad` são 20 × N medições, cada uma lendo
  `getBoundingClientRect`. O engasgo cai exatamente no carregamento, quando a thread já é a mais
  disputada. Um `refresh()`, depois que o layout parar de mudar.
- **Lenis com `scroll-behavior: smooth` no CSS** — dois donos do scroll disputando a mesma
  posição. O repo força `scroll-behavior: auto`.
- **`100vh` num stage pinado no celular** — a barra de URL retrátil muda `vh` e tudo re-mede.
  `100svh` para texto, `100lvh` para mídia de fundo, para nunca aparecer uma faixa vazia.
- **Parallax acima de ~120px de deslocamento** — a camada descola do conteúdo e a borda da
  imagem aparece. O overscan necessário é `2 × travel`; a partir daí você está pagando pixels
  para esconder um erro.

## Verificação

- [ ] `markers: true` removido de todo trigger antes de commitar
- [ ] Um `refresh()` depois das fontes; marcadores não se deslocam ao recarregar com cache frio
- [ ] Strict Mode: montar/desmontar duas vezes não duplica trigger — `ScrollTrigger.getAll().length`
      igual antes e depois
- [ ] `prefers-reduced-motion: reduce` no DevTools (Rendering → Emulate CSS media): capítulos
      empilham, toda a copy visível, Lenis não instanciado, nenhum pin
- [ ] 375px de largura: nenhum pin de várias telas, travel pela metade, `scrollMobile` menor
- [ ] Performance com 4× CPU throttle: sem barras de Layout durante o scrub
- [ ] Redimensionar a janela no meio de uma seção pinada não salta nem sobrepõe
- [ ] Navegar por âncora para o meio da página: nada fica invisível esperando um trigger
- [ ] Teclado: `Tab` e `PageDown` chegam ao fim da página — nenhum pin intercepta teclas
