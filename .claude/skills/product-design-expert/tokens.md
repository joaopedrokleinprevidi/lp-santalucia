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

## Bloco `@theme`

> **Este bloco é o ponto de partida de uma página nova, não o estado deste repositório.**
> `src/styles/index.css` já publica um `@theme` com cinco degraus de tipo (`hero` 34→72,
> `chapter` 30→52, `statement` 26→52, `lead` 16→20, `eyebrow` 11), ajustados contra a headline
> real mais longa de cada papel. Colar o bloco abaixo por cima reflui todas as manchetes do site.
>
> Num repositório com `@theme`: **acrescente só os degraus que faltam** — `display`, `metric`,
> `title`, `body`, `body-sm`, `label`, `caption` — e deixe os cinco existentes como estão. Num
> projeto novo: use o bloco inteiro.

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
