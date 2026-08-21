# Motion Prompts — Landing Page Santa Lúcia

> Prompts de animação (Google Flow / Veo) para os stills gerados no GPT.
> Ordem idêntica à do `design/image-prompts.md` e à do `design/landing-blueprint.md`.
> Fonte da verdade de cor, voz e fatos: `design/design-system.json`.
> Regras técnicas: `.claude/skills/video-to-website/SKILL.md` e `decision.md`.

**Convenção de nomes assumida** (o `image-prompts.md` ainda não foi escrito — quando for, tem
que adotar esta convenção ou este arquivo quebra):
`design/stills/NN-<id>.png` → `design/flow/NN-<id>.mp4` → `public/frames/<id>/frame_%04d.webp`

---

## Por que os prompts são tão restritos

Estas restrições não são estilo. Todas vêm da etapa de extração de frames, e cada uma existe
porque quebrá-la produz um defeito visível na página final.

**Clipe de 4 a 6 segundos.** Na taxa nativa do Flow (25 fps), 6 s dá 150 frames — exatamente o
piso da técnica. Abaixo de 150 o movimento estroba ao ser raspado. Acima de ~300 o payload deixa
de valer o que pesa, e para chegar lá é preciso decimar, o que estroba de novo. Seis segundos é o
ponto em que os dois problemas desaparecem ao mesmo tempo.

**UM movimento contínuo, sem corte.** O scroll mapeia linearmente para o índice do frame. Um
corte no meio do clipe vira um salto de imagem no meio da rolagem, e o usuário lê isso como bug,
não como edição. Dois movimentos no mesmo clipe produzem um tranco na emenda. A câmera também
nunca inverte o sentido dentro do mesmo clipe: movimento monotônico ou nada.

**Sem tremor de câmera.** Raspar frame a frame amplifica micro-instabilidade em jitter. O que na
reprodução normal passa como "câmera na mão" vira vibração quando o dedo do usuário controla a
linha do tempo. Onde o briefing pede naturalidade (a caminhada da 04), pede-se *gentle natural
bob*, nunca *handheld*.

**Nunca pedir loop.** Vídeo gerado deriva: o último frame quase nunca encontra o primeiro. Um
loop mal fechado pisca a cada volta. A sequência descansa no último frame e fica lá — por isso
**o último frame de cada clipe é escolhido como imagem**, não como sobra. Ele é o que fica mais
tempo na tela, é o poster de `prefers-reduced-motion` e é o que aparece num screenshot.

**Assunto dentro da área segura.** O canvas desenha com `IMAGE_SCALE = 0.85`, então os ~7,5%
externos de cada borda são preenchimento. Qualquer coisa que importe ali é cortada. Some-se o
crop 9:16 do mobile e a regra fica: **o que carrega a cena vive no terço central**.

**Movimento lento, sempre.** Vídeo gerado não tem motion blur real — cada frame é nítido e
independente. Sob scrub rápido, frames nítidos em sequência estrobam em vez de borrar. Por isso
`FRAME_SPEED` fica em **1.8** (extremo baixo da faixa) e todo prompt pede *slow, deliberate*.

**Sem rosto humano reconhecível em nenhum clipe.** Não temos nomes nem CRMV da equipe
(`unverified` no design system). Mãos, antebraços de jaleco roxo, ombros e silhuetas fora de foco
apenas. Isso está no style anchor, não em cada prompt, justamente para não ser esquecido.

**Sem texto em nenhum frame.** Modelo generativo renderiza texto como garatuja plausível, e
diacrítico português piora. Toda a copy é DOM. Placa, monitor e etiqueta entram sempre no
`EXCLUDE`.

---

## Decisão de técnica — e as três mudanças em relação ao blueprint

A skill impõe teto duro: **no máximo 2 sequências de frames em canvas na página inteira**. O
blueprint autorizava uma só (a 06) mas pedia `<video>` com `currentTime` raspado pelo scroll em
outras seis (01, 02, 04, 07, 08, 11). O `decision.md` classifica `currentTime` como técnica
legada: o iOS Safari não busca frame de vídeo comprimido com precisão e visivelmente treme. Esta
página é de tráfego majoritariamente mobile. Então o pedido do blueprint, executado ao pé da
letra, entregaria tremor para a maioria dos visitantes.

