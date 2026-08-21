---
name: frontend-design
description: Use when choosing the visual character of a page or fixing one that reads as templated — font pairing, texture, atmosphere, colour proportion inside a fixed brand palette. Fase 10b — parece template, esta generico, sem personalidade, AI slop, direcao estetica, tom, escolha de fonte, par tipografico, grain, ruido, gradiente mesh, sombra tingida, proporcao de area entre as cores, zona de cor, acento cortado em portugues.
argument-hint: [secao-ou-pagina]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

# Frontend Design — caráter e diferenciação

| | |
|---|---|
| **ENTRADA** | `src/styles/index.css` com o `@theme` já publicado por `product-design-expert`; `design/design-system.json` (hexes travados, classe tipográfica observada, motifs); `design/creative-direction.json` (Experience Score, capítulo dono de cada WOW); `design/renders/` |
| **SAÍDA** | as quatro respostas da seção 1 escritas em comentário acima do `@theme`, mais a sequência de zonas de cor; `--font-display` e `--font-sans` escolhidas, verificadas em pt-BR e importadas; a camada de textura implementada em `src/styles/index.css` |
| **ANTES** | `product-design-expert` (Fase 10a) — degraus, grade, espaço e paleta já existem |
| **DEPOIS** | `landing-motion-expert` (Fase 10c) — a linguagem de motion se apoia no caráter já fixado |

Esta skill decide **o caráter**: o que torna a página reconhecível e o que a impede de parecer
gerada. Não decide o sistema. Se a pergunta é "qual o valor", é de outra skill. Se é "qual o
caráter", é desta.

| Assunto | Dono |
|---|---|
| Escala tipográfica, espaçamento, grid, hierarquia, contraste, densidade | [product-design-expert](../product-design-expert/SKILL.md) |
| Quais hexes a marca tem | [brand-dna-extractor](../brand-dna-extractor/SKILL.md) |
| **Qual família, quanta textura, qual composição, como a cor se distribui** | **esta skill** |
| Duração, ease, stagger, distância de deslocamento | [landing-motion-expert](../landing-motion-expert/SKILL.md) |
| Pin, scrub, parallax, disparo | [gsap-scrolltrigger-expert](../gsap-scrolltrigger-expert/SKILL.md) |
| Frame sequence em canvas | [video-to-website](../video-to-website/SKILL.md) |
| Creative Budget e quais WOW moments a página ganha | [creative-direction-expert](../creative-direction-expert/SKILL.md) |
| A rubrica do Experience Score | `CLAUDE.md` |

## 1. A decisão estética vem antes do código

Quatro perguntas. Cada uma produz um artefato escrito. Nenhuma linha de CSS antes das quatro.

| Pergunta | Artefato que sai | Onde ele vive |
|---|---|---|
| **Propósito** — quem chega, em que estado, e o que precisa acontecer? | uma frase sem adjetivo: "mulher de 30–55 decidindo se agenda; precisa ver o resultado antes do preço" | comentário no topo do capítulo, ou `src/data/site.ts` |
| **Tom** — qual extremo? | nome da direção + 3 substantivos concretos: "editorial: papel, filete, silêncio" | comentário acima do `@theme` em `src/styles/index.css` |
| **Restrição** — o que a marca já fixou? | lista fechada: hexes, logo, tipo obrigatório. Tudo fora da lista é livre | tokens no `@theme` |
| **Diferenciação** — o que a pessoa lembra 10 minutos depois? | uma frase com o momento e o capítulo onde ele acontece | o Major WOW do `CLAUDE.md` |

A regra dura: **sem as quatro respostas escritas, o resultado converge para o default do
framework.** Isso não é azar — é o que sobra quando nenhuma decisão foi tomada: `font-sans`,
`rounded-lg`, `shadow-md`, `border-gray-200`. O "AI slop" é a ausência de escolha tornada visível.

### Escolher o tom

Escolha uma direção e execute inteira; misturar duas produz o mesmo cinza de não ter escolhido
nenhuma.

