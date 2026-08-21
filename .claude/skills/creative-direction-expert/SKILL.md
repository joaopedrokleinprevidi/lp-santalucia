---
name: creative-direction-expert
description: Use when setting or auditing how ambitious a landing page experience is, before anything is animated. Fases 5 e 7: fixa Experience Score, Creative Budget, profundidade em viewports, pontos de complexidade por capitulo, banda da curva 20-40-60-80-100, quais WOW moments a pagina ganha e o fallback de cada um, capitulo de silencio, variedade de entrada, Motion ROI. Tambem quando a pagina parece toda igual ou sem momento marcante.
argument-hint: [passe-1-ou-2] [experience-score-alvo]
allowed-tools: Read, Glob, Grep, Write, Edit, Bash(node *)
---

# Direção criativa — Fases 5 e 7

| | |
|---|---|
| **ENTRADA** | Passe 1: `design/briefing.json`, `design/design-system.json`, `design/pesquisa.md` e `design/inventario.json` — o inventário de mídia da Fase 1, incluindo quais MP4 existem em `assets-source/`. Passe 2: `design/estrutura.md` e `src/data/story.ts` — o story map com `share`, `scroll` e `scrollMobile` já repartidos |
| **SAÍDA** | `design/creative-direction.json`. Passe 1 grava `experienceScore`, `rationale`, `depth` total, `budget` e a contagem de WOW por tier; passe 2 completa `chapters[]` e `wow[]` |
| **ANTES** | Passe 1: `niche-research` (Fase 4). Passe 2: `copy-conversao` (Fase 6b) |
| **DEPOIS** | Passe 1: `estrutura-secoes` (Fase 6a). Passe 2: `prompt-imagem` (Fase 8a) |

Esta skill escreve **um arquivo e nenhum componente**. Sem ele, três capítulos seguidos entram
com o mesmo fade, o momento mais caro cai no hero e a curva só desce dali em diante, e ninguém
consegue dizer se a página está pronta porque não existe número para comparar.

A regra de ouro: *você decide o quê e o quanto; nunca o como.* Se estiver escrevendo
`gsap.to(...)`, saiu do papel.

## Por que duas passadas

O `scroll` de cada capítulo só existe depois que `estrutura-secoes` reparte o `share`. Então a
direção roda partida em duas:

| Passe | Quando | O que fica decidido |
|---|---|---|
| **1 — orçamento** | Fase 5, antes da estrutura | `experienceScore`, `depth` **total** (uma faixa, não por capítulo), `budget`, quantos WOW de cada tier a página ganha e em que **posição** da curva o pico cai |
| **2 — ratificação** | Fase 7, com o story map pronto | `band` de cada capítulo pela posição medida do meio dele, `points`, `entrance`, `silence`, e a qual capítulo cada WOW pertence |

