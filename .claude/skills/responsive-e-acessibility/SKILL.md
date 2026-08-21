---
name: responsive-e-acessibility
description: Use when an implementation is finished and needs the final gate, or when reviewing any interface for mobile, keyboard and screen-reader behaviour — auditoria de responsividade e acessibilidade, breakpoints, alvos de toque, ordem de foco, semântica, contraste, alt text, prefers-reduced-motion, peso em conexão lenta. Responsive audit, accessibility review, WCAG AA, keyboard navigation, focus order, screen reader, touch targets, contrast ratio, reduced motion, slow 3G/4G budget.
argument-hint: [rota-ou-componente] [--fix]
allowed-tools: Read, Grep, Glob, Bash, Edit, Write, mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_resize, mcp__plugin_playwright_playwright__browser_snapshot, mcp__plugin_playwright_playwright__browser_press_key, mcp__plugin_playwright_playwright__browser_evaluate, mcp__plugin_playwright_playwright__browser_take_screenshot, mcp__plugin_playwright_playwright__browser_console_messages, mcp__plugin_playwright_playwright__browser_network_requests
---

# Auditoria de responsividade e acessibilidade

Esta skill é o portão final. Ela não sugere melhorias: ela **aprova ou reprova**, e quando
reprova cita o passo do protocolo, o arquivo, o que foi observado e o critério violado.

Três desfechos por achado, e só três:

- **BLOQUEIO** — alguém não consegue usar a página. Reprova o build. Sem exceção negociável.
- **RESSALVA** — degrada a experiência sem impedir o uso. Registra-se, não bloqueia.
- **NÃO AUDITÁVEL** — o passo não pôde rodar: servidor fora do ar, rota em 404, build quebrado,
  script devolvendo exceção. Não é aprovação. Um passo não auditável **reprova**, com o motivo
  no lugar do achado — senão a ausência de achado vira selo de qualidade.

APROVADO exige as duas coisas: nenhum bloqueio **e** os dez passos efetivamente rodados.
"Aprovado com ressalvas graves" não existe.

### Classificação — a severidade não é julgada, é lida

Sem esta tabela o portão é decorativo: basta registrar tudo como ressalva e aprovar. Um achado
que não caiba em nenhuma linha é BLOQUEIO por padrão — a dúvida corre a favor de quem não
consegue usar a página. A causa de cada critério está no passo que o produz.

| Passo | BLOQUEIO | RESSALVA |
|---|---|---|
| A1 | `h1` repetido, `main` ausente ou duplicado, heading pulado, `nav` sem rótulo distinto, `aria-labelledby` órfão | — |
| A2 | overflow horizontal; texto de leitura abaixo de 16px; beats sobrepostos; CTA atrás da barra de URL | conteúdo que some por falta de altura, previsto no media query |
| A2b | alvo abaixo de 44px **sem** 24px de folga; menos de 8px entre alvos; menos de 24px da borda | alvo de 24 a 44px **com** ≥24px de folga — a exceção por espaçamento da SC 2.5.8 |
| A3 | zoom bloqueado; rolagem nos dois eixos a 320px; conteúdo perdido ao ampliar | — |
| A4 | `invisiveis` não vazia; `tabindex` positivo; foco preso fora de modal; `Escape` que não fecha; alvo do skip link sem `tabIndex={-1}` | — |
| A5 | texto invisível; `<video>` no DOM; stage ainda sticky; card fora da tela; spinner congelado | razão `scrollMobile / scroll` fora de 0,68–0,75 |
| A6 | texto abaixo de 4.5:1, ou de 3:1 se ≥24px; informação transmitida só por cor | anel de foco entre 3:1 e 4.5:1 — 3:1 é o mínimo da SC 1.4.11, 4.5:1 é regra de texto |
| A7 | LCP > 2.5s; CLS > 0.1; imagem sem `width`/`height`; vídeo baixado além de 1,5 viewport | `saveData` não honrado; `og-image.jpg` acima de 300 KB |
| A8 | `<img>` sem `alt`; controle sem nome acessível; focável sob `aria-hidden` | alt começando com "Foto de"; alt repetindo a legenda logo abaixo |

## Entrada