| Direção | O que ela fixa em concreto | Como ela falha |
|---|---|---|
| Editorial | serif de alto contraste no display, grotesca no corpo, numeração de seção, filete de 1px, coluna quebrada por imagem sangrando | vira "blog" se o corpo também for serif |
| Brutalista | grotesca 800–900 em 10–15vw, `border-radius: 0`, sombra zero, hairline de 1px na tinta, contraste ≥12:1 sem meio-tom, cortes secos (`duration: 0.2, ease: none`) | vira "template de agência" acima de 2 cores |
| Luxo silencioso | 90% da área numa cor só, accent <3%, caixa alta com tracking ≥0.2em, transições ≥0.9s, vinheta em vez de borda | vira "vazio" se a densidade nunca subir em nenhum capítulo |
| Art déco / salão | simetria central deliberada, didone, small caps, filete metálico de 1px, escala 1.02–1.06 no reveal | vira "casamento" se o ouro virar preenchimento em vez de filete |
| Orgânico / material | grain visível no fundo, cantos irregulares via `clip-path`, fotografia sangrando, sem contêiner | vira "artesanal amador" se a tipografia também for irregular |

## 2. Marca fixa, direção livre

A paleta é **entrada, não saída**. Direção criativa não é inventar cor — é composição,
tipografia, movimento e textura. As quatro seguem livres com os hexes travados.

| Variável | Livre? | Quem manda |
|---|---|---|
| Hex das cores, logo, nome, grafia | Não | manual da marca |
| **Proporção de área entre as cores** | **Sim** | esta skill |
| **Família tipográfica** (salvo tipo obrigatório) | **Sim** | esta skill |
| **Composição**: alinhamento, assimetria, sangria, densidade | **Sim** | esta skill |
| **Textura**: grain, malha, filete, sombra, vinheta | **Sim** | esta skill |
| **Movimento**: duração, ease, direção, o que entra primeiro | **Sim** | landing-motion + gsap |

A prova, com roxo + amarelo e sem mudar um dígito de hex: **editorial** põe o roxo como tinta
sobre papel quase-branco e o amarelo em <2% da área, em número de seção e filete;
**brutalista** chapa o roxo em 90% da área com amarelo em caixa alta a 14vw por cima, raio 0 e
corte seco; **art déco** usa roxo profundo com vinheta radial e reduz o amarelo a um filete de
gradiente de 1px, como foil, com didone e fade lento em `scale: 1.04`. Três páginas
irreconhecíveis entre si, mesma marca.

### Distribuição de área é a alavanca principal

