---
name: "audit-responsivo"
description: "Use when auditing a finished implementation for mobile, tablet and viewport behaviour, before the accessibility gate. Portao 11a de responsividade, breakpoint, 375px sem scroll horizontal, overflow, texto abaixo de 16px, alvo de toque 44px, reflow 320px, zoom bloqueado, 100svh, densidade de motion por breakpoint, pin que vira sticky, parallax com amplitude reduzida, trilho horizontal que vira vertical, razao scrollMobile/scroll."
argument-hint: "[rota-ou-componente] [--fix]"
allowed-tools: Read, Grep, Glob, Bash, Edit, Write, mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_resize, mcp__plugin_playwright_playwright__browser_snapshot, mcp__plugin_playwright_playwright__browser_evaluate, mcp__plugin_playwright_playwright__browser_take_screenshot
---

# Portão de responsividade

| | |
|---|---|
| **ENTRADA** | rota ou componente a auditar; `src/styles/index.css` (`@theme`); `design/creative-direction.json` (`scroll` e `scrollMobile` por capítulo); servidor em `npm run dev` |
| **SAÍDA** | `design/laudo-responsivo.md` — BLOQUEIOS, RESSALVAS, NÃO AUDITÁVEL, VEREDITO |
| **ANTES** | `landing-motion-expert` (Fase 10c) e todos os especialistas que ele roteou — `gsap-scrolltrigger-expert`, `video-decisao` e sua cadeia, `motion-ui-expert` |
| **DEPOIS** | `audit-acessibilidade` (Fase 11b) |

Fase 11a de [landing-page-factory](../landing-page-factory/SKILL.md). Primeiro dos três portões
porque toda correção de layout muda tamanho de fonte e posição de elemento, e as duas coisas
mudam contraste e ordem de foco: auditar acessibilidade sobre um layout ainda instável obriga a
auditar de novo.

Este portão não sugere melhorias. Ele **aprova ou reprova**, e quando reprova cita o passo, o
arquivo, o que foi observado e o critério violado.

- **BLOQUEIO** — alguém não consegue usar a página no aparelho dele. Reprova o build.
- **RESSALVA** — degrada sem impedir. Registra-se, não bloqueia.
- **NÃO AUDITÁVEL** — o passo não rodou: servidor fora do ar, rota em 404, script com exceção.
  Não é aprovação, e reprova junto — senão ausência de achado vira selo de qualidade.

Sem argumento, audita a página inteira. Com `--fix`, corrija **depois** de emitir o laudo e
**rode o protocolo inteiro de novo**: mudar um `clamp()` para caber em 360 muda o reflow e a
folga entre alvos. Portão que conserta sem remedir não é portão; vale o último laudo.

### Classificação — a severidade é lida, não julgada

Achado que não caiba em nenhuma linha é BLOQUEIO por padrão: a dúvida corre a favor de quem não
consegue usar a página.

| Passo | BLOQUEIO | RESSALVA |
|---|---|---|
| R1 | overflow horizontal; texto de leitura abaixo de 16px; beats sobrepostos; CTA atrás da barra de URL | conteúdo que some por falta de altura, previsto no media query |
| R2 | alvo abaixo de 44px **sem** 24px de folga; menos de 8px entre alvos; menos de 24px da borda | alvo de 24 a 44px **com** ≥24px de folga — a exceção por espaçamento da SC 2.5.8 |
| R3 | zoom bloqueado; rolagem nos dois eixos a 320px; conteúdo perdido ao ampliar | — |
| R4 | pin com `pin-spacer` ativo abaixo de 768; parallax acima de ±40px no mobile; trilho horizontal preso ao scroll y no touch | razão `scrollMobile / scroll` fora de 0,68–0,75 |

---

## Os breakpoints deste projeto

Tailwind 4 sem nenhum override de `--breakpoint-*`: valem os cinco defaults, e a contagem de uso
abaixo é a real. Não invente um sexto.

| Token | px | Uso real | O que muda de fato |
|---|---|---|---|
| base | 0–639 | 360 / 375 / 390 / 414 é o tráfego real | coluna única; `--chapter-scroll-mobile`; vídeo 1280×720; beats em série |
| `sm` | 640 | 13 ocorrências | tipo e gaps; nada estrutural |
| `md` | **768** | 20 ocorrências | **única fronteira estrutural**: `.chapter` troca para `--chapter-scroll-desktop`, `useIsDesktop()` vira true, o vídeo passa a 1920×1080, beats compartilham a stage |
| `lg` | 1024 | 4 ocorrências | nav inline substitui hambúrguer + sheet |
| `xl` | 1280 | 2 ocorrências | trilho de capítulos à direita; = `max-width` do `.container-editorial` |
| `2xl` | 1536 | 0 | não usado |

