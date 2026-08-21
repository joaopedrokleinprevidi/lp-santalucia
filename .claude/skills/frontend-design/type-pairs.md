# Pares tipográficos

Catálogo estendido, matriz de pareamento e o procedimento de verificação de português.
As regras de escolha estão na [seção 4 do SKILL.md](SKILL.md).

## Matriz de pareamento

O par funciona quando as duas famílias são de **classes diferentes**. Dentro da mesma classe,
a diferença lê como erro de importação.

| Display | Corpo que combina | Corpo que briga |
|---|---|---|
| Serif de alto contraste (Cormorant, Playfair, Bodoni) | grotesca neutra (Inter, Work Sans) | outra serif |
| Serif quente/torta (Fraunces, Newsreader) | grotesca técnica (IBM Plex Sans) | serif quente |
| Grotesca de display (Archivo, Bricolage, Syne) | serif de leitura (Newsreader, Lora) ou grotesca neutra de altura-x diferente | outra grotesca de largura parecida |
| Didone extremo (Bodoni Moda) | grotesca de altura-x alta, peso 400 | qualquer coisa com contraste |

Exceção legítima ao "duas classes": **uma família em dois tamanhos ópticos**. Fraunces com
`opsz` alto no display e `opsz` baixo no corpo é um par, não uma repetição — os desenhos são
diferentes de verdade.

## Catálogo — display

| Família | Caráter | Direção estética | Pacote |
|---|---|---|---|
| Cormorant Garamond | contraste alto, hastes finas, ar aristocrático | luxo silencioso, editorial | `@fontsource/cormorant-garamond` |
| Playfair Display | didone, terminais em gota | moda, beleza, revista | `@fontsource-variable/playfair-display` |
| Bodoni Moda | didone no limite, tamanho óptico | capa, título único gigante | `@fontsource-variable/bodoni-moda` |
| Instrument Serif | peso único (400), italiana moderna, sidebearings largas | hero de uma linha | `@fontsource/instrument-serif` |
| Fraunces | eixos `SOFT` e `WONK`, quente e levemente torta | artesanal, orgânico, comida | `@fontsource-variable/fraunces` |
| Newsreader | serif de tela com `opsz`, boa em corpo e display | editorial de leitura longa | `@fontsource-variable/newsreader` |
| Lora | serif de contraste médio, calorosa, muito legível | conteúdo, saúde, confiança | `@fontsource-variable/lora` |
| Bricolage Grotesque | contraforma estranha, largura variável | tecnologia com voz, cultura | `@fontsource-variable/bricolage-grotesque` |
| Archivo | grotesca de sinalização, aguenta 900 e caixa alta enorme | brutalista, marquee, stats | `@fontsource-variable/archivo` |
| Syne | geométrica excêntrica, larguras irregulares entre pesos | arte, cultura, evento | `@fontsource-variable/syne` |
| DM Serif Display | serif de display sólida, peso único, sem frescura | headline curta com muito contraste de tamanho | `@fontsource/dm-serif-display` |
| Libre Baskerville | transicional clássica, altura-x alta | institucional, jurídico, tradicional | `@fontsource/libre-baskerville` |

## Catálogo — corpo

| Família | Caráter | Pacote |
|---|---|---|
| Inter | neutra de UI, altura-x alta, ótima em 14–18px | `@fontsource-variable/inter` |
| IBM Plex Sans | neutra com detalhe (`a`, `l`), levemente técnica | `@fontsource-variable/ibm-plex-sans` |
| Work Sans | grotesca de leitura, mais quente que Inter | `@fontsource-variable/work-sans` |
| Public Sans | neutra institucional, sem personalidade — de propósito | `@fontsource-variable/public-sans` |
| Figtree | geométrica amigável, boa em CTA e label | `@fontsource-variable/figtree` |
| Manrope | geométrica com terminais cortados, boa em números | `@fontsource-variable/manrope` |
| Newsreader | serif de tela para parágrafo longo sob display sans | `@fontsource-variable/newsreader` |
| Source Sans 3 | humanista, excelente em texto denso | `@fontsource-variable/source-sans-3` |

## Catálogo — mono (só com função)

Números tabulares em tabela de preço, código, etiquetas de dado. Nunca como display "porque
parece técnico".

| Família | Pacote |
|---|---|
| JetBrains Mono | `@fontsource-variable/jetbrains-mono` |
| IBM Plex Mono | `@fontsource/ibm-plex-mono` |
| Space Mono | `@fontsource/space-mono` |

## Pares já validados

| Direção | Display | Corpo | Onde |
|---|---|---|---|
| Luxo silencioso / beleza | Cormorant Garamond 400/600/700 | Inter Variable | este projeto (`src/styles/index.css`) |
| Editorial contemporâneo | Bricolage Grotesque Variable | Newsreader Variable | — |
| Brutalista | Archivo Variable 700–900 | Public Sans Variable | — |
| Artesanal | Fraunces Variable (`SOFT` alto) | IBM Plex Sans Variable | — |
| Moda / didone | Playfair Display Variable | Work Sans Variable | — |

**Não repita o par do último projeto por conforto.** Isso é o mesmo mecanismo do slop com um
default privado. Cada marca refaz a escolha.

## Verificar o pacote antes de importar

Nem toda família do Google tem build variável no Fontsource, e o nome da família CSS de um
pacote variável **não é** o nome do pacote:

