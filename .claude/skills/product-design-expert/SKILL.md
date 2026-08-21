---
name: product-design-expert
description: Use when defining or auditing a page's visual system — type scale, spacing, grid, hierarchy, palette, contrast, design tokens, section composition. Fase 10a — escala tipografica, tamanho de fonte, clamp, espacamento, margem, grade, coluna, medida em ch, hierarquia de leitura, paleta, contraste 4.5:1, tokens no @theme, composicao de secao. Tambem quando a pagina esta apertada, sem hierarquia ou com todas as secoes iguais.
argument-hint: [secao-ou-arquivo] [o-que-corrigir]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

# Product Design — sistema visual

| | |
|---|---|
| **ENTRADA** | `design/design-system.json` (cores medidas, classe tipográfica observada); `src/data/site.ts` e `design/landing-blueprint.md` (a copy real — a palavra mais longa fixa o teto do hero); `design/creative-direction.json` (Experience Score); `src/styles/index.css` se já houver `@theme` |
| **SAÍDA** | `src/styles/index.css` com o `@theme` completo (escala de tipo, espaço, grade, paleta, tokens de forma); o padrão de composição de cada seção anotado em `design/landing-blueprint.md`; e **os componentes de capítulo escritos** em `src/components/chapters/`, um por seção de `src/data/story.ts`, com o TSX do padrão escolhido ([composition.md](composition.md)) e os alvos `data-*` já marcados — é este arquivo que `landing-motion-expert` (10c) recebe para animar |
| **ANTES** | Fase 9, a parada manual: depois de `prompt-imagem` (8a) e `prompt-animacao` (8b), o dev gerou e entregou as mídias em `design/renders/`, já copiadas para `assets-source/`. Nenhuma skill roda entre a 8b e esta |
| **DEPOIS** | `frontend-design` (Fase 10b) escolhe famílias e textura **sobre** estes tokens |

Esta skill define os números que o resto do time consome. Nada aqui é opinião — cada valor é
verificável no DevTools.

**Possui:** escala de tipo, escala de espaço, grade, níveis de leitura, paleta e contraste, padrão de
composição por seção, tokens em `@theme`.
**Não possui:** o caráter estético e *quais* famílias ([frontend-design](../frontend-design/SKILL.md)
decide a família; esta skill decide os degraus e o pareamento), ordem das seções
([estrutura-secoes](../estrutura-secoes/SKILL.md)), curvas e durações
([landing-motion-expert](../landing-motion-expert/SKILL.md)), comportamento por breakpoint
([audit-responsivo](../audit-responsivo/SKILL.md)).

## Ordem das decisões

Cada passo consome o anterior; inverter produz retrabalho. **1.** Escala tipográfica — os degraus e o
pareamento de famílias. **2.** Escala de espaço + grade — base 4px, colunas, medida de leitura em
`ch`. **3.** Paleta — rampa da cor de marca, neutros e contrastes medidos. **4.** Hierarquia — três
níveis por seção, com tamanho/peso/cor apenas. **5.** Composição — o padrão de cada seção e a regra
de não-repetição. Só então escreva markup.

Os tokens completos ficam em [tokens.md](tokens.md); o TSX de cada padrão de seção, em
[composition.md](composition.md).

## 1. Escala tipográfica