**Altura também é breakpoint.** `ChapterJourney` usa
`[@media(min-width:768px)_and_(min-height:840px)]` porque a stage tem exatamente `100svh` e o
lead só cabe quando há altura. O caso que quebra primeiro não é o celular em pé: é o celular
deitado (844×390), que tem largura de tablet e nenhuma altura. Sempre que um beat empilha mais de
dois blocos dentro de `100svh`, a condição é de altura, não de largura.

**Condicione ao que a condição realmente é.** Largura para layout, altura para empilhamento,
`(pointer: fine)` para hover, `(prefers-reduced-motion)` para movimento. Um `min-width: 768px`
fazendo o trabalho de `(hover: hover)` é um bug esperando um tablet.

Tipografia resolvida por largura, espaçamento, densidade de motion e as medidas dos alvos que já
existem no repo: [sistema.md](sistema.md). Três constantes de lá valem nos passos abaixo: **16px**
de piso para texto de leitura, **24px** para afastamento da borda, **0,68–0,75** para a razão
`scrollMobile / scroll`.

## Adaptação de motion por breakpoint

*Como* adaptar é de `gsap-scrolltrigger-expert` (`gsap.matchMedia`, sticky em vez de pin) e de
`landing-motion-expert` (orçamento por seção). Esta tabela é o **estado final** que o portão
confere, com o passo que pega cada linha.

| Efeito no desktop | Abaixo de 768 precisa estar | Pego em | Porque |
|---|---|---|---|
| Stage sticky com 4 beats simultâneos | mesma stage, beats em série — um sai, o outro entra | R1 | 640px de altura não comportam título mais grade de quatro; eles se sobrepõem. É o `if (isMobile)` de `ChapterExperience` |
| Pin via `ScrollTrigger.pin` | `position: sticky` puro em CSS | R4 | o pin-spacer é medido em JS, e a barra de URL do Android muda a altura no meio da rolagem: o pin salta. Sticky não mede nada |
| Parallax ±120px | ≤ ±40px, ou removido | R4 | em 640px de altura, 120px de deslocamento tiram o elemento do enquadramento antes de ele ser lido |
| Scrub de N telas | 0,68 N a 0,75 N | R4 | o mesmo capítulo custa três vezes mais dedo no celular do que roda de scroll no desktop |
| Trilho horizontal preso ao scroll y | pilha vertical, ou `scroll-snap-type: x mandatory` nativo | R4 | arrastar para baixo para mover algo para o lado é um gesto brigando com o outro; e o card fora da tela nunca recebe foco |
| Cursor magnético (`useMagnetic`) | ausente, por `(pointer: fine)` | R2 | um tablet de 1024px tem largura de desktop e nenhum ponteiro |
| Canvas de 1920 com 300 frames | set de 1280, ou só o poster | — | é peso: cai em `audit-performance` |
| `100vh` | `100svh` para texto, `100lvh` para mídia | R1 | com `100vh` a barra de URL cobre o CTA. O projeto já separa os dois em `.chapter__stage` e `.chapter__media` |

---

## Protocolo

Cinco passos, R0 a R4. Cite o código em cada achado (`[R2]`): é o que torna o laudo rastreável e
é o que a tabela de classificação usa para atribuir severidade.

### R0 — Contexto

Leia o `@theme` de `src/styles/index.css` e o componente sob auditoria; liste os controles
interativos; se o diff mexeu num `clamp()`, recalcule contra [sistema.md](sistema.md) antes de
medir. Sem reprovação própria: é a linha de base dos outros quatro passos.

### R1 — Viewports

**Rodar:** `browser_resize` em 360×640, **375×667**, 390×844, 844×390 (paisagem), 768×1024,
1024×768 e 1440×900. Em cada um, role o capítulo inteiro.

Dois scripts, **um `browser_evaluate` cada**. Não os cole juntos: as duas expressões começam com
`[`, e sem ponto e vírgula a segunda vira acesso de índice na primeira — o bloco morre em
`SyntaxError` e a viewport passa como se estivesse limpa.

```js
// Overflow horizontal: quem é o culpado.
[...document.querySelectorAll('*')]
  .filter((el) => el.scrollWidth > document.documentElement.clientWidth + 1)
  .map((el) => `${el.tagName}.${el.className}`.slice(0, 90))
```

