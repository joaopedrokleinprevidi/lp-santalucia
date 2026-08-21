---
name: product-design-expert
description: Use when someone asks to design, redesign, audit or fix the visual layer of an interface — type scale, spacing, grid, layout, visual hierarchy, colour palette, contrast ratio, design tokens or section composition. Define escala tipográfica, tamanho de fonte, espaçamento, margem, grade, layout, hierarquia de leitura, paleta, contraste, tokens e composição de seção. Também quando alguém diz que a página está desalinhada, apertada, sem hierarquia, com tudo do mesmo tamanho ou "todas as seções iguais".
argument-hint: [secao-ou-arquivo] [o-que-corrigir]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

# Product Design — sistema visual

Esta skill define os números que o resto do time consome: a escala tipográfica, a grade, o
espaçamento, a paleta e o padrão de composição de cada seção. Nada aqui é opinião — cada valor
é verificável no DevTools.

**Possui:** escala de tipo, escala de espaço, grade, níveis de leitura, paleta e contraste,
padrão de composição por seção, tokens em `@theme`.
**Não possui:** o caráter estético e a escolha das famílias (`frontend-design` decide *quais*
fontes; esta skill decide os *degraus* e o pareamento), ordem das seções (storytelling), curvas e
durações (motion), breakpoints de comportamento (responsive). Entrega os tokens; os outros
especialistas os consomem.

## Ordem das decisões

Nesta ordem, sempre. Cada passo consome o anterior; inverter produz retrabalho.

1. **Escala tipográfica** — os degraus e o pareamento de famílias.
2. **Escala de espaço + grade** — a base 4px, as colunas, a medida de leitura em `ch`.
3. **Paleta** — a rampa da cor de marca, os neutros e os contrastes medidos.
4. **Hierarquia** — três níveis por seção, com tamanho/peso/cor apenas.
5. **Composição** — o padrão de cada seção e a regra de não-repetição.

Só então escreva markup. Os tokens completos ficam em [tokens.md](tokens.md); o código TSX de
cada padrão de seção, em [composition.md](composition.md).

## 1. Escala tipográfica

> **Antes de escrever qualquer degrau, leia o `@theme` que já existe.** Uma escala em produção
> foi ajustada contra a headline real mais longa de cada papel. Substituí-la por esta tabela
> reflui todas as manchetes do site e invalida a tabela de valores resolvidos de
> [responsive-e-acessibility](../responsive-e-acessibility/SKILL.md). A regra é **estender, nunca
> sobrescrever**: adicione o degrau que falta, mantenha os que já servem.
>
> O que este repositório já publica em `src/styles/index.css` — cinco degraus, todos menores que
> a referência abaixo:
>
> | Token | Valor em produção | @360 | @1440 | teto |
> |---|---|---|---|---|
> | `--text-hero` | `clamp(2.125rem, 4.5vw, 4.5rem)` | 34px | 64.8px | 72px a 1600 |
> | `--text-chapter` | `clamp(1.875rem, 3.5vw, 3.25rem)` | 30px | 50.4px | 52px |
> | `--text-statement` | `clamp(1.625rem, 3.4vw, 3.25rem)` | 26px | 49px | 52px |
> | `--text-lead` | `clamp(1rem, 1.15vw, 1.25rem)` | 16px | 16.6px | 20px a 1739 |
> | `--text-eyebrow` | `0.6875rem` | 11px | 11px | — |
>
> O piso de 16px em `--text-lead` é deliberado e não sobe: abaixo dele o Safari do iOS dá zoom ao
> focar um campo. Faltam `display`, `metric`, `title`, `body`, `body-sm`, `label` e `caption` — é
> aí que a tabela de referência entra.

A referência completa: onze degraus, do display ao caption, para uma página que ainda não tem
escala. Todo degrau fluido é `clamp()` com mínimo em 360px e máximo atingido em 1440px. A
fórmula do termo do meio:

```
slope = (maxPx - minPx) / (maxVw - minVw)
preferido = ((minPx - slope * minVw) / 16)rem + (slope * 100)vw
```