**Mudança 1 — a 04 sobe para canvas frames.** O blueprint já descreve a 04 como "capítulo pinado,
clipe raspado frame a frame pelo scroll" e "o scroll é a caminhada". Essa é a definição literal de
sequência de frames. Só o rótulo da entrega estava errado. Canvas honra a intenção declarada; o
`currentTime` a trairia no aparelho de quem mais importa. É promoção de técnica, não desvio de
direção.

**Mudança 2 — 01, 07 e 11 descem para `<video>` tocado uma vez.** Nas três, o conteúdo é o
*movimento de câmera*, não uma transformação no tempo. Aplicando o teste honesto do `decision.md`
— "se isto fosse um still com uma boa legenda, o que se perderia?" — o que se perde é atmosfera,
e atmosfera é exatamente o caso de uso do loop. Tocam uma vez na entrada (`IntersectionObserver`),
não repetem, e descansam no último frame. Imagem idêntica, um terço do custo de decodificação.

**Mudança 3 — 02 e 08 mantêm scrub, mas só onde ele funciona.** Nessas duas a mudança ao longo do
tempo **é** o conteúdo (o balcão indo de 3h às 15h; o cão indo de molhado a seco), e o blueprint
constrói argumento em cima disso. Ganham `currentTime` raspado em `≥1024px` e tocam uma vez no
mobile. Custo declarado: **a delícia escondida da 08 — rolar para cima molha o cachorro de novo —
existe só no desktop.** Ela não vale um slot de canvas que o MAJOR precisa, e no mobile ela nunca
funcionaria de forma confiável de qualquer jeito.

Os dois slots de canvas ficam com a **04** (pico narrativo) e a **06** (pico técnico), separadas
pela 05, que não tem vídeo nenhum. É exatamente o "duas, se a página for longa e elas estiverem
distantes" que o `decision.md` permite.

---

## STYLE ANCHOR

Colar **verbatim** na primeira linha de todo prompt de movimento, e ser o mesmo do
`image-prompts.md`. Se ele mudar na seção 5, as seções 1–4 deixam de combinar. Muda tudo ou não
muda nada.

```
Professional editorial photography for a Brazilian 24-hour veterinary hospital. Shot on a 35mm
lens at f/2.0, shallow depth of field, natural motion, no color grading and no filter look.
Warm palette: deep violet #482A78 and #2D1542 appear only as painted walls, upholstery and
signage; amber #FCB400 appears only as practical light sources, uniform trim and equipment
accents; cream #F7F4EF as floors, tile and daylight. Clean uncluttered composition with generous
negative space. Real materials — brushed steel, matte paint, glass, fabric. No recognizable human
faces anywhere: hands, forearms in violet scrubs, shoulders, or out-of-focus silhouettes only.
```

---

# 01 · `agora`

| | |
|---|---|
| **Still** | `design/stills/01-agora.png` — fachada à noite, vista da calçada oposta |
| **Técnica** | `loop <video>` — toca uma vez na entrada, descansa no último frame |
| **Duração** | **6 s** |
| **Budget** | 30% · small |

```
[STYLE ANCHOR]

MOTION: one continuous slow forward dolly across a wet asphalt street toward the glass entrance
of a veterinary hospital at night, ending on the sidewalk directly in front of the glass. The
door stays closed for the entire clip.
CAMERA: slow push in, constant speed, locked horizon, tripod-smooth, no handheld float, no tilt
SUBJECT ACTION: none — the street is empty of people and cars. Only a thin blade of amber light
spilling from under the closed glass door onto the wet pavement, faint drizzle catching the
violet sign glow, and the reflection on the asphalt lengthening as the camera approaches.
PACE: slow, deliberate, 6 seconds, single continuous take, the final second almost completely
still
EXCLUDE: cuts, the door opening, people, cars, camera shake, zoom snaps, lens flare bursts, text
overlays, readable signage, speed ramps, rain becoming heavy
```

**Por que este movimento.** O hero cria a pergunta e não gasta a resposta. A porta abre **uma vez
na página inteira**, na 04. Se ela abrir aqui, o gesto do MAJOR já foi queimado aos 11% do scroll
— foi o pior erro da proposta original e é o que este prompt existe para não repetir.