```js
// Texto de leitura abaixo de 16px — o piso que evita o zoom automático do Safari iOS.
// Só nós folha com texto: um wrapper herda `font-size` sem nunca pintar um glifo.
[...document.querySelectorAll('p,li,a,button,label,input,td,dd')]
  .filter((el) => !el.firstElementChild && el.textContent.trim().length > 12)
  .filter((el) => el.checkVisibility({ opacityProperty: true, visibilityProperty: true }))
  .map((el) => ({ px: parseFloat(getComputedStyle(el).fontSize), el }))
  .filter(({ px }) => px < 16)
  .map(({ px, el }) => `${px}px — ${el.outerHTML.slice(0, 70)}`)
```

Rótulos são exceção declarada: `--text-eyebrow` (11px) e a nav do desktop ficam abaixo do piso de
propósito. Toda outra linha que o segundo script devolver é achado.

**Reprova se:** o primeiro array não volta vazio (`body` tem `overflow-x: clip`, então a barra
nunca aparece — o vazamento fica invisível e continua empurrando o layout); o segundo devolve
qualquer coisa fora dos rótulos declarados; dois beats se sobrepõem em 390×844 ou 844×390; um CTA
fica atrás da barra de URL (sintoma de `100vh` onde devia ser `100svh`).

O emulador acerta a largura e erra a chrome do navegador. A verificação de entrega da Fase 12 é
**375px em aparelho real**, e é lá que o `100vh` esquecido aparece.

### R2 — Alvos de toque

**Rodar:** em 390×844, com o capítulo na tela.

```js
// Mede as três coisas de uma vez: tamanho, folga até o vizinho e distância da borda.
(() => {
  const SEL =
    'a[href],button:not([disabled]),input:not([disabled]),select:not([disabled]),textarea:not([disabled]),summary,[tabindex]:not([tabindex="-1"])'

  // Visível de verdade (cadeia de ancestrais inclusa) e dentro da tela agora.
  // Não use `offsetParent !== null`: é null em todo `position: fixed`, o que exclui
  // a barra fixa, o sheet e o skip link — metade dos alvos da página.
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
O piso aqui é **44**: o tráfego é polegar em movimento dentro de WebView de Instagram, e a área de
contato do polegar mede 11–12 mm, perto de 45 CSS px. Um alvo de 24px não é pequeno demais para
ser tocado — é pequeno demais para ser tocado sem mirar, e ninguém mira rolando. Os 8px de folga e
os 24px de borda saem do mesmo raciocínio: o custo do erro de mira não é simétrico, e acertar
"Fechar" no lugar de "Agendar" encerra a visita.

**Reprova se:** o script devolve qualquer linha em viewport de toque. O rótulo do motivo já dá a
severidade: `pequeno`, `vizinho` e `borda` são BLOQUEIO; `pequeno-com-folga` é RESSALVA, e só
deixa de ser registrado quando o alvo existe exclusivamente sob `(pointer: fine)`.

### R3 — Zoom e reflow

**Rodar:** `browser_resize` não aplica zoom e não existe ferramenta aqui que aplique. Use a
equivalência: sobre uma janela de 1280px, zoom de N% deixa a mesma largura em CSS px que
redimensionar para `1280 ÷ (N / 100)`. Logo 200% → **640×1024** e 400% → **320×1024**. Role a
página inteira nos dois. Depois confira o meta viewport com `Grep` em `index.html`.

A equivalência não é perfeita — o zoom real também escala densidade de pixel — mas cobre o que a
WCAG 1.4.10 mede, que é largura em CSS px: a 320 CSS px o conteúdo precisa caber em uma coluna,
sem rolagem nos dois eixos ao mesmo tempo, e nada pode sumir.

**Reprova se:** aparece rolagem horizontal a 320px; algum conteúdo desaparece ou fica inacessível
ao ampliar; `user-scalable=no` ou `maximum-scale` menor que 5 está no meta viewport — é WCAG 1.4.4
e é bloqueio automático, sem discussão.

### R4 — Densidade de motion

**Rodar:** em 390×844 e em 1440×900, com a página rolada até o meio de cada capítulo.

```js
// 1) Nenhum pin-spacer pode existir no mobile; 2) a razão de scroll de cada capítulo.
(() => {
  const spacers = [...document.querySelectorAll('.pin-spacer')].map((s) => s.className)
  const capitulos = [...document.querySelectorAll('section.chapter')].map((c) => ({
    id: c.id,
    telas: +(c.getBoundingClientRect().height / innerHeight).toFixed(2),
  }))
  return { largura: innerWidth, spacers, capitulos }
})()
```

Rode nas duas larguras e divida `telas` do mobile pelo `telas` do desktop, capítulo a capítulo. A
faixa canônica é **0,68–0,75**, de
[estrutura-secoes](../estrutura-secoes/SKILL.md#pacing) — este portão confere e não renegocia. As razões atuais dos sete capítulos estão em
[sistema.md](sistema.md#densidade-de-motion).

**Observar, ainda:** amplitude de translate/parallax ≤40px abaixo de 768 (meça o `transform` no
meio da timeline); nenhum trilho horizontal movido por scroll y no touch; nenhum efeito de hover
ou magnético alcançável sem `(pointer: fine)`.

**Reprova se:** `spacers` não volta vazio em 390px; a razão de qualquer capítulo cai fora de
0,68–0,75 (RESSALVA); um parallax passa de 40px no mobile; o trilho horizontal continua preso ao
scroll y no touch.

---

## Laudo

Saída obrigatória, em `design/laudo-responsivo.md`. Sem ele não houve auditoria.

```
PARECER — audit-responsivo
Alvo: <rota ou componente>   Build: <branch/commit>
Perfis: 360×640 · 375×667 · 390×844 · 844×390 · 768×1024 · 1024×768 · 1440×900 ·
        reflow 640/320 · densidade de motion em 390 e 1440