| Token | 360px | 1440px | `clamp()` | line-height | letter-spacing | Papel |
|---|---|---|---|---|---|---|
| `--text-display` | 48 | 128 | `clamp(3rem, 1.333rem + 7.41vw, 8rem)` | 1.02 | -0.03em | marquee, statement único da página |
| `--text-hero` | 40 | 88 | `clamp(2.5rem, 1.5rem + 4.44vw, 5.5rem)` | 1.04 | -0.02em | h1, uma vez |
| `--text-chapter` | 32 | 56 | `clamp(2rem, 1.5rem + 2.22vw, 3.5rem)` | 1.06 | -0.015em | h2 de seção |
| `--text-statement` | 28 | 44 | `clamp(1.75rem, 1.417rem + 1.48vw, 2.75rem)` | 1.18 | -0.01em | frase de fecho, citação |
| `--text-metric` | 40 | 72 | `clamp(2.5rem, 1.833rem + 2.96vw, 4.5rem)` | 0.95 | -0.02em | números que contam (tabular) |
| `--text-title` | 20 | 24 | `clamp(1.25rem, 1.167rem + 0.37vw, 1.5rem)` | 1.30 | -0.005em | h3, título de item |
| `--text-lead` | 17 | 22 | `clamp(1.0625rem, 0.958rem + 0.46vw, 1.375rem)` | 1.55 | 0 | parágrafo de abertura |
| `--text-body` | 16 | 18 | `clamp(1rem, 0.958rem + 0.19vw, 1.125rem)` | 1.65 | 0 | corpo |
| `--text-body-sm` | 14 | 14 | `0.875rem` | 1.60 | 0 | descrição de item, nota |
| `--text-label` | 12 | 12 | `0.75rem` | 1 | +0.14em | tag, caixa alta |
| `--text-eyebrow` | 11 | 11 | `0.6875rem` | 1 | +0.24em | abertura de capítulo, caixa alta |
| `--text-caption` | 13 | 13 | `0.8125rem` | 1.45 | +0.01em | legenda, crédito, rodapé |

**Bandas de line-height** (para qualquer degrau novo): ≥56px → 0.92–1.0 · 28–56px → 1.04–1.18 ·
22–28px → 1.2–1.3 · 13–22px → 1.45–1.65 · qualquer tamanho em caixa alta de rótulo → 1.