**Teste de aceitação.** A porta está fechada no frame 1 e no último frame. O último frame funciona
sozinho como poster do hero, com espaço negativo no terço inferior para a H1, o telefone em escala
`stat` e o par segmentado. Se a porta abre, se aparece gente, ou se o último frame tem hotspot
brilhante onde a copy vai entrar → regenerar.

---

# 02 · `dois-caminhos`

| | |
|---|---|
| **Still** | `design/stills/02-dois-caminhos.png` — balcão da recepção, frontal, câmera travada |
| **Técnica** | `currentTime` raspado em `≥1024px` · `loop <video>` toca-uma-vez no mobile |
| **Duração** | **6 s** |
| **Budget** | 45% · medium |

```
[STYLE ANCHOR]

MOTION: a single locked-off time-lapse of one veterinary reception desk as the light travels from
3am to 3pm — amber practical light at the start, cool blue window light in the middle, full cream
daylight at the end. The room, the desk and the furniture never change.
CAMERA: completely static, tripod locked, no pan, no push, no parallax, no breathing
SUBJECT ACTION: only light and time — the amber ceiling fixture fades out as daylight climbs the
violet wall behind the desk; occasional soft motion-blurred streaks of people crossing frame,
never resolving into a recognizable figure; a shadow rotating slowly across the cream floor
PACE: slow and even, 6 seconds, one continuous take, light changing at a constant rate with no
jump or flicker
EXCLUDE: cuts, camera movement of any kind, flicker, strobing, recognizable faces, readable
signage, screens with readable content, speed ramps, the room being rearranged
```

**Por que este movimento.** É a tese da página inteira dita em uma imagem: *é a mesma sala, muda o
motivo pelo qual você entra nela*. O scroll controla a hora do dia. Densidade de movimento
baixíssima, densidade de ideia altíssima — e é o segundo clipe mais barato de produzir.

**Nota de gramática.** Esta seção está deliberadamente fora da continuidade de câmera 01 → 04 →
11. Câmera travada e tempo comprimido é outra gramática, e é o que impede a página de virar cinco
planos iguais de alguém andando pela clínica.

**Teste de aceitação.** A mudança de luz é monotônica: nenhum frame é mais escuro que o anterior.
Nenhuma pessoa chega a ser reconhecível em nenhum frame. Móvel nenhum se move. Se a câmera derivar
um pixel que seja, o efeito "mesma sala" morre → regenerar.

---

# 03 · `e-grave`

| | |
|---|---|
| **Still** | `design/stills/03-e-grave.png` — corredor visto de quem acabou de entrar |
| **Técnica** | **still apenas** — sem clipe, sem prompt de Flow |
| **Duração** | — |
| **Budget** | 15% · small por escolha |

**Não gerar clipe.** A seção declara que o WOW é a ausência de movimento; pagar produção de Flow
aqui seria pagar por nada. Em beat 9, no auge do medo, espetáculo é ofensivo — e a economia desta
seção é o que financia a sequência de frames da 06.

**A coreografia é tipográfica, em DOM.** Conforme o scroll avança, a linha ativa da lista de sinais
fica em 100% de opacidade e todas as outras caem para 18%. Ler a lista é ler com lanterna. Na
última linha, tudo acende de uma vez junto com o botão de ligar. Easing `cubic-bezier(0.16, 1,
0.3, 1)`.

**Teste de aceitação.** O still tem profundidade longa e uma faixa central escura o bastante para
a lista em tipo grande passar em contraste AA sem scrim adicional. Nenhuma luminária cai atrás de
uma linha de texto.

---

# 04 · `primeiros-minutos` — ◆ MAJOR

| | |
|---|---|
| **Still** | `design/stills/04-primeiros-minutos.png` — POV do tutor na calçada, diante do vidro fechado |
| **Técnica** | **canvas frames** — `pin` + `scrub`, `FRAME_SPEED = 1.8` |
| **Duração** | **6 s** → 150 frames |
| **Budget** | 100% · o MAJOR da página |

