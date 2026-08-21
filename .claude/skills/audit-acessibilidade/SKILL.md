---
name: audit-acessibilidade
description: Use when auditing an interface for keyboard, screen reader and WCAG AA compliance after the responsive gate. Portao 11b de acessibilidade, contraste 4.5:1 e 3:1, ordem de foco, focus-visible, skip link, armadilha de foco, semantica, hierarquia de headings, h1 unico, landmark, alt text util, aria-hidden, role=img, nome acessivel, prefers-reduced-motion, scrim sobre video, leitor de tela, navegacao por teclado, WCAG AA.
argument-hint: [rota-ou-componente] [--fix]
allowed-tools: Read, Grep, Glob, Bash, Edit, Write, mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_resize, mcp__plugin_playwright_playwright__browser_snapshot, mcp__plugin_playwright_playwright__browser_press_key, mcp__plugin_playwright_playwright__browser_evaluate, mcp__plugin_playwright_playwright__browser_take_screenshot
---

# Portão de acessibilidade

| | |
|---|---|
| **ENTRADA** | rota ou componente; `design/laudo-responsivo.md` **aprovado**; `src/styles/index.css` (`@theme` e o bloco de reduced motion); servidor em `npm run dev` |
| **SAÍDA** | `design/laudo-acessibilidade.md` — BLOQUEIOS, RESSALVAS, NÃO AUDITÁVEL, VEREDITO |
| **ANTES** | `audit-responsivo` (Fase 11a) |
| **DEPOIS** | `audit-performance` (Fase 11c) |

Fase 11b de [landing-page-factory](../landing-page-factory/SKILL.md). Roda depois do responsivo
porque contraste, ordem de foco e overflow só existem em runtime, e o runtime muda a cada correção
de layout. Ler o código-fonte não encontra nenhum dos três.

- **BLOQUEIO** — alguém não consegue usar a página. Reprova o build.
- **RESSALVA** — degrada sem impedir. Registra-se, não bloqueia.
- **NÃO AUDITÁVEL** — o passo não rodou. Não é aprovação, e reprova junto.

Com `--fix`, corrija **depois** do laudo e rode o protocolo inteiro de novo: uma correção de
contraste muda tamanho de fonte, que muda reflow, que muda alvo de toque — nesse caso
`audit-responsivo` também precisa rodar outra vez. Vale sempre o último laudo.

### Classificação — a severidade é lida, não julgada

Achado que não caiba em nenhuma linha é BLOQUEIO por padrão.

| Passo | BLOQUEIO | RESSALVA |
|---|---|---|
| A1 | `h1` repetido, `main` ausente ou duplicado, heading pulado, `nav` sem rótulo distinto, `aria-labelledby` órfão, `lang` errado | — |
| A2 | `invisiveis` não vazia; `tabindex` positivo; foco preso fora de modal; `Escape` que não fecha; alvo do skip link sem `tabIndex={-1}` | anel de foco entre 3:1 e 4.5:1 — 3:1 é o mínimo da SC 1.4.11 |
| A3 | `<img>` sem `alt`; controle sem nome acessível; focável sob `aria-hidden` | alt começando com "Foto de"; alt repetindo a legenda logo abaixo |
| A4 | texto abaixo de 4.5:1, ou de 3:1 se ≥24px; informação transmitida só por cor; scrim insuficiente no frame mais claro | ícone decorativo abaixo de 3:1 |
| A5 | texto invisível sob reduced motion; `<video>` no DOM; stage ainda sticky; card fora da tela; spinner congelado | transição de 250–400ms que sobreviveu ao guard |

## Os números da WCAG AA

| Alvo | Mínimo | Vale para |
|---|---|---|
| Texto normal | **4.5:1** | qualquer texto abaixo de 24px, e abaixo de 18.66px em bold |
| Texto grande | **3:1** | ≥24px, ou ≥18.66px em bold (SC 1.4.3) |
| Não-texto | **3:1** | borda de controle, ícone informativo, anel de foco (SC 1.4.11) |
| Redimensionar | 200% sem perda | SC 1.4.4 — medido em `audit-responsivo` R3 |

### Pares medidos desta paleta

