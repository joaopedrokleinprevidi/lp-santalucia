---
name: skill-builder
description: Use when creating a new skill, rewriting or auditing an existing one, or deciding which skill owns a piece of knowledge. Cria, audita e reescreve skills do Claude Code — frontmatter, densidade operacional, arquivos de apoio, fronteiras entre as skills do projeto.
argument-hint: [nome-da-skill | caminho/para/SKILL.md]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(ls *), Bash(wc *)
---

# Skill Builder

Uma skill é um SOP executável. Ela não existe para explicar um assunto — existe para que a
mesma tarefa produza o mesmo resultado na terça e na sexta, com um modelo diferente e um
contexto diferente.

O critério aqui é **densidade operacional**: cada linha ou é um número, ou é código, ou é uma
decisão com causa. Prosa que não muda o output é peso morto que consome contexto.

Referência técnica completa de frontmatter, invocação, `allowed-tools`, `context: fork`, hooks e
troubleshooting: [reference.md](reference.md). Leia antes de configurar qualquer campo.

## Quando uma skill é a resposta

| Situação | Onde vai |
|---|---|
| Claude precisa saber **sempre** (convenção, stack, regra do repo) | `CLAUDE.md` |
| Claude precisa saber **ao fazer X**, e X repete | Skill |
| Detalhe de referência dentro de X, consultado às vezes | Arquivo de apoio da skill |
| Vale para um arquivo só | Comentário no código |

Antes de criar: rode `ls .claude/skills/` e leia a `description` das existentes. Se uma delas já
cobre 70% do escopo, **estenda** em vez de criar. Duas skills sobrepostas divergem em três meses
e ninguém sabe qual está certa.

---

## Modo 1 — Criar skill nova

### Passo 0: infira antes de perguntar

Esta skill roda em sessões autônomas. Um interrogatório de 6 rodadas trava o trabalho. A regra:
**se dá para inferir do repositório ou do pedido, infira, execute, e declare a suposição no fim.**

| Pergunta | Onde inferir a resposta | Bloqueante? |
|---|---|---|
| O que a skill faz | Do pedido do usuário e do material de origem | Não |
| Nome | Do escopo — minúsculas, hífens, ≤64 chars, igual ao diretório | Não |
| Gatilho / palavras-chave | Do vocabulário do pedido + das descriptions das skills vizinhas | Não |
| Passo a passo | Do código existente que faz a tarefa hoje (`src/`, `scripts/`) | Não |
| Entradas e saídas | Dos caminhos reais do repo — `src/data/site.ts`, `public/`, `src/generated/` | Não |
| Guardrails | Dos anti-patterns já documentados nas skills irmãs e no CLAUDE.md | Não |

Só três coisas são genuinamente bloqueantes, porque nenhuma leitura de repositório resolve:

1. **Destino externo com custo ou efeito irreversível** — chave de API, envio, deploy, geração
   paga. Pergunte qual conta, qual orçamento.
2. **Escolha de produto sem sinal no repo** — quando duas arquiteturas plausíveis levam a skills
   incompatíveis e nada no código indica a preferência.
3. **Contradição entre o pedido e o CLAUDE.md** — pergunte qual vence, não escolha sozinho.

Fora disso: prossiga. Se uma suposição estiver errada, corrigir uma skill custa uma edição;
travar a sessão custa a sessão inteira.

### Passo 0b: declare as suposições no fim

Ao terminar, feche com este bloco. Ele substitui as perguntas que não foram feitas:

```
## Suposições
- Nome `<x>`: derivado de <razão>. Renomeie o diretório e o campo `name` juntos se mudar.
- Gatilho: description cobre "<frase>", "<frase>". Se disparar demais, estreite aqui.
- Entradas: assumi <caminho> como fonte. Não existia <alternativa>.
- Escopo: NÃO cobre <x> — isso é de <outra-skill>.
```

### Passo 1: escolha o tipo