```
[STYLE ANCHOR]

MOTION: one unbroken first-person walking shot at eye height that begins on the sidewalk facing
the closed glass door, the glass door slides open once, and the camera keeps moving forward
without ever stopping — through the entrance, past the reception desk, hands reaching in to take
a pet carrier, down a corridor, ending as the exam-room light fades up at the end of the walk.
CAMERA: forward travelling shot, constant walking speed, gentle natural bob, never reversing,
never stopping, never turning back, locked horizon
SUBJECT ACTION: the glass door slides open a single time; a pair of hands and violet-scrubbed
forearms enter frame to receive a pet carrier and lift it out of shot; amber practical lights
pass overhead; the exam-room fixture fades up in the final second
PACE: slow purposeful walking pace, 6 seconds, one continuous take from first frame to last,
strictly monotonic forward motion
EXCLUDE: cuts, scene changes, recognizable faces, the camera stopping or reversing, hard handheld
shake, whip pans, doors closing behind the camera, readable signage, monitors with readable
content, speed ramps, other people walking toward camera
```

**Continuidade obrigatória.** O primeiro frame deste clipe é o **último frame da 01**: mesma
lente, mesma altura, mesma distância do vidro, mesma chuva. Gerar o still da 04 a partir do último
frame da 01, não de um prompt independente. Se o visitante perceber um salto de posição entre as
duas seções, a página inteira perde a costura que justifica ter feito o hero parar na calçada.

**Mecânica.** Capítulo pinado. Cada um dos cinco passos do protocolo entra em texto no momento em
que a câmera passa pelo lugar físico correspondente: *Você chega* na calçada, *O pet entra na
frente do papel* nas mãos recebendo a caixa, *Triagem* no balcão, *Exame no mesmo prédio* no
corredor, *A conversa* na sala de exame acendendo. O scroll é a caminhada.

**Extração — desktop (1440 px, 25 fps, 150 frames)**

```bash
mkdir -p public/frames/primeiros-minutos
npx ffmpeg -i "design/flow/04-primeiros-minutos.mp4" \
  -vf "fps=25,scale=1440:-1:flags=lanczos" \
  -c:v libwebp -quality 72 \
  "public/frames/primeiros-minutos/frame_%04d.webp"
```

**Extração — mobile (720 px, 25 fps, 150 frames)**

```bash
mkdir -p public/frames/primeiros-minutos-mobile
npx ffmpeg -i "design/flow/04-primeiros-minutos.mp4" \
  -vf "fps=25,scale=720:-1:flags=lanczos" \
  -c:v libwebp -quality 60 \
  "public/frames/primeiros-minutos-mobile/frame_%04d.webp"
```

Manter 25 fps nas duas: derrubar fps no mobile levaria abaixo do piso de 150 frames e o movimento
estrobaria justamente onde o hardware é pior. A economia sai da **largura e da qualidade**, nunca
da contagem de frames. Verificar com `ls public/frames/primeiros-minutos | wc -l` (= 150) e
`du -sh public/frames/primeiros-minutos`.

**Teste de aceitação.** Nenhum corte em 150 frames. A câmera nunca para nem recua. Nenhum rosto
reconhecível em nenhum frame. Mãos e patas anatomicamente sãs a 100% de zoom nos frames em que
aparecem — é o ponto de falha usual e o mais caro aqui, porque as mãos recebendo a caixa são o
frame emocional da seção. Se as mãos morfarem, regenerar o still e refazer o clipe; não aceitar.

---

# 05 · `estrutura`

| | |
|---|---|
| **Still** | `design/stills/05-estrutura.svg` — corte axonométrico do prédio, ilustrado |
| **Técnica** | **still apenas** — animação em DOM/SVG, sem Flow |
| **Duração** | — |
| **Budget** | 50% · small |

**Não gerar clipe.** Única troca de registro visual da página: sai fotografia, entra ilustração
axonométrica na paleta da marca (paredes `#603084`, luz `#FCB400`, chão `#F7F4EF`). Quatro
capítulos seguidos na mesma gramática fotográfica viram papel de parede.

**Movimento em DOM.** As salas acendem uma a uma, de baixo para cima, conforme o scroll sobe:
laboratório → imagem → centro cirúrgico → UTI → internação canina → internação felina → farmácia.
`stagger` 0,10 s, easing `cubic-bezier(0.16, 1, 0.3, 1)`. Parallax sutil entre andares. Patas em
lilás `#E4D2F0` a 10% derivando lentamente atrás do corte.

**Teste de aceitação.** O prédio inteiro é legível em 375 px de largura sem zoom. Cada sala tem um
rótulo em texto DOM, nunca dentro do SVG como path de letra — o rótulo precisa ser selecionável e
traduzível.

---

# 06 · `quando-precisa-ficar` — PICO TÉCNICO