BLOQUEIOS (n)
1. [R1] src/components/chapters/ChapterCare.tsx:88 — trilho vaza 42px à direita em
   375×667; overflow-x: clip esconde a barra e o layout segue empurrado. → limitar o
   trilho ao container e mover o excedente para wrap.

RESSALVAS (n)
1. [R4] ChapterClinic — razão scrollMobile/scroll em 0,77, acima da faixa 0,68–0,75.

NÃO AUDITÁVEL (n)
1. [R3] reflow a 320px não rodou: o dev server caiu no meio da sessão.

VEREDITO: REPROVADO
```

`VEREDITO: APROVADO` exige BLOQUEIOS **e** NÃO AUDITÁVEL vazios. Ressalvas podem constar e ainda
assim aprovar — é para isso que a lista existe.

## Checklist de aprovação

Uma linha por passo; o portão só fecha com as cinco preenchidas. "Não rodei" não é valor válido:
é NÃO AUDITÁVEL, e reprova.

| Passo | Precisa ser verdade | Prova |
|---|---|---|
| R0 | todo `clamp()` tocado pelo diff foi recalculado | valores conferidos contra [sistema.md](sistema.md) |
| R1 | sem overflow horizontal e sem texto de leitura abaixo de 16px; beats não se sobrepõem; nada atrás da barra de URL | os dois scripts vazios nas sete viewports |
| R2 | todo alvo ≥44×44 — ou ≥24×24 com 24px livres em volta; ≥8px entre alvos; ≥24px da borda; nenhuma ação só por hover ou long-press | script do R2 vazio em 390×844 |
| R3 | reflow a 320px sem rolagem nos dois eixos e sem conteúdo perdido; zoom não bloqueado | 640×1024 e 320×1024 rolados inteiros; meta viewport sem `user-scalable=no` e sem `maximum-scale` < 5 |
| R4 | nenhum pin-spacer abaixo de 768; razão `scrollMobile / scroll` entre 0,68 e 0,75; parallax ≤40px no mobile; trilho horizontal virou pilha ou snap nativo | script do R4 nas duas larguras |

## Anti-patterns

- **Breakpoint novo para resolver um caso** — cinco já existem e vinte arquivos assumem 768. Um
  sexto conserta uma tela e desalinha as outras dezenove.
- **`100vh` em qualquer lugar** — no Android a barra de URL come 60–110px e leva o CTA junto.
- **`min-width: 768px` para desligar hover** — um iPad tem 1024px e nenhum ponteiro.
- **Encolher a fonte para caber em 360** — abaixo de 16px o Safari iOS dá zoom ao focar um campo e
  desloca o layout. Quebre a linha, corte a palavra, reduza o padding: a fonte é o último recurso.
- **`display: none` no mobile para "resolver" o overflow** — o conteúdo existe no desktop, então
  importava; some justo para quem tem menos tela.
- **Rebaixar um bloqueio a ressalva para fechar o portão** — a severidade sai da tabela, não do
  prazo. Um bloqueio renomeado continua impedindo alguém de usar a página.
- **Aprovar um passo que não rodou** — servidor fora do ar devolve zero achados, e zero achados
  lidos como aprovação transformam um build quebrado em selo de qualidade. É NÃO AUDITÁVEL.