```bash
npm view @fontsource-variable/fraunces version
npm i @fontsource-variable/fraunces
# a família CSS declarada é "Fraunces Variable", não "Fraunces"
grep -o "font-family:[^;]*" node_modules/@fontsource-variable/fraunces/index.css | head -1
# subsets disponíveis (latin cobre U+00C0–U+00FF, que é o que o português precisa)
ls node_modules/@fontsource-variable/fraunces/files | head
```

Errar o nome da família é silencioso: a página renderiza inteira na fallback e parece só um
pouco pior. Confirme no DevTools → Elements → Computed → `font-family` renderizada.

## Português: o procedimento completo

O bloco necessário é U+00C0–U+00FF: `á â ã à é ê í ó ô õ ú ü ç` mais as maiúsculas
`Á Â Ã À É Ê Í Ó Ô Õ Ú Ç`. O subset `latin` cobre isso. O que quebra na prática:

### 1. Cobertura declarada

```js
document.fonts.check("700 4rem 'Fraunces Variable'", 'ÁÂÃÀÉÊÍÓÔÕÚÜÇáâãàéêíóôõúüç')
```

Responde pelo `unicode-range` do `@font-face`. Não sabe se o glifo existe dentro do arquivo —
por isso o passo 2 é obrigatório.

### 2. Cobertura real

Renderize numa headline real, no tamanho real, com a família aplicada:

```
AÇÃO ÓTIMA — coração, não, você, três
```

Procure por: glifo faltando (retângulo), e principalmente **troca de família no meio da
palavra** — o `Ç` de `AÇÃO` numa serif diferente do resto é o sintoma de fallback parcial.
Famílias de caixa alta única e boa parte das foundries independentes enviam ASCII e nada mais.

### 3. Acento clipado por line-height

`line-height: 0.9` num serif de display corta o til de `Ã` e a cedilha de `Ç`.

- Piso para display acentuado: **1.02**. O valor por degrau (`--text-display` 1.02,
  `--text-hero` 1.04) é publicado por
  [product-design-expert](../product-design-expert/SKILL.md#1-escala-tipográfica) — não o
  redeclare aqui, e não baixe a banda dele para o fundo quando a copy tem acento.
- Quanto maior o contraste da família, mais alto o acento sobe e mais folga ela pede. Uma
  Bodoni Moda pede o teto da banda; uma Lora se resolve no piso.

### 4. Acento clipado pela máscara de reveal

`.word` em `src/styles/index.css` usa `overflow: hidden` para o reveal por linha, e compensa
descendentes com `padding-bottom: 0.16em; margin-bottom: -0.16em`. O topo não está compensado —
com display acentuado o til desaparece durante a animação:

```css
.word {
  padding-top: 0.12em;
  margin-top: -0.12em;
}
```

Teste rolando devagar com `AÇÃO` numa headline, não com a página parada: o clipe só aparece
enquanto a linha está viajando.

### 5. Caixa alta acentuada precisa de mais entrelinha

Em marquee ou eyebrow em caixa alta, `Ã` e `Õ` ocupam acima da altura de capitular. Se o
elemento tem `overflow: hidden`, adicione `padding-top: 0.08em` ou suba o `line-height` para
1.1. Um marquee que corta o til lê como bug, não como estilo.

## Carregamento e custo

O teto de 3 pesos por família e `font-synthesis-weight: none` são regras do
product-design-expert; aqui fica só o preço de cada escolha.

- Cada peso estático pesa ~25–45 KB em woff2 no subset latin. Três pesos de display são
  ~75–135 KB antes de o primeiro texto aparecer.
- Importe pesos, não a família: `@import '@fontsource/cormorant-garamond/600.css'`. O
  `@import '@fontsource/cormorant-garamond'` puxa todos os pesos e todos os subsets.
- **Variável compensa a partir de 3 pesos usados** ou quando o eixo é animado. Abaixo disso o
  arquivo estático é menor.
- **`typographyReady()`** (`src/lib/gsap.ts`) espera `document.fonts.ready` com teto de 1600ms
  antes de medir linhas. Um display pesado que chega depois disso faz `lineGroups()` medir as
  quebras da fallback, e o reveal anima linhas que não existem mais. Se o display passa de
  ~120 KB, pré-carregue:

```html
<link rel="preload" as="font" type="font/woff2" crossorigin
      href="/assets/cormorant-garamond-latin-600-normal.woff2" />
```

## Ajustes ópticos

As bandas de `letter-spacing` e `line-height` por tamanho são do
[product-design-expert](../product-design-expert/SKILL.md) (seção 1) — use as de lá, não invente
outras aqui. O que muda por **família** é isto:

| Situação | Ajuste |
|---|---|
| Serif com `opsz` (Fraunces, Newsreader, Bodoni Moda) | deixe `font-optical-sizing: auto` — é o default e faz o trabalho; o desenho já compensa o tamanho |
| Didone de contraste extremo (Bodoni Moda, Playfair) em display | fique no piso da banda de tracking negativo, não no teto: as hastes finas somem antes das grossas quando as letras encostam |
| Grotesca de largura variável (Bricolage, Syne) | tracking 0 no display — o desenho já varia a largura; somar tracking desmancha o ritmo que é o caráter dela |
| Fraunces | `SOFT` e `WONK` são o caráter. `WONK: 1` só em display; no corpo a itálica torta cansa |
| Aspas e travessão | `"` e `-` são sinal de descuido. Use `"…"` e `—` |