| | |
|---|---|
| **Still** | `design/stills/06-quando-precisa-ficar.png` — internação à noite, janela alta à esquerda |
| **Técnica** | **canvas frames** — `pin` + `scrub`, `FRAME_SPEED = 1.8`, com gate |
| **Duração** | **6 s** → 150 frames |
| **Budget** | 90% · medium narrativo, pico técnico |

```
[STYLE ANCHOR]

MOTION: a single completely locked shot of a veterinary inpatient room at night in which nothing
moves except the light — the tall window on the left goes from black at 2am, to deep blue at 6am,
to warm gold at 8am, and the violet walls warm toward cream as it does.
CAMERA: absolutely static, tripod locked, no push, no pan, no parallax, no breathing, no drift
SUBJECT ACTION: a medium dog sleeping curled under a blanket in an open kennel, breathing slowly
and evenly, never lifting its head and never waking; a vital-signs monitor at the back of the
room pulsing a steady green line; dust drifting slowly through the window light
PACE: very slow, 6 seconds, one continuous take, light changing at a constant rate, ending held
on the gold window
EXCLUDE: cuts, camera movement, the dog waking, lifting its head, standing or looking at the
door, people entering frame, readable numbers or text on the monitor, alarms, flicker, speed
ramps
```

**Por que esta é a sequência de frames que a página mais merece.** É o único clipe impossível de
substituir por still — a mudança ao longo do tempo **é** o conteúdo — e ao mesmo tempo o mais
barato de extrair: câmera imóvel, cena escura, pouco detalhe, WebP pequeno por frame. O scroll é o
relógio. Rolar é a madrugada passando.

**Handoff de fundo.** A transição roxo-noite → creme-dia da página inteira acontece **dentro**
deste capítulo, dirigida pelo mesmo `progress` do scrub, com a `curva-swoosh` da marca e nunca em
linha reta. O capítulo 07 já nasce em outro dia.

**Os três boletins** entram em pontos fixos do progresso, sobre o clipe: `03h20` em ~20%, `05h50`
em ~55%, `07h40` em ~90%. Texto de processo, nunca de prognóstico.

**Extração — desktop (1600 px, 25 fps, 150 frames)**

```bash
mkdir -p public/frames/quando-precisa-ficar
npx ffmpeg -i "design/flow/06-quando-precisa-ficar.mp4" \
  -vf "fps=25,scale=1600:-1:flags=lanczos" \
  -c:v libwebp -quality 72 \
  "public/frames/quando-precisa-ficar/frame_%04d.webp"
```

**Extração — mobile (720 px, 25 fps, 150 frames)**

```bash
mkdir -p public/frames/quando-precisa-ficar-mobile
npx ffmpeg -i "design/flow/06-quando-precisa-ficar.mp4" \
  -vf "fps=25,scale=720:-1:flags=lanczos" \
  -c:v libwebp -quality 58 \
  "public/frames/quando-precisa-ficar-mobile/frame_%04d.webp"
```

Esta seção aguenta 1600 px no desktop, mais que a 04, porque a cena é escura e de baixo detalhe:
o mesmo `-quality 72` produz arquivo bem menor por frame. Se o `du -sh` passar de 3,5 MB, **cair
para `fps=20` antes de mexer na qualidade** — aqui a perda de 30 frames custa pouco, porque não
há movimento de assunto para estrobar, só luz.

**Gate obrigatório.** Só inicializa acima de um limiar de conexão e de viewport, via
`IntersectionObserver`. Abaixo do limiar, e sob `prefers-reduced-motion`, entrega o **último
frame** (a janela dourada) mais os cinco itens em texto e os três boletins estáticos. Nenhuma
informação se perde.

**Teste de aceitação — o mais importante da página.** **O cão está dormindo no frame 1 e no frame
150.** Ele não levanta a cabeça, não olha para a porta, não acorda. Desfecho clínico encenado é
proibido pelo design system, e um print dessa tela vira promessa. Se ele acordar em qualquer
frame → regenerar, sem discussão. Além disso: nenhum número legível no monitor, nenhuma deriva de
câmera, e o último frame precisa funcionar como imagem final da seção, porque é o que fica na tela
sob reduced-motion e em conexão fraca.

---

# 07 · `ala-felina`

