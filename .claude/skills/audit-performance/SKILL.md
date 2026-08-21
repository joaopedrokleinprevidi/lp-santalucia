---
name: "audit-performance"
description: "Use when measuring page weight and Core Web Vitals on the production build before deploy. Portao 11c de performance, LCP 2.5s, CLS 0.1, INP 200ms, orcamento de peso em MB por Experience Score, 4G lento, caminho critico, lazy loading, IntersectionObserver, custo de frame sequence, AVIF ou WebP, og:image, fonte com display swap e subset, preload de fonte, Lighthouse local, throttle de CPU 4x, peso no celular."
argument-hint: "[rota] [--fix]"
allowed-tools: Read, Grep, Glob, Bash, Edit, Write, mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_evaluate, mcp__plugin_playwright_playwright__browser_network_requests, mcp__plugin_playwright_playwright__browser_console_messages
---

# Portão de performance

| | |
|---|---|
| **ENTRADA** | `design/laudo-acessibilidade.md` **aprovado**; `design/creative-direction.json` (`experienceScore` e `budget.mediaDesktopMB` / `mediaMobileMB` / `eagerMB`); os derivados pesáveis em `public/media/` e `public/frames/`, com as durações em `src/generated/media.ts`; build de produção rodando em `npm start` |
| **SAÍDA** | `design/laudo-performance.md` — BLOQUEIOS, RESSALVAS, NÃO AUDITÁVEL, VEREDITO |
| **ANTES** | `audit-acessibilidade` (Fase 11b) |
| **DEPOIS** | `publicar-lp` (Fase 12) |

Fase 11c de [landing-page-factory](../landing-page-factory/SKILL.md). É o último dos três portões
porque só o build final tem os números reais: correção de contraste troca imagem, correção de
reflow troca rendição, e as duas mudam o peso.

**Meça no `npm run build && npm start`, nunca no `npm run dev`.** O dev server serve ESM sem bundle nem
minificação; o número que ele dá não existe em produção e aprova uma página que reprova no ar.

- **BLOQUEIO** — a página não carrega a tempo em rede móvel, ou pula na cara de quem lê.
- **RESSALVA** — desperdício medido que não impede a leitura.
- **NÃO AUDITÁVEL** — build quebrado, Lighthouse com exceção, preview fora do ar. Reprova junto.

### Classificação — a severidade é lida, não julgada

| Passo | BLOQUEIO | RESSALVA |
|---|---|---|
| P1 | LCP > 2.5s; CLS > 0.1; INP > 500ms | INP entre 200 e 500ms |
| P2 | caminho crítico acima de 400 KB; peso total acima do orçamento do Experience Score | JS da rota dobrou entre dois builds sem feature nova |
| P3 | imagem sem `width`/`height`; `loading="lazy"` na imagem do LCP; imagem servida acima de `maxRenderWidth` | falta de AVIF onde só há WebP; `og-image` acima de 300 KB |
| P4 | vídeo ou frame sequence baixado a mais de 1,5 viewport de distância; segunda frame sequence adjacente | `saveData` não honrado |
| P5 | fonte sem `font-display: swap`; request para `fonts.gstatic.com` | peso ou subset carregado e não usado |

## Orçamento de peso

Peso **transferido** da primeira visita, tudo somado, com cache frio. O número que manda para o
projeto em mãos é `budget.mediaDesktopMB` / `mediaMobileMB` de `design/creative-direction.json`,
fixado por `creative-direction-expert` na Fase 5. A tabela abaixo é a referência quando o campo
não existe: as linhas ★★★★☆ e ★★★★★ são as de `prompt-animacao` (Fase 8b) e as três de baixo
escalam a partir delas.

| Experience Score | Desktop | Mobile | Frame sequences | Vídeo |
|---|---|---|---|---|
| ★☆☆☆☆ | 1,5 MB | 0,8 MB | 0 | nenhum |
| ★★☆☆☆ | 3 MB | 1,5 MB | 0 | nenhum |
| ★★★☆☆ | 5 MB | 2 MB | 0 | 1 loop curto |
| ★★★★☆ (padrão) | **8 MB** | **3,5 MB** | até 1 | até 3 capítulos |
| ★★★★★ | 12 MB | 5 MB | até 2, e distantes | orçado como produção |

