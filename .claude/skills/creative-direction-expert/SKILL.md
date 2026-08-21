---
name: creative-direction-expert
description: Use when someone asks to plan, direct or raise the quality of a landing page experience before anything is animated — fixing the Experience Score, distributing the Creative Budget across the page, choosing which WOW moments the page earns, deciding where motion goes and where silence goes, and handing the plan to the implementation specialists. Direção criativa de landing page: Experience Score, Creative Budget, WOW moment, momento memorável, ritmo, quanto scroll cada capítulo recebe, densidade de motion, onde fica o silêncio, e o handoff para quem implementa. Também quando alguém diz que a página está "toda igual", "cansativa", "animada demais" ou "sem nenhum momento marcante".
argument-hint: [brief-ou-rota] [experience-score-alvo]
allowed-tools: Read, Glob, Grep, Write, Bash(node *), Skill
---

# Creative Direction

Esta skill escreve **um arquivo e nenhum componente**: `design/creative-direction.json`. Ele não
é lido sozinho por ninguém — é você que entrega o recorte dele a cada especialista, na ordem do
CLAUDE.md (ver [handoff.md](handoff.md)).

Sem ele, cada capítulo é dirigido a partir de um palpite novo: três seções seguidas entram com
o mesmo fade, o momento mais caro da página cai no hero e a curva de intensidade só desce
depois disso, e ninguém consegue dizer se a página está pronta porque não existe número para
comparar. O CLAUDE.md já define a ordem dos especialistas e o que cada um responde. O que
falta — e é o que está aqui — é **quanto** cada trecho da página recebe, **medido**.

A regra de ouro desta skill: *você decide o quê e o quanto; nunca o como.* Se estiver
escrevendo `gsap.to(...)`, saiu do papel.

## Entrada

| Insumo | Onde | Se faltar |
|---|---|---|
| Objetivo do cliente e a ação que a página cobra | prompt do usuário | pergunte — sem CTA definido não há curva |
| Capítulos e copy | `src/data/site.ts` | leia o arquivo; nunca invente seção |
| Inventário de mídia | `src/generated/media.ts` e `public/media/` | Glob `public/media/**/*.mp4`, e o node do checklist para o peso |

### Esta skill roda em duas passadas

O CLAUDE.md coloca a direção criativa **antes** de `landing-storytelling-director`, e o `scroll`
de cada capítulo só existe depois que aquela skill reparte o `share`. Então:

| Passada | Quando | O que fica decidido |
|---|---|---|
| **1 — orçamento** | antes do storytelling | `experienceScore`, `depth` **total** (uma faixa, não por capítulo), `budget`, quantos WOW de cada tier a página ganha e em que **posição** da curva o pico cai |
| **2 — ratificação** | depois que o story map existe | `band` de cada capítulo (pela posição medida do meio dele), `points`, `entrance`, `silence`, e a qual capítulo cada WOW pertence |

`scroll` e `scrollMobile` **nunca são inventados aqui**: eles vêm do `share` do story map, pela
função `scrollProp()` daquela skill. Se um capítulo não couber no teto de pontos da sua band, a
saída é cortar pontos ou devolver ao storytelling para repartir o `share` de novo. Duas skills
escrevendo `scroll` produzem duas páginas de altura diferente, e nenhuma das duas erra sozinha.

## A unidade de medida: ponto de complexidade

Tudo aqui é contado em pontos. Um ponto é **um alvo distinto que a timeline do capítulo
escreve** — não um `tween`, não uma linha de código.

| O que a cena tem | Pontos |
|---|---|
| Um `Reveal` de copy (linhas ou bloco) | 1 |
| Um elemento com transform scrubado (parallax, rail, thread, push-in de mídia) | 1 |
| Um contador ou índice que avança | 1 |
| Um grupo que entra junto em stagger (pilares, detalhes) | 1 |
| Marquee | 1 |
| Vídeo em loop ou `once` de fundo | 2 |
| Troca de cor de fundo ou veil scrubada | 2 |
| Rail, stack ou ciclo com N itens cronometrados um a um | N |
| Vídeo com `currentTime` dirigido por scroll | 4 |
| Sequência de frames em canvas | 6 |

Conte à mão pela tabela — não existe comando que conte pontos, porque um ponto é um alvo e um
alvo costuma receber dois ou três tweens. O comando abaixo conta **tweens e reveals**, que é
outra coisa: serve só para comparar capítulos entre si e achar o que cresceu sem ninguém notar.
Um número alto aqui pede uma recontagem à mão, não é um veredito.