**Piso de 1.02 sobrepõe a banda quando o display carrega acento.** Em pt-BR o til de `Ã` e a
cedilha de `Ç` sobem acima da altura de capitular, e a 0.92 a linha de cima os corta. Por isso os
dois degraus acima ficam em 1.02 e 1.04 em vez do fundo da banda, e é o que `--text-hero` já
publica neste repositório. A causa completa e o teste visual são de
[frontend-design](../frontend-design/type-pairs.md#3-acento-clipado-por-line-height); o valor é
daqui.

**Bandas de letter-spacing:** ≥48px → -0.03 a -0.02em · 28–48px → -0.02 a -0.01em · 16–28px →
-0.005 a 0 · <14px caixa alta → +0.12 a +0.24em (quanto menor, mais tracking).
Causa: o tracking padrão da fonte foi desenhado para 16px. A 64px ele abre buracos entre letras;
a 11px em caixa alta as hastes das maiúsculas colidem.

### Pareamento display + corpo

- **Duas famílias, no máximo.** Uma terceira só se for mono, e só para números tabulares.
- Display serif de alto contraste + corpo sans (o projeto: Cormorant Garamond + Inter Variable).
- A display só desce até `--text-title` (20px). Abaixo disso um serif de alto contraste perde as
  hastes finas em telas 1x — Cormorant a 15px some.
- Se as duas famílias forem sans, exija contraste estrutural: eixos diferentes (grotesk vs.
  geométrica) **ou** diferença de peso ≥300 (400 vs. 700). Sem isso parece erro de importação.
- **Três pesos por família, no máximo.** A 16px ninguém distingue 500 de 600 — o quarto peso não
  cria nível, só entra na página.
- `font-synthesis-weight: none` no `body`. Falso negrito engorda a haste sem redesenhar a letra.
- Contadores GSAP levam `font-variant-numeric: tabular-nums`; sem isso a largura dos dígitos
  muda a cada frame do count-up e o bloco inteiro treme.

**Regra de densidade:** no máximo 4 degraus visíveis na mesma seção — três níveis de leitura mais
um caption. Um quinto degrau significa que a seção tem duas ideias.

## 2. Escala de espaço

Base **4px**. É o padrão do Tailwind 4 (`--spacing: 0.25rem`), então toda utilidade numérica já
cai na escala. Degraus permitidos: 4, 8, 12, 16, 24, 32, 48, 64, 96, 128, 160, 192, 256.
Nunca um valor entre degraus — `mt-[27px]` é a assinatura de layout ajustado no olho.

### Ritmo vertical por relação

| Relação | Desktop | Mobile |
|---|---|---|
| eyebrow → título | 16–24px | 12–16px |
| título → lead | 24–32px | 20–24px |
| lead → CTA | 32–40px | 28–32px |
| item → item (grid) | 24–32px x, 24–40px y | 20–24px |
| bloco → bloco dentro da seção | 64–96px | 48–56px |
| seção → seção | 128–192px | 80–96px |

**Lei de proximidade, em número:** o espaço entre grupos precisa ser ≥1.6× o maior espaço interno
do grupo. Abaixo disso os dois grupos leem como um só, e é aí que alguém propõe um card para
"separar" — o card é uma solução de 1px para um problema de 32px.

### Padding de seção por breakpoint

O `padding-inline` não é uma tabela de degraus: é `clamp(24px, 5vw, 64px)`, já implementado em
`.container-editorial` (`src/styles/index.css`) junto com `max-width: 1280px`. Use a classe; não
recrie o container por seção e não invente uma margem de 80px que ela não produz.

| Viewport | padding-inline resolvido | padding-block |
|---|---|---|
| <640 | 24px (o piso do clamp) | 72–96px |
| 640–1023 | 32–51px | 96–120px |
| 1024–1279 | 51–64px | 120–160px |
| ≥1280 | 64px (o teto do clamp) | 160–192px |

**Dentro de um palco de 100svh** (capítulos com `.chapter__stage`) o padding é vertical e
relativo ao viewport, não em px: `pt-[12svh]` para limpar a barra de navegação (≥72px) e
`pb-[12svh]` para o cue de scroll. Use `svh`, **não `vh`** — `vh` resolve contra o viewport
*grande*, então dentro de um palco de `100svh` ele produz um padding maior que os 12% pedidos
exatamente no celular com a barra de URL visível, que é o caso que o padding existia para
resolver. Texto vive no `svh`; a mídia (`.chapter__media`) preenche o `lvh` para que a barra
retraindo nunca revele uma faixa de fundo.

### Medida de leitura

| Papel | Medida |
|---|---|
| corpo longo | 60–75ch (ótimo 66) |
| lead / intro | 45–55ch |
| headline display | 12–20ch, nunca >24ch |
| coluna de grid de 3–4 itens | 30–40ch |
| caption / label | 30–40ch |
| mobile a 360px | 35–45ch (é o que sobra com 24px de padding) |

Use `ch`, não px: `max-w-[65ch]`. A unidade acompanha a fonte; um `max-width` em px não. Com Inter
a 16px, 65ch ≈ 590px — o mesmo bloco em `max-w-[720px]` chega a ~79ch e o olho começa a errar o
retorno de linha. Acima de 75ch a taxa de erro sobe; abaixo de 45ch o texto quebra a cada três
palavras.

Duas ressalvas antes de confiar no número: `1ch` é o avanço do glifo `0`, não a largura média de
caractere (≈0.57em na Inter, ≈0.5em em grotescas mais estreitas), e ele resolve contra a fonte **do
próprio elemento** — o mesmo `65ch` num título em `font-display` dá outra largura. Meça a família
antes de fixar o teto:

```js
// 1ch em em, para a fonte que o elemento realmente usa.
const el = document.querySelector('p')
const fs = parseFloat(getComputedStyle(el).fontSize)
const probe = Object.assign(document.createElement('span'), { textContent: '0'.repeat(100) })
probe.style.cssText = 'position:absolute;visibility:hidden;white-space:pre;font:inherit'
el.append(probe)
console.log((probe.getBoundingClientRect().width / 100 / fs).toFixed(3) + 'em por ch')
probe.remove()
```

## 3. Grade

Colunas e gutter são decisão desta skill. A margem não é — ela vem do `clamp()` da
`.container-editorial` e está aqui só como valor resolvido, para conferência.

| Viewport | Colunas | Gutter | Margem (resolvida) | Conteúdo máx. |
|---|---|---|---|---|
| 0–639 | 4 | 16px | 24px | viewport − 48px |
| 640–1023 | 8 | 24px | 32–51px | viewport − 2× margem |
| 1024–1279 | 12 | 24px | 51–64px | viewport − 2× margem |
| ≥1280 | 12 | 32px | 64px | 1280px |

```tsx
<div className="grid grid-cols-4 gap-x-4 sm:grid-cols-8 sm:gap-x-6 lg:grid-cols-12 xl:gap-x-8">
```

**Proporções de split:** 5/7 ou 7/5 de 12. Nunca 6/6 — com metades exatas nada domina, o olho
escolhe a esquerda por hábito e a intenção da seção se perde. Texto editorial ocupa 5 ou 6 de 12.

### Quebrar a grade de propósito

Três casos legítimos. Fora deles, quebra é erro.

1. **Sangria de 1–2 colunas** além da margem, em **um** elemento por seção. Dois elementos
   sangrando na mesma seção e a grade deixa de existir como referência — aí a quebra parece bug.
2. **Sobreposição** de mídia e tipo entre **8% e 15%** da largura. Abaixo de 8% lê como
   desalinhamento acidental; acima de 20% a legibilidade cai e precisa de scrim.
3. **Alinhamento óptico**: aspas de abertura, ícones circulares e letras redondas (O, C, S)
   precisam de -1 a -3px além da margem para *parecerem* alinhadas. Alinhamento geométrico exato
   parece torto — é assim que o olho funciona.

Full-bleed real: `relative left-1/2 w-screen -translate-x-1/2`. Funciona porque o `body` do
projeto tem `overflow-x: clip` — sem isso, `100vw` inclui a largura da barra de rolagem e cria
~15px de rolagem horizontal no desktop. Tem de ser `clip`, nunca `hidden`: `hidden` transforma o
`body` em contêiner de rolagem e os `.chapter__stage` passam a grudar nele em vez de no viewport,
o que mata todo o mecanismo de capítulo. `clip` recorta sem criar contêiner de rolagem.

## 4. Hierarquia — três níveis, sem caixa

Toda seção tem exatamente três níveis de leitura. Eles se distinguem por **tamanho, peso e cor**.
Borda, fundo e sombra não são níveis: são contêineres, e contêiner é decisão de composição.

| Nível | O que é | Tamanho | Peso | Cor |
|---|---|---|---|---|
| N1 | o que a pessoa leva se ler só uma coisa | `--text-chapter` ou acima | 600 | 100% (`ink` / branco) |
| N2 | a frase que explica o N1 | `--text-lead` | 400 | 75–80% |
| N3 | a etiqueta que classifica | `--text-eyebrow`/`--text-body-sm` | 500 | acento, ou 60% sobre foto, ou `--color-body-soft` sobre claro |

Regras numéricas:

- Razão de tamanho N1:N2 ≥ **2.2× no ponto de 1440px** (aqui: 50.4 ÷ 16.6 = 3.0). No mobile a
  razão cai para ~1.9× e isso é correto, não um defeito: o corpo não desce de 16px, então o topo
  é que teria de encolher. A 360px quem carrega a hierarquia é a diferença de peso, de família e
  de cor — por isso a regra dos dois eixos abaixo não é opcional.
- N2:N3 ≥ 1.3× — ou o N3 troca tamanho por caixa alta + tracking, que é o que a escala já prevê.
- Cada par adjacente difere em **≥2 dos 3 eixos**. Só tamanho não basta: dois textos de pesos e
  cores iguais em 24px e 18px leem como o mesmo nível mal alinhado.
- Sobre foto: rampa 100 / 80 / 60. **Nunca abaixo de 60%.** Medido sobre o pior caso real — foto
  clara sob o scrim de ink a 0.86 — o branco a 60% dá 5.1:1, a 55% dá 4.55:1 e a 50% dá 4.05:1.
  50% reprova; 55% passa com folga nenhuma e some assim que a foto muda. 60% é o piso com margem.
- **Sobre fundo claro, opacidade não serve para atenuar texto.** O ink a 60% sobre branco dá
  4.48:1 e reprova por pouco; a 55%, 3.83:1. Atenue trocando a **cor**, não o alfa —
  `--color-body-soft` (#666666) dá 5.74:1 e é por isso que ele existe como token.
- O eyebrow acima do título é a única inversão de posição permitida: ele lê como etiqueta, não
  como manchete, porque tamanho e caixa mudam ao mesmo tempo.

```tsx
<div className="max-w-[42ch]">
  {/* N3 — 11px, caixa alta, +0.24em, acento */}
  <Eyebrow className="text-rose-soft">{consultation.eyebrow}</Eyebrow>

  {/* N1 — 32→56px, display, 600, 100% */}
  <h2 className="text-chapter mt-6 font-display font-semibold text-balance text-white">
    {consultation.title}
  </h2>

  {/* N2 — 17→22px, sans, 400, 80% */}
  <p className="text-lead mt-6 max-w-[48ch] text-white/80">{consultation.lead}</p>
</div>
```

**Quando um card é legítimo** (e só então): 3+ itens paralelos que precisam de superfície
clicável ≥44×44px, ou conteúdo que alguém vai *ler para copiar* (endereço, horário, telefone)
por cima de imagem movimentada — no projeto, `.glass-panel`, tingido com o ink a 0.86 em vez de
branco, porque branco translúcido sobre foto clara perde o recorte.

## 5. Cor

### Construir a rampa a partir de uma cor de marca

Onze degraus, com a luminosidade em OKLCH fixada por degrau e a croma modulada — croma constante
produz neon nos degraus claros e lama nos escuros.

| Degrau | 50 | 100 | 200 | 300 | 400 | 500 | 600 | 700 | 800 | 900 | 950 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| L | .97 | .94 | .89 | .82 | .74 | .66 | .58 | .49 | .40 | .31 | .24 |
| C × base | .14 | .26 | .45 | .68 | .88 | 1.0 | .98 | .88 | .74 | .58 | .44 |

```css
@theme inline {
  --color-brand: #e95d79;
  --color-brand-500: oklch(from var(--color-brand) 0.66 calc(c * 1) h);
  --color-brand-700: oklch(from var(--color-brand) 0.49 calc(c * 0.88) h);
  /* … os onze degraus em tokens.md */
}
```

`oklch(from …)` (relative color syntax) exige Safari 16.4+, Chrome 119+, Firefox 128+. Abaixo
disso, calcule os onze valores uma vez e cole os hex — o resultado é idêntico e não custa suporte.

**Neutros:** duas rampas, nunca uma. Um neutro quente que carrega 2–6% da croma da marca (o
`--color-canvas-soft: #f8f5f6` do projeto é branco puxado para o rosa) e o ink da marca para
texto. Cinza puro `#808080` ao lado de uma paleta quente puxa para o verde por contraste
simultâneo e parece sujo.

### Contraste mínimo — medido, não estimado

| Conteúdo | Mínimo | Regra |
|---|---|---|
| corpo <24px (ou <18.66px em 700) | **4.5:1** | WCAG AA 1.4.3 |
| ≥24px, ou ≥18.66px em peso 700 | **3:1** | manchete, métrica |
| ícone/borda/estado que carrega informação, anel de foco | **3:1** | WCAG AA 1.4.11 |
| público 45+ ou leitura longa | 7:1 | AAA, opcional |

Pares reais do projeto, já medidos:

| Par | Razão | Serve para |
|---|---|---|
| `#1f1f1f` sobre `#ffffff` | 16.5:1 | corpo |
| `#666666` sobre `#ffffff` | 5.74:1 | corpo secundário, com margem sobre o AA |
| `#e95d79` sobre `#ffffff` | **3.33:1** | só ≥24px, ícone e anel de foco. **Nunca corpo** |
| `#e95d79` sobre `#32151e` | 5.0:1 | corpo sobre o ink |
| `#f7a5b8` sobre `#32151e` | 8.74:1 | eyebrow e acento sobre o ink |
| `#f7a5b8` sobre `#ffffff` | 1.90:1 | nada textual — só forma |
| `#e8b98a` sobre `#32151e` | 9.3:1 | hairline e detalhe (sobre branco dá 1.79:1) |

A cor de marca falhar no corpo é normal e não é motivo para trocá-la: é motivo para o corpo ser
neutro e o rosa aparecer em manchete, foco e ação.

### Proporção dominante / secundária / acento

60 / 30 / 10 — mas a versão operável é esta:

- **Dominante (60%)**: fundo e superfícies. Um valor por zona de cor, e a zona muda entre seções.
- **Secundária (30%)**: tipo e mídia.
- **Acento (10%)**: **no máximo um elemento acentuado por dobra**. Reservado para a ação primária
  e o indicador de estado atual. Se o acento aparece 3× na mesma tela ele virou secundária e o
  CTA some junto.
- O acento nunca é fundo de área >15% do viewport — acima disso ele deixa de sinalizar.

## 6. Composição de seção

| Padrão | Quando | Métrica | Como falha |
|---|---|---|---|
| **Centrado** | hero, statement de uma frase, CTA final. Conteúdo curto, sem elemento secundário | título 12–24ch, lead 45–55ch, 1 CTA | três seções centradas seguidas apagam a âncora esquerda; a leitura desacelera e a página vira deck |
| **Split 5/7** | texto + prova visual em relação 1:1 | 5+7 de 12, gutter 32–64px, texto ≤45ch | 6/6 mata a hierarquia; no mobile decida explicitamente qual metade vem primeiro |
| **Alinhado à borda** | legenda sobre mídia, contato, número de capítulo | conteúdo em 30–40% da largura, inset ≥24px mobile / ≥64px desktop | sem scrim direcional o texto encosta no ruído da foto |
| **Full-bleed** | a mídia é o argumento; marquee 10–15vw | 100vw com `overflow-x: clip` no ancestral | dois full-bleed seguidos removem a referência de margem e a página parece sem grade |
| **Sobreposto** | passagem entre capítulos, tipo sobre mídia, número atrás do título | sobreposição 8–15%, par com contraste ≥3:1 | <8% parece erro de alinhamento; >20% fica ilegível |

**Não repetir consecutivos.** Duas seções vizinhas precisam diferir no eixo de alinhamento **e**
na densidade. Densidade = área de tipo ÷ área do viewport: leve <25%, densa >45%.

| Seção n | Sucessores válidos |
|---|---|
| Centrado | Split, Borda, Full-bleed |
| Split 5/7 | Split 7/5 (só se a densidade mudar), Centrado, Borda, Sobreposto |
| Borda | Centrado, Split, Full-bleed |
| Full-bleed | Split, Borda, Centrado |
| Sobreposto | Centrado, Split |

Em 6 seções: no mínimo 4 padrões distintos, nenhum usado mais de 2×.

TSX de cada padrão, com os blocos reais do projeto (`Chapter`, `.beat`, `.container-editorial`,
`Scrim`): [composition.md](composition.md).

## 7. Tokens em Tailwind 4

Tudo que vira utilidade vai em `@theme`. O que só é lido por CSS ou por JS (como
`--chapter-scroll-desktop`) fica em `:root` ou inline no elemento — em `@theme` geraria uma
utilidade morta.

```css
@theme {
  --color-ink: #32151e;
  --color-rose: #e95d79;

  --font-display: 'Cormorant Garamond', ui-serif, Georgia, serif;
  --font-sans: 'Inter Variable', 'Inter', ui-sans-serif, system-ui, sans-serif;

  --text-chapter: clamp(2rem, 1.5rem + 2.22vw, 3.5rem);
  --text-chapter--line-height: 1.06;
  --text-chapter--letter-spacing: -0.015em;
}
```

Mapa de prefixo → utilidade: `--color-*` → `bg-* text-* border-*` · `--font-*` → `font-*` ·
`--text-*` → `text-*` (com os sufixos `--line-height`, `--letter-spacing`, `--font-weight`) ·
`--spacing` → toda a escala numérica · `--breakpoint-*` → variantes responsivas ·
`--container-*` → `max-w-*` · `--radius-*`, `--shadow-*`, `--ease-*`.

O modificador de opacidade (`text-white/80`) funciona com qualquer valor porque o Tailwind 4
resolve em `color-mix(in oklab, …)` — inclusive com `oklch()`. Bloco completo em
[tokens.md](tokens.md).

## Experience Score → parâmetros

O score está definido no CLAUDE.md. Aqui está o que ele significa em números de design:

| Score | Famílias | Degraus em uso | Padrões de composição | Quebras de grade |
|---|---|---|---|---|
| ★★☆☆☆ | 1 | 5 | 2 | 0 |
| ★★★☆☆ | 2 | 7 | 3 | 1 |
| ★★★★☆ | 2 | 9 | 4 | 2–3 |
| ★★★★★ | 2 (+1 mono) | 11 | 5 | 3–5, uma estrutural |

## Anti-patterns

- **Não centralize três seções seguidas** — sem âncora à esquerda o olho refaz a busca do início
  de linha a cada bloco; a página lê como slides, não como documento.
- **Não use card, borda ou sombra para separar itens já a 32px+ de distância** — a borda cria uma
  segunda referência de alinhamento que compete com a grade.
- **Não empilhe sombra + borda + fundo no mesmo elemento** — três marcadores para uma separação.
  Escolha um: sombra = profundidade, borda = recorte, fundo = agrupamento.
- **Não meça contraste sobre gradiente num ponto só** — um gradiente de dois matizes atrás de
  texto muda a razão ao longo da linha; meça nos dois extremos, porque a ponta clara reprova
  sozinha. (*Qual* gradiente é aceitável é decisão de
  [frontend-design](../frontend-design/SKILL.md), não desta skill.)
- **Não passe de 2 famílias ou 3 pesos** — a 16px ninguém distingue 500 de 600, então o peso extra
  não cria nível, só entra no bundle.
- **Não defina `font-size` em `vw` puro** — abaixo de 360px o texto some e o zoom não recupera.
  `clamp()` com mínimo em `rem` sempre.
- **Não use `max-width` em px para corpo** — 720px a 18px dá 80+ch. Use `ch`.
- **Não use `text-align: justify`** — sem hifenização o navegador abre rios de espaço branco.
  `text-wrap: pretty` com margem direita irregular lê melhor.
- **Não use opacidade <60% em texto sobre foto** — sobre o scrim de ink a 0.86, o branco a 50% dá
  4.05:1 e reprova o AA; 60% dá 5.1:1 e é o piso com margem.
- **Não use cinza puro numa paleta quente** — contraste simultâneo o puxa para o verde.
- **Não pinte área grande com o acento** — acima de 15% do viewport ele para de sinalizar ação e
  o CTA no mesmo tom desaparece.
- **Não peça ênfase animando `font-size`, `letter-spacing` ou `margin`** — cada frame refaz o
  layout de todas as line boxes na main thread. Se o efeito couber em `transform`/`opacity`, use
  isso; se for tracking de verdade, que não tem equivalente no compositor, o efeito não vale o
  custo — troque por outro. Curvas e durações são de
  [landing-motion-expert](../landing-motion-expert/SKILL.md).

## Verificação

Rodável, não opinável.

- [ ] `grep -Rn -- "--font-\|font-family" src/styles/` retorna no máximo 2 famílias declaradas.
- [ ] `grep -Rn "text-\[" src/` — cada ocorrência é uma exceção justificada, não um degrau novo.
- [ ] Filtro de cinza no console: `document.documentElement.style.filter = 'grayscale(1)'`. Os
      três níveis de leitura ainda se distinguem. Se não, a hierarquia estava apoiada em cor.
- [ ] DevTools → Elements → Accessibility: todo par texto/fundo ≥4.5:1 (corpo) e ≥3:1 (≥24px).
      Sobre foto, meça com o scrim aplicado e no quadro mais claro do vídeo, não no poster.
- [ ] Medida do corpo, com o snippet de `ch` acima: 60–75 no desktop, 35–45 a 360px.
- [ ] Nenhum degrau novo criado por valor arbitrário — o `@theme` existente foi estendido, não
      reescrito; `git diff src/styles/index.css` não altera nenhum `clamp()` já publicado.
- [ ] Nenhum padrão de composição repetido em seções consecutivas; ≥4 padrões distintos em 6.
- [ ] No máximo um elemento em cor de acento por dobra.
- [ ] Todo espaçamento cai num degrau da escala 4px.

Zoom, alvos de toque, ordem de foco e comportamento em 844×390 são o portão final e ficam com
[responsive-e-acessibility](../responsive-e-acessibility/SKILL.md) — que carrega os números
oficiais do projeto. Não duplique os limites aqui: eles divergem de propósito (44×44 com 24px de
folga é regra de lá, não daqui) e duas fontes de verdade produzem uma auditoria que aprova o que a
outra reprova.