**O orçamento total não é o que precisa caber em 2,5s.** O que precisa caber é o **caminho
crítico**. O throttling móvel do Lighthouse entrega 1,6 Mbps de descida, ou 200 KB por segundo;
descontando 150 ms de RTT por conexão e o tempo de parse, cabem cerca de **400 KB** antes dos 2,5s
de LCP. Os 3,5 MB restantes do orçamento mobile chegam depois, sob demanda — é para isso que
existem `loading="lazy"` e o `IntersectionObserver`. Um orçamento de 3,5 MB baixado inteiro no
primeiro segundo é a mesma coisa que não ter orçamento: 3,5 MB a 200 KB/s são 17,5 segundos.

| Métrica | Alvo | Reprova em | Causa mais comum aqui |
|---|---|---|---|
| LCP | ≤ 2,5s | > 2,5s | poster do hero sem preload, ou `<video>` promovido a LCP |
| CLS | ≤ 0,1 | > 0,1 | `<img>` sem `width`/`height`; fonte trocando com métrica diferente |
| INP | ≤ 200ms | > 500ms | trabalho de scroll na main thread; ScrollTrigger duplicado |

O navegador não considera o primeiro quadro de um `<video>` tão cedo quanto uma imagem
decodificada: trocar o poster do hero por vídeo piora o LCP mesmo servindo menos bytes.

---

## Protocolo

Seis passos, P0 a P5. Cite o código em cada achado (`[P3]`).

### P0 — Build e contexto

```bash
npm run build && npm start
```

Leia o `experienceScore` de `design/creative-direction.json` para fixar a linha do orçamento, e
guarde a saída do build: ela lista o tamanho de cada chunk. Registre-a no laudo — é a série
histórica que torna "o JS dobrou" um fato e não uma impressão.

**NÃO AUDITÁVEL se** o build falha. Não meça o dev server como substituto.

### P1 — Core Web Vitals

```bash
npx lighthouse http://localhost:4173 --only-categories=performance \
  --output=json --output-path=./design/lighthouse.json --quiet
```

O padrão do Lighthouse já é o perfil móvel: 150 ms RTT, 1,6 Mbps de descida, **CPU 4× mais
lenta**. Não desligue nenhum dos três — a página é rápida na sua máquina por construção; o teste
existe para o aparelho de quem chega pelo Instagram.

INP não sai de teste de laboratório, porque depende de interação real. Meça-o à mão: role a página
inteira com o throttle de CPU 4× ligado no DevTools e observe se o scrub acompanha o dedo. Um
`long task` acima de 200ms durante o scroll aparece no painel Performance como bloco vermelho.

**Reprova se:** LCP > 2,5s; CLS > 0,1; INP > 500ms; qualquer `long task` acima de 200ms no scroll.

### P2 — Caminho crítico e peso total

**Rodar:** `browser_navigate` no preview, depois `browser_network_requests`.

```js
// Peso por tipo e o que entrou antes do LCP. `transferSize` é o que passou no fio;
// `encodedBodySize` ignora o ganho de compressão e infla o número.
(() => {
  const lcp = performance.getEntriesByType('largest-contentful-paint').at(-1)?.startTime ?? Infinity
  const r = performance.getEntriesByType('resource')
  const kb = (n) => Math.round(n / 1024)
  const soma = (list) => kb(list.reduce((t, e) => t + (e.transferSize || 0), 0))
  return {
    criticoKB: soma(r.filter((e) => e.startTime < lcp)),
    totalKB: soma(r),
    porTipo: Object.fromEntries(
      [...new Set(r.map((e) => e.initiatorType))].map((t) => [t, soma(r.filter((e) => e.initiatorType === t))]),
    ),
    lcpMs: Math.round(lcp),
  }
})()
```

`budget.eagerMB` da Fase 5 é a mídia que carrega **sem gate**; ela entra inteira no caminho
crítico. Se `eagerMB` sozinho já passa de 0,4 MB, o LCP não fecha em 2,5s a 200 KB/s, e o achado
volta para a direção criativa cortar mídia eager — não para o build espremer bytes.

