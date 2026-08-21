# Textura e atmosfera — receitas

O código de toda camada de textura. Quando cada uma entra, o limite numérico de cada uma e os
três portões de contraste estão na [seção 5 do SKILL.md](SKILL.md) — aqui fica a implementação.

## Tabela de custo

| Técnica | Custo | Quando o custo aparece |
|---|---|---|
| Grain SVG estático | rasteriza 1 tile, depois 1 layer composta | nunca durante o scroll |
| Gradiente de malha | paint puro | só se os stops ou `background-position` mudarem |
| Grade de pontos / filetes | paint puro | no resize, se a largura for fracionária |
| `box-shadow` blur ≤ 40px | paint no elemento | a cada frame, se o elemento anima algo que não é `transform` |
| Vinheta / scrim radial | paint puro, 1 layer | nunca |
| `mix-blend-mode` em camada de viewport | compositor relê o backdrop | a cada frame em que o backdrop muda (vídeo = sempre) |
| `backdrop-filter` | compositor relê e desfoca o backdrop | a cada frame em que o backdrop muda |
| `filter: blur()` em elemento grande | repaint + blur por frame | sempre que o elemento ou o pai anima |
| Grain animado (flicker) | repaint do tile por frame | sempre — só fora de reduced-motion |
| Ruído em `<canvas>` | 1 draw por resize | no resize; nunca no scroll se cacheado |

Regra de orçamento: **no máximo um `backdrop-filter` visível por vez, e nunca em elemento de
viewport inteira.** Medição: DevTools → Rendering → Frame Rendering Stats, rolando a página
inteira. Abaixo de 55fps sustentado, remova a camada mais recente e meça de novo.

## Grain — o mais barato e o mais eficaz

```css
.grain::after {
  content: '';
  position: fixed;
  inset: 0;
  z-index: 60;
  pointer-events: none;
  opacity: 0.045;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='160' height='160' filter='url(%23n)'/%3E%3C/svg%3E");
}
```

O `feTurbulence` é rasterizado **uma vez** num tile de 160×160; depois é uma camada composta e
nada mais. `position: fixed` sobre um capítulo com vídeo deixa o grão parado enquanto a cena se
move — é isso que lê como película.

Tile ≥128px: abaixo disso o padrão se repete visivelmente. `opacity` acima de 0.06 lê como sujeira
na fotografia, não como textura, e o corpo abaixo dela perde contraste mensurável.

## Gradiente de malha

```css
.mesh {
  background-color: var(--color-canvas-soft);
  background-image:
    radial-gradient(60% 45% at 12% 18%, color-mix(in oklab, var(--color-rose) 26%, transparent) 0%, transparent 70%),
    radial-gradient(50% 40% at 88% 8%, color-mix(in oklab, var(--color-gilt) 22%, transparent) 0%, transparent 65%),
    radial-gradient(70% 60% at 50% 100%, color-mix(in oklab, var(--color-ink) 18%, transparent) 0%, transparent 72%);
}
```

Um mesh varia o fundo ao longo da seção, então o texto por cima tem contraste variável. Meça no
blob mais escuro **e** no ponto mais claro; se um dos dois reprova, baixe as porcentagens do
`color-mix` até os dois passarem, em vez de escurecer o texto.

## Grade de pontos e filetes

```css
.dot-grid {
  background-image: radial-gradient(
    circle at 1px 1px,
    color-mix(in oklab, var(--color-ink) 14%, transparent) 1px,
    transparent 0
  );
  background-size: 24px 24px;
}
```

Alpha ≤0.16 — acima disso a grade compete com o texto em vez de dar chão. Para régua de colunas,
desenhe com `border-inline-start` em elementos reais, não com `background-size: calc(100% / 12)`:
em largura não-inteira as linhas caem em pixel fracionário e tremem no resize.

## Sombra com intenção

```css
--shadow-card: 0 1px 2px rgb(50 21 30 / 0.04), 0 12px 32px -12px rgb(50 21 30 / 0.16);
```

- **Contato**: blur 1–2px, alpha 0.04–0.08. É o que faz o objeto tocar a superfície.
- **Ambiente**: blur 24–48px, spread negativo, alpha 0.12–0.24. É o que dá altura.
- **Tinja com a tinta da marca** (`rgb(50 21 30)` = `--color-ink`), nunca preto puro. Preto sobre
  fundo quente lê como sujeira.

Uma sombra estática é paint pago uma vez. Se o elemento anima **só** `transform`, a camada inteira
— sombra junto — é composta e nada repinta, por mais alto que seja o blur. O que custa é animar a
própria sombra: aí o retângulo é repintado a cada frame, e com blur >40px isso derruba o alvo de
60fps. Para "levantar" no hover, cruze duas camadas de sombra com `opacity` em vez de interpolar
o blur.

