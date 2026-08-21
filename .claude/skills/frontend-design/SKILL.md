---
name: frontend-design
description: Use when choosing the visual character of a page or fixing one that reads as templated — font pairing, texture, atmosphere, composition, "this looks like AI slop", "parece template", "está genérico", "sem personalidade", direção estética, identidade visual, escolha de fonte, grain, ruído, gradiente, paleta da marca. Covers the aesthetic decision that precedes code, distinctive type pairs with full Portuguese accent support, and how to be unmistakable inside a fixed brand palette.
argument-hint: [secao-ou-pagina]
---

# Frontend Design — caráter e diferenciação

Esta skill decide **o caráter** de uma página: o que a torna reconhecível e o que a impede de
parecer gerada. Não decide o sistema.

| Assunto | Dono |
|---|---|
| Escala tipográfica, espaçamento, grid, hierarquia, contraste, densidade | [product-design-expert](../product-design-expert/SKILL.md) |
| Quais hexes a marca tem | [brand-dna-extractor](../brand-dna-extractor/SKILL.md) |
| Qual família, quanta textura, qual composição, como a cor se distribui | esta skill |
| Duração, ease, stagger, distância de deslocamento | [landing-motion-expert](../landing-motion-expert/SKILL.md) |
| Pin, scrub, parallax, disparo | [gsap-scrolltrigger-expert](../gsap-scrolltrigger-expert/SKILL.md) |
| Frame sequence em canvas | [video-to-website](../video-to-website/SKILL.md) |
| Creative Budget e quais WOW moments a página ganha | [creative-direction-expert](../creative-direction-expert/SKILL.md) |
| A rubrica do Experience Score | `CLAUDE.md` |

Se a pergunta é "qual o valor", é da outra skill. Se é "qual o caráter", é desta.

---

## 1. A decisão estética vem antes do código

Quatro perguntas. Cada uma produz um artefato escrito. Nenhuma linha de CSS antes das quatro.

| Pergunta | Artefato que sai | Onde ele vive |
|---|---|---|
| **Propósito** — quem chega, em que estado, e o que precisa acontecer? | Uma frase sem adjetivo: "mulher de 30–55 decidindo se agenda; precisa ver o resultado antes do preço." | comentário no topo do capítulo, ou `src/data/site.ts` |
| **Tom** — qual extremo? | Nome da direção + 3 substantivos concretos: "editorial: papel, filete, silêncio" | comentário acima do `@theme` em `src/styles/index.css` |
| **Restrição** — o que a marca já fixou? | Lista fechada: hexes, logo, tipo obrigatório. Tudo fora da lista é livre | tokens no `@theme` |
| **Diferenciação** — o que a pessoa lembra 10 minutos depois? | Uma frase descrevendo o momento e o capítulo onde ele acontece | o Major WOW do `CLAUDE.md` |

A regra dura: **sem as quatro respostas escritas, o resultado converge para o default do
framework.** Isso não é tendência nem azar — é o que sobra quando nenhuma decisão foi tomada:
`font-sans`, `rounded-lg`, `shadow-md`, `border-gray-200`. O "AI slop" é a ausência de escolha
tornada visível.

### Escolher o tom

Cada direção fixa coisas concretas. Escolha uma e execute inteira; misturar duas produz o
mesmo cinza de não ter escolhido nenhuma.

| Direção | O que ela fixa em concreto | Como ela falha |
|---|---|---|
| Editorial | serif de alto contraste no display, grotesca no corpo, numeração de seção, filete de 1px, coluna quebrada por imagem sangrando | vira "blog" se o corpo também for serif |
| Brutalista | grotesca de peso 800–900 em 10–15vw, `border-radius: 0`, sombra zero, hairline de 1px na tinta, contraste ≥ 12:1 (sem meio-tom entre fundo e texto), cortes secos (`duration: 0.2, ease: none`) | vira "template de agência" acima de 2 cores |
| Luxo silencioso | 90% da área numa cor só, accent < 3%, caixa alta com tracking ≥ 0.2em, transições ≥ 0.9s, vinheta em vez de borda | vira "vazio" se a densidade nunca subir em nenhum capítulo |
| Art déco / salão | simetria central deliberada, didone, small caps, filete metálico de 1px, escala 1.02–1.06 no reveal | vira "casamento" se o ouro virar preenchimento em vez de filete |
| Orgânico / material | fundo com grain visível, cantos irregulares via `clip-path`, fotografia sangrando, sem contêiner | vira "artesanal amador" se a tipografia também for irregular |