**Reprova se:** `criticoKB` passa de 400; `totalKB` passa da linha do Experience Score; algum
recurso entrou antes do LCP sem ser necessário para pintá-lo (fonte de capítulo, vídeo, ícone de
rodapé).

### P3 — Imagens

`AVIF` e `WebP` não são alternativas de gosto. A regra:

| Formato | Quando | Porque |
|---|---|---|
| **AVIF** | primeira fonte de toda fotografia | 30–50% menor que WebP na mesma qualidade percebida em foto |
| **WebP** | segunda fonte, sempre | Safari só ganhou AVIF em 16.4; WebP roda em tudo desde 2020 |
| **JPEG** | fallback do `<img>` **e** o `og:image` | WhatsApp e Facebook não renderizam prévia de AVIF/WebP de forma confiável |
| **PNG** | só transparência com poucas cores | em fotografia é 5–10× o peso do AVIF |
| **SVG** | logo, ícone, filete | vetor não tem rendição por largura, e o arquivo costuma ficar abaixo de 2 KB |

AVIF suaviza grão fino em qualidade baixa: numa foto com textura de pele ou película, compare o
recorte a 100% antes de aceitar o ganho de bytes.

Ordem no `<picture>`: AVIF, depois WebP, depois o `<img>` com JPEG. O navegador para na primeira
que entende — inverter a ordem entrega WebP a quem podia receber AVIF.

**Reprova se:** alguma `<img>` está sem `width`/`height` (é CLS garantido); a imagem do LCP tem
`loading="lazy"` (o lazy atrasa a descoberta e o LCP piora de forma medível) em vez de
`fetchpriority="high"`; alguma imagem abaixo da dobra está **sem** `loading="lazy"` e
`decoding="async"`; a largura servida passa do `maxRenderWidth` que o inventário registrou;
`public/media/og-image.jpg` passa de 300 KB — acima disso o WhatsApp não renderiza a prévia e o
link chega sem imagem, no principal canal de chegada deste público.

### P4 — Vídeo e frame sequence

Toda mídia pesada carrega por gate, não por existir no DOM.

```tsx
// rootMargin de 150% = uma viewport e meia de antecedência. Menos que isso e o
// vídeo chega depois da seção; mais e você paga por quem nunca rolou até lá.
useEffect(() => {
  const el = ref.current
  if (!el) return
  const io = new IntersectionObserver(
    ([entry]) => { if (entry.isIntersecting) { setShouldLoad(true); io.disconnect() } },
    { rootMargin: '150% 0px' },
  )
  io.observe(el)
  return () => io.disconnect()
}, [])
```

**O custo de uma frame sequence:** 300 frames × 8–25 KB em AVIF são **2,4 a 7,5 MB**, e eles
chegam todos ou a animação engasga no meio. Uma sequência consome quase o orçamento mobile inteiro
do ★★★★☆ sozinha — é por isso que o teto é **uma** por página, e duas só num ★★★★★ e em capítulos
distantes. Além dos bytes há memória: 300 bitmaps decodificados em largura total derrubam a aba no
iOS.

**Reprova se:** algum `<video>` ou set de frames é baixado a mais de 1,5 viewport de distância;
existe uma segunda frame sequence adjacente à primeira; o vídeo é baixado com
`navigator.connection.saveData` ligado (RESSALVA — a API é só do Chromium, então é melhoria
progressiva). WebViews, `saveData` e o que muda em rede móvel real:
[rede-e-webviews.md](rede-e-webviews.md).

### P5 — Fontes

Fontes vêm de `@fontsource-variable`, **auto-hospedadas**. Um request para `fonts.gstatic.com`
adiciona DNS, TLS e uma conexão nova no caminho crítico, e a família ainda chega depois.

- `font-display: swap` em toda `@font-face`. Sem ele o texto fica invisível até a fonte chegar
  (o "flash of invisible text"), e o LCP espera o download inteiro.
- **Subset `latin` + `latin-ext`.** pt-BR precisa de `ã õ ç á é í ó ú â ê ô à`. Cada subset extra é
  um arquivo baixado; `cyrillic`, `greek` e `vietnamese` nunca são usados aqui.
