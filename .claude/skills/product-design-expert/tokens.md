# Tokens — bloco completo

O que vira utilidade do Tailwind vai em `@theme`. O que só é lido por CSS ou JS fica em `:root`
ou inline no elemento: em `@theme` geraria uma classe que ninguém usa e ainda aparece no
autocomplete.

## Gerador de `clamp()` fluido

```ts
/**
 * clamp() fluido entre dois pontos de viewport. Entrada em px, saída em rem+vw.
 * O mínimo e o máximo ficam em rem para que o zoom do navegador continue funcionando —
 * um tamanho em vw puro não responde a zoom e falha o WCAG 1.4.4.
 */
export function fluid(minPx: number, maxPx: number, minVw = 360, maxVw = 1440): string {
  const slope = (maxPx - minPx) / (maxVw - minVw)
  const intercept = (minPx - slope * minVw) / 16
  return `clamp(${minPx / 16}rem, ${intercept.toFixed(3)}rem + ${(slope * 100).toFixed(2)}vw, ${maxPx / 16}rem)`
}

fluid(32, 56) // "clamp(2rem, 1.500rem + 2.22vw, 3.5rem)"
```

Rode uma vez, cole o resultado no `@theme`. Não deixe a chamada em runtime: o valor é constante
e o CSS precisa dele em build.

## Medir `1ch` da família real

A medida de leitura em `ch` (60–75 no corpo) só é confiável depois de saber quanto vale `1ch` na
fonte **daquele** elemento: `1ch` é o avanço do glifo `0`, não a largura média de caractere, e
resolve contra a família do próprio elemento. Cole no console com a página carregada:

```js
const el = document.querySelector('p')
const fs = parseFloat(getComputedStyle(el).fontSize)
const probe = Object.assign(document.createElement('span'), { textContent: '0'.repeat(100) })
probe.style.cssText = 'position:absolute;visibility:hidden;white-space:pre;font:inherit'
el.append(probe)
console.log((probe.getBoundingClientRect().width / 100 / fs).toFixed(3) + 'em por ch')
probe.remove()
```

Inter dá ≈0.57em por `ch`; grotescas mais estreitas ficam perto de 0.5em. O mesmo `max-w-[65ch]`
num título em `font-display` produz outra largura — meça antes de fixar o teto de um papel novo.

## Os onze degraus e as bandas