---

## 2. Marca fixa, direção livre — resolvendo a tensão

A regra geral de direção estética é "escolha um extremo e nunca convirja para o meio". Aqui as
landings são de marcas reais com identidade fechada: roxo e amarelo da Santa Lucia, `#32151e` e
`#e95d79` da Beleza Completa. Parece que a regra não cabe. Cabe — desde que se entenda onde a
direção mora.

**A paleta é entrada, não saída.** Direção criativa não é inventar cor. Direção criativa é
composição, tipografia, movimento e textura. Essas quatro são livres mesmo com os hexes travados.

| Variável | Livre? | Quem manda |
|---|---|---|
| Hex das cores | Não | manual da marca |
| Logo, nome, grafia | Não | manual da marca |
| **Proporção de área entre as cores** | **Sim** | esta skill |
| **Família tipográfica** (salvo tipo obrigatório) | **Sim** | esta skill |
| **Composição**: alinhamento, assimetria, sangria, densidade | **Sim** | esta skill |
| **Movimento**: duração, ease, direção, o que entra primeiro | **Sim** | landing-motion + gsap |
| **Textura**: grain, malha, filete, sombra, vinheta | **Sim** | esta skill |

### A prova: três páginas, os mesmos dois hexes

Roxo + amarelo, sem mudar um dígito de hex:

- **Editorial** — roxo como tinta de texto sobre papel quase-branco; amarelo só em número de
  seção e filete de 1px (menos de 2% da área). Serif de alto contraste. Reveal por linha subindo
  de máscara.
- **Brutalista** — roxo chapado em 90% da área; amarelo em caixa alta a 14vw por cima. Raio 0,
  sombra 0. Grotesca em 900. Corte seco entre seções, marquee horizontal.
- **Salão art déco** — roxo profundo com vinheta radial; amarelo reduzido a um filete de
  gradiente de 1px, como foil. Simetria central. Didone com small caps. Fade lento com `scale: 1.04`.

Três páginas irreconhecíveis entre si. Mesma marca. A direção estava na proporção, na família,
na composição e no movimento — nunca no hex.

### Distribuição de área é a alavanca principal

É o que mais muda o caráter e é sempre livre. Duas distribuições da mesma paleta:

- `90 / 8 / 2` — uma cor domina, accent quase invisível. Lê como caro e contido.
- `50 / 30 / 20` — nenhuma domina. Lê como template.