`scroll` e `scrollMobile` **nunca são inventados aqui**: vêm do `scrollProp()` de
[estrutura-secoes](../estrutura-secoes/SKILL.md#convertendo-share-em-scroll), que é a dona dos
dois números. Capítulo que não couber no teto de pontos da sua band devolve o problema para lá
repartir o `share` de novo. Duas skills escrevendo `scroll` produzem duas páginas de altura
diferente, e nenhuma das duas erra sozinha.

## A unidade de medida: ponto de complexidade

Um ponto é **um alvo distinto que a timeline do capítulo escreve** — não um `tween`, não uma
linha de código.

| O que a cena tem | Pontos |
|---|---|
| Um reveal de copy (linhas ou bloco) | 1 |
| Um elemento com transform scrubado (parallax, rail, thread, push-in de mídia) | 1 |
| Um contador ou índice que avança | 1 |
| Um grupo que entra junto em stagger | 1 |
| Marquee | 1 |
| Vídeo em loop ou `once` de fundo | 2 |
| Troca de cor de fundo ou veil scrubada | 2 |
| Rail, stack ou ciclo com N itens cronometrados um a um | N |
| Vídeo com `currentTime` dirigido por scroll | 4 |
| Sequência de frames em canvas | 6 |

Conte à mão pela tabela. Não existe comando que conte pontos, porque um alvo costuma receber
dois ou três tweens e um tween dentro de um `forEach` de 5 cartões conta 1 no texto e 5 no
runtime. Os comandos que **comparam** capítulos entre si estão em [handoff.md](handoff.md).

**Profundidade de scroll** é a segunda medida: `1 + scroll` viewports por capítulo, somados. O
`1` é o stage sticky que todo capítulo ocupa (`.chapter` em `src/styles/index.css`). Em runtime:
`document.body.scrollHeight / window.innerHeight`.

## Passo 1 — Fixar o Experience Score

**Artefato:** `"experienceScore": 4` no topo do JSON, mais a justificativa em uma frase. O
default do projeto é ★★★★☆; ★★★★★ exige que o cliente tenha material (vídeo, fotografia
dirigida) que sustente o pico.

Um nível só é atingido quando **todas** as colunas batem. Não é média: um item abaixo da faixa
derruba o nível.

| Score | WOW major | medium | small | Entradas distintas | Profundidade (desktop) | Pontos totais | Mídia (desktop / mobile) | Scroll storytelling |
|---|---|---|---|---|---|---|---|---|
| ★☆☆☆☆ | 0 | 0 | ≤3 | 1 | 2–4 | ≤6 | ≤1,5 MB / ≤0,8 MB | não |
| ★★☆☆☆ | 0 | 0–1 | 3–5 | 2 | 4–8 | 6–14 | ≤2,5 MB / ≤1,2 MB | não |
| ★★★☆☆ | 0 | 1–2 | 5–8 | 3 | 8–14 | 14–26 | ≤4 MB / ≤2 MB | opcional |
| ★★★★☆ | 1 | 2 | 6–10 | ≥4 | 14–26 | 26–42 | ≤8 MB / ≤3,5 MB | ≥3 capítulos com stage sticky |
| ★★★★★ | 1–2 | 2–3 | 8–14 | ≥6 | 26–36 | 42–60 | ≤12 MB / ≤5 MB | a página inteira |

Duas restrições valem em qualquer nível:

- **Antes do primeiro scroll, no máximo 4 MB carregados.** Na referência é o filme do hero
  (3,2 MB desktop / 1,0 MB mobile); o resto entra por `IntersectionObserver` com
  `rootMargin: '150% 0px'`.
- **Piso de reduced motion:** com `prefers-reduced-motion: reduce` a página degrada, mas **nunca
  abaixo de ★★☆☆☆** — toda copy visível, toda seção alcançável, todo item de rail acessível. O
  stylesheet entrega o piso (`[data-rail] { flex-wrap: wrap }`, `.chapter { height: auto }`); o
  seu trabalho é não planejar nada que dependa de motion para existir.

Acima de 36 viewports a página deixa de ser experiência e vira travessia: o CTA está no fim e
cada viewport a mais é uma chance de sair antes dele. Passou de 36, corte um capítulo — não
corte a qualidade dos que ficam.

## Passo 2 — Distribuir o Creative Budget

**Artefato:** `chapters[]`, cada um com `scroll`, `points` e `band`.

A progressão 20 → 40 → 60 → 80 → 100 do CLAUDE.md é uma curva sobre a **profundidade acumulada**,
não sobre o número de seções. A `band` é lida pela posição do **meio** do capítulo na
profundidade total, e cada faixa tem teto de pontos:

| Faixa (posição do meio do capítulo) | Intensidade | Pontos por capítulo | O que cabe |
|---|---|---|---|
| 0–20% — abertura | 20 | 4–7 | Mask reveal da headline, um filme em loop, o scroll cue. Nenhum WOW major. |
| 20–45% — primeira escalada | 40 | 5–11 | O primeiro WOW medium, e o capítulo de silêncio logo antes dele. |
| 45–65% — platô | 60 | 5–9 | O corpo do argumento. Entradas variadas, nenhum WOW novo. |
| 65–85% — pico | 80–100 | 9–14 | O WOW major vive aqui, e só aqui. |
| 85–100% — fecho | 30–40 | 3–5 | Silêncio, a linha final e o CTA. |

Calibração medida na página de referência deste repositório. As colunas `scroll` /
`scrollMobile` são do story map, repetidas só para as colunas de ponto e faixa terem contra o
que ser lidas; divergiu, a do story map vence.

| Capítulo | `scroll` / `scrollMobile` | Profundidade | Meio em | Pontos | Faixa |
|---|---|---|---|---|---|
| `inicio` | 4.6 / 3.2 | 5,6 | 8% | 6 | abertura |
| `experiencia` | 3.2 / 2.2 | 4,2 | 22% | 5 | escalada (silêncio) |
| `jornada` | 6 / 4.2 | 7,0 | 38% | 11 | escalada (WOW medium) |
| `consulta` | 4 / 2.8 | 5,0 | 55% | 6 | platô |
| `cuidado` | 4.8 / 3.4 | 5,8 | 71% | 13 | **pico (WOW major)** |
| `clinica` | 3.2 / 2.4 | 4,2 | 85% | 5 | fecho (silêncio) |
| `agendar` | 2 / 1.4 | 3,0 | 96% | 4 | fecho |
| | | **34,8** | | **50** | ★★★★★ |

Duas regras derivadas dessa tabela:

- **Scroll por beat:** 0,8–1,0 viewport para cada beat que o capítulo revela; mínimo 2, máximo 6.
  `jornada` gasta 6 para 7 beats — 0,86 por beat. `agendar` tem um beat e recebe o piso.
- **Mobile entre 0,68 e 0,75 × desktop** — faixa de
  [estrutura-secoes](../estrutura-secoes/SKILL.md#pacing). Aqui ela só é conferida: capítulo
  fora dela conta outra história no celular, e aí é outro plano, não um ajuste.

## Passo 3 — Alocar os WOW moments

**Artefato:** o array `wow`, cada entrada com `tier`, `chapters`, `cost` e — obrigatoriamente —
`fallback`.

| Tier | Custo típico | Scroll que consome | Quando vale |
|---|---|---|---|
| **major** | 0–8 MB, 80–160 LOC, risco alto de regressão | 4–6 viewports | Quando a mudança ao longo do tempo **é** o conteúdo. 0 MB quando o mecanismo é CSS puro — o pin de `.chapter` não baixa nada. |
| **medium** | 0–1,5 MB, 40–70 LOC | 3–5 viewports | Quando o conteúdo já tem estrutura (sequência ordenada, itens comparáveis) e o motion só a torna legível. |
| **small** | 0 KB, 5–30 LOC | 0 viewports | Quando cabe dentro de um beat que já existe. Nunca cria beat próprio. |

**Quantos de cada** é a linha do Score no Passo 1, não um número fixo por tier. Um filme de
fundo conta como medium mesmo quando ninguém o chamou de WOW — se pesa megabytes e o capítulo
depende dele, ocupa uma vaga de medium.

Os 22 momentos do catálogo, com custo em KB, LOC e viewports, o especialista dono e o fallback
de cada um: [wow-catalog.md](wow-catalog.md).

**Um WOW sem `fallback` descrito não entra no plano.** Sem ele, quem usa reduced motion recebe
uma página com buraco exatamente no momento principal — e é sempre o principal, porque é o único
que dependia inteiramente de movimento. Fallback é frase concreta: *"a sequência de frames vira
o poster do frame 1, e os cinco princípios viram uma lista empilhada"*, não *"degrada com
elegância"*.

Ordem de alocação, nesta sequência:

1. Escolha o **major** e prenda-o na faixa de pico (65–85%). Ele define o resto.
2. Escolha os **medium** separados do major por pelo menos um capítulo. A exceção é o device
   recorrente — um filme de fundo em três capítulos é uma entrada só, e um deles pode ser o do
   pico; o que se proíbe é **dois momentos distintos** colados.
3. Distribua os **small** nos capítulos que sobraram, inclusive nos de silêncio, onde um único
   small é exatamente o suficiente.
4. Some `cost.kb` e compare com `budget.mediaDesktopMB`. Todo arquivo de mídia tem de estar em
   alguma entrada ou em `eagerMB`. MB sem dono é MB que ninguém corta quando o orçamento
   estoura, porque ninguém sabe de quem ele é.

## Passo 4 — Mapa de densidade

**Artefato:** `silence: true` em pelo menos um capítulo a cada quatro.

- **No máximo dois capítulos consecutivos acima de 7 pontos.** O terceiro fica em ≤5.
- **Silêncio é definido, não sentido:** ≤5 pontos e ≤3,2 viewports de `scroll`, sem WOW major nem
  medium próprio, sem troca de cor de fundo scrubada. Um device recorrente que só passa por ali
  não descaracteriza o silêncio — mas os pontos dele contam, e são 2.

Na referência os capítulos de silêncio são `experiencia` e `clinica`, um antes de cada pico.
**O pico só é percebido como pico porque o capítulo anterior baixou.** Uma página em que todos
os capítulos têm 9 pontos não é intensa, é plana num nível alto: o visitante calibra pelo que
acabou de ver, e depois de três capítulos densos o quarto denso é o normal.

Scroll morto **com um beat parado no frame** é tempo de leitura — é o que `TRAVEL_SHARE = 0.28`
compra em `ChapterJourney`, onde 72% do orçamento de cada passo é o cartão parado sendo lido.
Scroll morto **sem nada parado no frame** é uma página que parece travada.

## Passo 5 — Variedade de entrada

**Artefato:** `entrance` de cada capítulo, diferente do anterior.

- Nenhuma entrada se repete em **capítulos consecutivos**.
- Nenhuma entrada aparece mais de **duas vezes** na página inteira.
- ★★★★☆ exige ≥4 tipos distintos; ★★★★★ exige ≥6.

As sete da referência, todas distintas: `mask-lines-late` (linhas sobem da máscara, mas só a 65%
e 86% do capítulo), `intro-exit-stagger` (o intro sai por cima, os pilares sobem em stagger),
`rail-steps` (rail horizontal que avança em passos, não em glide), `media-push-in`
(`scale: 1 → 1.05`), `cycle-replace` (um item por vez substitui o anterior), `media-pull-back`
(`scale: 1.08 → 1`), `block-settle` (bloco único, sem stagger — a página para de falar).

`consulta` e `clinica` usam o mesmo mecanismo com o sinal invertido. Isso conta como duas
entradas, e só funciona porque há dois capítulos entre elas. Coladas, seriam a mesma coisa duas
vezes.

## Passo 6 — Teste de Motion ROI

**Artefato:** as animações reprovadas saem do plano antes do handoff. Nenhuma sobrevive "para
decidir depois" — decidir depois significa implementada.

> Se eu remover esta animação e deixar o elemento no estado final, **o que o visitante perde?**

**Aprova** se a resposta couber em uma frase e for uma destas: perde a **ordem de leitura** (três
elementos chegam ao olho no mesmo instante); perde **informação** (o movimento é o conteúdo —
o rail que avança, o número que conta); perde **continuidade** (duas cenas trocam com corte
seco); perde **hierarquia** (o secundário chega junto com o principal).

| Resposta que reprova | Por quê |
|---|---|
| "Fica parado / sem vida / sem charme" | Não é perda do visitante, é preferência do autor. Custa FPS e não paga. |
| "Não perde nada, mas não atrapalha" | Toda animação atrapalha alguma coisa: peso, bateria, uma leitura no meio do scroll. |
| "Perde consistência com as outras seções" | Consistência decide **qual** animação usar, nunca **se** deve existir uma. |
| "Perde o efeito impressionante" | Se o custo é a legibilidade, o impressionante é o problema. |

O último caso já aconteceu aqui: `cuidado` tinha o filme scrubado, era a coisa mais
impressionante da página **e** a razão de a copy por cima ser ilegível — nenhum contraste se
sustenta contra um fundo que muda a cada pixel de scroll. O filme desceu de major para medium
(`<ChapterFilm mode="loop">`), o capítulo passou a carregar o peso por composição, e continua
sendo o pico.

**Ordem de corte** quando a soma de pontos estoura a faixa do Score:

1. Smalls nos capítulos de silêncio que não sejam o único small do capítulo.
2. Qualquer entrada que já apareceu duas vezes na página.
3. O medium que duplica o mecanismo do outro medium.
4. Um capítulo inteiro, começando pelo de menor profundidade no platô.

Nunca corte reduzindo a qualidade do major — um WOW major mal executado custa o mesmo download
e não entrega nada.

## Passo 7 — O artefato

`design/creative-direction.json` tem três blocos. A interface `CreativeDirection` completa, o
JSON preenchido da página de referência, a tabela de delegação e as medições de cobrança estão
em [handoff.md](handoff.md).

| Bloco | Campos | Quem preenche |
|---|---|---|
| topo | `experienceScore`, `rationale`, `depth`, `budget` (`pointsTotal`, `mediaDesktopMB`, `mediaMobileMB`, `eagerMB`) | passe 1 |
| `chapters[]` | `id`, `band`, `scroll`, `scrollMobile`, `points`, `entrance`, `silence`, `roi` | passe 2 — `scroll` e `scrollMobile` copiados do story map |
| `wow[]` | `chapters[]`, `tier`, `pattern`, `cost` (`kb`, `loc`, `viewports`), `fallback`, `owner` | tier e contagem no passe 1; capítulo dono e `cost` no passe 2 |

Três armadilhas que o formato esconde: `cost.viewports` **não se soma** entre dois momentos do
mesmo capítulo (o capítulo tem um orçamento de scroll só; um medium dentro de um major é o
mecanismo dele, não uma segunda entrada); um device repetido em três capítulos é **uma** entrada
com três ids, não três entradas; e os `small` não entram no array — mas a rubrica os conta, então
liste-os por capítulo antes de delegar.

Ao delegar, passe **o recorte do JSON que é do especialista**, não o arquivo inteiro — um
motion-ui que recebe o plano de scroll inteiro começa a opinar sobre pinagem.

## Anti-patterns

- **WOW major no hero** — a curva só pode descer dali, a página vira 100/80/60/40/20 e o CTA
  chega no ponto mais fraco de todos.
- **Vídeo scrubado atrás de copy que precisa ser lida** — o fundo muda a cada pixel de scroll e
  nenhum contraste se sustenta contra um fundo que nunca para.
- **Dois WOW major na mesma página** — o segundo não é percebido como surpresa, é percebido como
  o motivo de a página estar pesada.
- **Animar um capítulo só porque o vizinho tem** — sem o capítulo baixo, o pico não lê como pico.
- **Repetir a entrada em capítulos consecutivos** — duas repetições bastam para o visitante
  aprender o padrão, e a partir daí ele para de olhar para a chegada dos elementos.
- **Esticar `scroll` de um capítulo sem beats para revelar** — scroll sem nada parado no frame lê
  como página travada, não como respiro.
- **Planejar WOW sem fallback** — o buraco cai exatamente no momento principal.
- **Medir orçamento em sensação** — "parece longa" não é revisável; `scrollHeight / innerHeight` é.
- **Escrever `gsap` nesta skill** — uma direção que já escolheu o `ease` não é direção, é
  implementação com opinião.

## Checklist

- [ ] `design/creative-direction.json` existe e valida contra `CreativeDirection`.
- [ ] Todo item de `wow` tem `fallback` com uma frase concreta.
- [ ] `sum(points)` e `sum(1 + scroll)` dentro das faixas da linha do Score.
- [ ] Nenhum capítulo passa do teto de pontos da sua `band`.
- [ ] No máximo dois capítulos consecutivos acima de 7 pontos.
- [ ] Pelo menos um `silence: true` a cada quatro capítulos.
- [ ] Nenhum `entrance` repetido em capítulos consecutivos; nenhum mais de duas vezes.
- [ ] Todo capítulo tem `roi`, e nenhum `roi` começa com "fica".
- [ ] A contagem de major, medium e small bate com a linha do Score — inclusive os small, que não
      entram no array `wow` e por isso passam despercebidos. Liste-os por capítulo antes de delegar.
- [ ] `sum(scrollMobile) / sum(scroll)` entre 0,68 e 0,75, e nenhum capítulo isolado fora disso.
- [ ] Nenhum arquivo de vídeo sem dono em `wow` ou em `eagerMB`.
- [ ] Depois da implementação, os três números de cobrança de [handoff.md](handoff.md) batem.
- [ ] Com `prefers-reduced-motion: reduce` no DevTools, a página lê inteira e nenhum WOW deixou
      espaço vazio no lugar dele.
- [ ] Network com throttle 4G: o que carrega antes do primeiro scroll fica em ≤4 MB.