Rota ou componente a auditar. Sem argumento, audita a página inteira em `src/App.tsx`. Antes de
qualquer coisa leve o servidor no ar — `npm run dev` para A0–A6 e A8, `npm run preview` para A7:
ordem de foco, overflow e contraste sobre vídeo só existem em runtime, e ler o código-fonte não
encontra nenhum dos três.

Com `--fix`, corrija **depois** de emitir o laudo, nunca antes — o laudo é o registro do que
estava errado. Corrigido, **rode o protocolo inteiro de novo** e emita um segundo laudo: uma
correção de contraste muda tamanho de fonte, que muda reflow, que muda alvo de toque. Portão que
conserta sem remedir não é portão, e o veredito válido é sempre o do último laudo.

---

## O sistema deste projeto

Números lidos de `src/styles/index.css`, `vite.config.ts` e `scripts/prepare-assets.mjs`. Não
invente breakpoints: este projeto roda Tailwind 4 sem nenhum override de `--breakpoint-*`, então
valem os cinco defaults, e a contagem de uso abaixo é a real.

| Token | px | Uso real | O que muda de fato |
|---|---|---|---|
| base | 0–639 | 360 / 390 / 414 é o tráfego real | coluna única; `--chapter-scroll-mobile`; vídeo 1280×720; beats em série |
| `sm` | 640 | 13 ocorrências | tipo e gaps; nada estrutural |
| `md` | **768** | 20 ocorrências | **única fronteira estrutural**: `.chapter` troca para `--chapter-scroll-desktop`, `useIsDesktop()` vira true, o vídeo passa a 1920×1080, beats compartilham a stage |
| `lg` | 1024 | 4 ocorrências | nav inline substitui hambúrguer + sheet |
| `xl` | 1280 | 2 ocorrências | trilho de capítulos à direita; = `max-width` do `.container-editorial` |
| `2xl` | 1536 | 0 | não usado — não introduza um sexto |

**Altura também é breakpoint.** `ChapterJourney` usa
`[@media(min-width:768px)_and_(min-height:840px)]` porque a stage tem exatamente `100svh` e o
lead só cabe quando há altura. O caso que quebra primeiro não é o celular em pé: é o celular
deitado (844×390), que tem largura de tablet e nenhuma altura. Sempre que um beat empilha mais
de dois blocos dentro de `100svh`, a condição é de altura, não de largura.

### Tipografia, espaçamento, densidade de motion