## Vinheta e scrim sobre fotografia

O jeito de deixar texto legível sobre imagem sem inventar um card. Já existe no projeto em
`src/components/ui/Scrim.tsx`:

```css
background-image: radial-gradient(
  120% 90% at 50% 50%,
  rgb(50 21 30 / 0.5) 0%,
  rgb(50 21 30 / 0.78) 62%,
  rgb(50 21 30 / 0.92) 100%
);
```

- Tinja com a tinta da marca, nunca preto puro — preto sobre fotografia quente cinza a pele.
- Três stops no mínimo. Dois produzem uma borda visível onde o gradiente termina.
- Para copy lateral (não centralizada), troque o radial por um linear no eixo do texto — com os
  mesmos quatro stops, é a variante `left` do `Scrim`:
  `linear-gradient(100deg, rgb(50 21 30 / 0.93) 0%, rgb(50 21 30 / 0.78) 30%, rgb(50 21 30 / 0.32) 62%, rgb(50 21 30 / 0.06) 100%)`.
  Um linear de dois stops (`0.85` → `transparent`) é o erro comum: a borda onde ele termina fica
  visível sobre fotografia clara.
- **Meça o contraste no ponto mais claro do scrim**, não na média. 4.5:1 para corpo, 3:1 para
  texto ≥ 24px bold.

## Duotone em fotografia

Amarra fotos de origens diferentes na paleta da marca sem reeditá-las.

```css
.duotone {
  position: relative;
  filter: grayscale(1) contrast(1.1);
}
.duotone::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    to bottom,
    color-mix(in oklab, var(--color-ink) 85%, transparent),
    color-mix(in oklab, var(--color-rose) 55%, transparent)
  );
  mix-blend-mode: color;
  pointer-events: none;
}
```

Custo: `filter` + `mix-blend-mode` num elemento estático é pago uma vez. Sobre **vídeo em
reprodução** é por frame — nesse caso aplique o duotone no encode (`ffmpeg`, no
`scripts/prepare-assets.mjs`) em vez de no navegador.

## Filete metálico (foil)

Substitui a borda cinza de 1px por algo que pertence à marca. Já existe como `.rule-gilt`:

```css
.rule-gilt {
  background-image: linear-gradient(
    to right,
    color-mix(in srgb, var(--color-gilt) 60%, transparent),
    color-mix(in srgb, var(--color-gilt) 12%, transparent)
  );
}
```

Aplicado num elemento de `height: 1px`. O degradê ao longo do filete é o que o faz ler como
foil e não como `border`. Para foil em volta de um card, use `border: 1px solid transparent` com
`background-origin: border-box` e dois `background-image` empilhados — mas só se o card já
existir por outro motivo; foil não justifica criar contêiner.

## Grain animado (tremor de película)

```css
@keyframes film {
  0%   { transform: translate3d(0, 0, 0); }
  25%  { transform: translate3d(-2%, 1%, 0); }
  50%  { transform: translate3d(1%, -2%, 0); }
  75%  { transform: translate3d(-1%, -1%, 0); }
  100% { transform: translate3d(0, 0, 0); }
}

.grain::after {
  /* tile 12% maior que a viewport para o deslocamento nunca revelar borda */
  inset: -6%;
  animation: film 0.6s steps(4) infinite;
}
```

Anime **`transform`**, nunca `background-position`: transform é composto, background-position
repinta o tile. `steps(4)` em vez de interpolação contínua — grão que desliza suavemente lê como
erro; grão que salta lê como película.

Obrigatório desligar:

```css
@media (prefers-reduced-motion: reduce) {
  .grain::after { animation: none; }
}
```

O bloco global de reduced-motion em `src/styles/index.css` já zera `animation-duration` para
tudo — confirme que a nova camada está coberta por ele antes de duplicar a regra.

## Transparências em camada

Profundidade sem sombra e sem blur. Três camadas do mesmo tom com alphas diferentes, deslocadas
uma em relação à outra.

O atalho errado é `background: <cor>, <cor>` — o shorthand só aceita cor na **última** camada,
então a declaração inteira é inválida e o navegador a descarta em silêncio. Empilhe gradientes,
que são imagens e podem ocupar qualquer camada:

```css
.stack {
  background-color: var(--color-canvas);
  background-image:
    linear-gradient(color-mix(in oklab, var(--color-ink) 4%, transparent) 0 100%),
    linear-gradient(color-mix(in oklab, var(--color-ink) 8%, transparent) 0 100%);
  background-size: 100% 100%, 96% 100%;
  background-position: 0 0, 2% 6px;
  background-repeat: no-repeat;
}
```