> **Leia o `@theme` que já existe antes de escrever qualquer degrau.** Uma escala em produção foi
> ajustada contra a headline real mais longa de cada papel; substituí-la reflui todas as manchetes e
> invalida a tabela de valores resolvidos de [audit-responsivo](../audit-responsivo/SKILL.md). A
> regra é **estender, nunca sobrescrever**: acrescente o degrau que falta e deixe os que já servem.
> Os cinco degraus que este repositório publica, com os valores resolvidos em 360 e 1440, estão em
> [tokens.md](tokens.md#os-cinco-degraus-já-em-produção-neste-repositório).

A referência para uma página que ainda não tem escala: onze degraus, todo degrau fluido em `clamp()`
com mínimo em 360px e máximo em 1440px. O gerador `fluid(minPx, maxPx)` está em
[tokens.md](tokens.md#gerador-de-clamp-fluido) — rode uma vez e cole o resultado, não deixe a chamada
em runtime.

Os onze degraus com `clamp()`, line-height, letter-spacing e papel de cada um, as bandas de
line-height e de letter-spacing por faixa de tamanho, e o piso de 1.02 que o til de `Ã` e a cedilha
de `Ç` impõem aos dois degraus do topo estão em
[tokens.md](tokens.md#os-onze-degraus-e-as-bandas).

### Pareamento display + corpo

- **Duas famílias, no máximo** — uma terceira só se for mono, e só para números tabulares. Display
  serif de alto contraste + corpo sans (o projeto: Cormorant Garamond + Inter Variable).
- A display só desce até `--text-title` (20px): abaixo disso um serif de alto contraste perde as
  hastes finas em telas 1x — Cormorant a 15px some.
- Se as duas forem sans, exija contraste estrutural: eixos diferentes (grotesk vs. geométrica) **ou**
  diferença de peso ≥300 (400 vs. 700). Sem isso parece erro de importação.
- **Três pesos por família, no máximo.** A 16px ninguém distingue 500 de 600 — o quarto peso não cria
  nível, só entra na página. `font-synthesis-weight: none` no `body`: falso negrito engorda a haste
  sem redesenhar a letra.
- Contadores GSAP levam `font-variant-numeric: tabular-nums`; sem isso a largura dos dígitos muda a
  cada frame do count-up e o bloco inteiro treme.

**Regra de densidade:** no máximo 4 degraus visíveis na mesma seção — três níveis de leitura mais um
caption. Um quinto degrau significa que a seção tem duas ideias.

## 2. Escala de espaço

Base **4px**, o padrão do Tailwind 4 (`--spacing: 0.25rem`), então toda utilidade numérica já cai na
escala. Degraus permitidos: 4, 8, 12, 16, 24, 32, 48, 64, 96, 128, 160, 192, 256. Nunca um valor
entre degraus: `mt-[27px]` é a assinatura de layout ajustado no olho.

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

O `padding-inline` não é uma tabela de degraus: é `clamp(24px, 5vw, 64px)`, já implementado em
`.container-editorial` junto com `max-width: 1280px`. Use a classe; não recrie o container por seção
e não invente uma margem de 80px que ela não produz. O `padding-block` acompanha o viewport: 72–96px
abaixo de 640, 96–120px até 1023, 120–160px até 1279, 160–192px a partir de 1280.

**Dentro de um palco de 100svh** (`.chapter__stage`) o padding é vertical e relativo ao viewport:
`pt-[12svh]` e `pb-[12svh]`, em `svh` e nunca em `vh`. A causa e a divisão `svh` para texto / `lvh`
para mídia estão em [tokens.md](tokens.md#padding-vertical-dentro-de-um-palco).

### Medida de leitura

| Papel | Medida |
|---|---|
| corpo longo | 60–75ch (ótimo 66) |
| lead / intro | 45–55ch |
| headline display | 12–20ch, nunca >24ch |
| coluna de grid de 3–4 itens | 30–40ch |
| caption / label | 30–40ch |
| mobile a 360px | 35–45ch (é o que sobra com 24px de padding) |

Use `ch`, não px: `max-w-[65ch]`. A unidade acompanha a fonte; um `max-width` em px não. Com Inter a
16px, 65ch ≈ 590px — o mesmo bloco em `max-w-[720px]` chega a ~79ch e o olho começa a errar o retorno
de linha. Ressalva: `1ch` é o avanço do glifo `0` e resolve contra a fonte **do próprio elemento**, o
que muda o número num título em `font-display`. Meça com o snippet de
[tokens.md](tokens.md#medir-1ch-da-família-real) antes de fixar o teto de um papel novo.

## 3. Grade

Colunas e gutter são decisão desta skill. A margem vem do `clamp()` da `.container-editorial` e
está aqui só como valor resolvido, para conferência.

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

Três quebras legítimas da grade; fora delas, quebra é erro:

1. **Sangria de 1–2 colunas** além da margem, em **um** elemento por seção. Com dois sangrando na
   mesma seção a grade deixa de existir como referência, e aí a quebra parece bug.
2. **Sobreposição** de mídia e tipo entre **8% e 15%** da largura. Abaixo de 8% lê como
   desalinhamento acidental; acima de 20% a legibilidade cai e precisa de scrim.
3. **Alinhamento óptico**: aspas de abertura, ícones circulares e letras redondas (O, C, S) pedem -1
   a -3px além da margem para *parecerem* alinhados. Geométrico exato parece torto.

Full-bleed real: `relative left-1/2 w-screen -translate-x-1/2`, e o `body` precisa de
`overflow-x: clip` — sem ele `100vw` inclui a barra de rolagem e cria ~15px de rolagem horizontal.
Tem de ser `clip`, nunca `hidden`: `hidden` transforma o `body` em contêiner de rolagem e os
`.chapter__stage` passam a grudar nele em vez de no viewport, o que mata o mecanismo de capítulo
inteiro.

## 4. Hierarquia — três níveis, sem caixa

Toda seção tem exatamente três níveis de leitura, distinguidos por **tamanho, peso e cor**. Borda,
fundo e sombra não são níveis: são contêineres, e contêiner é decisão de composição.

| Nível | O que é | Tamanho | Peso | Cor |
|---|---|---|---|---|
| N1 | o que a pessoa leva se ler só uma coisa | `--text-chapter` ou acima | 600 | 100% (`ink` / branco) |
| N2 | a frase que explica o N1 | `--text-lead` | 400 | 75–80% |
| N3 | a etiqueta que classifica | `--text-eyebrow`/`--text-body-sm` | 500 | acento, ou 60% sobre foto, ou `--color-body-soft` sobre claro |

- Razão N1:N2 ≥ **2.2× a 1440px** (aqui: 50.4 ÷ 16.6 = 3.0). No mobile cai para ~1.9× e isso é
  correto: o corpo não desce de 16px, então o topo é que teria de encolher. A 360px quem carrega a
  hierarquia é peso, família e cor — por isso a regra dos dois eixos não é opcional.
- N2:N3 ≥ 1.3× — ou o N3 troca tamanho por caixa alta + tracking, que a escala já prevê.
- Cada par adjacente difere em **≥2 dos 3 eixos**. Só tamanho não basta: dois textos de mesmo peso e
  cor em 24px e 18px leem como o mesmo nível mal alinhado.
- Sobre foto: rampa 100 / 80 / 60, **nunca abaixo de 60%**. Sobre o scrim de ink a 0.86, branco a 60%
  dá 5.1:1, a 55% dá 4.55:1 e a 50% dá 4.05:1 — 50% reprova e 55% some assim que a foto muda.
- **Sobre fundo claro, opacidade não serve para atenuar texto.** Ink a 60% sobre branco dá 4.48:1 e
  reprova; a 55%, 3.83:1. Atenue trocando a **cor**: `--color-body-soft` (#666666) dá 5.74:1.
- O eyebrow acima do título é a única inversão de posição permitida — tamanho e caixa mudam ao mesmo
  tempo, então ele lê como etiqueta, não como manchete.

O TSX dos três níveis está em [composition.md](composition.md#3-alinhado-à-borda). **Um card só é
legítimo** com 3+ itens paralelos que precisem de superfície clicável ≥44×44px, ou com conteúdo que
alguém vai *ler para copiar* (endereço, horário, telefone) sobre imagem movimentada — no projeto,
`.glass-panel`, tingido com ink a 0.86, porque branco translúcido sobre foto clara perde o recorte.

## 5. Cor

Rampa de onze degraus a partir da cor de marca, com a luminosidade em OKLCH fixada por degrau e a
croma modulada — croma constante produz neon nos degraus claros e lama nos escuros.

O bloco `@theme inline` com os onze degraus, a camada semântica por zona de cor e o script que extrai
os hex resolvidos (para alvos abaixo de Safari 16.4 / Chrome 119, sem `oklch(from …)`) estão em
[tokens.md](tokens.md#rampa-de-cor-a-partir-de-uma-cor-de-marca). **Onde a marca cai na rampa:** meça
pela luminosidade, não presuma, senão a rampa fica com dois tons quase idênticos.

**Neutros:** duas rampas, nunca uma. Um neutro quente que carrega 2–6% da croma da marca (o
`--color-canvas-soft: #f8f5f6` é branco puxado para o rosa) e o ink da marca para texto. Cinza puro
`#808080` ao lado de uma paleta quente puxa para o verde por contraste simultâneo e parece sujo.

| Conteúdo | Mínimo | Regra |
|---|---|---|
| corpo <24px (ou <18.66px em 700) | **4.5:1** | WCAG AA 1.4.3 |
| ≥24px, ou ≥18.66px em peso 700 | **3:1** | manchete, métrica |
| ícone/borda/estado que carrega informação, anel de foco | **3:1** | WCAG AA 1.4.11 |
| público 45+ ou leitura longa | 7:1 | AAA, opcional |

Os sete pares já medidos deste projeto (`#e95d79` sobre branco dá **3.33:1** e por isso nunca é
corpo; `#f7a5b8` sobre o ink dá 8.74:1) estão em [tokens.md](tokens.md#pares-de-contraste-medidos).
A cor de marca falhar no corpo é normal e não é motivo para trocá-la: é motivo para o corpo ser
neutro e o rosa aparecer em manchete, foco e ação.

**Proporção 60 / 30 / 10, na versão operável:** dominante = fundo e superfícies, um valor por zona de
cor, e a zona muda entre seções; secundária = tipo e mídia; acento = **no máximo um elemento
acentuado por dobra**, reservado à ação primária e ao indicador de estado atual. Se o acento aparece
3× na mesma tela ele virou secundária e o CTA some junto, e ele nunca é fundo de área >15% do
viewport. *Qual* proporção acima desse teto a página adota é de
[frontend-design](../frontend-design/SKILL.md).

## 6. Composição de seção

| Padrão | Quando | Métrica | Como falha |
|---|---|---|---|
| **Centrado** | hero, statement de uma frase, CTA final. Conteúdo curto, sem elemento secundário | título 12–24ch, lead 45–55ch, 1 CTA | três centradas seguidas apagam a âncora esquerda; a página vira deck |
| **Split 5/7** | texto + prova visual em relação 1:1 | 5+7 de 12, gutter 32–64px, texto ≤45ch | 6/6 mata a hierarquia; no mobile decida qual metade vem primeiro |
| **Alinhado à borda** | legenda sobre mídia, contato, número de capítulo | conteúdo em 30–40% da largura, inset ≥24px mobile / ≥64px desktop | sem scrim direcional o texto encosta no ruído da foto |
| **Full-bleed** | a mídia é o argumento; marquee 10–15vw | 100vw com `overflow-x: clip` no ancestral | dois seguidos removem a referência de margem |
| **Sobreposto** | passagem entre capítulos, tipo sobre mídia, número atrás do título | sobreposição 8–15%, par com contraste ≥3:1 | <8% parece erro de alinhamento; >20% fica ilegível |

**Não repetir consecutivos.** Duas seções vizinhas diferem no eixo de alinhamento **e** na densidade
— densidade = área de tipo ÷ área do viewport: leve <25%, densa >45%. Nenhum padrão se segue a si
mesmo; em 6 seções, no mínimo 4 padrões distintos e nenhum usado mais de 2×.

A matriz de sucessão completa, o TSX de cada padrão com os blocos reais do projeto (`Chapter`,
`.beat`, `.container-editorial`, `Scrim`), o script que mede densidade e a tabela de colapso no
mobile estão em [composition.md](composition.md#matriz-de-sucessão).

## 7. Tokens em Tailwind 4

Tudo que vira utilidade vai em `@theme`. O que só é lido por CSS ou por JS (como
`--chapter-scroll-desktop`) fica em `:root` ou inline no elemento — em `@theme` geraria uma utilidade
morta que ainda aparece no autocomplete. Mapa de prefixo → utilidade: `--color-*` →
`bg-* text-* border-*` · `--font-*` → `font-*` · `--text-*` → `text-*` (com os sufixos
`--line-height`, `--letter-spacing`, `--font-weight`) · `--spacing` → toda a escala numérica ·
`--breakpoint-*` → variantes responsivas · `--container-*` → `max-w-*` · `--radius-*`, `--shadow-*`,
`--ease-*`.

O bloco `@theme` completo, a camada semântica com `[data-zone='dark']`, a tabela do que fica fora do
`@theme` e as armadilhas do Tailwind 4 (valor arbitrário, `text-balance`, fonte variável) estão em
[tokens.md](tokens.md).

## Experience Score → parâmetros

O score está definido no `CLAUDE.md`. Aqui está o que ele significa em números de design:

| Score | Famílias | Degraus em uso | Padrões de composição | Quebras de grade |
|---|---|---|---|---|
| ★★☆☆☆ | 1 | 5 | 2 | 0 |
| ★★★☆☆ | 2 | 7 | 3 | 1 |
| ★★★★☆ | 2 | 9 | 4 | 2–3 |
| ★★★★★ | 2 (+1 mono) | 11 | 5 | 3–5, uma estrutural |

## Anti-patterns

- **Não centralize três seções seguidas** — sem âncora à esquerda o olho refaz a busca do início de
  linha a cada bloco, e a página lê como slides, não como documento.
- **Não use card, borda ou sombra para separar itens já a 32px+** — a borda cria uma segunda
  referência de alinhamento que compete com a grade.
- **Não empilhe sombra + borda + fundo no mesmo elemento** — três marcadores para uma separação.
  Escolha um: sombra = profundidade, borda = recorte, fundo = agrupamento.
- **Não meça contraste sobre gradiente num ponto só** — a razão muda ao longo da linha e a ponta
  clara reprova sozinha; meça nos dois extremos.
- **Não passe de 2 famílias ou 3 pesos** — a 16px ninguém distingue 500 de 600: o peso extra não cria
  nível, só entra no bundle.
- **Não defina `font-size` em `vw` puro** — abaixo de 360px o texto some e o zoom não recupera.
- **Não use `max-width` em px para corpo** — 720px a 18px dá 80+ch. Use `ch`.
- **Não use `text-align: justify`** — sem hifenização o navegador abre rios de espaço branco.
- **Não use opacidade <60% em texto sobre foto** — sobre o scrim de ink a 0.86, 50% dá 4.05:1.
- **Não use cinza puro numa paleta quente** — contraste simultâneo o puxa para o verde.
- **Não pinte área grande com o acento** — acima de 15% do viewport ele para de sinalizar ação e o
  CTA no mesmo tom desaparece.
- **Não peça ênfase animando `font-size`, `letter-spacing` ou `margin`** — cada frame refaz o layout
  de todas as line boxes na main thread. Se couber em `transform`/`opacity`, use isso; se for
  tracking de verdade, que não tem equivalente no compositor, troque o efeito.

## Verificação

- [ ] `grep -Rn -- "--font-\|font-family" src/styles/` retorna no máximo 2 famílias declaradas.
- [ ] `grep -Rn "text-\[" src/` — cada ocorrência é exceção justificada, não um degrau novo.
- [ ] `document.documentElement.style.filter = 'grayscale(1)'`: os três níveis ainda se distinguem.
      Se não, a hierarquia estava apoiada em cor.
- [ ] DevTools → Elements → Accessibility: todo par texto/fundo ≥4.5:1 (corpo) e ≥3:1 (≥24px). Sobre
      foto, meça com o scrim aplicado e no quadro mais claro do vídeo, não no poster.
- [ ] Medida do corpo com o snippet de [tokens.md](tokens.md#medir-1ch-da-família-real): 60–75ch no
      desktop, 35–45ch a 360px.
- [ ] `git diff src/styles/index.css` não altera nenhum `clamp()` já publicado.
- [ ] Nenhum padrão de composição repetido em seções consecutivas; ≥4 padrões distintos em 6.
- [ ] No máximo um elemento em cor de acento por dobra; todo espaçamento num degrau da escala 4px.

Zoom, alvos de toque, reflow em 320px e 844×390 são de
[audit-responsivo](../audit-responsivo/SKILL.md); ordem de foco, `focus-visible` e nome acessível são
de [audit-acessibilidade](../audit-acessibilidade/SKILL.md). Não duplique os limites aqui: eles
divergem de propósito (44×44 com 24px de folga é regra de lá) e duas fontes de verdade produzem uma
auditoria que aprova o que a outra reprova.