A referência para uma página que ainda não tem escala. Todo degrau fluido em `clamp()`, com mínimo
em 360px e máximo em 1440px. Num repositório que já publica `@theme`, acrescente só o que falta —
ver o aviso em [Bloco `@theme`](#bloco-theme).

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
22–28px → 1.2–1.3 · 13–22px → 1.45–1.65 · caixa alta de rótulo → 1.

**Piso de 1.02 sobrepõe a banda quando o display carrega acento.** Em pt-BR o til de `Ã` e a cedilha
de `Ç` sobem acima da altura de capitular, e a 0.92 a linha de cima os corta — por isso os dois
degraus do topo ficam em 1.02 e 1.04. A causa e o teste visual são de
[frontend-design](../frontend-design/type-pairs.md#3-acento-clipado-por-line-height); o valor é daqui.

**Bandas de letter-spacing:** ≥48px → -0.03 a -0.02em · 28–48px → -0.02 a -0.01em · 16–28px → -0.005
a 0 · <14px em caixa alta → +0.12 a +0.24em. Causa: o tracking padrão da fonte foi desenhado para
16px. A 64px ele abre buracos entre letras; a 11px em caixa alta as hastes colidem.

## Bloco `@theme`

> **Este bloco é o ponto de partida de uma página nova, não o estado deste repositório.**
> `src/styles/index.css` já publica um `@theme` com cinco degraus de tipo, ajustados contra a
> headline real mais longa de cada papel. Colar o bloco abaixo por cima reflui todas as manchetes
> do site.
>
> Num repositório com `@theme`: **acrescente só os degraus que faltam** — `display`, `metric`,
> `title`, `body`, `body-sm`, `label`, `caption` — e deixe os cinco existentes como estão. Num
> projeto novo: use o bloco inteiro.

### Os cinco degraus já em produção neste repositório

| Token | Valor em produção | @360 | @1440 | teto |
|---|---|---|---|---|
| `--text-hero` | `clamp(2.125rem, 4.5vw, 4.5rem)` | 34px | 64.8px | 72px a 1600 |
| `--text-chapter` | `clamp(1.875rem, 3.5vw, 3.25rem)` | 30px | 50.4px | 52px |
| `--text-statement` | `clamp(1.625rem, 3.4vw, 3.25rem)` | 26px | 49px | 52px |
| `--text-lead` | `clamp(1rem, 1.15vw, 1.25rem)` | 16px | 16.6px | 20px a 1739 |
| `--text-eyebrow` | `0.6875rem` | 11px | 11px | — |

Todos os cinco são **menores** que a referência de onze degraus do
[SKILL.md](SKILL.md#1-escala-tipográfica): o teto de cada um foi baixado até a palavra mais longa
da copy real caber sem hífen. O piso de 16px em `--text-lead` não sobe — abaixo dele o Safari do
iOS dá zoom ao focar um campo.

```css
@import 'tailwindcss';

@import '@fontsource/cormorant-garamond/400.css';
@import '@fontsource/cormorant-garamond/600.css';
@import '@fontsource/cormorant-garamond/700.css';
@import '@fontsource-variable/inter';

@theme {
  /* ---- Famílias -------------------------------------------------------- */
  --font-display: 'Cormorant Garamond', ui-serif, Georgia, 'Times New Roman', serif;
  --font-sans: 'Inter Variable', 'Inter', ui-sans-serif, system-ui, -apple-system, sans-serif;

  /* ---- Escala tipográfica ---------------------------------------------- */
  /* 1.02 é o piso para display acentuado: a 0.92 a linha de cima corta o til de Ã. */
  --text-display: clamp(3rem, 1.333rem + 7.41vw, 8rem);
  --text-display--line-height: 1.02;
  --text-display--letter-spacing: -0.03em;

  --text-hero: clamp(2.5rem, 1.5rem + 4.44vw, 5.5rem);
  --text-hero--line-height: 1.04;
  --text-hero--letter-spacing: -0.02em;

  --text-chapter: clamp(2rem, 1.5rem + 2.22vw, 3.5rem);
  --text-chapter--line-height: 1.06;
  --text-chapter--letter-spacing: -0.015em;

  --text-statement: clamp(1.75rem, 1.417rem + 1.48vw, 2.75rem);
  --text-statement--line-height: 1.18;
  --text-statement--letter-spacing: -0.01em;

  --text-metric: clamp(2.5rem, 1.833rem + 2.96vw, 4.5rem);
  --text-metric--line-height: 0.95;
  --text-metric--letter-spacing: -0.02em;

  --text-title: clamp(1.25rem, 1.167rem + 0.37vw, 1.5rem);
  --text-title--line-height: 1.3;
  --text-title--letter-spacing: -0.005em;

  /* O piso nunca desce de 1rem: abaixo de 16px o Safari do iOS dá zoom ao focar um campo. */
  --text-lead: clamp(1.0625rem, 0.958rem + 0.46vw, 1.375rem);
  --text-lead--line-height: 1.55;

  --text-body: clamp(1rem, 0.958rem + 0.19vw, 1.125rem);
  --text-body--line-height: 1.65;

  --text-body-sm: 0.875rem;
  --text-body-sm--line-height: 1.6;

  --text-label: 0.75rem;
  --text-label--line-height: 1;
  --text-label--letter-spacing: 0.14em;

  --text-eyebrow: 0.6875rem;
  --text-eyebrow--line-height: 1;
  --text-eyebrow--letter-spacing: 0.24em;

  --text-caption: 0.8125rem;
  --text-caption--line-height: 1.45;
  --text-caption--letter-spacing: 0.01em;

  /* ---- Marca ------------------------------------------------------------ */
  --color-ink: #32151e;
  --color-ink-soft: #4a202b;
  --color-rose: #e95d79;
  --color-rose-soft: #f7a5b8;
  --color-canvas: #ffffff;
  --color-canvas-soft: #f8f5f6;
  --color-surface: #fffdfd;
  --color-body: #1f1f1f;
  --color-body-soft: #666666;   /* 5.74:1 sobre branco */
  --color-line: #e8e2e4;
  --color-success: #5fa870;
  --color-gilt: #e8b98a;        /* filete: 9.3:1 sobre o ink, 1.79:1 sobre branco */

  /* ---- Forma e tempo ---------------------------------------------------- */
  --radius-card: 24px;
  --radius-pill: 999px;
  --shadow-card: 0 1px 2px rgb(50 21 30 / 0.04), 0 12px 32px -12px rgb(50 21 30 / 0.16);
  --shadow-lift: 0 2px 4px rgb(50 21 30 / 0.06), 0 24px 48px -16px rgb(50 21 30 / 0.24);
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-out-quint: cubic-bezier(0.22, 1, 0.36, 1);

  /* ---- Container -------------------------------------------------------- */
  --container-editorial: 1280px;
}
```

`--spacing` fica no default do Tailwind 4 (`0.25rem`): a base 4px já é a escala inteira, e
redefini-la invalida toda a documentação da biblioteca para quem ler o código depois.

## Rampa de cor a partir de uma cor de marca

A luminosidade fica fixa por degrau e a croma é modulada — croma constante produz neon nos degraus
claros e lama nos escuros.

| Degrau | 50 | 100 | 200 | 300 | 400 | 500 | 600 | 700 | 800 | 900 | 950 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| L | .97 | .94 | .89 | .82 | .74 | .66 | .58 | .49 | .40 | .31 | .24 |
| C × base | .14 | .26 | .45 | .68 | .88 | 1.0 | .98 | .88 | .74 | .58 | .44 |

```css
@theme inline {
  --color-brand: #e95d79;

  --color-brand-50:  oklch(from var(--color-brand) 0.97 calc(c * 0.14) h);
  --color-brand-100: oklch(from var(--color-brand) 0.94 calc(c * 0.26) h);
  --color-brand-200: oklch(from var(--color-brand) 0.89 calc(c * 0.45) h);
  --color-brand-300: oklch(from var(--color-brand) 0.82 calc(c * 0.68) h);
  --color-brand-400: oklch(from var(--color-brand) 0.74 calc(c * 0.88) h);
  --color-brand-500: oklch(from var(--color-brand) 0.66 calc(c * 1.00) h);
  --color-brand-600: oklch(from var(--color-brand) 0.58 calc(c * 0.98) h);
  --color-brand-700: oklch(from var(--color-brand) 0.49 calc(c * 0.88) h);
  --color-brand-800: oklch(from var(--color-brand) 0.40 calc(c * 0.74) h);
  --color-brand-900: oklch(from var(--color-brand) 0.31 calc(c * 0.58) h);
  --color-brand-950: oklch(from var(--color-brand) 0.24 calc(c * 0.44) h);
}
```

`@theme inline` faz a utilidade conter o **valor** do token em vez de `var(--color-brand-500)`. A
diferença só importa quando o token depende de outra variável que muda de valor em algum escopo
aninhado — é o caso da camada semântica abaixo, e é por isso que ela precisa de `inline`. Aqui,
com `--color-brand` fixo, `@theme` puro produziria o mesmo resultado; `inline` está no bloco por
consistência com a camada semântica, não porque a rampa quebre sem ele.

**Onde a marca cai na rampa:** meça, não presuma. Uma cor com L≈0.68 não é o degrau 500 por
decreto — compare a luminosidade e encaixe-a no degrau mais próximo, senão a rampa fica com dois
tons quase idênticos.

**Extrair os hex resolvidos** (para quando o alvo inclui Safari <16.4 ou Chrome <119):

```js
// Cole no console com a página carregada. Copie a saída para o CSS.
const probe = document.createElement('span')
document.body.append(probe)
const L = [0.97, 0.94, 0.89, 0.82, 0.74, 0.66, 0.58, 0.49, 0.4, 0.31, 0.24]
const K = [0.14, 0.26, 0.45, 0.68, 0.88, 1.0, 0.98, 0.88, 0.74, 0.58, 0.44]
L.forEach((l, i) => {
  probe.style.color = `oklch(from #e95d79 ${l} calc(c * ${K[i]}) h)`
  console.log(i, getComputedStyle(probe).color)
})
probe.remove()
```

**Neutros:** duas rampas. A quente sai da mesma matriz com `calc(c * 0.03)` — 2–6% da croma da
marca. Cinza puro ao lado de uma paleta quente puxa para o verde por contraste simultâneo.

## Pares de contraste medidos

Os mínimos por papel (4.5:1 no corpo, 3:1 em ≥24px e em anel de foco) estão no
[SKILL.md](SKILL.md#5-cor). Aqui ficam os pares deste projeto, já calculados — use estes em vez de
estimar de novo.

| Par | Razão | Serve para |
|---|---|---|
| `#1f1f1f` sobre `#ffffff` | 16.5:1 | corpo |
| `#666666` sobre `#ffffff` | 5.74:1 | corpo secundário |
| `#e95d79` sobre `#ffffff` | **3.33:1** | só ≥24px, ícone e anel de foco. **Nunca corpo** |
| `#e95d79` sobre `#32151e` | 5.0:1 | corpo sobre o ink |
| `#f7a5b8` sobre `#32151e` | 8.74:1 | eyebrow e acento sobre o ink |
| `#f7a5b8` sobre `#ffffff` | 1.90:1 | nada textual — só forma |
| `#e8b98a` sobre `#32151e` | 9.3:1 | hairline e detalhe (sobre branco dá 1.79:1) |

Sobre o scrim de ink a 0.86, branco a 60% dá 5.1:1, a 55% dá 4.55:1 e a 50% dá 4.05:1 — 60% é o piso
com margem para texto sobre fotografia.

## Camada semântica

Nunca chame `--color-brand-700` direto no componente. Papéis mudam por zona de cor; degraus não.

```css
:root {
  --surface: var(--color-canvas);
  --surface-raised: var(--color-canvas-soft);
  --text-strong: var(--color-ink);
  --text-default: var(--color-body);
  --text-muted: var(--color-body-soft);   /* 5.74:1 sobre branco — o limite */
  --accent: var(--color-rose);
  --border-hairline: var(--color-line);
}

/* Zona escura: a mesma seção, invertida. Só os papéis mudam. */
[data-zone='dark'] {
  --surface: var(--color-ink);
  --surface-raised: var(--color-ink-soft);
  --text-strong: #ffffff;
  --text-default: rgb(255 255 255 / 0.8);
  --text-muted: rgb(255 255 255 / 0.6);   /* nunca abaixo de 0.6 */
  --accent: var(--color-rose-soft);       /* 8.74:1 sobre o ink */
  --border-hairline: rgb(255 255 255 / 0.25);
}
```

Esses `--surface`, `--text-muted` etc. estão em `:root`, **não** em `@theme` — sozinhos eles não
geram nenhuma utilidade, e `bg-surface` não existe. A ponte é um segundo bloco, e ele tem de ser
`inline`: sem `inline` a utilidade emitiria `var(--color-surface)`, que é resolvido uma vez em
`:root` e não acompanharia a redefinição dentro de `[data-zone='dark']`.

```css
@theme inline {
  --color-surface: var(--surface);
  --color-surface-raised: var(--surface-raised);
  --color-text-strong: var(--text-strong);
  --color-text-default: var(--text-default);
  --color-text-muted: var(--text-muted);
  --color-accent: var(--accent);
  --color-border-hairline: var(--border-hairline);
}
```

A partir daí `bg-surface`, `text-muted` e `border-border-hairline` existem, e a mesma marcação
inverte inteira ao ganhar `data-zone="dark"` no ancestral — sem uma única variante `dark:` no JSX.

O acento **troca de degrau** entre as zonas. `--color-rose` sobre branco dá 3.33:1 e sobre o ink
dá 5.0:1; `--color-rose-soft` sobre branco dá 1.90:1. É a mesma cor de marca em dois papéis, não
duas marcas. Trocar a zona sem trocar o degrau do acento é como a maioria das inversões reprova o
contraste.

## Padding vertical dentro de um palco

`.chapter__stage` tem 100svh e o padding é relativo ao viewport: `pt-[12svh]` para limpar a
navegação (≥72px) e `pb-[12svh]` para o cue de scroll.

Use `svh`, **não `vh`**: `vh` resolve contra o viewport *grande* e produziria um padding maior que
os 12% pedidos justamente no celular com a barra de URL visível, que é o caso que ele existia para
resolver. Texto vive no `svh`; a mídia (`.chapter__media`) preenche o `lvh`, para que a barra
retraindo nunca revele uma faixa de fundo.

## Fora do `@theme`

| Valor | Onde | Por quê |
|---|---|---|
| `--chapter-scroll-desktop` / `-mobile` | inline no `<section>` | é dado de capítulo, não token; muda por instância |
| altura de palco, insets de safe zone | `@layer components` | é geometria de layout, não escala compartilhada |
| durações e easings de timeline GSAP | `src/lib/` em TS | o GSAP lê números, não strings CSS |
| cores de estado de um único componente | o próprio componente | um token global usado uma vez é acoplamento sem ganho |

## Armadilhas do Tailwind 4

- **Valor arbitrário é dívida.** `text-[27px]` não aparece em nenhuma auditoria de escala. Se o
  degrau é necessário, ele vira token; se é exceção, comente por quê na mesma linha.
- **Modificador de opacidade** (`text-white/80`, `bg-ink/86`) resolve em
  `color-mix(in oklab, …)` e funciona com hex, `oklch()` e `var()`. Preferível a declarar um
  token novo para cada nível de transparência.
- **`text-balance` só em títulos curtos** — o Chromium desiste do balanceamento acima de 6 linhas,
  então em parágrafo a declaração não faz nada e só ocupa espaço. Corpo usa `text-wrap: pretty`,
  já aplicado em `@layer base` para `p`, e `balance` já está em `h1, h2, h3`: não repita nenhum
  dos dois como utilidade no JSX.
- **Fontes variáveis**: onde existir uma variável, importe só ela — `@fontsource-variable/inter`
  entrega o eixo de peso inteiro num arquivo, e somar `400.css`/`600.css` estáticos da *mesma*
  família baixa as duas coisas. Pesos estáticos continuam corretos para uma família sem versão
  variável ou quando só 2–3 pesos são usados, que é o caso da Cormorant aqui (400/600/700).