| | |
|---|---|
| **Still** | `design/stills/07-ala-felina.png` — parede frontal, vidro jateado à esquerda, gato à direita |
| **Técnica** | `loop <video>` — toca uma vez na entrada, descansa no último frame |
| **Duração** | **5 s** |
| **Budget** | 60% · small |

```
[STYLE ANCHOR]

MOTION: one continuous lateral travelling shot moving right along a wall, passing through the
wall itself in a single unbroken move — from a warm, busy canine side seen through frosted glass,
to a quiet, bright feline side in natural window light.
CAMERA: slow dolly left-to-right, constant speed, locked horizon, no push, no tilt, no reversing
SUBJECT ACTION: on the left, blurred warm silhouettes moving behind frosted glass, progressively
losing saturation and focus as they exit frame; on the right, a cat sitting motionless on a
cushion in soft window light, coming into sharp focus as the camera arrives, blinking once slowly
PACE: slow, even, 5 seconds, one continuous take, single direction, never reversing
EXCLUDE: cuts, the cat standing, jumping or leaving frame, recognizable human faces, dogs in
sharp focus, camera shake, zoom, readable signage, speed ramps
```

**Por que este movimento.** A seção afirma que são dois mundos separados e o clipe faz o visitante
atravessar a separação. O movimento **é** o argumento: ninguém lê que as alas são separadas, a
pessoa as vê se separando. É também a primeira luz natural da página — a 06 já amanheceu.

**Teste de aceitação.** O atravessamento da parede é contínuo e sem corte: não pode haver um frame
em que a imagem troca. O gato não sai do quadro. O último frame — gato nítido, silêncio, luz de
janela — é o poster da seção e precisa funcionar sozinho.

---

# 08 · `rotina-e-estetica`

| | |
|---|---|
| **Still** | `design/stills/08-rotina-e-estetica.png` — sala de banho e tosa em luz de manhã |
| **Técnica** | `currentTime` raspado em `≥1024px` · `loop <video>` toca-uma-vez no mobile |
| **Duração** | **6 s** |
| **Budget** | 65% · medium |

```
[STYLE ANCHOR]

MOTION: one continuous shot of a medium dog in a bright grooming tub going from soaking wet to
fully dry and fluffed, in a single unbroken take — water sheeting off the coat, foam rising, a
dryer lifting the fur, the coat settling clean and full.
CAMERA: very slow push in, constant speed, locked horizon, minimal movement — the transformation
carries the shot, not the camera
SUBJECT ACTION: the dog stands calmly throughout and never shakes off or jumps; hands and
violet-scrubbed forearms work at the edge of frame; soap bubbles rise slowly through the entire
clip; the coat visibly changes from wet-flat to dry-full at a constant rate
PACE: slow and steady, 6 seconds, one continuous take, the wet-to-dry change progressing evenly
with no jump
EXCLUDE: cuts, the dog shaking off, recognizable human faces, splashing that obscures the
subject, clinical or surgical equipment in frame, camera shake, readable signage, speed ramps
```

**Por que este movimento.** É o destino do `É rotina` do hero e o ponto onde o público de rotina
converte. Creme e âmbar dominando, roxo reduzido a acento, nada de clínico no quadro — quem chega
aqui pelo atalho do hero precisa aterrissar num ambiente que não pareça pronto-socorro.

**A delícia escondida, e seu custo.** Em `≥1024px`, como o clipe é raspado, **rolar para cima
molha o cachorro de novo**. Ninguém avisa; quem descobre, brinca. No mobile o clipe toca uma vez e
para — o iOS não busca frame comprimido com precisão, e forçar isso entregaria tremor para a
maioria do tráfego. Perda aceita conscientemente: o slot de canvas que salvaria a delícia é o slot
que o MAJOR precisa.

**Teste de aceitação.** A transição molhado → seco é monotônica: nenhum frame no meio está mais
molhado que o anterior, ou o scrub reverso deixa de ler como "molhando de novo". O cão nunca se
sacode. Nenhum rosto humano. Bolhas presentes do primeiro ao último frame.

---

# 09 · `depoimentos`

| | |
|---|---|
| **Still** | nenhum |
| **Técnica** | **zero mídia** |
| **Budget** | 20% · small |

**Nenhum clipe e nenhuma imagem, por declaração explícita.** A página foi cinemática por oito
capítulos; aqui o visitante precisa **ler**, e vídeo atrás de texto de depoimento rouba a leitura.