```bash
node -e "
const fs = require('fs'), dir = 'src/components/chapters';
for (const f of fs.readdirSync(dir)) {
  const s = fs.readFileSync(dir + '/' + f, 'utf8');
  const tweens = (s.match(/\.(from|fromTo|to)\(/g) ?? []).length;
  const reveals = (s.match(/select:/g) ?? []).length;
  console.log(f.padEnd(26), 'tweens', tweens, 'reveals', reveals);
}"
```

Ele conta texto estático: um tween dentro de um `forEach` de 5 cartões conta 1, e no runtime são
5. É mais uma razão para a contagem que vale ser a da tabela.

**Profundidade de scroll** é a segunda medida: `1 + scroll` viewports por capítulo, somados.
O `1` é o stage sticky que todo capítulo ocupa (ver `.chapter` em `src/styles/index.css`);
o resto é o orçamento que você aloca. Verificação em runtime:

```js
document.body.scrollHeight / window.innerHeight   // profundidade total da página
```

## Passo 1 — Fixar o Experience Score

**Artefato:** uma linha no topo do JSON — `"experienceScore": 4` — e a justificativa em uma
frase. O default do projeto é ★★★★☆; subir para ★★★★★ exige que o cliente tenha material
(vídeo, fotografia dirigida) que sustente o pico.

A rubrica abaixo é medível. Um nível só é atingido quando **todas** as colunas batem — não é
uma média, e um único item abaixo da faixa derruba o nível.

| Score | WOW major | medium | small | Entradas distintas | Profundidade (desktop) | Pontos totais | Mídia (desktop / mobile) | Scroll storytelling |
|---|---|---|---|---|---|---|---|---|
| ★☆☆☆☆ | 0 | 0 | ≤3 | 1 | 2–4 | ≤6 | ≤1,5 MB / ≤0,8 MB | não |
| ★★☆☆☆ | 0 | 0–1 | 3–5 | 2 | 4–8 | 6–14 | ≤2,5 MB / ≤1,2 MB | não |
| ★★★☆☆ | 0 | 1–2 | 5–8 | 3 | 8–14 | 14–26 | ≤4 MB / ≤2 MB | opcional |
| ★★★★☆ | 1 | 2 | 6–10 | ≥4 | 14–26 | 26–42 | ≤8 MB / ≤3,5 MB | ≥3 capítulos com stage sticky |
| ★★★★★ | 1–2 | 2–3 | 8–14 | ≥6 | 26–36 | 42–60 | ≤12 MB / ≤5 MB | a página inteira |

Duas restrições que valem em qualquer nível:

- **Antes do primeiro scroll**, no máximo **4 MB** carregados. Nesta referência é o filme do
  hero (3,2 MB desktop / 1,0 MB mobile); todo o resto entra por `IntersectionObserver` com
  `rootMargin: '150% 0px'` — ver `src/components/media/ChapterFilm.tsx`.
- **Piso de reduced motion:** com `prefers-reduced-motion: reduce`, uma página de qualquer Score
  degrada — mas **nunca abaixo de ★★☆☆☆**: toda copy visível, toda seção alcançável, todo item
  de um rail acessível. Uma página que já nasceu ★☆☆☆☆ ou ★★☆☆☆ não muda de nível. O stylesheet
  já entrega o piso (`[data-rail] { flex-wrap: wrap }`, `[data-progress-row] { display: none }` e
  `.chapter { height: auto }`, em `src/styles/index.css`); o seu trabalho é não planejar nada que
  dependa de motion para existir.

Acima de 36 viewports a página deixa de ser uma experiência e vira uma travessia: o CTA está no
fim, e cada viewport adicionada é mais uma chance de sair antes dele. Se a soma passar de 36,
corte um capítulo — não corte a qualidade dos que ficam.

## Passo 2 — Distribuir o Creative Budget

**Artefato:** a lista de capítulos do JSON, cada um com `scroll`, `points` e `band`.

O `scroll` vem do `share` do story map, já convertido. O que esta skill decide é a `band` de cada
capítulo e **quantos pontos cabem nela**.

A progressão 20 → 40 → 60 → 80 → 100 do CLAUDE.md é uma curva sobre a **profundidade acumulada**,
não sobre o número de seções. A `band` de um capítulo é lida pela posição do seu **meio** na
profundidade total, e cada faixa tem um teto de pontos por capítulo:

