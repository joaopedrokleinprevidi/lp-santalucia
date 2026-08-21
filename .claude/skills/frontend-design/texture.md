# Textura e atmosfera — receitas estendidas

As quatro receitas base (grain, malha, grade de pontos, sombra) estão na
[seção 5 do SKILL.md](SKILL.md). Aqui ficam as demais, com custo e limite.

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