`90 / 8 / 2` lê como caro e contido: uma cor domina e o accent é quase invisível. `50 / 30 / 20`
lê como template: nenhuma domina. O teto de 15% de área para o accent é do
[product-design-expert](../product-design-expert/SKILL.md#5-cor) — não o redefina, use-o. O que
esta skill decide é **qual** proporção acima dele a página adota.

Trocar a proporção troca o contraste junto: quando a dominante vira a escura, todo texto que era
tinta sobre papel passa a ser papel sobre tinta. Meça o corpo nos dois estados antes de fechar a
distribuição — 4.5:1 em ambos, ou a direção não é adotável. O script de medição de área está em
[texture.md](texture.md#medir-a-proporção-de-área).

### Quando a marca não definiu um tom

Derive, não invente: `color-mix(in oklab, …)` entre dois hues saturados mantém a croma no meio do
caminho, e **nunca** interpole uma cor de marca até o branco — a faixa lavada do midpoint em sRGB
é literalmente a cor do slop. Receitas e a exceção do `in srgb` estão em
[texture.md](texture.md#derivar-tom-da-marca-sem-inventar-cor).

## 3. AI slop — o padrão, a causa, a substituição

| Padrão | Por que aparece | Por que falha | Em vez disso |
|---|---|---|---|
| Inter/Roboto/system como **display** | é o `font-sans` default do Tailwind e nunca foi escolhido | Inter foi desenhada para UI em 13–16px: contraste baixo, terminais retos. Aos 4rem a headline fica lisa e sem autor | serif de display ou grotesca com contraforma própria. Inter segue ótima **no corpo** — é o que este projeto faz |
| `linear-gradient(135deg,#667eea,#764ba2)` | é o gradiente de todo template desde 2019 | o midpoint em sRGB dessatura para cinza-lavanda | cor chapada da marca; se precisar de degradê, `in oklab`, parando em 60% |
| Card com glassmorphism | esconde a falta de hierarquia atrás de blur | `backdrop-filter` obriga o compositor a reler o backdrop enquanto ele muda — sobre vídeo em scroll isso come o alvo de 60fps | texto direto no fundo. Exceção sancionada: `.glass-panel` (ink 86%, blur 18px), quando há dado para **ler** sobre foto. `.glass-card` (branco 8%) **não** é sancionada — branco translúcido sobre mídia é padrão de template |
| Hero centralizado com blob de gradiente atrás | é o layout que não exige nenhuma decisão | centro + blob é a composição de 100.000 páginas; não sobra nada para o olho descobrir | assimetria: copy no terço externo, mídia sangrando pela borda oposta |
| Ícone dentro de círculo pastel | preenche o vazio de um card de feature | o pastel não pertence a paleta nenhuma — é um tint arbitrário | número da seção em display (`001`), ou o ícone sozinho na tinta da marca |
| "Trusted by" com logos em cinza | é o bloco de prova social pronto | logo a 40% de opacidade não prova nada e vira borrão | um número com fonte (`+2.400 atendimentos desde 2019`) ou uma frase com nome de quem disse |
| `border: 1px solid` cinza em tudo | separa blocos sem pensar em espaçamento | a borda faz o trabalho que o espaço deveria fazer; lê como planilha | espaço (product-design-expert) ou filete com intenção — ver `.rule-gilt` |
| `border-radius: 8px` universal | é `rounded-lg`, o default | raio único em botão, card e imagem é ausência de decisão. Raio é escala | 2–3 raios com papel: `--radius-card: 24px`, `--radius-pill: 999px`, 4px só no focus ring |
| `shadow-md` do Tailwind | é o jeito rápido de "levantar" algo | preto puro sobre fundo quente lê como sujeira e não tem camada de contato | duas camadas tingidas com a tinta da marca — ver a tabela da seção 5 |

Nenhum é proibido por feiura. Todos são proibidos por serem **o que acontece quando ninguém
escolheu.**

## 4. Tipografia: escolher o par

Quantas famílias, quantos pesos e quanto tracking por tamanho é do
[product-design-expert](../product-design-expert/SKILL.md#pareamento-display--corpo). Aqui a
pergunta é **qual** família, e por quê esta e não a vizinha.

A resposta começa pelo contraste de **classe**, não de peso: serif de display + grotesca de corpo,
ou grotesca larga de display + serif de leitura. Duas grotescas diferentes leem como erro de
importação — a diferença entre elas é pequena demais para o leitor atribuir a uma decisão.

| Direção (seção 1) | Display | Corpo |
|---|---|---|
| Editorial | Instrument Serif | Work Sans |
| Brutalista | Archivo 700–900 | Public Sans |
| Luxo silencioso | Cormorant Garamond (é o display deste projeto) | Inter |
| Art déco / salão | Playfair Display ou Bodoni Moda | Work Sans |
| Orgânico / material | Fraunces (`SOFT` alto) | IBM Plex Sans |

Inverter as classes é um par igualmente válido e soa diferente: Bricolage Grotesque + Newsreader
é editorial contemporâneo, não editorial clássico.

Catálogo completo (12 display, 8 corpo, 3 mono), matriz de pareamento, pares validados, nome de
pacote, custo de carregamento e ajustes ópticos por família estão em
[type-pairs.md](type-pairs.md). Nomes de pacote saem de lá — não os digite de memória, e confirme
com `npm view` antes de importar: nem toda família do Google tem build variável no Fontsource.

### Português é um filtro real

O subset `latin` cobre U+00C0–U+00FF, então o arquivo quase nunca é o problema. O que elimina
candidatas é isto:

1. **Display de caixa alta única sem maiúsculas acentuadas.** Famílias no estilo Bebas e boa parte
   das foundries independentes enviam ASCII e nada mais. `AÇÃO` cai para a fallback no meio da
   palavra, e a troca de família dentro de uma palavra é visível. Reprova a família — não é
   ajustável.
2. **Acento clipado pelo `line-height`.** O piso para display acentuado é **1.02**, e quem publica
   o valor por degrau é
   [product-design-expert](../product-design-expert/SKILL.md#1-escala-tipográfica). Quanto maior o
   contraste da família, mais alto o acento sobe e mais folga ela pede — é por isso que a escolha
   da família muda o número, e é por isso que este teste vive aqui.
3. **Máscara de reveal cortando por cima.** `.word` compensa descendentes
   (`padding-bottom: 0.16em; margin-bottom: -0.16em`) mas não o topo — o til some durante o reveal.
   Compense espelhado com `padding-top: 0.12em; margin-top: -0.12em`.

Teste obrigatório, numa headline real e no tamanho real, **rolando** — o clipe da máscara só
aparece enquanto a linha viaja:

```
AÇÃO ÓTIMA — coração, não, você, três
```

Procedimento completo (cobertura declarada por `document.fonts.check`, cobertura real, caixa alta
acentuada em marquee) em [type-pairs.md](type-pairs.md#português-o-procedimento-completo).
`check()` responde pelo `unicode-range` do `@font-face` e não sabe se o glifo existe dentro do
arquivo, então o teste visual nunca é dispensável.

## 5. Textura e atmosfera

Fundo chapado é a segunda assinatura mais forte de página gerada, depois da fonte. Toda camada
tem custo e limite; o código de cada uma está em [texture.md](texture.md).

| Camada | Quando entra | Limite numérico | Custo |
|---|---|---|---|
| Grain SVG | fundo chapado em ≥1 capítulo; é a mais barata e a mais eficaz | tile ≥128px, `opacity` 0.03–0.06 | rasteriza 1 tile, depois só compõe; 0 no scroll |
| Gradiente de malha | zona clara que precisa de profundidade sem foto | 3 blobs, `color-mix` 18–26% | paint puro enquanto os stops não mudam |
| Grade de pontos | dar chão a bloco de dado ou grid | alpha ≤0.16, célula 24px | paint puro |
| Sombra em duas camadas | superfície que precisa tocar o chão | contato 1–2px/alpha 0.04–0.08 + ambiente 24–48px/alpha 0.12–0.24, tingida com `--color-ink` | paint 1×; caro só se a sombra for animada |
| Vinheta / scrim | texto sobre fotografia (`src/components/ui/Scrim.tsx`) | ≥3 stops, tingido com a tinta da marca | paint puro |
| Duotone, filete foil, camadas translúcidas | amarrar fotos de origens diferentes, substituir borda cinza | ver [texture.md](texture.md) | paint estático |
| `backdrop-filter` | um painel com dado para ler sobre mídia | no máximo 1 visível, **nunca** em elemento de viewport inteira | compositor relê o backdrop a cada frame em que ele muda |

**Máximo de duas camadas simultâneas por página.** Grain + malha + grade ao mesmo tempo vira
ruído, não atmosfera.

Três portões, dentro do fluxo:

- **Contraste sob a textura.** Se a camada cobre parágrafo, meça o corpo depois de aplicá-la:
  4.5:1 no ponto **mais claro** do grão e no blob mais escuro do mesh, nunca na média. Se reprova,
  baixe a porcentagem do `color-mix` — não escureça o texto.
- **Nunca anime `background-position` nem os stops** — repinta o retângulo inteiro a cada frame.
  Ponha cada blob num filho absoluto e anime `transform`, como `ChapterFinale.tsx` já faz com a
  keyframe `aurora`.
- **Toda textura animada para sob `prefers-reduced-motion`.** O bloco global de
  `src/styles/index.css` zera `animation-duration`; confirme que a camada nova está coberta.

Não adicione `mix-blend-mode` numa camada de viewport inteira sobre vídeo — o compositor passa a
ler o backdrop e o vídeo perde o caminho de overlay de hardware. Se quiser blend, meça: DevTools →
Rendering → Frame Rendering Stats; abaixo de 55fps durante o scroll, volte para `opacity`.

## 6. Regras de scroll-driven

A mecânica de frame sequence é de
[video-to-website](../video-to-website/SKILL.md); quantos tipos de entrada a página precisa ter é
da rubrica de Experience Score do
[creative-direction-expert](../creative-direction-expert/SKILL.md) (★★★★☆ pede ≥4, ★★★★★ pede ≥6);
pin e scrub são do [gsap-scrolltrigger-expert](../gsap-scrolltrigger-expert/SKILL.md); variedade
de layout é do [product-design-expert](../product-design-expert/SKILL.md); ritmo e stagger são do
[landing-motion-expert](../landing-motion-expert/SKILL.md). Não renegocie nenhum. Sobram três
decisões de caráter.

**Sem cards.** Em página scroll-driven o texto senta direto no fundo. A legibilidade vem de peso
(600+), de `text-shadow` quando necessário (`.text-shadow-cinema`) e de escolher o ponto do vídeo
onde o fundo está limpo. Contêiner só quando há dado para ler, não para agrupar. O eyebrow usa
`--color-body-soft` (#666) sobre canvas ou branco a 70% sobre mídia — não uma opacidade arbitrária
no meio do caminho, que é o que produz o cinza sem dono.

**O teto tipográfico é medido, não decretado.** A tabela de video-to-website manda hero em
`clamp(3.5rem, 12vw, 11rem)` — número tirado de headline curta em inglês com grotesca. Em
português, num serif de display, quem fixa o teto é a palavra mais longa da copy real: por isso
`--text-hero` deste projeto é `clamp(2.125rem, 4.5vw, 4.5rem)`, um quarto do teto sugerido. Pegue
a palavra mais longa que a copy usa de verdade ("exclusivamente", "transformação"), meça contra a
coluna em 1280px e em 375px, e baixe o teto até ela caber sem hífen. Anote o teto medido e a
palavra que o fixou junto ao token — sem isso o próximo passo devolve o número da tabela.

**Zonas de cor.** A sequência do fundo entre capítulos é decisão de caráter: claro → escuro →
accent → claro dá respiro e clímax; claro → claro → claro é a página que ninguém lembra. Fixe a
sequência antes de implementar e escreva-a junto do tom, na seção 1. O código do handoff é do
[landing-motion-expert](../landing-motion-expert/patterns.md) — não o reescreva aqui.

O que esta skill acrescenta é o portão: o handoff troca o fundo debaixo de um texto que **não**
mudou. Meça o corpo nas duas pontas. Se um estado reprova em 4.5:1, tweene a cor do texto na mesma
timeline em vez de aceitar meio capítulo ilegível.

## 7. Checklist "isto parece genérico?"

- [ ] A cor dominante ocupa ≥60% da área e o accent <15%
      ([script](texture.md#medir-a-proporção-de-área)). Três primeiras empatadas = sem direção.
- [ ] `document.documentElement.style.filter = 'grayscale(1)'` — a hierarquia sobrevive. Se some,
      a cor estava carregando a estrutura no lugar de tamanho, peso e espaço.
- [ ] `document.documentElement.style.setProperty('--font-display','Georgia, serif')` — a página
      perde caráter visível. Se ficou igual, o caráter estava só no arquivo da fonte.
- [ ] `grep -rn "rounded-lg\|shadow-md\|shadow-lg\|border-gray\|slate-\|gray-" src/` volta vazio.
- [ ] Nenhum `h1`/`h2`/`h3` caindo em `--font-sans`; 2 a 3 raios distintos; toda sombra tingida com
      a tinta da marca, nenhuma em `rgb(0 0 0 / …)`.
- [ ] `grep -rn "feTurbulence\|grain\|radial-gradient\|repeating-" src/` — existe ao menos uma
      camada de textura, e no máximo duas simultâneas.
- [ ] Nenhum `backdrop-filter` em elemento de viewport inteira; no máximo um visível por vez.
- [ ] Três capturas de capítulos consecutivos: nenhum padrão de composição repetido entre vizinhos;
      ao menos um elemento sangra pela borda; nenhum hero centralizado com blob atrás.
- [ ] `AÇÃO ÓTIMA — coração, não, você, três` na headline real, no tamanho real, rolando: sem troca
      de família no meio da palavra, sem til cortado. `line-height` do display ≥1.02.
- [ ] Grain e malha não derrubam o corpo abaixo de 4.5:1, medido no ponto mais claro da textura.
- [ ] Nas duas pontas de cada handoff de zona, o corpo passa em 4.5:1. `#666`
      (`--color-body-soft`) é o texto mais claro aceitável sobre canvas.
- [ ] Toda animação de textura para sob `prefers-reduced-motion`.

## Anti-patterns

- **Escolher a paleta antes do propósito** — a cor vira decoração e nada mais decorre dela.
- **Trocar o hex da marca para "melhorar a estética"** — a página deixa de ser da marca; a direção
  mora na proporção, na família, na composição e na textura.
- **Duas famílias da mesma classe** — duas grotescas ou duas serifs leem como erro de importação.
- **Textura por cima de tudo** — grain sobre corpo derruba contraste; a camada de atmosfera fica
  atrás do conteúdo ou em `opacity` ≤0.06.
- **Animar `background-position` de um mesh** — repinta o retângulo inteiro por frame; mova um
  filho com `transform`.
- **`backdrop-filter` em camada de viewport inteira** — o compositor relê o backdrop enquanto ele
  muda; sobre vídeo em scroll isso custa o alvo de 60fps.
- **Gradiente de cor de marca até o branco** — o midpoint em sRGB atravessa uma faixa lavada que é
  exatamente a cor do slop. Interpole `in oklab` e pare em 60%.
- **Copiar a direção do último projeto** — é o mesmo mecanismo do slop, com um default privado.
  Cada marca refaz as quatro perguntas da seção 1.