O teto de 15% de área para o accent é do
[product-design-expert](../product-design-expert/SKILL.md#5-cor) — não o redefina aqui, use-o. O
que esta skill decide é **qual** proporção acima dele a página adota, e isso muda o caráter mais
do que qualquer outra alavanca. O script de medição está no checklist da seção 7.

Trocar a proporção troca o contraste junto: quando a dominante vira a escura, todo texto que era
tinta sobre papel passa a ser papel sobre tinta. Meça o corpo nos dois estados antes de fechar a
distribuição — 4.5:1 em ambos, ou a direção não é adotável.

### Quando a marca não definiu um tom

Derive, não invente. Entre **dois hues saturados**, `color-mix`/gradiente em `oklab` mantém a
croma no meio do caminho e em `srgb` o midpoint dessatura — por isso a interpolação de marca é
sempre `in oklab`. Misturar uma cor com `transparent` é o caso barato: os dois espaços entregam
quase a mesma rampa, e é por isso que `.rule-gilt` no projeto usa `in srgb` sem prejuízo.

```css
/* tint da marca — não é uma cor nova, é a mesma cor com menos presença */
background-color: color-mix(in oklab, var(--color-rose) 12%, transparent);
border-color: color-mix(in oklab, var(--color-ink) 18%, transparent);
```

E o inverso do erro clássico: **nunca crie `linear-gradient(180deg, var(--brand), white)`.**
Interpolar uma cor saturada até o branco em sRGB atravessa uma faixa lavada — que é exatamente
a cor do slop. Se precisar de degradê da marca, interpole `in oklab` e pare em 60%, sem chegar
ao branco.

---

## 3. AI slop — o padrão, a causa, a substituição

| Padrão | Por que aparece | Por que falha | Em vez disso |
|---|---|---|---|
| Inter/Roboto/system como **display** | é o `font-sans` default do Tailwind e nunca foi escolhido | Inter foi desenhada para UI em 13–16px: contraste baixo, terminais retos, sem voz. Aos 4rem a headline fica lisa e sem autor | serif de display ou grotesca com contraforma própria. Inter segue ótima **no corpo** — é o que este projeto faz |
| `linear-gradient(135deg,#667eea,#764ba2)` sobre branco | é o gradiente de todo template desde 2019 | o midpoint em sRGB dessatura para cinza-lavanda; é literalmente a cor de "gerado" | cor chapada da marca; se precisar de degradê, `in oklab`, parando em 60% |
| Card com glassmorphism | esconde a falta de hierarquia atrás de blur | `backdrop-filter` obriga o compositor a reler o backdrop enquanto ele muda — sobre vídeo em scroll isso come o alvo de 60fps. E o texto ganha um contêiner que não precisava | texto direto no fundo; hierarquia por tamanho/peso/cor. Exceção sancionada: `.glass-panel` (ink 86%, blur 18px) — dado que alguém precisa **ler** sobre fotografia. `.glass-card` (branco 8%, blur 14px, usado em `ChapterExperience.tsx`) é a exceção **não** sancionada: branco translúcido sobre mídia é o padrão de template. Troque por `.glass-panel` ou por texto direto com `.text-shadow-cinema` |
| Hero centralizado com blob de gradiente atrás | é o layout que não exige nenhuma decisão | centro + blob é a composição de 100.000 páginas; não sobra nada para o olho descobrir | assimetria: copy no terço externo, mídia sangrando pela borda oposta |
| Ícone dentro de círculo pastel | preenche o vazio de um card de feature | o círculo pastel não pertence a paleta nenhuma — é um tint arbitrário de uma cor que já existe | número da seção em display (`001`), ou o ícone sozinho na tinta da marca, sem contêiner |
| "Trusted by" com logos em cinza | é o bloco de prova social pronto | logo dessaturado a 40% de opacidade não prova nada, e todos viram o mesmo borrão | um número com fonte (`+2.400 atendimentos desde 2019`) ou uma frase com nome de quem disse |
| `border: 1px solid` cinza-clara em tudo | separa blocos sem pensar em espaçamento | a borda faz o trabalho que o espaço deveria fazer; o resultado lê como planilha | espaço (sistema do product-design-expert) ou filete com intenção — ver `.rule-gilt` |
| `border-radius: 8px` universal | é `rounded-lg`, o default | um raio único em botão, card e imagem é ausência de decisão. Raio é escala: quanto maior a superfície, maior o raio | 2–3 raios com papel definido. Aqui: `--radius-card: 24px`, `--radius-pill: 999px`, e 4px só no focus ring |
| `shadow-md` do Tailwind | é o jeito rápido de "levantar" algo | preto puro (`rgb(0 0 0 / 0.1)`) sobre fundo quente lê como sujeira, e o default não tem camada de contato | duas camadas tingidas com a tinta da marca — ver `--shadow-card` na seção 5 |

Nenhum desses padrões é proibido por feiura. Todos são proibidos por serem **o que acontece
quando ninguém escolheu.**

---

## 4. Tipografia: escolher o par

Quantas famílias, quantos pesos e quanto tracking por tamanho é do
[product-design-expert](../product-design-expert/SKILL.md#pareamento-display--corpo). Aqui a
pergunta é outra: **qual** família, e por quê esta e não a vizinha.

A resposta começa pelo contraste de **classe**, não de peso. Serif de display + grotesca de
corpo, ou grotesca larga de display + serif de leitura. Duas grotescas diferentes leem como erro
de importação, não como escolha — a diferença entre elas é pequena demais para o leitor atribuir
a uma decisão.

### Do tom à família

A direção escolhida na seção 1 já elimina quase todas as candidatas. O atalho:

| Direção (seção 1) | Display | Corpo |
|---|---|---|
| Editorial | Instrument Serif | Work Sans |
| Brutalista | Archivo 700–900 | Public Sans |
| Luxo silencioso | Cormorant Garamond (é o display deste projeto) | Inter |
| Art déco / salão | Playfair Display ou Bodoni Moda | Work Sans |
| Orgânico / material | Fraunces (`SOFT` alto) | IBM Plex Sans |

Inverter as classes é um par igualmente válido e soa diferente: grotesca de display + serif de
leitura (Bricolage Grotesque + Newsreader) é editorial contemporâneo, não editorial clássico.

Catálogo completo (12 display, 8 corpo, 3 mono), matriz de pareamento por classe, pares já
validados, custo de carregamento e ajustes ópticos por família estão em
[type-pairs.md](type-pairs.md). Nomes de pacote saem de lá — não os digite de memória.

**Confirme antes de importar** — nem toda família do Google tem build variável no Fontsource:

```bash
npm view @fontsource-variable/fraunces version
```

### Português é um filtro real

O subset `latin` do Google Fonts cobre U+00C0–U+00FF, então o arquivo quase nunca é o problema.
O que elimina candidatas é isto:

1. **Display de caixa alta única sem maiúsculas acentuadas.** Famílias no estilo Bebas e boa
   parte das foundries independentes enviam ASCII e nada mais. `AÇÃO` cai para a fallback no
   meio da palavra, e a troca de família dentro de uma palavra é visível. Isso reprova a
   família — não é ajustável.
2. **Acento clipado pelo `line-height`.** O piso para display acentuado é **1.02**, e quem
   publica o valor por degrau é
   [product-design-expert](../product-design-expert/SKILL.md#1-escala-tipográfica) — aqui fica só
   a causa. Quanto maior o contraste da família, mais alto o acento sobe e mais folga ela pede: é
   por isso que a escolha da família muda o número, e é por isso que este teste vive nesta skill.
3. **Máscara de reveal cortando por cima.** `.word` compensa descendentes
   (`padding-bottom: 0.16em; margin-bottom: -0.16em`) mas não o topo — o til some durante o
   reveal. Compense espelhado com `padding-top: 0.12em; margin-top: -0.12em`.

Teste obrigatório, numa headline real e no tamanho real — não com a página parada, e sim
rolando, porque o clipe da máscara só aparece enquanto a linha viaja:

```
AÇÃO ÓTIMA — coração, não, você, três
```

Procedimento completo (cobertura declarada via `document.fonts.check`, cobertura real, caixa
alta acentuada em marquee) em [type-pairs.md](type-pairs.md). `check()` responde pelo
`unicode-range` do `@font-face` e não sabe se o glifo existe dentro do arquivo, então o teste
visual nunca é dispensável.

### Custo

O teto de 3 pesos por família é do product-design-expert. O que interessa aqui é o preço de
cada escolha:

- Cada peso estático pesa ~25–45 KB em woff2 no subset latin — três pesos de display são
  ~75–135 KB antes de o primeiro texto aparecer.
- Variável compensa a partir de 3 pesos usados ou quando o eixo é animado; abaixo disso o
  estático é menor.
- Importe pesos, não a família inteira: `@import '@fontsource/cormorant-garamond/600.css'`.
- `typographyReady()` em `src/lib/gsap.ts` espera as faces por no máximo 1600ms antes de medir
  as linhas. Um display pesado que chega depois disso faz o reveal medir as quebras da fallback.

---

## 5. Textura e atmosfera

Fundo chapado é a segunda assinatura mais forte de página gerada, depois da fonte. Cada receita
abaixo tem custo medido e um limite.

### Grain — o mais barato e o mais eficaz

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

Custo: o `feTurbulence` é rasterizado **uma vez** num tile de 160×160; depois é uma camada
composta e nada mais. `position: fixed` sobre um capítulo com vídeo deixa o grão parado enquanto
a cena se move — é isso que lê como película.

Limites: tile ≥ 128px (abaixo disso o padrão se repete visivelmente), `opacity` 0.03–0.06 —
acima de 0.06 lê como sujeira na fotografia, não como textura, e o corpo de texto abaixo dela
perde contraste mensurável. Se a camada cobre parágrafo, meça o corpo depois de aplicá-la: 4.5:1
no ponto mais claro do grão, não na média. Não adicione
`mix-blend-mode: overlay` numa camada de viewport inteira sobre vídeo — o compositor passa a ler
o backdrop e o vídeo perde o caminho de overlay de hardware. Se quiser blend, meça: DevTools →
Rendering → Frame Rendering Stats; abaixo de 55fps durante o scroll, volte para `opacity`.

Grain animado (tremor de película) só fora de `prefers-reduced-motion`.

### Gradiente de malha

```css
.mesh {
  background-color: var(--color-canvas-soft);
  background-image:
    radial-gradient(60% 45% at 12% 18%, color-mix(in oklab, var(--color-rose) 26%, transparent) 0%, transparent 70%),
    radial-gradient(50% 40% at 88% 8%, color-mix(in oklab, var(--color-gilt) 22%, transparent) 0%, transparent 65%),
    radial-gradient(70% 60% at 50% 100%, color-mix(in oklab, var(--color-ink) 18%, transparent) 0%, transparent 72%);
}
```

Custo: paint puro, zero durante o scroll enquanto o elemento não muda.

Contraste: um mesh varia o fundo ao longo da seção, então o texto por cima tem contraste
variável. Meça no blob mais escuro **e** no ponto mais claro; se um dos dois reprova, baixe as
porcentagens do `color-mix` até os dois passarem, em vez de escurecer o texto.

**Nunca anime `background-position` ou os stops** — repinta o retângulo inteiro a cada frame.
Para movimento, ponha cada blob num filho absoluto e anime `transform`. É o que
`ChapterFinale.tsx` já faz com a keyframe `aurora` (`translate3d` + `scale`).

### Grade de pontos e filetes

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

Custo: paint puro. Alpha ≤ 0.16 — acima disso a grade compete com o texto em vez de dar chão.

Para régua de colunas, desenhe com `border-inline-start` em elementos reais, não com
`background-size: calc(100% / 12)`: em largura não-inteira as linhas caem em pixel fracionário e
tremem no resize.

### Sombra com intenção

Uma sombra real tem duas partes:

```css
--shadow-card: 0 1px 2px rgb(50 21 30 / 0.04), 0 12px 32px -12px rgb(50 21 30 / 0.16);
```

- **Contato**: 1–2px de blur, alpha 0.04–0.08. É o que faz o objeto tocar a superfície.
- **Ambiente**: blur 24–48px, spread negativo, alpha 0.12–0.24. É o que dá altura.
- **Tinja com a tinta da marca** (`rgb(50 21 30)` = `--color-ink`), nunca preto puro. Preto sobre
  fundo quente lê como sujeira.

Custo: uma sombra estática é paint pago uma vez. Se o elemento anima **só** `transform`, a
camada inteira — sombra junto — é composta e nada repinta, por mais alto que seja o blur. O que
custa é animar a própria sombra (`box-shadow` no hover, blur crescendo): aí o retângulo é
repintado a cada frame, e com blur > 40px isso derruba o alvo de 60fps. Para "levantar" no
hover, cruze duas camadas de sombra com `opacity` em vez de interpolar o blur.

### Mais receitas

Vinheta e scrim sobre fotografia (ver `src/components/ui/Scrim.tsx`), duotone, filete metálico,
transparências em camada, recorte irregular, ruído em canvas e tremor de película estão em
[texture.md](texture.md), cada um com custo e limite. O cursor custom é do
[motion-ui-expert](../motion-ui-expert/SKILL.md).

---

## 6. Regras de scroll-driven

Valem quando a página é uma sequência de capítulos com scroll no comando. A mecânica de frame
sequence é de [video-to-website](../video-to-website/SKILL.md#non-negotiables); quantos tipos de
entrada a página precisa ter é da rubrica de Experience Score do
[creative-direction-expert](../creative-direction-expert/SKILL.md) (★★★★☆ pede ≥4, ★★★★★ pede
≥6); pin e scrub são do
[gsap-scrolltrigger-expert](../gsap-scrolltrigger-expert/SKILL.md). Não repita nem renegocie
nenhum dos três. Variedade de layout é do
[product-design-expert](../product-design-expert/SKILL.md) (seção 6, composição); ritmo e
stagger são do [landing-motion-expert](../landing-motion-expert/SKILL.md). O que segue é só o
caráter — as três decisões que sobram para esta skill.

**Sem cards.** Em página scroll-driven, o texto senta direto no fundo. A legibilidade vem de
peso (600+), de `text-shadow` quando necessário (`.text-shadow-cinema`) e de escolher o ponto do
vídeo onde o fundo está limpo. Contêiner só quando há dado para ler, não para agrupar. O eyebrow
que acompanha esse texto usa `--color-body-soft` (#666) sobre canvas ou branco a 70% sobre
mídia — não uma opacidade arbitrária no meio do caminho, que é o que produz o cinza sem dono.

**O teto tipográfico é medido, não decretado.** A tabela de video-to-website manda hero em
`clamp(3.5rem, 12vw, 11rem)` — número tirado de headline curta em inglês com grotesca. Em
português, num serif de display, quem fixa o teto é a palavra mais longa da copy real, e ela
vence o número da tabela: por isso `--text-hero` deste projeto é `clamp(2.125rem, 4.5vw,
4.5rem)`, um quarto do teto sugerido. Procedimento: pegue a palavra mais longa que a copy usa
de verdade ("exclusivamente", "transformação"), meça contra a coluna em 1280px e em 375px, e
baixe o teto até ela caber sem hífen. Hero que parte uma palavra ao meio é pior que hero menor.
Anote o teto medido e a palavra que o fixou junto ao token — sem isso o próximo passo devolve o
número da tabela.

**Zonas de cor.** O fundo muda de dono entre capítulos, e a sequência é uma decisão de caráter:
claro → escuro → accent → claro dá respiro e clímax; claro → claro → claro é a página que ninguém
lembra. Fixe a sequência antes de implementar e escreva-a junto do tom, na seção 1.

O código do handoff é do
[landing-motion-expert](../landing-motion-expert/patterns.md) (seção 7) — token resolvido para
hex antes do tween, `overwrite`, `gsap.context()` com `revert()` no cleanup. Não o reescreva
aqui.

O que esta skill acrescenta é o portão: o handoff troca o fundo debaixo de um texto que **não**
mudou. Meça o corpo nas duas pontas, clara e escura. Se um dos estados reprova em 4.5:1, tweene
a cor do texto na mesma timeline em vez de aceitar meio capítulo ilegível — e escolha a cor
final pensando nos dois, não só no estado que você tinha na tela quando decidiu.

A tabela de entradas, o stagger e as distâncias de deslocamento são de
[landing-motion-expert](../landing-motion-expert/SKILL.md#linguagem-de-motion), e as receitas
(contador, marquee, parallax, troca de fundo) de
[patterns.md](../landing-motion-expert/patterns.md). A tabela equivalente em
[choreography.md](../video-to-website/choreography.md) **não vale aqui**: ela serve uma página de
canvas frame sequence e os cursos dela são o dobro. Não duplique nenhuma das duas.

---

## 7. Checklist "isto parece genérico?"

Cada item é verificável. Rode no console do navegador com a página aberta, ou no terminal.

**Cor**

```js
// proporção de área por cor de fundo no viewport atual
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

- [ ] A cor dominante ocupa ≥ 60% da área. Se as três primeiras estão empatadas, não há direção.
- [ ] O accent ocupa < 15%. Acima disso deixou de ser accent.

**Hierarquia sem cor**

```js
document.documentElement.style.filter = 'grayscale(1)'
```

- [ ] A hierarquia sobrevive. Se some, a cor estava carregando a estrutura no lugar de
      tamanho, peso e espaço.

**Caráter sem a fonte**

```js
document.documentElement.style.setProperty('--font-display', 'Georgia, serif')
```

- [ ] A página perde caráter visível. Se ficou igual, o caráter estava só no arquivo da fonte —
      não na composição, não na textura, não no movimento.

**Defaults**

```bash
grep -rn "rounded-lg\|shadow-md\|shadow-lg\|border-gray\|slate-\|gray-" src/ | wc -l
grep -rn "font-family\|font-display\|font-sans" src/components/ src/styles/
```

- [ ] Zero utilitário de cor genérica (`gray-`, `slate-`) — todas as cores vêm do `@theme`.
- [ ] Nenhum `h1`/`h2`/`h3` caindo em `--font-sans`.
- [ ] Contagem de raios distintos entre 2 e 3. Um só, e sendo 8px, é ausência de decisão.
- [ ] Toda sombra é tingida com a tinta da marca; nenhuma é `rgb(0 0 0 / …)`.

**Textura**

```bash
grep -rn "feTurbulence\|grain\|radial-gradient\|repeating-" src/
```

- [ ] Existe pelo menos uma camada de textura ou atmosfera na página inteira.
- [ ] Nenhum `backdrop-filter` em elemento de viewport inteira; no máximo um visível por vez.

**Composição**

- [ ] Três capturas de capítulos consecutivos: nenhum padrão de composição repetido entre
      vizinhos (a contagem exata por página é do product-design-expert, seção 6).
- [ ] Ao menos um elemento sangra pela borda ou quebra a coluna.
- [ ] Nenhum hero centralizado com blob atrás.

**Português**

- [ ] `AÇÃO ÓTIMA — coração, não, você, três` renderizado na headline real, no tamanho real,
      sem troca de família no meio da palavra e sem til cortado.
- [ ] `line-height` do display ≥ 1.02.

**Acessibilidade** (bloqueante — a textura não pode custar leitura)

Os portões estão dentro do fluxo, no ponto de cada decisão: a proporção de cor na seção 2, o
grain e o mesh na seção 5, o handoff de zona na seção 6. Isto aqui é só a re-verificação final.

- [ ] Grain e malha não derrubam o contraste do corpo abaixo de 4.5:1. Meça no ponto mais claro
      da textura, não na média.
- [ ] Toda animação de textura para sob `prefers-reduced-motion`.
- [ ] Nas duas pontas de cada handoff de zona de cor, o corpo passa em 4.5:1.
- [ ] `#666` (`--color-body-soft`) é o texto mais claro aceitável sobre canvas.

---

## Anti-patterns

- **Escolher a paleta antes do propósito** — a cor vira decoração e nada mais decorre dela.
- **Trocar o hex da marca para "melhorar a estética"** — a página deixa de ser da marca; a
  direção mora na proporção, na família, na composição e na textura.
- **Duas famílias da mesma classe** — duas grotescas ou duas serifs leem como erro de
  importação, não como par.
- **Textura por cima de tudo** — grain sobre texto de corpo derruba contraste; a camada de
  atmosfera fica atrás do conteúdo ou em `opacity` ≤ 0.06.
- **Animar `background-position` de um mesh** — repinta o retângulo inteiro por frame; mova um
  filho com `transform`.
- **`backdrop-filter` em camada de viewport inteira** — o compositor relê o backdrop enquanto ele
  muda; sobre vídeo em scroll isso custa o alvo de 60fps.
- **Copiar a direção do último projeto** — é o mesmo mecanismo do slop, só que com um default
  privado. Cada marca refaz as quatro perguntas da seção 1.