| Tipo | O que faz | Frontmatter típico |
|---|---|---|
| **Tarefa** | Executa um fluxo com início e fim | `argument-hint`, às vezes `allowed-tools` |
| **Referência** | Injeta conhecimento que Claude aplica ao trabalho corrente | só `name` + `description` |
| **Especialista** | Detém uma decisão do pipeline (as 8 deste projeto) | `description` com gatilho de auto-invocação |

Skill de referência **nunca** leva `context: fork` — o subagente recebe diretrizes sem tarefa e
devolve vazio.

### Passo 2: frontmatter

- `name` — igual ao nome do diretório. Em skill de projeto o comando vem do **diretório**; o
  `name` é só o rótulo exibido na listagem. Divergir não quebra `/nome-do-diretório` — cria duas
  verdades, com o menu mostrando um nome que ninguém consegue digitar. (Só em skill de plugin o
  `name` manda no último segmento do comando.)
- `description` — formato "Use when someone asks to X, Y, or Z", em inglês, com uma frase em
  português contendo os termos que o usuário deste projeto realmente digita ("capítulo",
  "rolagem", "acessibilidade"). O matching melhora com as duas línguas presentes.
  Máximo ~350 caracteres. O teto do produto é 1.536 por entrada, mas a listagem inteira cabe em
  ~1% da janela de contexto: com 13 skills, passar de 350 rouba espaço das vizinhas — e o corte
  começa pelas skills menos invocadas. Ver [reference.md](reference.md).
- `argument-hint` — se aceita argumento. Aparece no autocomplete do menu `/`.
- `allowed-tools` — se o escopo é restrito de verdade. Não use como segurança: ele **adiciona**
  auto-aprovações durante o turno que invoca a skill, e o grant expira na próxima mensagem do
  usuário. O campo que de fato *tira* uma ferramenta da mão do modelo é `disallowed-tools`.
- `disable-model-invocation: true` — só se a skill gasta dinheiro, envia algo ou é destrutiva.
- `context: fork` + `agent` — só para tarefa autocontida e verbosa.

Campo que você não precisa é campo que mente sobre o comportamento da skill. Omita.

### Passo 3: escreva com densidade

Estrutura de uma skill de tarefa:

1. **Uma frase de enquadramento** — o que muda no mundo quando ela roda.
2. **Entrada** — arquivos, argumentos, o que ler se faltar informação (caminho real, não "os dados").
3. **Não-negociáveis** — tabela numerada de thresholds. Todo valor verificável.
4. **Fluxo** — passos numerados, cada um com o comando ou o código que executa.
5. **Acessibilidade dentro do passo em que ela acontece** — nunca numa seção de rodapé.
6. **Anti-patterns com causa.**
7. **Checklist de verificação** — itens observáveis, não intenções.