| Faixa (posição do meio do capítulo) | Intensidade | Pontos por capítulo | O que cabe |
|---|---|---|---|
| 0–20% — abertura | 20 | 4–7 | Mask reveal da headline, um filme em loop, o scroll cue. Nenhum WOW major. |
| 20–45% — primeira escalada | 40 | 5–11 | O primeiro WOW medium, e o primeiro capítulo de silêncio logo antes dele. |
| 45–65% — platô | 60 | 5–9 | O corpo do argumento. Entradas variadas, nenhum WOW novo. |
| 65–85% — pico | 80–100 | 9–14 | O WOW major vive aqui, e só aqui. |
| 85–100% — fecho | 30–40 | 3–5 | Silêncio, a linha final e o CTA. |

Calibração — esta é a página de referência deste repositório, medida, não estimada. As colunas
`scroll` e `scrollMobile` são as do story map de `landing-storytelling-director`, repetidas aqui
só para as colunas de ponto e faixa terem contra o que ser lidas; se as duas divergirem, a do
story map vence.

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
  `jornada` gasta 6 para 7 beats (título, 5 cartões, fecho) — 0,86 por beat. `agendar` tem um
  beat e recebe o piso.
- **Mobile é 0,68–0,75 × desktop, na prop `scroll`** — faixa de
  [landing-storytelling-director](../landing-storytelling-director/SKILL.md#pacing), que é dona
  dos dois números. Aqui ela só é conferida: um capítulo fora dela conta uma história diferente no
  celular, e aí é outro plano, não um ajuste. Na referência a razão medida vai de 0,69 a 0,75
  (`clinica` é a única acima de 0,72, porque 3.2 já é o piso útil para um capítulo de silêncio).

```tsx
// O plano vira props. Nada aqui é decidido no componente.
<Chapter id="cuidado" scroll={4.8} scrollMobile={3.4} labelledBy="chapter-cuidado-title" ref={ref}>
```

## Passo 3 — Alocar os WOW moments

**Artefato:** o array `wow` do JSON. Cada entrada declara `tier`, `chapters`, `cost` e —
obrigatoriamente — `fallback`.

| Tier | Custo típico | Scroll que consome | Quando vale |
|---|---|---|---|
| **major** | 0–8 MB, 80–160 LOC, risco alto de regressão | 4–6 viewports | Quando a mudança ao longo do tempo **é** o conteúdo, e a página tem um único assunto que merece isso. 0 MB quando o mecanismo é CSS puro — o pin de `.chapter` não baixa nada. |
| **medium** | 0–1,5 MB, 40–70 LOC | 3–5 viewports | Quando o conteúdo já tem estrutura (uma sequência ordenada, itens comparáveis) e o motion apenas a torna legível. |
| **small** | 0 KB, 5–30 LOC | 0 viewports | Quando cabe dentro de um beat que já existe. Nunca cria beat próprio. |

**Quantos de cada** é a linha do Score na rubrica do Passo 1, não um número fixo por tier: um
★★★★☆ leva 1 major / 2 medium / 6–10 small; um ★★★★★ leva 1–2 major / 2–3 medium / 8–14 small.
Um filme de fundo conta como medium mesmo quando ninguém o chamou de WOW — se ele pesa
megabytes e o capítulo depende dele, ele ocupa uma vaga de medium.

O catálogo completo — 22 momentos com custo em KB, LOC e viewports, o especialista dono de cada
um e o fallback de reduced motion — está em [wow-catalog.md](wow-catalog.md).

**Um WOW sem `fallback` descrito não entra no plano.** Sem ele, quem usa reduced motion recebe
uma página com um buraco exatamente onde estava o momento principal — e é sempre o momento
principal, porque é o único que dependia inteiramente de movimento. O fallback é uma frase
concreta: *"a sequência de frames vira o poster do frame 1, e os cinco princípios viram uma
lista empilhada"*, não *"degrada com elegância"*.

Ordem de alocação, nesta sequência:

1. Escolha o **major** primeiro e prenda-o na faixa de pico (65–85%). Ele define o resto.
2. Escolha os **medium** de forma que fiquem separados do major por pelo menos um capítulo. A
   exceção é o device recorrente — um filme de fundo que aparece em três capítulos é uma entrada
   só, e nada impede que um deles seja o do pico; o que se proíbe é **dois momentos distintos**
   colados.
3. Distribua os **small** nos capítulos que sobraram — inclusive nos de silêncio, onde um
   único small é exatamente o suficiente.
4. Some `cost.kb` do array e compare com `budget.mediaDesktopMB`. Todo arquivo que aparece no
   node de peso do checklist tem de estar em alguma entrada ou em `eagerMB`. MB sem dono é MB
   que ninguém corta quando o orçamento estoura, porque ninguém sabe de quem ele é.

## Passo 4 — Mapa de densidade

**Artefato:** o campo `silence: true` em pelo menos um capítulo a cada quatro.

Duas regras, ambas conferíveis contando a coluna de pontos:

- **No máximo dois capítulos consecutivos acima de 7 pontos.** O terceiro fica em ≤5.
- **Pelo menos um capítulo de silêncio a cada quatro.** Silêncio é definido, não sentido:
  **≤5 pontos e ≤3,2 viewports de `scroll`**, sem WOW major nem medium próprio do capítulo, e
  sem troca de cor de fundo scrubada. Um device recorrente que só passa por ali (o filme de
  fundo da página, por exemplo) não descaracteriza o silêncio — mas os pontos dele contam, e
  são 2, o que já consome quase metade do teto.

Na referência, os dois capítulos de silêncio são `experiencia` e `clinica` — os dois com
`scroll={3.2}` e 5 pontos, um antes de cada pico. Não é coincidência: **o pico só é percebido
como pico porque o capítulo anterior baixou.** Uma página em que todos os capítulos têm 9
pontos não é intensa, é plana num nível alto — o visitante calibra pelo que acabou de ver, e
depois de três capítulos densos o quarto denso é o normal.

Scroll morto merece uma distinção: scroll morto **com um beat parado no frame** é tempo de
leitura, e é o que `TRAVEL_SHARE = 0.28` compra em `ChapterJourney` — 72% do orçamento de cada
passo é o cartão parado, sendo lido. Scroll morto **sem nada parado no frame** é uma página que
parece travada.

## Passo 5 — Variedade de entrada

**Artefato:** o campo `entrance` de cada capítulo, com um valor diferente do capítulo anterior.

- Nenhuma entrada se repete em **capítulos consecutivos**.
- Nenhuma entrada aparece mais de **duas vezes** na página inteira.
- ★★★★☆ exige ≥4 tipos distintos; ★★★★★ exige ≥6.

As sete entradas da referência, todas distintas:

| Capítulo | Entrada | Mecanismo |
|---|---|---|
| `inicio` | `mask-lines-late` | Linhas sobem da máscara, mas só a 65% e 86% do capítulo |
| `experiencia` | `intro-exit-stagger` | O intro sai por cima; os pilares sobem em stagger |
| `jornada` | `rail-steps` | Rail horizontal que avança em passos, não em glide |
| `consulta` | `media-push-in` | A mídia empurra para dentro, `scale: 1 → 1.05` |
| `cuidado` | `cycle-replace` | Um princípio por vez substitui o anterior |
| `clinica` | `media-pull-back` | A mídia recua, `scale: 1.08 → 1` |
| `agendar` | `block-settle` | Bloco único, sem stagger — a página para de falar |

`consulta` e `clinica` usam o mesmo mecanismo com o sinal invertido. Isso conta como duas
entradas, e só funciona porque há dois capítulos entre elas. Coladas, seriam a mesma coisa
duas vezes.

## Passo 6 — Teste de Motion ROI

**Artefato:** as animações reprovadas saem do plano antes do handoff. Nenhuma sobrevive "para
decidir depois" — decidir depois significa implementada.

Para cada animação, uma pergunta só:

> Se eu remover esta animação e deixar o elemento no estado final, **o que o visitante perde?**

**Aprova** se a resposta for uma destas — uma basta, e precisa caber em uma frase:

- Perde a **ordem de leitura**: sem ela, três elementos chegam ao olho no mesmo instante.
- Perde **informação**: o movimento é o conteúdo (o rail que avança, o número que conta, o
  thread que mede o progresso).
- Perde **continuidade**: sem ela, duas cenas trocam com um corte seco.
- Perde **hierarquia**: sem ela, o secundário chega junto com o principal.

**Reprova**, e a animação é cortada, se a resposta for qualquer uma destas:

| Resposta | Por que reprova |
|---|---|
| "Fica parado / sem vida / sem charme" | Não é uma perda do visitante, é uma preferência do autor. Custa FPS e não paga. |
| "Não perde nada, mas não atrapalha" | Toda animação atrapalha alguma coisa: peso, bateria, uma leitura no meio do scroll. |
| "Perde consistência com as outras seções" | Consistência decide **qual** animação usar, nunca **se** deve existir uma. |
| "Perde o efeito impressionante" | Se o custo é a legibilidade, o impressionante é o problema. |

O último caso já aconteceu neste repositório: `cuidado` tinha o filme scrubado — scroll
dirigindo o playhead — e era a coisa mais impressionante da página **e** a razão de as palavras
por cima serem ilegíveis, porque o fundo mudava a cada pixel de scroll e nenhum contraste se
sustenta contra um fundo que nunca para. O filme desceu de major para medium: virou fundo com
playhead próprio (`<ChapterFilm mode="loop">`), o capítulo passou a carregar o peso por
composição — um princípio por vez, sozinho no centro do quadro — e continua sendo o pico da
página. O comentário no topo de `src/components/chapters/ChapterCare.tsx` registra a decisão.

**Ordem de corte** quando a soma de pontos estoura a faixa do Score, nesta sequência:

1. Smalls nos capítulos de silêncio que não sejam o único small do capítulo.
2. Qualquer entrada que já apareceu duas vezes na página.
3. O medium que duplica o mecanismo do outro medium.
4. Um capítulo inteiro, começando pelo de menor profundidade no platô.

Nunca corte reduzindo a qualidade do major — um WOW major mal executado custa o mesmo download
e não entrega nada.

## Passo 7 — Escrever o artefato e delegar

**Artefato:** `design/creative-direction.json`, validando contra esta interface.

```ts
// design/creative-direction.d.ts
export type Tier = 'major' | 'medium' | 'small'
export type Band = 'abertura' | 'escalada' | 'plato' | 'pico' | 'fecho'

export interface WowMoment {
  tier: Tier
  /**
   * Ids dos capítulos em src/data/site.ts. Um device repetido é UMA entrada com N capítulos:
   * é assim que o visitante percebe (na terceira vez é a textura da página, não uma surpresa)
   * e é assim que ele ocupa uma vaga só na contagem do Score.
   */
  chapters: readonly string[]
  /** Nome do momento no catálogo — ver wow-catalog.md. */
  pattern: string
  /**
   * Custo declarado antes de implementar, para poder ser cobrado depois.
   * `viewports` não se soma entre dois momentos do mesmo capítulo: o capítulo tem um orçamento
   * de scroll só. Se um medium mora dentro de um major, ele é o mecanismo do major, não uma
   * segunda entrada.
   */
  cost: { kb: number; loc: number; viewports: number }
  /** Obrigatório. O que a cena vira sob prefers-reduced-motion, em uma frase concreta. */
  fallback: string
  /** Especialista que implementa. Ver handoff.md. */
  owner: string
}

export interface ChapterPlan {
  id: string
  band: Band
  /** Viewports além do primeiro screen — vira <Chapter scroll={...}>. Vem do story map. */
  scroll: number
  /** Vem do story map, na faixa 0.68–0.75 × scroll que landing-storytelling-director fixa.
   *  Padrão 0.70; sobe só quando 0.70 cai abaixo do piso útil de ~2 viewports. */
  scrollMobile: number
  /** Soma pela tabela de pontos de complexidade. */
  points: number
  /** Diferente do capítulo anterior; no máximo duas vezes na página. */
  entrance: string
  /** true quando points <= 5 e scroll <= 3.2 e não há WOW aqui. */
  silence: boolean
  /** Uma frase: o que o visitante perde se este capítulo não animar. */
  roi: string
}

export interface CreativeDirection {
  experienceScore: 1 | 2 | 3 | 4 | 5
  rationale: string
  /** Soma de (1 + scroll) de todos os capítulos. */
  depth: number
  budget: { pointsTotal: number; mediaDesktopMB: number; mediaMobileMB: number; eagerMB: number }
  chapters: ChapterPlan[]
  wow: WowMoment[]
}
```

O JSON preenchido para a página de referência, a tabela de delegação (o que cada especialista
recebe e o que precisa devolver) e o formato do handoff estão em [handoff.md](handoff.md).

Delegue na ordem do CLAUDE.md. Ao chamar cada especialista, passe **o recorte do JSON que é
dele**, não o arquivo inteiro — um especialista de motion-ui que recebe o plano de scroll
inteiro começa a opinar sobre pinagem.

## Anti-patterns

- **Não gaste o WOW major no hero** — a curva de intensidade só pode descer a partir dali, a
  página vira 100/80/60/40/20 e o CTA chega no ponto mais fraco de todos.
- **Não scrube vídeo atrás de copy que precisa ser lida** — o fundo muda a cada pixel de scroll
  e nenhum contraste se sustenta contra um fundo que nunca para.
- **Não coloque dois WOW major na mesma página** — o segundo não é percebido como surpresa; é
  percebido como o motivo de a página estar pesada.
- **Não dê animação a um capítulo só porque o vizinho tem** — sem o capítulo baixo, o pico não
  lê como pico; o visitante calibra pelo que acabou de ver.
- **Não repita a entrada em capítulos consecutivos** — duas repetições bastam para o visitante
  aprender o padrão, e a partir daí ele para de olhar para a chegada dos elementos.
- **Não estenda o `scroll` de um capítulo sem beats para revelar** — scroll sem nada parado no
  frame lê como página travada, não como respiro.
- **Não planeje um WOW sem fallback** — o buraco cai exatamente no momento principal, porque é
  o único que dependia inteiramente de movimento.
- **Não meça o orçamento em sensação** — "parece longa" não é revisável; `scrollHeight /
  innerHeight` é.
- **Não escreva `gsap` nenhuma nesta skill** — o plano precisa sobreviver a uma troca de
  biblioteca, e uma direção que já escolheu o `ease` não é uma direção, é uma implementação
  com opinião.

## Checklist de verificação

Rodável, antes de considerar a direção entregue.

- [ ] `design/creative-direction.json` existe e valida contra `CreativeDirection`.
- [ ] Todo item de `wow` tem `fallback` preenchido com uma frase concreta.
- [ ] `sum(points)` dentro da faixa da linha do Score na rubrica.
- [ ] `sum(1 + scroll)` dentro da faixa de profundidade da mesma linha.
- [ ] Nenhum capítulo passa do teto de pontos da sua `band`.
- [ ] No máximo dois capítulos consecutivos acima de 7 pontos.
- [ ] Pelo menos um `silence: true` a cada quatro capítulos.
- [ ] Nenhum `entrance` repetido em capítulos consecutivos; nenhum mais de duas vezes.
- [ ] Todo capítulo tem `roi` preenchido, e nenhum `roi` começa com "fica".
- [ ] A contagem de major, medium e small bate com a linha do Score — inclusive os small, que não
      entram no array `wow` e por isso são os que passam despercebidos. Liste-os por capítulo
      antes de delegar.
- [ ] `sum(scrollMobile) / sum(scroll)` entre 0,68 e 0,75, e nenhum capítulo isolado fora disso.
- [ ] Nenhum arquivo de vídeo em `budget` que não apareça em algum `wow` ou no `eagerMB` — MB sem
      dono é MB que ninguém vai cortar quando o orçamento estourar.

Depois que a implementação existir, os mesmos números precisam bater no código:

```bash
# Razão mobile/desktop de cada capítulo — precisa ficar entre 0.68 e 0.75
node -e "
const fs = require('fs');
const dir = 'src/components/chapters';
for (const f of fs.readdirSync(dir)) {
  const s = fs.readFileSync(dir + '/' + f, 'utf8');
  const d = s.match(/scroll=\{([\d.]+)\}/), m = s.match(/scrollMobile=\{([\d.]+)\}/);
  if (d && m) console.log(f.padEnd(26), d[1], m[1], (m[1] / d[1]).toFixed(2));
}"

# Peso de mídia por rendição — compare com budget.mediaDesktopMB / mediaMobileMB
node -e "
const fs = require('fs'), p = 'public/media';
const rows = fs.readdirSync(p).filter(f => f.endsWith('.mp4'))
  .map(f => [fs.statSync(p + '/' + f).size, f]).sort((a, b) => b[0] - a[0]);
for (const [s, f] of rows) console.log((s / 1e6).toFixed(2).padStart(6), 'MB', f);
const sum = t => (rows.filter(r => r[1].includes(t)).reduce((a, r) => a + r[0], 0) / 1e6).toFixed(2);
console.log('desktop', sum('-desktop'), 'MB / mobile', sum('-mobile'), 'MB');"
```

`node` em vez de `find -printf`: `-printf` é extensão do GNU find e não existe no `find` do
Windows, que é a máquina onde estes projetos rodam.

No navegador, com `npm run dev` servindo:

- [ ] `document.body.scrollHeight / window.innerHeight` dentro da faixa.
- [ ] Com `prefers-reduced-motion: reduce` ativo no DevTools, a página lê inteira, todo item de
      rail está alcançável e nenhum WOW deixou um espaço vazio no lugar dele.
- [ ] Aba Network com throttle 4G: o que carrega antes do primeiro scroll fica em ≤4 MB.