| Par | Razão | Serve para |
|---|---|---|
| `--color-body` #1f1f1f sobre branco | 16.5:1 | qualquer texto |
| `--color-ink` #32151e sobre branco | 16.6:1 | qualquer texto |
| `--color-body-soft` #666 sobre branco | 5.7:1 | corpo — é o cinza mais claro aceitável |
| `--color-body-soft` sobre `--color-canvas-soft` | 5.3:1 | corpo |
| branco sobre `--color-rose` #e95d79 | **3.3:1** | só texto grande. Botão rosa com label de 15px reprova |
| `--color-rose-soft` #f7a5b8 sobre branco | **1.9:1** | nada. Reprova como anel de foco em capítulo claro |
| `--color-rose` sobre branco | 3.3:1 | anel de foco, bordas, ícones |
| `--color-gilt` #e8b98a sobre `--color-ink` | 9.3:1 | texto e filete |
| `--color-gilt` sobre branco | **1.8:1** | só filete. Nunca texto |
| `--color-success` #5fa870 sobre branco | **2.9:1** | nada sozinho — nem ícone informativo |

---

## Protocolo

Seis passos, A0 a A5. Cite o código em cada achado (`[A2]`): é o que torna o laudo rastreável e é
o que a tabela de classificação usa para atribuir severidade.

### A0 — Contexto

Leia `src/App.tsx` (landmarks), o bloco de reduced motion de `src/styles/index.css` e o
componente sob auditoria; liste controles interativos e imagens. Sem reprovação própria.

### A1 — Árvore semântica

**Rodar:** `browser_snapshot` na rota, com a página no topo.

```js
// Cada capítulo precisa de nome, e o nome precisa existir.
[...document.querySelectorAll('section.chapter')].map((s) => {
  const id = s.getAttribute('aria-labelledby')
  return `${s.id}: ${!id ? 'SEM aria-labelledby' : document.getElementById(id) ? 'ok' : `aponta para #${id}, que não existe`}`
})
```

**Reprova se:** existe mais de um `h1`; um nível é pulado (h2 → h4); `main` ausente ou duplicado;
duas `nav` sem rótulo distinto; `lang` diferente de `pt-BR`; o script devolve algo fora de `ok`
(um `aria-labelledby` órfão é pior que nenhum — a seção fica sem nome e o defeito não aparece em
lugar nenhum); um bloco com `--text-hero` é `<p>` ou `<div>` — o leitor lista heading, não tamanho.

### A2 — Teclado e foco

**Rodar:** com o foco no topo do documento, `browser_press_key` com `Tab` até o rodapé,
registrando cada parada. E o script abaixo, que acha as paradas invisíveis.

```js
// Ordem do DOM — igual à ordem de foco só porque este projeto proíbe `tabindex`
// positivo; quem confirma é o `Tab` manual. Nunca filtre por `offsetParent !== null`:
// é null em todo `position: fixed`, e o skip link deste projeto é fixed — filtrar
// assim apaga da lista o primeiro item que este passo verifica.
(() => {
  const FOCUSABLE =
    'a[href],area[href],button:not([disabled]),input:not([disabled]),select:not([disabled]),' +
    'textarea:not([disabled]),summary,audio[controls],video[controls],[contenteditable]:not([contenteditable="false"]),' +
    '[tabindex]:not([tabindex="-1"])'
  const seen = { ordem: [], invisiveis: [] }

  for (const el of document.querySelectorAll(FOCUSABLE)) {
    // Sem caixa de layout (`display: none` em algum ancestral): já saiu da ordem de foco.
    if (el.getClientRects().length === 0) continue
    // `visibility: hidden` também tira da ordem de foco, e é herdado. É o que o
    // `autoAlpha` do GSAP escreve: comportamento correto, não achado.
    if (getComputedStyle(el).visibility === 'hidden') continue

    const rotulo = `${el.tagName} "${(el.textContent || el.ariaLabel || '').trim().slice(0, 40)}"`

    // Sobra o caso perigoso: opacidade zero em algum ponto da cadeia, com o controle
    // focável. `opacityProperty` olha os ancestrais; `getComputedStyle` devolveria 1.
    if (el.checkVisibility({ opacityProperty: true })) seen.ordem.push(`${seen.ordem.length}: ${rotulo}`)
    else seen.invisiveis.push(`${rotulo} — ${el.outerHTML.slice(0, 70)}`)
  }
  return seen
})()
```