Custo zero — é tudo paint estático. É a alternativa correta ao glassmorphism quando não há nada
atrás para desfocar, e quase nunca há.

## Recorte irregular

`clip-path` quebra o retângulo sem imagem extra:

```css
.bleed {
  clip-path: polygon(0 0, 100% 4%, 100% 100%, 0 96%);
}
```

Custo: composto, gratuito enquanto estático. Animar `clip-path` entre polígonos com o **mesmo
número de vértices** é interpolável e barato; entre `circle()` e `polygon()` não é — o navegador
não interpola e o corte pula. É por isso que o handoff do hero em
[choreography.md](../video-to-website/choreography.md) usa `circle()` ou `inset()` do começo ao
fim, nunca uma mistura.

## Ruído em canvas

Só quando o grain SVG não basta — ruído colorido, ou densidade que precisa variar por seção.

```ts
function paintNoise(canvas: HTMLCanvasElement, alpha = 12): void {
  const ctx = canvas.getContext('2d')
  if (!ctx) return
  const { width, height } = canvas
  const image = ctx.createImageData(width, height)
  for (let i = 0; i < image.data.length; i += 4) {
    const v = (Math.random() * 255) | 0
    image.data[i] = image.data[i + 1] = image.data[i + 2] = v
    image.data[i + 3] = alpha
  }
  ctx.putImageData(image, 0, 0)
}
```

Pinte num canvas de **256×256** e repita com `background-image` a partir de `toDataURL()`, ou
deixe o canvas em `position: fixed` com `image-rendering: pixelated` esticado. Pintar um canvas
de viewport inteira custa `width × height` iterações — em 2560×1440 são 3,7 milhões de pixels e
o frame é perdido. 256×256 são 65 mil.

Repinte só no `resize`, nunca no scroll.

## Derivar tom da marca sem inventar cor

Quando a marca fixou os hexes mas não o tom, derive a partir do que já existe. Entre **dois hues
saturados**, `color-mix`/gradiente em `oklab` mantém a croma no meio do caminho; em `srgb` o
midpoint dessatura — por isso a interpolação de marca é sempre `in oklab`. Misturar uma cor com
`transparent` é o caso barato: os dois espaços entregam quase a mesma rampa, e é por isso que
`.rule-gilt` usa `in srgb` sem prejuízo.

```css
/* tint da marca — não é uma cor nova, é a mesma cor com menos presença */
background-color: color-mix(in oklab, var(--color-rose) 12%, transparent);
border-color: color-mix(in oklab, var(--color-ink) 18%, transparent);
```

E o inverso do erro clássico: **nunca crie `linear-gradient(180deg, var(--brand), white)`.**
Interpolar uma cor saturada até o branco em sRGB atravessa uma faixa lavada — que é exatamente a
cor do slop. Se precisar de degradê da marca, interpole `in oklab` e pare em 60%, sem chegar ao
branco.

## Medir a proporção de área

A dominante precisa de ≥60% da área do viewport e o accent de <15%. Cole no console com a página
aberta, no capítulo que quiser medir:

```js
const area = new Map()
for (const el of document.querySelectorAll('*')) {
  const r = el.getBoundingClientRect()
  if (r.width < 4 || r.height < 4) continue
  const bg = getComputedStyle(el).backgroundColor
  if (bg === 'rgba(0, 0, 0, 0)') continue
  area.set(bg, (area.get(bg) ?? 0) + r.width * r.height)
}
console.table([...area].sort((a, b) => b[1] - a[1]).slice(0, 6))
```

Se as três primeiras linhas estão empatadas, não há direção — a página está no `50 / 30 / 20` que
lê como template.

## Cursor custom

O cursor custom é do [motion-ui-expert](../motion-ui-expert/patterns.md) — a implementação
completa está lá, com o escopo por `[data-cursor]` e o `cursor: revert` que devolve o cursor do
sistema dentro de link, botão e campo. Não reimplemente aqui.

O que esta skill decide é só se ele **entra**: um cursor custom é textura, e vale a mesma regra
do resto da seção — se a página já carrega grain e mesh, ele é a terceira camada e não entra.

## Onde a textura não entra

- Atrás de texto de corpo. Grain sobre parágrafo derruba contraste e cansa.
- Em campo de formulário. Fundo texturizado num `input` lê como erro de renderização.
- Em mais de duas camadas simultâneas. Grain + malha + grade ao mesmo tempo vira ruído, não
  atmosfera. Escolha duas por página, no máximo.