Aplique a [Densidade operacional](#densidade-operacional) abaixo enquanto escreve, não depois.

### Passo 4: arquivos de apoio

Passou de 500 linhas, ou tem bloco de referência consultado só às vezes: separe.

| Arquivo | Conteúdo | Exemplo neste projeto |
|---|---|---|
| `reference.md` | Tabelas de campo, API, troubleshooting | `skill-builder/reference.md` |
| `decision.md` | "Qual técnica usar" — tabela comparativa + orçamento | `video-to-website/decision.md` |
| `<tema>.md` | Um subsistema completo com código | `video-to-website/react-port.md` |

Todo arquivo de apoio precisa de um link markdown relativo no SKILL.md. Arquivo não referenciado
nunca carrega — ele não é opcional, é invisível.

### Passo 5: registre e teste

Adicione a skill na seção correspondente do `CLAUDE.md` (nome, gatilho, uma linha do que faz,
onde a saída cai). Depois:

1. **Linguagem natural** — diga uma frase que deveria disparar. Não disparou: falta palavra-chave
   na `description`. Teste 3 frases diferentes.
2. **Direto** — `/nome-da-skill` com argumento de teste. Confira se `$ARGUMENTS` / `$0` substitui.
3. **Sem argumento** — a skill precisa se comportar, não travar.
4. **Orçamento** — `/context` mostra o tamanho da listagem já depois do corte; `/doctor` mostra
   quem são os maiores contribuintes. Description truncada é a causa mais comum de skill que
   existe e não dispara.

---

## Densidade operacional

Este é o critério de qualidade deste workspace. Sete testes; cada um tem número.

| Teste | Reprova quando | Aprova quando |
|---|---|---|
| **1. Número no lugar de adjetivo** | "tipografia grande", "premium", "fluido" | `clamp(3.5rem, 12vw, 11rem)`, `stagger: 0.12`, `≥800vh` |
| **2. Código real** | pseudocódigo, `// sua lógica aqui`, JS genérico com CDN | TSX que compila para React 19 + GSAP 3.15 + Lenis 1.3 + Tailwind 4 |
| **3. Tabela de decisão** | lista solta de opções | tabela com a coluna "quando usar" |
| **4. Anti-pattern com causa** | "nunca anime margin" | "nunca anime `margin` — dispara layout na main thread; use `transform`" |
| **5. Tamanho** | SKILL.md ≥500 linhas | <500 linhas, resto em arquivo de apoio referenciado |
| **6. Acessibilidade no fluxo** | seção "Acessibilidade" no rodapé | o guard de `prefers-reduced-motion` no passo em que ele roda |
| **7. Verificação executável** | "garanta qualidade premium" | "rode `npm run build`", "375px: seções empilham, ≤550vh" |

Reprovou em 3 ou mais: reescreva a skill. Remendo em documento de filosofia produz documento de
filosofia com números soltos.

### O que conta como "código real"

Código real vem da stack instalada e traz o *porquê* onde a linha é contraintuitiva. Padrão do
repositório, extraído de `src/hooks/useChapterTimeline.ts`:

```tsx
import { useLayoutEffect } from 'react'

import { gsap, seal } from '../lib/gsap'

// Dentro de useChapterTimeline(ref, build): `ref` é o RefObject do capítulo e
// `builderRef` guarda o `build` atual sem entrar nas deps do efeito.
useLayoutEffect(() => {
  const scope = ref.current
  if (!scope) return

  const mm = gsap.matchMedia(scope)

  mm.add('(prefers-reduced-motion: no-preference)', () => {
    const tl = gsap.timeline({
      defaults: { ease: 'none' },
      scrollTrigger: {
        trigger: scope,
        start: 'top top',
        end: 'bottom bottom',
        // Lenis já fornece o easing; um scrub numérico por cima lê como lag.
        scrub: true,
        invalidateOnRefresh: true,
      },
    })

    builderRef.current(seal(tl), scope)
  })

  return () => mm.revert()
}, [ref])
```

Quatro coisas o tornam real e nenhuma é opcional:

- **`gsap.matchMedia` com escopo** — sob `prefers-reduced-motion` nenhuma propriedade chega ao
  DOM, e seletor de string dentro de um capítulo não alcança outro.
- **`mm.revert()` no cleanup** — React 19 Strict Mode monta duas vezes; sem isso todo
  ScrollTrigger é registrado em dobro e a página briga com ela mesma.
- **O import vem de `src/lib/gsap`**, nunca de `'gsap'` direto — `registerPlugin(ScrollTrigger)`
  roda uma vez lá; um segundo registro em outro arquivo duplica o plugin.
- **O comentário explica a causa**, não o que a linha faz.

Escrever exemplo em vanilla JS com GSAP via CDN é erro de fato aqui: uma segunda cópia registra
um segundo ScrollTrigger e os dois disputam o mesmo scroller.

### Regras de forma

- Sem formatação vertical (uma palavra por linha com linha em branco entre elas). Triplica o
  arquivo sem adicionar informação.
- Sem lista de marcas ("Apple, Linear, Stripe") sem dizer **o que** copiar de cada uma.
- Sem emoji decorativo.
- Sem repetir o CLAUDE.md. A skill complementa; se o texto já está lá, referencie.
- Não invente assinatura de API. Sem certeza: escreva o padrão sem fingir precisão.

---

## Modo 2 — Auditar uma skill existente

Leia o arquivo inteiro antes de propor qualquer mudança. Nunca audite uma skill que você não leu.

### Camada 1 — estrutura

- [ ] `name` bate com o nome do diretório
- [ ] `description` tem as palavras que alguém realmente diria, em PT e EN
- [ ] `description` específica o bastante para não disparar falso, ampla o bastante para pegar o caso real
- [ ] `disable-model-invocation: true` presente se a skill gera custo, envia ou destrói
- [ ] `argument-hint` presente se aceita argumento
- [ ] `allowed-tools` presente se o escopo é restrito
- [ ] `context: fork` só em tarefa autocontida — nunca em skill de referência
- [ ] Nenhum campo supérfluo
- [ ] Arquivos de apoio referenciados por link relativo, nenhum órfão
- [ ] Caminhos de arquivo existem de fato (`ls` neles)
- [ ] Chave de API em variável de ambiente, nunca no corpo

### Camada 2 — densidade operacional

Rode os sete testes acima e some. Para cada reprovação, registre a linha e o conserto:

- [ ] Teste 1 — zero adjetivos usados como threshold
- [ ] Teste 2 — todo padrão implementável tem código TSX da stack real
- [ ] Teste 3 — toda escolha entre alternativas está em tabela com "quando usar"
- [ ] Teste 4 — todo anti-pattern tem a causa
- [ ] Teste 5 — `wc -l SKILL.md` < 500
- [ ] Teste 6 — reduced-motion aparece dentro do fluxo
- [ ] Teste 7 — checklist final é observável

### Camada 3 — fronteira

- [ ] Não duplica conteúdo de outra skill (ver a tabela abaixo)
- [ ] Não duplica o CLAUDE.md
- [ ] Onde encosta em outra skill, aponta para ela com link em vez de repetir

### Formato do relatório

```
## Auditoria: <skill>  (<N> linhas)

Estrutura: <N>/11    Densidade: <N>/7    Fronteira: <N>/3

### Reprovações
1. [teste 2] linha 84 — "use uma animação suave" → sem código.
   Conserto: bloco `gsap.to(el, { y: 0, duration: 1, ease: 'expo.out' })`.
2. [teste 5] 546 linhas → mover "Referência de easing" para `easing.md`.

### Veredito
<Aprovada | Remendar (≤2 reprovações) | Reescrever (≥3)>
```

---

## As 14 skills deste projeto

Cada skill detém **uma** decisão. Para dividir escopo, a coluna que importa não é o que a skill
faz — é o que ela **não** decide, e para quem passa.

**Os 8 especialistas do pipeline.** A responsabilidade e a ordem deles estão no `CLAUDE.md` e não
se repetem aqui. Só a fronteira:

| Skill | Nunca decide — isso é de |
|---|---|
| `creative-direction-expert` | implementação → `landing-motion-expert` em diante |
| `landing-storytelling-director` | estética e código → `product-design-expert` |
| `product-design-expert` | animação → `landing-motion-expert` |
| `landing-motion-expert` | detalhe de timeline → `gsap-scrolltrigger-expert` |
| `gsap-scrolltrigger-expert` | o que o vídeo conta → `scroll-video-director` |
| `scroll-video-director` | scrubbing frame a frame → `video-to-website` |
| `motion-ui-expert` | motion de seção → `gsap-scrolltrigger-expert` |
| `responsive-e-acessibility` | direção criativa → `creative-direction-expert` |

**As 6 que o `CLAUDE.md` não lista.** Estas precisam da definição completa, porque não existe
outra fonte:

| Skill | Detém a decisão de | Nunca decide |
|---|---|---|
| `brand-dna-extractor` | o que os assets reais **provam**: hexes medidos, classe tipográfica observada, formas, motifs, fatos do negócio | qual família usar, qual rampa derivar, layout, motion |
| `frontend-design` | caráter: qual família, proporção de área entre as cores da marca, textura, atmosfera | escala, espaçamento, densidade e contraste (do `product-design-expert`); coreografia de scroll |
| `video-to-website` | pipeline canvas frame-sequence: extração, preload, port React | se o vídeo deveria existir |
| `ai-visual-prompt-director` | prompts de imagem/animação por seção, consistência visual | como a página anima |
| `landing-page-factory` | a orquestração ponta a ponta: fases, portões, credenciais, deploy | tudo que é conteúdo de fase — ela só chama a dona e confere o portão |
| `skill-builder` | como as skills são escritas, auditadas e divididas | qualquer decisão de produto |

> Confira contra `ls .claude/skills/` antes de confiar nesta tabela — o diretório manda.

### Regra de não-duplicação

Um fato mora em **um** lugar. Quando duas skills precisam dele, a dona escreve e a outra linka.

| Fato | Dona | As outras |
|---|---|---|
| Escala tipográfica, espaçamento, grade, rampa de cor, contraste, tokens | `product-design-expert` | Referenciam |
| Duração, ease, stagger, distância de deslocamento, variedade de entrada | `landing-motion-expert` | Consomem; nenhuma redefine um valor |
| `start`/`end`, `scrub`, pin, `refresh`, setup de GSAP + Lenis | `gsap-scrolltrigger-expert` | Linkam |
| `share`, `scroll`, `scrollMobile` e a razão 0,68–0,75 | `landing-storytelling-director` | `creative-direction-expert` ratifica; `responsive-e-acessibility` confere |
| Guard de `prefers-reduced-motion` no fluxo | Cada skill que anima, no seu passo | `responsive-e-acessibility` define o padrão e audita |
| Thresholds de frame, peso, `FRAME_SPEED` | `video-to-website` | Linkam `decision.md` |
| Ordem do pipeline de especialistas e o Experience Score | `CLAUDE.md` | Nenhuma repete |

Antes de escrever um bloco novo, pergunte: qual das 14 já é dona disto? Se alguma é, o bloco
vira um link de uma linha.

**O teste que pega a contradição cruzada:** quando duas skills citam o mesmo número, uma delas
tem de dizer de quem ele é. Se as duas escrevem o valor por extenso, elas vão divergir na
primeira edição — e a que estiver errada continuará parecendo autoritativa. Se as duas apontam
uma para a outra, ninguém é dona e o número não existe.

---

## Exemplo completo

Skill mínima e completa, calibrada para este repositório.

**Arquivo:** `.claude/skills/chapter-audit/SKILL.md`

````markdown
---
name: chapter-audit
description: Use when someone asks to audit a chapter, check a section's scroll timeline, or verify a chapter follows the motion rules. Audita um capítulo de src/components/chapters contra as regras de motion e acessibilidade do projeto.
argument-hint: [nome-do-capitulo]
allowed-tools: Read, Grep, Glob, Bash(npm run typecheck)
---

# Chapter Audit

Verifica um capítulo contra as regras de motion do projeto e devolve uma lista de correções
com arquivo e linha. Não edita nada — o relatório é a saída.

## Entrada

O capítulo `$0`. Sem argumento, audite todos os arquivos de `src/components/chapters/`.

## Passos

1. Leia `src/components/chapters/Chapter$0.tsx` e todo hook que ele importa de `src/hooks/`.
2. Verifique cada regra abaixo. Cite arquivo e linha em cada reprovação.

| # | Regra | Reprova quando |
|---|---|---|
| 1 | Timeline dentro de `gsap.matchMedia` com escopo | efeito sem guard de reduced motion |
| 2 | Cleanup chama `mm.revert()` ou `ctx.revert()` | cleanup ausente — Strict Mode registra em dobro |
| 3 | Só `transform` e `opacity` animados | `margin`, `top`, `width` na timeline — dispara layout na main thread |
| 4 | Entrada diferente da do capítulo anterior | mesma direção duas vezes seguidas |
| 5 | Altura do capítulo ≥ 300vh se tem pin | pin com scroll curto — a cena passa antes de ser lida |
| 6 | Copy é texto no DOM, em ordem de leitura | texto dentro de canvas ou imagem |
| 7 | Contraste do texto de corpo ≥ 4.5:1 sobre o fundo (WCAG AA) | abaixo de 4.5:1 — o número é de `responsive-e-acessibility` |

3. Rode `npm run typecheck`. Erro de tipo entra no relatório como reprovação bloqueante.

## Saída

## Auditoria: Chapter$0

**Reprovações**
- [regra N] `caminho:linha` — <o que está errado>
  Conserto: <mudança concreta>

**Aprovadas:** N/7  •  Typecheck: <ok | N erros>

## Notas

- Não conserte nada. Editar um capítulo é trabalho de `gsap-scrolltrigger-expert`.
- Capítulo sem timeline não reprova nas regras 1–5; anote "estático, fora de escopo".
- Não invente número de linha. Sem localizar, escreva "arquivo inteiro".
````

O que este exemplo demonstra: `argument-hint` + `$0`, `allowed-tools` coerente com uma skill que
só lê, tabela com critério de reprovação em vez de lista de boas práticas, template de saída, e
uma nota que impede a skill de invadir o escopo de outra.

---

## Anti-patterns

- **`description` genérica** ("ajuda com design") — não dispara quando precisa e dispara quando
  não precisa, porque o matching é feito sobre essa string e mais nada.
- **SKILL.md acima de 500 linhas** — carrega inteiro ao ser invocada; come o orçamento e o miolo
  do documento perde peso na atenção do modelo.
- **Copiar trecho do CLAUDE.md** — o CLAUDE.md já está no contexto. A cópia diverge no primeiro
  edit e passam a existir duas verdades.
- **`context: fork` em skill de referência** — o subagente recebe diretrizes sem tarefa e devolve
  vazio; parece bug de execução, é erro de frontmatter.
- **Arquivo de apoio sem link no SKILL.md** — arquivos de apoio não carregam sozinhos. Sem o
  link, ele não existe.
- **Lista de opções sem "quando usar"** — o modelo escolhe o primeiro item da lista, sempre, e o
  resultado vira loteria entre execuções.
- **Adjetivo como threshold** — "suave", "premium", "rápido" resolvem diferente a cada execução;
  a skill perde a única função que tinha, que é reprodutibilidade.
- **`` !`comando` `` com saída grande** — a saída inteira entra no prompt antes de a skill
  começar; um `ls -R` num repo grande queima milhares de tokens à toa.
- **Skill nova sobrepondo uma existente** — duas fontes de verdade divergem e ninguém sabe qual
  seguir. Estenda a que já existe.
- **`name` diferente do diretório** — em skill de projeto o comando continua saindo do diretório,
  então o menu exibe um rótulo que ninguém consegue digitar. O bug não aparece em teste, porque
  `/nome-do-diretório` funciona; aparece quando outra pessoa procura a skill pelo nome exibido.

---

## Verificação

Antes de dar a skill por pronta:

- [ ] `wc -l SKILL.md` < 500
- [ ] `name` == nome do diretório; `ls` confirma
- [ ] Todo caminho citado no corpo existe (`ls` em cada um)
- [ ] Todo arquivo de apoio tem link markdown relativo no SKILL.md
- [ ] Todo threshold é um número, não um adjetivo
- [ ] Todo padrão implementável tem código TSX da stack real (React 19, GSAP 3.15, Lenis 1.3, Tailwind 4)
- [ ] Todo anti-pattern tem "porque Y acontece"
- [ ] Reduced motion aparece no passo em que roda, não no rodapé
- [ ] Nenhum bloco duplica outra das 13 skills nem o CLAUDE.md
- [ ] A skill dispara em 3 frases naturais diferentes
- [ ] Bloco `## Suposições` escrito, se algo foi inferido