**Observar:** `ordem` batendo com a ordem visual e **`invisiveis` vazia**. Na passada manual: o
primeiro `Tab` revela o skip link; o anel `:focus-visible` aparece em todo fundo onde o controle
existe; `Escape` fecha o sheet e devolve o foco ao gatilho; `Tab` dentro dele circula e não escapa.

**Reprova se:** `invisiveis` não volta vazia; existe `tabindex` positivo (salta na frente de tudo
e cria uma ordem que ninguém consegue prever); o skip link não é a parada `0`; o alvo do skip link
não tem `tabIndex={-1}` (sem isso o foco fica no `body`, o próximo `Tab` volta para a nav e o skip
link vira decoração); o foco fica preso fora de um modal; um controle só é alcançável depois de um
gesto de ponteiro.

**A armadilha da página scroll-driven:** um CTA que só aparece a 66% do capítulo. Escondido com
`opacity: 0`, ele continua **plenamente focável e plenamente invisível** — é exatamente o que cai
em `invisiveis`. Use `autoAlpha`: o GSAP escreve `visibility: hidden` junto, e isso o tira da
ordem de foco até o reveal rodar. Depois teste o outro lado: chegue ao CTA só com `Espaço` + `Tab`.

### A3 — Nomes acessíveis e alt

**Rodar:**

```js
// Toda imagem sem alt, e todo controle sem nome acessível. Um botão só de ícone pode
// ser nomeado por `aria-labelledby` ou pelo `alt` de um <img> filho — sem esses dois
// testes o script acusa como anônimo um controle que tem nome.
[...document.images].filter((i) => !i.hasAttribute('alt')).map((i) => i.currentSrc)
  .concat([...document.querySelectorAll('a[href],button,[role="button"]')]
    .filter((el) => !(el.textContent.trim() || el.getAttribute('aria-label') ||
      el.getAttribute('aria-labelledby') || el.getAttribute('title') ||
      [...el.querySelectorAll('img[alt]')].some((i) => i.alt.trim())))
    .map((el) => el.outerHTML.slice(0, 80)))
```

**Alt útil responde "o que se perde se a imagem não carregar?"** — não "o que aparece na imagem".
Três pares, e o teto de trabalho é 125 caracteres:

| Cena | Inútil | Útil |
|---|---|---|
| Fachada | `alt="fachada da clínica"` | `alt="Fachada na Rua Honório Hermeto, com letreiro iluminado e porta de vidro no nível da calçada."` — quem chega precisa **reconhecer o lugar da calçada** |
| Antes e depois | `alt="antes e depois"` | `alt="Antes e depois de harmonização facial: à esquerda, sulcos marcados ao redor da boca; à direita, contorno suavizado."` |
| Ícone dentro de botão com label | `alt="seta"` | `aria-hidden="true"` — o botão já tem nome; a seta é reforço visual |