- **Variável em vez de estática.** Um arquivo variável cobre 300–700 no peso de pouco mais que uma
  estática; quatro estáticas são quatro downloads.
- **Preload só da face do LCP**, com `crossorigin` mesmo em mesma origem — sem ele o navegador
  baixa a fonte duas vezes, porque a requisição do preload e a do CSS entram em caches diferentes:

```html
<link rel="preload" as="font" type="font/woff2" crossorigin
      href="/assets/fonts/hero-latin-variable.woff2">
```

**Reprova se:** falta `font-display: swap`; existe request para `fonts.gstatic.com` ou
`fonts.googleapis.com`; um peso ou subset é baixado e nenhum glifo dele é pintado; o preload está
sem `crossorigin`.

---

## Laudo

Saída obrigatória, em `design/laudo-performance.md`.

```
PARECER — audit-performance
Alvo: <rota>   Build: <branch/commit>   Medido em: npm run build && npm start
Perfil: Lighthouse móvel padrão — 150 ms RTT, 1,6 Mbps, CPU 4×
Orçamento: ★★★★☆ → 8 MB desktop / 3,5 MB mobile

BLOQUEIOS (n)
1. [P3] src/components/media/Picture.tsx:44 — poster do hero com loading="lazy".
   LCP em 3,1s. → fetchpriority="high" e preload no <head>; sem lazy no LCP.

RESSALVAS (n)
1. [P4] ChapterFilm baixa o clipe com saveData ligado. 1,2 MB cobrados de quem
   pediu economia.

NÃO AUDITÁVEL (n)
1. [P1] Lighthouse não rodou: `npm run build` saiu com erro. LCP e CLS
   não foram medidos — ausência de achado aqui não é ausência de defeito.

VEREDITO: REPROVADO
```

## Checklist de aprovação

| Passo | Precisa ser verdade | Prova |
|---|---|---|
| P0 | `npm run build` passa e o `npm start` está no ar; tamanho de cada chunk registrado | saída do build colada no laudo |
| P1 | LCP ≤ 2,5s, CLS ≤ 0,1, nenhum long task acima de 200ms no scroll | `design/lighthouse.json` mais a passada manual com CPU 4× |
| P2 | caminho crítico ≤ 400 KB; total dentro da linha do Experience Score | script do P2 |
| P3 | toda `<img>` com `width`/`height`; LCP com `fetchpriority="high"` e sem lazy; resto com `loading="lazy"`; AVIF antes de WebP; `og-image` ≤ 300 KB | `du -h public/media/og-image.jpg` e o `<picture>` conferido |
| P4 | nada pesado baixado a mais de 1,5 viewport; no máximo uma frame sequence | `browser_network_requests` com a página no topo |
| P5 | `font-display: swap`; zero request para domínio de fonte externo; preload com `crossorigin` | aba Network filtrada por `font` |

## Anti-patterns

- **Medir no `npm run dev`** — sem bundle e sem minificação o número é ficção, e ele sempre
  favorece a aprovação.
- **`loading="lazy"` na imagem do LCP** — o lazy adia a descoberta do recurso justamente do
  elemento cuja pintura a métrica mede. Piora o LCP de forma medível.
- **`<video>` como elemento de LCP** — o navegador considera o primeiro quadro tarde. Poster
  primeiro, vídeo depois do gate.
- **Frame sequence em toda seção** — a segunda transforma a landing num download, e quem está no 4G
  fecha antes do primeiro frame.
- **Preload de fonte sem `crossorigin`** — a fonte é baixada duas vezes e o preload vira custo.
- **Desligar o throttle "porque a minha internet é boa"** — a sua internet não é a de ninguém que
  chega por link de WhatsApp em rede móvel.
- **Comprimir imagem até o artefato aparecer para caber no orçamento** — se não cabe, a seção tem
  mídia demais. Corte uma imagem inteira em vez de degradar cinco.
- **Rebaixar um bloqueio a ressalva para fechar o portão** — a severidade sai da tabela de
  classificação, não do prazo.
- **Aprovar um passo que não rodou** — build quebrado devolve zero achados, e zero achados lidos
  como aprovação transformam um build quebrado em selo de qualidade. É NÃO AUDITÁVEL.
