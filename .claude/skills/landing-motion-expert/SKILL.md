---
name: "landing-motion-expert"
description: "Use when routing a motion task to the right specialist, defining or auditing a project's motion language, or setting the per-section motion budget. Fase 10 — dona dos tokens de duracao, easing, stagger, distancia e story unit; roteia para gsap-scrolltrigger-expert, video-decisao e motion-ui-expert. Palavras-chave: linguagem de motion, jank, animacao inconsistente entre secoes, variedade de entrada, orcamento de motion, quanto tempo dura."
argument-hint: "[secao-ou-componente]"
---

# Linguagem de motion e roteamento — Fase 10

| | |
|---|---|
| **ENTRADA** | `design/creative-direction.json` (`band`, `points`, `entrance`, `silence` por capítulo); os componentes de capítulo que `product-design-expert` (Fase 10a) já escreveu em `src/components/chapters/`, com os alvos `data-*` marcados; `design/motion-prompts.md` e os arquivos em `design/renders/*.mp4` — a rota muda se existir MP4, e é aqui que se descobre que existe. `src/generated/media.ts` ainda não existe nesta altura: quem o escreve é `video-encode`, depois do roteamento |
| **SAÍDA** | `src/styles/index.css` com os tokens de duração e easing no bloco `@theme`, e a decisão escrita de qual especialista assume cada tarefa, capítulo por capítulo |
| **ANTES** | `frontend-design` (Fase 10b) fixou o caráter visual; `creative-direction-expert` (Fase 7) fixou onde o motion é caro |
| **DEPOIS** | `gsap-scrolltrigger-expert`, `video-decisao`, `motion-ui-expert` — um por vez; depois `audit-responsivo` (Fase 11a) |

Duas responsabilidades e nenhuma implementação: **rotear** o sintoma para o especialista dono e
**fixar os tokens** que todos eles usam. É isso que faz seis capítulos escritos em momentos
diferentes parecerem a mesma página.

A ordem dos especialistas e o Creative Budget estão no [CLAUDE.md](../../../CLAUDE.md) — não se
repetem aqui.

Antes de escolher a entrada de uma seção, leia o componente da seção **anterior** em
`src/components/chapters/`. A regra mais violada do projeto é duas seções consecutivas entrando
do mesmo jeito, e ela só se verifica lendo. Não invente.

## Tabela de roteamento