Tabelas em [sistema.md](sistema.md), junto com o que cada efeito de desktop precisa virar abaixo
de 768. Três constantes de lá são usadas pelos passos abaixo: **16px** de piso para texto de
leitura, **24px** de piso para afastamento da borda, e **0,68–0,75** para a razão
`scrollMobile / scroll` — faixa de
[landing-storytelling-director](../landing-storytelling-director/SKILL.md#pacing), que este
portão confere e não renegocia.

*Como* adaptar motion é de `gsap-scrolltrigger-expert` e `landing-motion-expert`; este portão só
confere o estado final, por uma regra: **condicione ao que a condição realmente é.** Largura para
layout, altura para empilhamento, `(pointer: fine)` para hover, `(prefers-reduced-motion)` para
movimento. Um `min-width: 768px` fazendo o trabalho de `(hover: hover)` é um bug esperando um tablet.

---

## prefers-reduced-motion — o que o portão exige

Sobrevivem **opacidade e cor abaixo de 250ms**. Morrem deslocamento, escala acima de 1.05,
rotação, blur animado, loop perpétuo e qualquer coisa presa ao scroll. A matriz linha a linha —
vinte tipos e onde cada um mora — está em [reduced-motion.md](reduced-motion.md#a-matriz).

Dois modos de falha, com a causa:

1. **Construir e depois desfazer.** Um `gsap.set(el, { autoAlpha: 0 })` fora do guard
   `matchMedia` deixa a copy invisível para sempre — o guard que a revelaria nunca roda. A
   varredura CSS (`.beat { opacity: 1 !important }`) é rede, não mecanismo: ninguém vai escrever
   `!important` para cada propriedade que uma timeline futura inventar. **Nada é escondido fora
   do guard.**
2. **A varredura global apagando informação.** O bloco `*{animation-duration:.001ms;
   animation-iteration-count:1;transition-duration:.001ms}` deste projeto zera foco e press sem
   perda — o estado final é idêntico ao animado. Mas ele também para **qualquer indicador cujo
   movimento é o conteúdo**: um spinner de progresso indeterminado congela no primeiro quadro e
   o único sinal de "ainda processando" desaparece. `motion-ui-expert` marca os sobreviventes com
   `[data-motion-safe='spin']` e `[data-motion-safe='fade']`; a auditoria confere que existe um
   sobrevivente para cada caso em que a transição *é* a informação — spinner, acordeão que só se
   entende abrindo, passo a passo que só se entende avançando.

---

## Protocolo de auditoria

Dez passos — A0 a A8, com A2b. Cada um tem o que rodar, o que observar e o critério de
reprovação. Cite o código do passo em cada achado (`[A4]`, `[A6]`): é o que torna o laudo
rastreável, e é o que a tabela de classificação usa para atribuir a severidade.

### A0 — Contexto

Leia `src/styles/index.css` (`@theme` e o bloco de reduced motion), `src/App.tsx` (landmarks) e o
componente sob auditoria; liste controles interativos e imagens; se o diff mexeu num `clamp()`,
recalcule antes. Sem reprovação própria: é a linha de base contra a qual os outros nove medem.

### A1 — Árvore semântica

**Rodar:** `browser_snapshot` na rota, com a página no topo.

**Observar:** um único `main`; `header`/`nav`/`footer` presentes; cada `nav` com `aria-label`
próprio quando há mais de uma; um único `h1`; a sequência de headings; `lang="pt-BR"` no `html`.

```js
// Cada capítulo precisa de nome, e o nome precisa existir.
[...document.querySelectorAll('section.chapter')].map((s) => {
  const id = s.getAttribute('aria-labelledby')
  return `${s.id}: ${!id ? 'SEM aria-labelledby' : document.getElementById(id) ? 'ok' : `aponta para #${id}, que não existe`}`
})
```

**Reprova se:** existe mais de um `h1`; um nível de heading é pulado (h2 → h4); `main` está
ausente ou duplicado; duas `nav` sem rótulo distinto; o script acima devolve qualquer coisa
diferente de `ok` (um `aria-labelledby` órfão é pior que nenhum — a seção fica sem nome e o
defeito não aparece em lugar nenhum); um bloco com `--text-hero` é um `<p>` ou `<div>` fazendo o
papel de título — estilo não é semântica, e o leitor de tela lista headings, não tamanhos.

### A2 — Viewports

**Rodar:** `browser_resize` em 360×640, 390×844, 844×390 (paisagem), 768×1024, 1024×768,
1440×900. Em cada um, role o capítulo inteiro.

**Observar:** overflow horizontal, texto cortado, alvo coberto pela chrome do navegador,
sobreposição de beats, imagem esticada.

Dois scripts, **um `browser_evaluate` cada um**. Não os cole juntos: as duas expressões começam
com `[`, e sem ponto e vírgula a segunda vira acesso de índice na primeira — o bloco inteiro
morre em `SyntaxError` e a viewport passa como se estivesse limpa.

```js
// overflow horizontal: quem é o culpado
[...document.querySelectorAll('*')]
  .filter((el) => el.scrollWidth > document.documentElement.clientWidth + 1)
  .map((el) => `${el.tagName}.${el.className}`.slice(0, 90))
```

```js
// texto de leitura abaixo de 16px — o piso que evita o zoom automático do Safari iOS.
// Só nós folha com texto: um wrapper herda `font-size` sem nunca pintar um glifo.
[...document.querySelectorAll('p,li,a,button,label,input,td,dd')]
  .filter((el) => !el.firstElementChild && el.textContent.trim().length > 12)
  .filter((el) => el.checkVisibility({ opacityProperty: true, visibilityProperty: true }))
  .map((el) => ({ px: parseFloat(getComputedStyle(el).fontSize), el }))
  .filter(({ px }) => px < 16)
  .map(({ px, el }) => `${px}px — ${el.outerHTML.slice(0, 70)}`)
```

Rótulos são exceção declarada: `--text-eyebrow` (11px) e a nav do desktop ficam abaixo desse piso
de propósito. Toda outra linha que o segundo script devolver é achado.

**Reprova se:** o primeiro array não volta vazio (`body` tem `overflow-x: clip`, então a barra não
aparece — o vazamento fica invisível e continua empurrando o layout); o segundo devolve qualquer
coisa fora dos rótulos declarados; dois beats se sobrepõem em 390×844 ou 844×390; um CTA fica
atrás da barra de URL (sintoma de `100vh` onde devia ser `100svh`).

### A2b — Alvos de toque

**Rodar:** em 390×844, com o capítulo na tela:

```js
// Mede as três coisas de uma vez: tamanho, folga até o vizinho e distância da borda.
(() => {
  const SEL =
    'a[href],button:not([disabled]),input:not([disabled]),select:not([disabled]),textarea:not([disabled]),summary,[tabindex]:not([tabindex="-1"])'

  // Visível de verdade (a cadeia de ancestrais inclusa) e dentro da tela agora.
  // Não use `offsetParent !== null`: é null em todo `position: fixed`, o que
  // exclui a barra fixa, o sheet e o skip link — metade dos alvos da página.
  const alvos = [...document.querySelectorAll(SEL)]
    .map((el) => ({ el, r: el.getBoundingClientRect() }))
    .filter(({ el, r }) =>
      el.checkVisibility({ opacityProperty: true, visibilityProperty: true }) &&
      r.width > 0 && r.bottom > 0 && r.top < innerHeight)

  // Espaço livre entre duas caixas; negativo quando elas se sobrepõem.
  const folga = (a, b) =>
    Math.max(b.left - a.right, a.left - b.right, b.top - a.bottom, a.top - b.bottom)

  return alvos
    .map(({ el, r }) => {
      // Aninhados fora: a "folga" de um filho para o pai é negativa e não significa nada.
      const vizinho = Math.min(
        Infinity,
        ...alvos
          .filter((o) => o.el !== el && !el.contains(o.el) && !o.el.contains(el))
          .map((o) => folga(r, o.r)),
      )
      const borda = Math.min(r.left, innerWidth - r.right, innerHeight - r.bottom)
      const motivos = []
      if (r.width < 44 || r.height < 44) motivos.push(vizinho >= 24 ? 'pequeno-com-folga' : 'pequeno')
      if (vizinho < 8) motivos.push('vizinho')
      if (borda < 24) motivos.push('borda')
      if (!motivos.length) return null
      return `${motivos.join('+')} — ${Math.round(r.width)}×${Math.round(r.height)}, ` +
        `vizinho ${Math.round(vizinho)}px, borda ${Math.round(borda)}px — ${el.outerHTML.slice(0, 70)}`
    })
    .filter(Boolean)
})()
```

**Por que 44 e não 24:** a WCAG 2.2 pede 24×24 CSS px no AA (SC 2.5.8) e 44×44 no AAA (SC 2.5.5).
O piso aqui é **44**: o tráfego é polegar em movimento dentro de WebView de Instagram, e a área
de contato do polegar mede 11–12 mm, perto de 45 CSS px. Um alvo de 24px não é pequeno demais
para ser tocado — é pequeno demais para ser tocado sem mirar, e ninguém mira rolando. Os 8px de
folga e os 24px de borda saem do mesmo raciocínio: o custo do erro de mira não é simétrico, e
acertar "Fechar" no lugar de "Agendar" encerra a visita. Abaixo de 44 só passa com 24px livres em
volta — é a própria exceção por espaçamento da SC 2.5.8.

As medidas dos alvos que já existem no repo estão em [sistema.md](sistema.md#alvos-de-toque-que-já-existem-no-repo).

**Reprova se:** o script devolve qualquer linha em viewport de toque. O rótulo do motivo já dá a
severidade: `pequeno`, `vizinho` e `borda` são BLOQUEIO; `pequeno-com-folga` é RESSALVA, e só
deixa de ser registrado quando o alvo existe exclusivamente sob `(pointer: fine)`.

### A3 — Zoom e reflow

**Rodar:** `browser_resize` não aplica zoom, e não existe ferramenta aqui que aplique. Use a
equivalência: sobre uma janela de 1280 px, zoom de N% deixa a mesma largura em CSS px que
redimensionar para `1280 ÷ (N / 100)`. Logo 200% → **640×1024** e 400% → **320×1024**. Rode os
dois e role a página inteira em cada um. Depois confira o meta viewport com `Grep` em
`index.html`. A equivalência não é perfeita — o zoom real também escala densidade de pixel — mas
cobre o que a 1.4.10 mede, que é largura em CSS px.

**Observar:** WCAG 1.4.10 — a 320 CSS px de largura o conteúdo precisa caber em uma coluna, sem
rolagem nos dois eixos ao mesmo tempo, e nada pode sumir.

**Reprova se:** aparece rolagem horizontal a 320 px; algum conteúdo desaparece ou fica inacessível
ao ampliar; `user-scalable=no` ou `maximum-scale` menor que 5 está no meta viewport (é WCAG
1.4.4 e é bloqueio automático, sem discussão).

### A4 — Teclado

**Rodar:** com o foco no topo do documento, `browser_press_key` com `Tab` repetidamente até o
rodapé, registrando cada parada.

```js
// Ordem do DOM — igual à ordem de foco só porque este projeto proíbe `tabindex`
// positivo; quem confirma é o `Tab` manual. E, separadamente, as paradas de foco
// invisíveis, que é o que o script existe para achar.
// Nunca filtre por `offsetParent !== null`: é null em todo `position: fixed`, e o
// skip link deste projeto é fixed (index.css) — filtrar assim apaga da lista
// justamente o primeiro item que este passo verifica.
(() => {
  const FOCUSABLE =
    'a[href],area[href],button:not([disabled]),input:not([disabled]),select:not([disabled]),' +
    'textarea:not([disabled]),summary,audio[controls],video[controls],[contenteditable]:not([contenteditable="false"]),' +
    '[tabindex]:not([tabindex="-1"])'
  const seen = { ordem: [], invisiveis: [] }

  for (const el of document.querySelectorAll(FOCUSABLE)) {
    // Sem caixa de layout (`display: none` em algum ancestral): já saiu da ordem
    // de foco, nada a verificar.
    if (el.getClientRects().length === 0) continue

    // `visibility: hidden` também tira da ordem de foco, e é herdado, então este
    // teste pega o ancestral junto. É o que o `autoAlpha` do GSAP escreve:
    // comportamento correto, não achado.
    if (getComputedStyle(el).visibility === 'hidden') continue

    const rotulo = `${el.tagName} "${(el.textContent || el.ariaLabel || '').trim().slice(0, 40)}"`

    // Sobra o caso perigoso: opacidade zero em algum ponto da cadeia, com o
    // controle plenamente focável. `opacityProperty` olha os ancestrais;
    // `getComputedStyle(el).opacity` devolveria 1 dentro de um pai com 0.
    if (el.checkVisibility({ opacityProperty: true })) seen.ordem.push(`${seen.ordem.length}: ${rotulo}`)
    else seen.invisiveis.push(`${rotulo} — ${el.outerHTML.slice(0, 70)}`)
  }
  return seen
})()
```

**Observar:** `ordem` batendo com a ordem visual e **`invisiveis` voltando vazia** — cada item
ali é um controle que recebe `Tab` sem aparecer na tela. Depois, na passada manual: o primeiro
`Tab` revela o skip link; o anel é visível em todos os fundos; `Escape` fecha o sheet e devolve o
foco ao botão que o abriu; `Tab` dentro do sheet circula e não escapa; a página inteira é legível
só com `Espaço`/`PageDown`.

**Reprova se:** `invisiveis` não volta vazia; existe `tabindex` positivo em qualquer lugar (ele
salta na frente de tudo e cria uma ordem que ninguém consegue prever); o skip link não é o item
`0` de `ordem`; o foco fica preso fora de um modal; o alvo do skip link não tem `tabIndex={-1}`
(sem isso o foco fica no `body` e o próximo `Tab` volta para a nav — o skip link vira
decoração); um controle só é alcançável depois de um gesto de ponteiro.

**A armadilha específica de página scroll-driven:** um CTA que só aparece a 66% do capítulo. Se
ele foi escondido com `opacity: 0`, ele continua **plenamente focável e plenamente invisível** —
é exatamente o que cai em `invisiveis`. Use `autoAlpha`: GSAP escreve `visibility: hidden` junto,
e isso tira o elemento da ordem de foco até o reveal rodar. Depois verifique o outro lado:
chegando ao CTA só com `Espaço` + `Tab`, sem mouse e sem trackpad.

### A5 — Reduced motion

**Rodar:** emule `prefers-reduced-motion: reduce` e **recarregue** (os guards rodam na montagem;
alternar sem recarregar não testa nada). Role a página inteira.

**Observar:** todo texto visível desde o início; nenhum capítulo mais alto que seu conteúdo;
nenhum vídeo em reprodução; cards do trilho todos na tela; Lenis não instanciado.

```js
// Nada pode restar invisível. Varra todo nó folha com texto dentro das stages —
// não uma lista de `data-*`, que envelhece a cada capítulo novo. Os alvos reais
// são os `.word__inner`, que é onde o GSAP escreve.
[...document.querySelectorAll('.chapter__stage *')]
  .filter((el) => !el.firstElementChild && el.textContent.trim())
  // Só o que ainda tem caixa de layout. `display: none` sai daqui de propósito:
  // o esconder deliberado deste projeto (`[data-progress-row]`) usa display, e
  // o bug que se caça — um `gsap.set` fora do guard — usa opacity/visibility.
  .filter((el) => el.getClientRects().length > 0)
  .filter((el) => !el.checkVisibility({ opacityProperty: true, visibilityProperty: true }))
  .map((el) => el.outerHTML.slice(0, 80))
```

**Reprova se:** o array acima não está vazio; existe `<video>` no DOM; a página continua mais
alta que a soma dos conteúdos (stage ainda sticky); um card do trilho está fora da viewport;
algum indicador de progresso indeterminado ficou parado — a varredura global congela `animation`
em todo `*`, e um spinner congelado não diz "processando", diz "travou".

### A6 — Contraste

**Rodar:** cada par texto/fundo. Para texto sobre mídia, veja o procedimento em
[alt-e-contraste.md](alt-e-contraste.md#texto-sobre-vídeo).

Mínimos WCAG AA: **4.5:1** para texto normal, **3:1** para texto grande (≥24px, ou ≥18.66px em
bold), **3:1** para bordas de controle, ícones informativos e anel de foco.

Pares medidos desta paleta:

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

**Reprova se:** qualquer par fica abaixo do mínimo da sua categoria; o anel de foco global
(`:focus-visible { outline: 2px solid var(--color-rose-soft) }`) é usado sobre fundo claro —
1.9:1 contra branco, e o `ChapterJourney` abre exatamente em branco; alguma informação é
transmitida só por cor (estado válido/inválido, ativo/inativo, disponível/indisponível).

### A7 — Peso e rede

**Rodar:** `npm run build && npm run preview` e meça **no preview**, nunca no `npm run dev`: o dev
server serve ESM sem bundle nem minificação, e o número que ele dá não existe em produção.

`npx lighthouse http://localhost:4173 --only-categories=performance` dá LCP e CLS já com o
throttling móvel padrão (150 ms RTT, 1,6 Mbps, CPU 4×). Nenhuma ferramenta desta skill aplica
throttling sozinha — `browser_network_requests` só lista o que já passou, e entra depois, para
responder *quais* recursos entraram e em que ordem.

**Observar:** o que entra antes do LCP, e o que entra sem ninguém pedir.

**Reprova se (BLOQUEIO):** LCP > 2.5s nesse perfil; CLS > 0.1; alguma imagem sem `width`/`height`
explícitos (é CLS garantido); algum `<video>` é baixado antes de estar a uma viewport e meia de
distância — `rootMargin: '150% 0px'` no `IntersectionObserver` de `ChapterFilm` é a referência.

**Registra como RESSALVA:** `<video>` baixado com `navigator.connection.saveData` ligado (a API é
só do Chromium, então isto é melhoria progressiva, não requisito); `public/media/og-image.jpg`
acima de 300 KB — passando disso o WhatsApp não renderiza a prévia e o link chega sem imagem,
que é o principal canal de chegada deste público. Hoje o arquivo tem 63 KB.

WebView de Instagram/WhatsApp, `saveData` e LCP: [alt-e-contraste.md](alt-e-contraste.md#conexão-lenta-e-webviews).

### A8 — Mídia e nomes acessíveis

**Rodar:**

```js
// Toda imagem sem alt, e todo controle sem nome acessível. Um botão só de ícone
// pode ser nomeado por `aria-labelledby` ou pelo `alt` de um <img> filho — sem
// esses dois testes o script acusa como anônimo um controle que tem nome.
[...document.images].filter((i) => !i.hasAttribute('alt')).map((i) => i.currentSrc)
  .concat([...document.querySelectorAll('a[href],button,[role="button"]')]
    .filter((el) => !(el.textContent.trim() || el.getAttribute('aria-label') ||
      el.getAttribute('aria-labelledby') || el.getAttribute('title') ||
      [...el.querySelectorAll('img[alt]')].some((i) => i.alt.trim())))
    .map((el) => el.outerHTML.slice(0, 80)))
```

**Observar:** cada `<img>` tem `alt` (mesmo que vazio); vídeo decorativo tem `aria-hidden="true"`
**e** `tabIndex={-1}`; canvas de sequência tem `role="img"` + `aria-label`; SVG que carrega
significado tem `role="img"` + `aria-label`, e SVG decorativo tem `aria-hidden`.

**Reprova se:** o array acima não está vazio (atributo `alt` ausente faz alguns leitores lerem o
nome do arquivo, e `journey-03.a1b2c3d4-640.webp` é a pior frase possível); um elemento focável
está sob `aria-hidden` (vira uma parada de foco que o leitor de tela anuncia como nada); um alt
começa com "Imagem de" / "Foto de"; um alt repete a legenda que está logo abaixo dele.

Alt para pet, equipe, fachada, procedimento e antes/depois, com o par inútil/útil de cada caso:
[alt-e-contraste.md](alt-e-contraste.md#alt-text).

---

## Laudo

Saída obrigatória. Sem ele não houve auditoria.

```
PARECER — responsive-e-acessibility
Alvo: <rota ou componente>   Build: <branch/commit>   Medido em: npm run preview
Perfis: 360×640 · 390×844 · 844×390 · 768×1024 · 1024×768 · 1440×900 · reflow 640/320 ·
        teclado · reduced motion · Lighthouse throttling móvel + CPU 4×

BLOQUEIOS (n)
1. [A6] src/components/ui/Action.tsx:31 — label de 15px em branco sobre --color-rose,
   3.3:1. Mínimo para texto normal é 4.5:1. → usar variant="solid" (ink/branco, 16.6:1).

RESSALVAS (n)
1. [A2] ChapterJourney em 844×390 — o lead some por falta de altura. Comportamento
   previsto pelo media query de altura; sem perda de informação.

NÃO AUDITÁVEL (n)
1. [A7] Lighthouse não rodou: `npm run preview` saiu com erro de build. LCP e CLS
   não foram medidos — ausência de achado aqui não é ausência de defeito.

VEREDITO: REPROVADO
```

`VEREDITO: APROVADO` exige BLOQUEIOS **e** NÃO AUDITÁVEL vazios. Ressalvas podem constar e ainda
assim aprovar — é para isso que a lista existe. Qualquer bloqueio, ou qualquer passo que não
rodou, força REPROVADO.

---

## Checklist de aprovação

Uma linha por passo, e o portão só fecha com as dez preenchidas. "Não rodei" não é um valor
válido: é NÃO AUDITÁVEL, e reprova junto com os bloqueios.

| Passo | Precisa ser verdade | Prova |
|---|---|---|
| A0 | todo `clamp()` tocado pelo diff foi recalculado | valores conferidos contra [sistema.md](sistema.md) |
| A1 | `lang="pt-BR"`, um `main`, um `h1`, nenhum heading pulado, cada `nav` com rótulo distinto, cada capítulo com `aria-labelledby` que resolve | `browser_snapshot` mais o script do A1 devolvendo só `ok` |
| A2 | sem overflow horizontal e sem texto de leitura abaixo de 16px; beats não se sobrepõem; nada atrás da barra de URL (`100svh` para texto, `100lvh` para mídia) | os dois scripts do A2 vazios em 360×640, 390×844, 844×390, 768×1024, 1024×768 e 1440×900 |
| A2b | todo alvo ≥44×44 — ou ≥24×24 com 24px livres em volta; ≥8px entre alvos; ≥24px da borda; nenhuma ação só por hover, long-press ou dois dedos | script do A2b vazio em 390×844 |
| A3 | reflow a 320px sem rolagem nos dois eixos, sem conteúdo perdido; zoom não bloqueado | 640×1024 e 320×1024 rolados inteiros; meta viewport sem `user-scalable=no` e sem `maximum-scale` < 5 |
| A4 | ordem de foco = ordem visual, sem `tabindex` positivo; skip link é a parada `0` e seu alvo tem `tabIndex={-1}`; modal prende o foco, `Escape` fecha e devolve ao gatilho; CTA final alcançável só com `Espaço` e `Tab` | `invisiveis` vazio **e** a passada manual de `Tab` até o rodapé |
| A5 | com reduced motion **e recarga**: todo texto visível, nenhum `<video>` no DOM, nenhum capítulo mais alto que seu conteúdo, nenhum card fora da tela, spinner ainda girando; nada escondido fora de um guard `matchMedia`; `scrollMobile / scroll` entre 0,68 e 0,75 | script do A5 vazio |
| A6 | todo par de texto em 4.5:1 — ou 3:1 se ≥24px; anel de foco ≥3:1 contra **todos** os fundos onde o controle aparece; nenhuma informação só por cor | tabela de pares do A6, e para texto sobre vídeo o procedimento de [alt-e-contraste.md](alt-e-contraste.md#texto-sobre-vídeo) medindo o **frame mais claro**, nunca o poster |
| A7 | LCP ≤ 2.5s e CLS ≤ 0.1; toda imagem com `width`/`height`; nenhum vídeo baixado a mais de 1,5 viewport | Lighthouse com throttling móvel e CPU 4×, sobre `npm run preview` — nunca sobre o dev server |
| A8 | todo `<img>` com `alt` (decorativa com `alt=""`, nenhum começando com "Foto de"); vídeo decorativo com `aria-hidden` e `tabIndex={-1}`; canvas com `role="img"` e label | script do A8 vazio |

---

## Anti-patterns

- **`opacity: 0` para esconder algo focável** — o controle continua na ordem de foco, invisível.
  Use `autoAlpha`, que escreve `visibility: hidden` junto.
- **`aria-hidden="true"` em elemento focável** — cria uma parada de foco que o leitor de tela
  anuncia como silêncio. Se esconde, tire da ordem: `tabIndex={-1}` junto, sempre.
- **`tabindex` positivo** — reordena o documento inteiro à frente de todo o resto; o próximo
  componente que alguém adicionar cai no lugar errado sem ninguém entender por quê.
- **Rebaixar um bloqueio a ressalva para fechar o portão** — a severidade sai da tabela de
  classificação, não do prazo. Um bloqueio renomeado continua impedindo alguém de usar a página;
  a única diferença é que agora ninguém vai procurá-lo.
- **Aprovar um passo que não rodou** — servidor fora do ar devolve zero achados, e zero achados
  lidos como aprovação transformam um build quebrado em selo de qualidade. É NÃO AUDITÁVEL.
- **Medir contraste sobre o poster do vídeo** — o poster é um frame escolhido a dedo; o texto
  precisa sobreviver ao frame mais claro que ele pode cobrir, em qualquer recorte de viewport.
- **`min-width: 768px` para desligar hover** — um iPad tem 1024px e nenhum ponteiro.
- **Breakpoint novo para resolver um caso** — cinco já existem e vinte arquivos assumem 768.
  Um sexto conserta uma tela e desalinha as outras dezenove.
- **`100vh` em qualquer lugar** — no Android a barra de URL come 60–110px e leva o CTA junto.