As sete regras e os onze casos deste tipo de projeto: [alt-e-scrim.md](alt-e-scrim.md#alt-text).

**Reprova se:** o array não volta vazio (`alt` ausente faz alguns leitores lerem o nome do
arquivo, e `journey-03.a1b2c3d4-640.webp` é a pior frase possível — decorativa é `alt=""`, nunca
ausente); um focável está sob `aria-hidden` (vira parada de foco que o leitor anuncia como
silêncio); um alt começa com "Imagem de" / "Foto de" ou repete a legenda logo abaixo; um canvas ou
SVG com significado está sem `role="img"` + `aria-label`; vídeo decorativo sem `tabIndex={-1}`.

### A4 — Contraste

**Rodar:** cada par texto/fundo contra a tabela da paleta. Para texto sobre vídeo ou foto, o
procedimento com `brightestUnder` e `scrimAlpha` está em
[alt-e-scrim.md](alt-e-scrim.md#texto-sobre-vídeo): amostre cinco posições do clipe, em 390×844 e
1440×900, e compare o pior frame com a opacidade real do gradiente ali.

**Reprova se:** qualquer par fica abaixo do mínimo da sua categoria; o anel de foco global
(`:focus-visible { outline: 2px solid var(--color-rose-soft) }`) é usado sobre fundo claro — 1.9:1
contra branco, e o `ChapterJourney` abre exatamente em branco; alguma informação é transmitida só
por cor (válido/inválido, ativo/inativo, disponível/indisponível); o alpha de scrim necessário no
frame mais claro é maior que o aplicado ali.

### A5 — Reduced motion

**Rodar:** emule `prefers-reduced-motion: reduce` e **recarregue** — os guards rodam na montagem,
e alternar sem recarregar não testa nada. Role a página inteira.

```js
// Nada pode restar invisível. Varra todo nó folha com texto dentro das stages — não
// uma lista de `data-*`, que envelhece a cada capítulo novo.
[...document.querySelectorAll('.chapter__stage *')]
  .filter((el) => !el.firstElementChild && el.textContent.trim())
  // Só o que ainda tem caixa de layout. `display: none` sai daqui de propósito: o
  // esconder deliberado deste projeto (`[data-progress-row]`) usa display, e o bug
  // que se caça — um `gsap.set` fora do guard — usa opacity/visibility.
  .filter((el) => el.getClientRects().length > 0)
  .filter((el) => !el.checkVisibility({ opacityProperty: true, visibilityProperty: true }))
  .map((el) => el.outerHTML.slice(0, 80))
```

**A matriz.** Sobrevivem **opacidade e cor abaixo de 250ms**. Morrem deslocamento, escala acima de
1.05, rotação, blur animado, loop perpétuo e qualquer coisa presa ao scroll.

| Tipo de animação | Sob reduced motion | Onde |
|---|---|---|
| Reveal de entrada (y + blur + opacity) | **não é construído**; nasce visível | `mm.add('(prefers-reduced-motion: no-preference)')` |
| Timeline scrubbada do capítulo | não existe | `useChapterTimeline` |
| Stage sticky | vira seção normal em fluxo | CSS: `.chapter{height:auto}` |
| Parallax de fundo | removido inteiro | — |
| Zoom lento de imagem (1.1 → 1) | removido; `scale: 1` | — |
| Scrub de vídeo / canvas | não construído; poster com `alt` | componente, antes do primeiro fetch |
| Vídeo de fundo em autoplay | o `<video>` **não é montado** | `useReducedMotion()` |
| Marquee / loop infinito | `animation: none` | CSS |
| Cross-fade entre beats | mantém a opacidade, perde o deslocamento | CSS/GSAP |
| Trilho horizontal | `flex-wrap: wrap` — nenhum card fica fora da tela | CSS `[data-rail]` |
| Contador animado | escreve o valor final direto | componente |
| Barra de progresso de leitura | escondida — progresso que não avança é ruído | `[data-progress-row]` |
| Scroll cue | escondido | CSS |
| Smooth scroll (Lenis) | não instanciado; scroll nativo | `SmoothScrollProvider` |
| Navegação por âncora | corte instantâneo (`immediate: true`) | `jumpTo` |
| Spinner indeterminado | **continua girando**, mais devagar | `[data-motion-safe='spin']` |
| Hover de cor / sombra | mantém | — |
| Press `scale(0.98)` | mantém a cor, perde a escala | `motion-reduce:active:scale-100` |
| Anel de foco | mantém (o estado final é idêntico) | — |
| Curtain / loader | opacidade sem transform | CSS inline no `index.html` |

Implementação, as três camadas e o código de cada linha: [reduced-motion.md](reduced-motion.md).

**Dois modos de falha.** (1) **Construir e depois desfazer:** um `gsap.set(el, { autoAlpha: 0 })`
fora do guard `matchMedia` deixa a copy invisível para sempre — o guard que a revelaria nunca
roda. Nada é escondido fora do guard. (2) **A varredura global apagando informação:** o bloco
`*{animation-duration:.001ms}` zera foco e press sem perda, mas congela todo indicador cujo
movimento *é* o conteúdo — um spinner parado não diz "processando", diz "travou".

**Reprova se:** o array não está vazio; existe `<video>` no DOM; a página continua mais alta que a
soma dos conteúdos (stage ainda sticky); um card do trilho está fora da viewport; um indicador
indeterminado ficou parado.

---

## Laudo

Saída obrigatória, em `design/laudo-acessibilidade.md`.

```
PARECER — audit-acessibilidade
Alvo: <rota ou componente>   Build: <branch/commit>
Passos: A1 semântica · A2 teclado · A3 nomes e alt · A4 contraste · A5 reduced motion

BLOQUEIOS (n)
1. [A4] src/components/ui/Action.tsx:31 — label de 15px em branco sobre --color-rose,
   3.3:1. Mínimo para texto normal é 4.5:1. → usar variant="solid" (ink/branco, 16.6:1).

RESSALVAS (n)
1. [A3] ChapterClinic — alt do retrato começa com "Foto de". Sem perda de informação,
   mas o leitor anuncia "imagem: foto de…".

NÃO AUDITÁVEL (n)
1. [A5] emulação de reduced motion não rodou: o dev server caiu na recarga.

VEREDITO: REPROVADO
```

`VEREDITO: APROVADO` exige BLOQUEIOS **e** NÃO AUDITÁVEL vazios.

## Checklist de aprovação

| Passo | Precisa ser verdade | Prova |
|---|---|---|
| A1 | `lang="pt-BR"`, um `main`, um `h1`, nenhum heading pulado, cada `nav` com rótulo distinto, cada capítulo com `aria-labelledby` que resolve | `browser_snapshot` mais o script do A1 devolvendo só `ok` |
| A2 | ordem de foco = ordem visual, sem `tabindex` positivo; skip link é a parada `0` e seu alvo tem `tabIndex={-1}`; modal prende o foco, `Escape` fecha e devolve ao gatilho; CTA final alcançável só com `Espaço` e `Tab` | `invisiveis` vazio **e** a passada manual até o rodapé |
| A3 | todo `<img>` com `alt` (decorativa com `alt=""`, nenhum começando com "Foto de"); todo controle com nome acessível; vídeo decorativo com `aria-hidden` e `tabIndex={-1}`; canvas com `role="img"` e label | script do A3 vazio |
| A4 | todo par de texto em 4.5:1 — ou 3:1 se ≥24px; anel de foco ≥3:1 contra **todos** os fundos onde o controle aparece; nenhuma informação só por cor | tabela de pares, e para texto sobre vídeo o procedimento de [alt-e-scrim.md](alt-e-scrim.md#texto-sobre-vídeo) medindo o **frame mais claro**, nunca o poster |
| A5 | com reduced motion **e recarga**: todo texto visível, nenhum `<video>` no DOM, nenhum capítulo mais alto que seu conteúdo, nenhum card fora da tela, spinner ainda girando; nada escondido fora de um guard `matchMedia` | script do A5 vazio |

## Anti-patterns

- **`opacity: 0` para esconder algo focável** — o controle continua na ordem de foco, invisível.
  Use `autoAlpha`, que escreve `visibility: hidden` junto.
- **`aria-hidden="true"` em elemento focável** — cria uma parada de foco que o leitor de tela
  anuncia como silêncio. Se esconde, tire da ordem: `tabIndex={-1}` junto, sempre.
- **`tabindex` positivo** — reordena o documento inteiro à frente de todo o resto; o próximo
  componente que alguém adicionar cai no lugar errado sem ninguém entender por quê.
- **Medir contraste sobre o poster do vídeo** — o poster é um frame escolhido a dedo; o texto
  precisa sobreviver ao frame mais claro que pode cobrir, em qualquer recorte de viewport.
- **`outline: none` com "o foco fica feio"** — o anel é o único jeito de saber onde se está sem
  mouse. Redesenhe o anel; não o apague.
- **Alt gerado a partir do nome do arquivo** — descreve o arquivo, não o motivo pelo qual a imagem
  está na página.
- **Rebaixar um bloqueio a ressalva para fechar o portão** — a severidade sai da tabela, não do
  prazo. Um bloqueio renomeado continua impedindo alguém de usar a página.
- **Aprovar um passo que não rodou** — servidor fora do ar devolve zero achados, e zero achados
  lidos como aprovação viram selo de qualidade sobre um build quebrado. É NÃO AUDITÁVEL.