| Sintoma / necessidade | Especialista | Por quê |
|---|---|---|
| Pin, scrub, parallax, progresso de seção, scroll horizontal | `gsap-scrolltrigger-expert` | Mapear scroll para progresso de timeline é medição de pixel, não estética |
| Existe MP4 no projeto | `video-decisao` primeiro — ver [rota de vídeo](#rota-de-vídeo) | Três técnicas custando de 300 KB a 8 MB. Errar é irreversível depois da extração |
| Hover, botão, card, modal, menu, form, loading, transição de página | `motion-ui-expert` | Responde a input direto: a escala é 0.12–0.4s, outra da do scroll |
| "Que seção vem antes de qual", pacing narrativo | `estrutura-secoes` | Ordem de seção é conversão. Animar a seção errada não conserta a ordem |
| Headline, rótulo de botão, FAQ | `copy-conversao` | Texto não é motion; motion sobre copy errada só move o erro |
| "Qual é o WOW", onde gastar o Creative Budget | `creative-direction-expert` | Decide *o que* merece motion caro antes de existir timeline |
| Espaçamento, tipografia, hierarquia, composição | `product-design-expert` | Hierarquia estática errada, animada, vira erro em movimento |
| Mobile, teclado, leitor de tela, reduced motion | `audit-responsivo` → `audit-acessibilidade` | Portão bloqueante. Nada fecha sem os dois passes |
| Durações/curvas inconsistentes; "parece template" | **esta skill** — [linguagem de motion](#linguagem-de-motion) | É problema de token. Chamar um especialista agora espalha a inconsistência |
| Jank, fps abaixo de 55, layout thrash | **esta skill** — [custo por propriedade](patterns.md#custo-por-propriedade), depois `gsap-scrolltrigger-expert` | Quase sempre é propriedade de layout animada. Troca-se a propriedade, não a timeline |
| ScrollTrigger duplicado, animação só quebra em dev | **esta skill** — [invariantes](patterns.md#invariantes-de-setup) | É Strict Mode sem `revert()`. Nenhum especialista de motion olha ciclo de vida do React |
| A animação não responde a "por que ela existe?" | ninguém — remova | Motion ROI, no CLAUDE.md |

**Um especialista por vez, na ordem do CLAUDE.md.** Chamar `gsap-scrolltrigger-expert` antes de
`estrutura-secoes` gasta uma timeline inteira numa seção que vai ser cortada.

### Rota de vídeo

`video-decisao` é a porta de entrada de qualquer coisa com vídeo: decide se o clipe fica, o que
ele diz e qual técnica o carrega. Ele passa para `video-encode` (o elemento `<video>`, renditions,
poster, gating) ou para `video-to-website` (frame sequence em canvas).

| Papel do vídeo na seção | Técnica | Quem assume |
|---|---|---|
| Ambiência atrás de copy que se lê por cima | `<video muted loop playsInline>` com IntersectionObserver | `video-encode` — referência viva em `src/components/media/ChapterFilm.tsx` |
| A mudança ao longo do tempo **é** a batida da história | canvas frame sequence | `video-to-website`, depois de ler [decision.md](../video-to-website/decision.md) |
| Clipe >30s de movimento essencial, página desktop-only | scrub de `currentTime` | `video-encode` — e aceite o stutter no iOS Safari, que não busca frame a frame em vídeo comprimido |

O teste honesto: *se esta seção fosse uma foto com uma boa legenda, o que se perderia?* Nada →
foto. Só o clima → loop. A transformação → frames.

Uma página recebe **uma** frame sequence. Duas se forem distantes. Três e a página vira download.

---

## Linguagem de motion

Esta skill é dona destes números. Nenhuma outra os redefine. Um valor fora das tabelas abaixo
precisa de justificativa escrita no código, ao lado da linha.

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

### Story units — em timeline scrubbed, duração é fração de scroll

Os capítulos são autorados em unidades de 0 a 1 (`seal()` em `src/lib/gsap.ts`), independentes
da altura em pixels:

| Story unit | Em `<Chapter scroll={4.8}>`, viewport 900px | Serve para |
|---|---|---|
| 0.01 | ~43 px de scroll | Um corte. É o mínimo — abaixo vira glitch, não transição |
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
corte um beat; nunca encolha a unidade. A razão `scrollMobile / scroll` (0,68–0,75) é de
`estrutura-secoes`; aqui ela só é consumida.

### Curvas de easing

| Nome | cubic-bezier | GSAP | Quando |
|---|---|---|---|
| `ease-out-expo` | `cubic-bezier(0.16, 1, 0.3, 1)` | `expo.out` | Entrada de copy e reveals. Chega quase instantâneo e assenta. Já é token em `src/styles/index.css` |
| `ease-out-quint` | `cubic-bezier(0.22, 1, 0.36, 1)` | `power4.out` | UI que precisa parecer rápida sem ser abrupta: menu, drawer, painel |
| `ease-out-quart` | `cubic-bezier(0.25, 1, 0.5, 1)` | `power3.out` | Deslocamento de elemento: hover, magnetic, card lift |
| `ease-out-cubic` | `cubic-bezier(0.33, 1, 0.68, 1)` | `power2.out` | Scale-up e fades onde expo parece exagerado |
| `ease-in-out-quint` | `cubic-bezier(0.83, 0, 0.17, 1)` | `power4.inOut` | Só para movimento que sai **e** volta: cortina, transição de página |
| `ease-out-back` | `cubic-bezier(0.34, 1.56, 0.64, 1)` | `back.out(1.7)` | Confirmação, toggle que "prende". Máximo um por página |
| `linear` | — | `none` | Scrubbed e loops, sempre |

Use o nome GSAP em JS e a variável CSS em CSS. Bezier literal solto no meio do código é o começo
de uma sétima curva que ninguém declarou.

- **Nunca `ease-in` numa entrada.** O elemento começa parado e só acelera no fim; o olho lê os
  primeiros 200ms como travamento, não como movimento.
- **Nunca `ease-in-out` em copy.** A rampa lenta nos dois lados faz a frase parecer que engasgou
  no meio. `in-out` é para algo que vai e volta.
- **Nunca easing sobre `scrub`.** A curva já é o dedo do usuário. Empilhar `expo.out` em cima do
  scroll faz a página parecer atrasada em relação ao gesto — por isso `useChapterTimeline.ts`
  fixa `defaults: { ease: 'none' }` e `scrub: true` sem número.

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
texto: o olho tenta ler na ordem em que as coisas surgem, e aleatório sabota isso.

### Distâncias de deslocamento

| Elemento | Distância | Por quê |
|---|---|---|
| Linha de headline em máscara | `yPercent: 110` | A máscara tem a altura da linha. Em px a regra quebra quando o `clamp()` da tipografia muda |
| Bloco de copy, lead, parágrafo | 22–26px de curso | Registra a chegada sem tirar a frase do lugar onde vai ser lida. O sinal é escolha da seção; a magnitude não negocia |
| Eyebrow / label | `x: -16px` | Entra lateralmente para não competir com o título que sobe ao mesmo tempo |
| Card, item de feature | `y: 40–60px` | Elementos grandes precisam de curso proporcional ou parecem estáticos |
| Item dentro de um stagger | `y: 24px` | O coletivo já carrega o movimento; 60px × 8 itens vira bagunça |
| Beat saindo de quadro | `y: -22 a -30px` + `autoAlpha: 0` | Sai contra a direção do scroll: lê como "isso já passou" |
| Hover lift | `y: -2 a -4px`, `scale` ≤ 1.02 | Acima disso o cursor descola do elemento e o hover pisca |
| Magnetic | cap de 5px (`useMagnetic.ts`) | A mão sente, ninguém percebe conscientemente |
| Parallax de mídia | 8–15% da altura do container | Em px fixos o efeito some no desktop e estoura no mobile |
| Stage / seção inteira | nunca translada | Use `autoAlpha` ou `clip-path`. Transladar a stage move junto tudo que está pinado dentro |

Regra que gera as outras: **deslocamento ≈ 0.4–0.8× a altura da linha do elemento.** Um h1 de
5.5rem subindo 22px parece que não se mexeu. Um label de 11px subindo 60px parece que caiu do teto.

### Opacidade

Sempre `autoAlpha`, nunca `opacity`. `autoAlpha` também escreve `visibility: hidden` no zero, o
que tira o elemento da árvore de acessibilidade e do hit-testing. Com `opacity: 0` puro um
elemento invisível continua tabulável e continua capturando cliques por cima do que está visível
— o bug mais chato de rastrear numa página com beats sobrepostos.

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
`scroll={4.8}` a 900px, que é o que `ChapterCare` usa (`at + CYCLE_BEAT - 0.03`). O beat que sai
começa a sair enquanto o próximo entra, e só isso. Overlap maior e as duas frases estão legíveis
ao mesmo tempo — nenhuma é lida.

**Variedade:** duas seções consecutivas nunca usam a mesma entrada. Leia a anterior e pegue outra
linha desta tabela:

| Entrada | Estado inicial | Duração | Ease | Serve para |
|---|---|---|---|---|
| `mask-lines` | `yPercent: 110`, `filter: blur(5px)`, por linha | 1.0, stagger 0.12 (`LINE_DURATION`) | `expo.out` | Headline de capítulo |
| `settle` | `y: ±22–26`, `autoAlpha: 0` | 0.9–1.1 (modo `block`) | `expo.out` | Lead, parágrafo, citação |
| `side-label` | `x: -16`, `autoAlpha: 0` | 0.6 | `expo.out` | Eyebrow, label, contador de seção |
| `stagger-rise` | `y: 24`, `autoAlpha: 0` | 0.6 + stagger da tabela acima | `power3.out` | Grade de cards, lista de features |
| `clip-wipe` | `clipPath: inset(100% 0 0 0)` | 1.2 | `power4.out` | Mídia, faixa, cortina que só abre |
| `scale-settle` | `scale: 0.94`, `autoAlpha: 0` | 0.6 | `power2.out` | Retrato, card único, selo |
| `counter` | valor final já no DOM como texto | 1.2 | `power1.out` | Número, preço, métrica |

Não use a tabela de [choreography.md](../video-to-website/choreography.md) aqui: ela serve uma
página de canvas frame sequence, os cursos dela (`y: 50–80`) são o dobro destes, e as duas não
podem valer ao mesmo tempo na mesma página.

**Repetição:** a mesma animação usada 3+ vezes deixa de ser motion e vira estilo. Isso é bom em
hover e ruim em reveal — reveal repetido é o que faz uma página cara parecer template.

---

## Setup

O setup de GSAP + Lenis é escrito uma vez e é de `gsap-scrolltrigger-expert`
([a tabela dele](../gsap-scrolltrigger-expert/SKILL.md#setup--a-ordem-importa)), com a
implementação viva em `src/lib/gsap.ts` e `src/components/providers/SmoothScrollProvider.tsx`.
**Não reescreva nenhum dos dois.** O que esta skill faz é conferir as oito invariantes de
[patterns.md](patterns.md#invariantes-de-setup) antes de aprovar qualquer motion — cada uma
quebra de um jeito específico, e o sintoma diz qual falhou.

O esqueleto de um capítulo em story units, que **é** desta skill, está em
[patterns.md](patterns.md#0-esqueleto-de-um-capítulo). Use `useChapterTimeline`, que já o faz.

## Anti-patterns

- **`will-change: transform` espalhado no CSS** — cada elemento vira camada de composição
  permanente; no mobile isso estoura a memória de GPU e o Safari descarta camadas, o que aparece
  como flash branco. O projeto declara em exatamente um seletor, `.word__inner`. O GSAP já promove
  sozinho o que está animando.
- **`setTimeout` / `setInterval` para animar** — não sincroniza com o refresh do display: em 120Hz
  dispara cedo e em aba de fundo o browser throttla para 1 tick por segundo. Use posição na
  timeline ou `gsap.delayedCall`.
- **`gsap.to()` dentro de um `onUpdate` de ScrollTrigger** — cria uma tween nova por frame, cada
  uma sobrescrevendo a anterior. Use `gsap.quickTo` (como `useMagnetic.ts`).
- **Um ScrollTrigger por item numa lista de 40** — cada um recalcula `start`/`end` em todo refresh
  e todo resize. Um trigger no container, com `stagger`.
- **`scrub: 1` numérico sobre Lenis** — o Lenis já é o amortecedor; um número em cima soma dois
  lags e a página parece atrasada em relação ao dedo. `scrub: true`.
- **Reveal que reverte no scroll para cima** — desescreve uma frase debaixo de quem está lendo, e
  faz um link de âncora cair no meio de copy que nunca foi encenada. Reveals tocam uma vez.
- **Informação que só existe no movimento** — um número que só aparece contando não existe para
  quem usa reduced motion. O valor final mora no DOM como texto; o counter só o anima.

A lista completa, incluindo `filter: blur(0px)` esquecido e `scrollTop` via JS, está em
[patterns.md](patterns.md#anti-patterns-de-implementação).

## Verificação

```bash
npm run typecheck && npm run lint && npm run dev
```

Com a página em `http://localhost:5173`:

- [ ] **Rendering > Frame Rendering Stats** — rolar um capítulo inteiro sem cair abaixo de 55 fps
- [ ] **Performance > CPU 4× slowdown** — o scrub ainda acompanha o gesto
- [ ] **Rendering > Paint flashing** — durante o scroll só a mídia repinta; texto piscando verde é
      propriedade de paint sendo animada
- [ ] **Rendering > Layer borders** — mais de ~30 camadas no mobile é `will-change` espalhado
- [ ] **Rendering > Emulate `prefers-reduced-motion: reduce`** e recarregar — toda a copy visível,
      capítulos empilhados, nenhum `style="transform…"` escrito pelo GSAP
- [ ] Navegar de ida e volta: contagem de triggers estável. Dobrar entre montagens = `revert()` faltando
- [ ] `Tab` do topo ao rodapé: ordem de foco = ordem de leitura, nenhum foco em elemento invisível
- [ ] Nenhuma duração entre 1.3s e 19s sem justificativa escrita ao lado
- [ ] Duas seções consecutivas não usam a mesma entrada
- [ ] Nenhum viewport com mais de 2 elementos em movimento simultâneo
- [ ] Story units convertidos com `scrollMobile`: a 360×640 nenhum beat de texto fica abaixo de 300px
- [ ] Toda timeline de capítulo passou por `seal()` — o último beat termina junto com o capítulo

```js
// Exponha temporariamente em src/lib/gsap.ts: Object.assign(window, { ScrollTrigger })
ScrollTrigger.getAll().length                      // estável entre navegações
document.documentElement.scrollWidth - innerWidth  // deve ser 0 em 375px
getComputedStyle(document.body).getPropertyValue('--ease-out-expo')
```

Receitas prontas, invariantes de setup, custo por propriedade e a tabela de sintoma → causa estão
em [patterns.md](patterns.md).