**Movimento em DOM.** Fundo creme sólido `#F7F4EF`, cards brancos radius 24px entrando pela
assinatura de motion da marca: revelação por máscara curva (`clip-path` no formato do swoosh),
nunca fade retangular. `stagger` 0,10 s. Divisor `regua-ambar-coracao` entre os cards. Parallax
sutil de velocidade entre a coluna e o fundo. Custo de banda: zero.

---

# 10 · `faq`

| | |
|---|---|
| **Still** | nenhum |
| **Técnica** | **zero mídia** |
| **Budget** | 10% · silêncio deliberado |

**Nenhum clipe e nenhuma imagem.** O momento mais frio da página, de propósito, logo antes do
fecho quente.

**Movimento em DOM.** Fundo creme com o motivo de patas em lilás `#E4D2F0` a 10% e deriva
lentíssima em `translateY` — textura, não animação. Único movimento funcional: o glifo de
expandir, que é o coração de contorno âmbar da marca, girando 45° na abertura em 0,2 s com
`cubic-bezier(0.16, 1, 0.3, 1)`, mais a altura do `<details>`. Custo de banda: zero.

---

# 11 · `contato`

| | |
|---|---|
| **Still** | `design/stills/11-contato.png` — mesma fachada da 01, mesmo ponto de vista, plano aberto |
| **Técnica** | `loop <video>` — toca uma vez na entrada, descansa no último frame |
| **Duração** | **6 s** |
| **Budget** | 45% · small |

```
[STYLE ANCHOR]

MOTION: one continuous slow backward dolly retreating down the street away from the veterinary
hospital at night, the whole block widening into frame, neighbouring windows dark, the hospital's
amber reception window staying lit until it is the only light on the street.
CAMERA: slow pull back, constant speed, locked horizon, tripod-smooth, ending held completely
still on the wide frame
SUBJECT ACTION: none — the street is empty of people and cars. Neighbouring windows are already
dark; only the amber reception window and the violet sign stay lit as the frame widens. Light
drizzle catching the glow on wet asphalt.
PACE: slow, deliberate, 6 seconds, one continuous take, the final second almost completely still
EXCLUDE: cuts, the camera moving forward at any point, people, cars, lights going out inside the
clinic, camera shake, text overlays, readable signage, speed ramps
```

**Continuidade obrigatória.** Mesma fachada, mesmo ponto de vista e mesma lente da 01 — plano
aberto. Gerar o still da 11 a partir do still da 01, alargando o enquadramento, nunca de um prompt
independente. O hero levou o visitante até a porta; o fecho o coloca do lado de fora e mostra o
que ele agora sabe que existe. Fecha o arco visual sem repetir o hero.

**Teste de aceitação.** É o recuo exato do percurso da 01: se as duas fachadas não forem
reconhecivelmente a mesma, o arco não fecha e o clipe vira decoração. A janela âmbar continua
acesa no último frame e é a única luz da rua. O último frame é a imagem final da página e o poster
de reduced-motion — precisa aguentar ser olhado sem se mover.

---

# Tabela-resumo

Frames estimados a 25 fps nativos do Flow. Pesos são estimativa de encode; verificar com
`du -sh` após a extração e ajustar `-quality` antes de mexer em `fps`.

| # | Seção | Técnica | Duração | Frames | Peso desktop | Peso mobile |
|---|---|---|---|---|---|---|
| 01 | `agora` | loop `<video>` | 6 s | — | 780 KB | 320 KB |
| 02 | `dois-caminhos` | scrub `≥1024` / loop mobile | 6 s | — | 680 KB | 300 KB |
| 03 | `e-grave` | still apenas | — | 1 | 180 KB | 85 KB |
| 04 | `primeiros-minutos` | **canvas frames** | 6 s | **150** | **4 400 KB** | **1 500 KB** |
| 05 | `estrutura` | still SVG + DOM | — | 1 | 60 KB | 60 KB |
| 06 | `quando-precisa-ficar` | **canvas frames** | 6 s | **150** | **3 000 KB** | **1 050 KB** |
| 07 | `ala-felina` | loop `<video>` | 5 s | — | 620 KB | 280 KB |
| 08 | `rotina-e-estetica` | scrub `≥1024` / loop mobile | 6 s | — | 720 KB | 310 KB |
| 09 | `depoimentos` | zero mídia | — | — | 0 | 0 |
| 10 | `faq` | zero mídia | — | — | 0 | 0 |
| 11 | `contato` | loop `<video>` | 6 s | — | 640 KB | 290 KB |
| | **TOTAL** | | | **300** | **11 080 KB ≈ 10,8 MB** | **4 195 KB ≈ 4,1 MB** |

**Verificação contra os tetos**

| Teto | Limite | Realizado | |
|---|---|---|---|
| Sequências de frames em canvas | 12 MB desktop · 5 MB mobile | **7,4 MB · 2,6 MB** | ✓ folga de 38% e 48% |
| Total de mídia da página (blueprint) | 12 MB desktop · 5 MB mobile | **10,8 MB · 4,1 MB** | ✓ |
| Nº de sequências de frames | máx. 2 | **2** (04, 06) | ✓ no teto |
| Sequência da 06 (teto do blueprint) | 3,5 MB desktop · 2,0 MB mobile | 3,0 MB · 1,05 MB | ✓ |
| Qualquer `<video>` individual | 900 KB | máx. 780 KB (01) | ✓ |
| Mídia acima da dobra | 400 KB | 0 KB — hero é poster estático até o LCP | ✓ |

**Nenhum rebaixamento por peso foi necessário.** As três mudanças de técnica em relação ao
blueprint (§"Decisão de técnica") vêm do teto de **duas** sequências de canvas e do
comportamento do iOS Safari, não do orçamento de bytes. A folga que sobra é margem para o
`du -sh` real vir acima da estimativa — e ela vem, tipicamente 10–15% acima em cenas claras. Se a
04 estourar, a ordem de corte é: largura 1440 → 1280 no desktop, depois `-quality 72` → `68`, e
só então `fps=25` → `20`. **Nunca cortar a contagem de frames abaixo de 150 em nenhuma das duas.**

**Receita única de encode para os cinco `<video>`** (01, 02, 07, 08, 11), duas renditions:

```bash
# desktop — 1440px
npx ffmpeg -i "design/flow/NN-<id>.mp4" -vf "scale=1440:-2" \
  -c:v libx264 -profile:v high -crf 26 -preset slow -pix_fmt yuv420p \
  -movflags +faststart -an "public/video/<id>-1440.mp4"

# mobile — 720px
npx ffmpeg -i "design/flow/NN-<id>.mp4" -vf "scale=720:-2" \
  -c:v libx264 -profile:v high -crf 30 -preset slow -pix_fmt yuv420p \
  -movflags +faststart -an "public/video/<id>-720.mp4"
```

`-an` porque nenhum clipe da página tem áudio. `+faststart` porque sem ele o vídeo não começa a
decodificar antes do download completo. Todos com `playsinline`, `muted`, `preload="none"` e
`IntersectionObserver` uma seção à frente — nada de vídeo antes do primeiro scroll.

---

## Checklist de revisão dos sete clipes

Rodar antes de comprometer qualquer extração. Rejeitar e regenerar sai mais barato que consertar
no CSS.

- [ ] Algum corte, mudança de cena ou salto de posição em qualquer clipe? → inutilizável para scrub
- [ ] A câmera inverte o sentido em algum ponto do mesmo clipe? → inutilizável
- [ ] Tremor de câmera perceptível ao avançar frame a frame? → inutilizável
- [ ] Rosto humano reconhecível em qualquer frame? → regenerar (não temos autorização de imagem)
- [ ] Texto, letra, logo ou número legível em placa, monitor ou etiqueta? → regenerar
- [ ] Mãos e patas anatomicamente sãs a 100% de zoom, especialmente na 04? → ponto de falha usual
- [ ] Cor da marca presente como objeto (parede, jaleco, luminária) e não como banho de cor?
- [ ] O último frame de cada clipe funciona como still? Ele fica mais tempo na tela que qualquer
      outro e é o poster de `prefers-reduced-motion`.
- [ ] O que carrega a cena está no terço central, sobrevivendo ao `IMAGE_SCALE` 0.85 e ao crop 9:16?
- [ ] 01 termina com a porta fechada? 04 começa exatamente onde 01 parou?
- [ ] **06 termina com o cão dormindo?** — bloqueante
- [ ] Alguma seção ficou 80% on-brand? Uma seção quase certa arrasta a página mais para baixo do
      que uma seção sem imagem nenhuma.
