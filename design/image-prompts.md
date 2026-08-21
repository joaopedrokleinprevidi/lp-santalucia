# Image Prompts — Landing Page Santa Lúcia

> Direção de imagem por seção. Deriva de `design/landing-blueprint.md` e de `design/design-system.json`.
> Cada still gerado aqui é o **primeiro frame** do clipe que o Google Flow vai animar e que o scroll vai raspar.
> Ao desenhar cada quadro a pergunta não foi "isto é bonito", foi **"o que se move aqui e onde cai a copy do DOM"**.

---

## Sumário de decisão

| # | Seção | Registro | O que entra | Proporção |
|---|---|---|---|---|
| 01 | `agora` | fotográfico noturno | **USAR FOTO REAL** | — |
| 02 | `dois-caminhos` | fotográfico noturno → diurno | prompt gerado | 16:9 |
| 03 | `e-grave` | fotográfico noturno | prompt gerado (still, sem clipe) | 16:9 |
| 04 | `primeiros-minutos` | fotográfico noturno | prompt gerado (+ quadro final opcional) | 16:9 |
| 05 | `estrutura` | **ilustrado axonométrico** | prompt gerado, âncora própria | 4:3 |
| 06 | `quando-precisa-ficar` | fotográfico noturno → amanhecer | prompt gerado | 16:9 |
| 07 | `ala-felina` | fotográfico diurno | prompt gerado | 16:9 |
| 08 | `rotina-e-estetica` | fotográfico diurno | prompt gerado | 16:9 |
| 09 | `depoimentos` | — | **sem imagem, declarado** | — |
| 10 | `faq` | — | **sem imagem, declarado** | — |
| 11 | `contato` | fotográfico noturno | **USAR FOTO REAL** | — |

**8 prompts de imagem escritos** — 7 seções (02, 03, 04, 05, 06, 07, 08) + 1 quadro final opcional para a 04.

**Nenhum 21:9 nesta página.** O 21:9 serve papel de faixa/textura, e as duas seções que ocupariam esse papel (09 e 10) estão declaradas sem mídia no blueprint. Forçar uma faixa aqui seria produzir uma imagem para preencher um slot de formato, que é exatamente o motion sem ROI que o CLAUDE.md manda remover.

---

## 1 · STYLE ANCHOR

Prefixado **literalmente, sem uma vírgula alterada**, em todos os prompts fotográficos (02, 03, 04, 06, 07, 08). Se ele mudar na seção 5, as seções 1 a 4 deixam de combinar — nesse caso, regerar todas ou não mexer em nenhuma.

```
STYLE ANCHOR — prepend verbatim, never edited mid-project:

Professional editorial photography for a veterinary clinic brand, documentary in register
and calm in mood. Soft diffused light with gentle falloff and no hard-edged shadows; every
light source is a practical fixture that belongs in the room. Shallow depth of field at
f/2.0, shot on a 50mm lens at eye level, only the natural falloff of the lens at the edges.
The brand palette exists in frame as real physical objects and never as a grade: warm cream
#F7F4EF on painted walls, tile, bedding and counter stone; deep violet #482A78 on painted
surfaces, scrub sleeves, cabinetry and powder-coated metal; amber #FCB400 as the glow of
lamps, as anodized trim on equipment and as a painted edge. Clean uncluttered composition,
generous negative space, few objects, nothing decorative added to fill the frame. People
appear only as hands, forearms in scrub sleeves, backs or out-of-focus silhouettes — never a
recognizable face. Natural skin, fur and material tones, neutral highlights with no color
cast, no color grading, no filter look, no bloom, no lens flare, no vignette added in post.
```

Três razões pelas quais este parágrafo é o que é:

1. **Luz, lente e paleta antes de assunto.** São os três termos que carregam consistência entre chamadas independentes do modelo. Assunto não carrega.
2. **A cor da marca está amarrada a objeto.** `violet on scrub sleeves and cabinetry` sobrevive à geração. `violet color scheme` produz uma lavagem roxa sobre a imagem inteira e denuncia IA em meio segundo. A regra 60/30/10 do design system vale para o quadro tanto quanto para o CSS.
3. **A âncora é agnóstica de hora do dia.** Ela diz *difusa e com falloff suave*, não *luz de dia*. A página vai de madrugada roxa (01–06) a manhã creme (07–08); a direção e a temperatura da luz entram na linha `DETAIL` de cada bloco, nunca na âncora. É o que permite congelá-la.

### EXCLUDE base — presente em todos os prompts, sem exceção

```
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels,
UI overlays, subtitles, extra limbs, distorted paws, malformed eyes, duplicated tails,
recognizable human faces, color grading, filter look, bloom, lens flare, oversaturation.
```

Modelo generativo renderiza texto como lixo plausível, e diacrítico português (`ção`, `Lúcia`, `matização`) piora o resultado. **Toda a copy é DOM** — é o que a torna selecionável, copiável, traduzível, indexável e legível por leitor de tela. O telefone e o endereço, em particular, nunca podem estar dentro de pixel.

### Regras globais de enquadramento

- **Crop 9:16 do mobile.** Assunto e espaço negativo têm que caber nos **60% centrais horizontais**. O que estiver fora disso não existe no celular, que é a maioria do tráfego desta página.
- **Área segura do canvas.** O renderizador da sequência de frames trabalha em `IMAGE_SCALE` 0.85: os ~7% externos de cada borda são padding. Nada que importe mora ali.
- **Gerar na maior resolução que a ferramenta oferecer.** O pipeline gera AVIF e WebP a partir do original; ele reduz, nunca amplia.
- **Nomes de arquivo:** `design/renders/NN-id-secao.png` (ex.: `design/renders/06-quando-precisa-ficar.png`). O clipe do Flow sai como `NN-id-secao.mp4`.

### Anexo de referência — o mecanismo, e onde ele quebra

Descrever cor em texto aproxima; anexar o arquivo entrega o hex. `warm cream #F7F4EF` é uma instrução que o modelo interpreta ao seu modo; o JPG do logo é uma amostra que ele lê. Por isso todo bloco que gera imagem traz uma tabela **ANEXAR**: os arquivos que sobem junto com o prompt, no mesmo envio, e o que cada um está ali para ensinar.

- **O logo entra em toda geração.** É o único arquivo que carrega `#603084`, `#482A78` e `#FCB400` medidos, e é ao lado do logo real no header que cada imagem vai ser vista.
- **Teto de 3 anexos — e quase todo bloco usa 2.** Com 4 ou mais o modelo faz média das composições e devolve colagem. O sintoma é a imagem voltar com o **enquadramento** da referência e outro assunto dentro: o crop do post, o balcão trocado por outra coisa no mesmo lugar. Quando aparecer, corte para o logo apenas e regere.
- **Anexo é referência de cor, luz e material. Nunca de enquadramento.** Quem define enquadramento é a linha `COMPOSITION`, porque é ela que reserva o espaço vazio onde a copy do DOM cai. Uma referência que sequestra o enquadramento destrói esse espaço e obriga a salvar a legibilidade com scrim — que é desperdiçar a foto.
- **`assets-source/fachada-clinica-rua.webp` não é anexo de nada.** Ela é usada direta nas seções 01 e 11 e alimenta o Flow como primeiro frame. Anexada a uma geração, puxa o quadro para perto do prédio real, e imagem gerada não pode afirmar um lugar que existe.
- **Uma conversa nova por seção.** Em thread longa o modelo passa a usar como referência implícita as imagens que ele mesmo acabou de gerar, e a 07 sai como a 06 com outro bicho dentro. A consistência vem da âncora congelada e dos mesmos anexos deliberados, nunca da memória da conversa.
- **Ordem do envio: anexos → prompt → frase de anexo.** A frase vai por último, depois do prompt, e está transcrita em todo bloco que gera imagem.

---

## 2 · Blocos por seção

---

### 01 · `agora` — hero

**Papel visual:** responder em meio segundo que tem gente acordada ali dentro, neste momento. O clipe é um dolly-in lento da calçada oposta até o vidro, e **para na calçada — a porta não abre.** A porta abre uma vez só na página inteira, na seção 04.

> ## USAR FOTO REAL — não gerar
> - **Desktop / full-bleed:** `assets-source/fachada-clinica-rua.webp`
> - **Mobile 9:16:** `assets-source/fachada-clinica-vertical.png`
> - **Poster de LCP:** `assets-source/fachada-clinica-thumb.jpg`

**ANEXAR: nada — e isso é decisão, não esquecimento.** Não há geração de imagem aqui, e anexo de referência só existe onde um modelo desenha pixel novo. O arquivo já é a foto certa; ele entra direto no layout e alimenta o Flow como primeiro frame. Vale o contrário também: `fachada-clinica-rua.webp` **não é anexada em nenhuma geração da página** — anexada, ela puxa qualquer quadro gerado para perto do prédio real, que é a única coisa que a página não pode inventar.

**Por que não se gera.** Isto é o prédio real, com a placa real, no endereço real. Uma fachada gerada é uma afirmação falsa sobre um lugar que existe — o tutor que já passou de carro na Jacob Luchesi reconhece a diferença, e a página perde exatamente a credencial que ela veio comprar. Vale para a 01 e para a 11.

**Tratamento (não é geração, é montagem):**
- Recortar o terço inferior (fila de carros estacionados, asfalto). Sobra o prédio, o letreiro, o selo 24h e a vitrine acesa.
- Graduar para noite/anoitecer, **preservando a vitrine e a marquise acesas**. É a janela âmbar que a seção 11 vai pagar no fecho; se ela apagar na graduação, o arco visual da página inteira deixa de fechar.
- **Composição para a copy:** a copy do hero cai no terço esquerdo, sobre o telhado baixo do vizinho e o céu. Manter o volume roxo do prédio do centro para a direita. Sem scrim de opacidade alta: se a legibilidade exigir scrim, o crop está errado.
- Alimentar o Flow com esta foto como primeiro frame (image-to-video). O movimento vai no `motion-prompts.md`.

**Pendência registrada:** a foto real é de dia, com céu encoberto. A graduação noturna resolve, mas **uma foto noturna real da fachada, com a vitrine acesa, é o único asset que faltaria para esta página ser perfeita** — é barato (um celular, 22h, calçada oposta) e melhora o hero e o fecho de uma vez. Solicitar ao cliente.

**Teste de aceitação:** refazer o crop se o selo 24h ou o letreiro saírem do quadro, se a graduação apagar a vitrine iluminada, ou se algum rosto ou placa de carro ficar legível.

**Salvar como:** nenhum still a salvar — a imagem já existe em `assets-source/`. Suba `assets-source/fachada-clinica-rua.webp` no Flow como primeiro frame e salve **o clipe que voltar** como `design/renders/01-agora.mp4`.

---

### 02 · `dois-caminhos`

**Papel visual:** dizer a tese da página inteira em uma imagem só — **é a mesma sala, muda o motivo pelo qual você entra nela.** Câmera travada, o scroll controla a hora do dia. O still é o quadro das 3h da manhã; o time-lapse leva até as 15h.

**Proporção:** 16:9 full-bleed.

**ANEXAR — 2 arquivos, no mesmo envio do prompt:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-santa-lucia-fundo-roxo.jpg` | o roxo `#603084` chapado em área grande — que é literalmente o que a metade superior deste quadro é — e o âmbar `#FCB400` exato do pendente | desenhar o logo, o coração ou as silhuetas de cão e gato dentro da cena |
| 2 | `assets-source/hero-veterinaria-com-gato.jpg` | como roxo e creme da marca se comportam numa sala clínica **fotografada**: parede clara, batente pintado de roxo, faixa âmbar no alto, inox com reflexo baixo e sem estouro | pessoa, rosto, gato, mesa de exame ou enquadramento — aqui a recepção está vazia |

Nenhum post entra na 02. Os quatro são alta-chave, quase brancos, e este quadro é 3h da manhã. E como esta é a **primeira imagem gerada da página** — a que todas as outras vão ter que casar — quanto menos referência, mais o resultado depende da âncora, que é o único elemento que as oito compartilham.

**Prompt completo:**

```
[STYLE ANCHOR verbatim]

SUBJECT: The empty reception counter of a veterinary clinic photographed straight on in the
middle of the night, with nobody in the room.
COMPOSITION: The counter runs horizontally across the lower third, dead centered and
perfectly level. The upper half is one calm unbroken wall of deep violet paint lit by a
single low amber pendant — that upper band is the negative space the eyebrow and headline
sit on. The middle band, across the whole width, stays even and low in contrast so two DOM
cards can land over it; keep the counter surface clear of props.
DETAIL: A cream stone countertop with a matte finish, one amber pendant lamp hanging low and
warm above it, dark glass panels behind the counter holding a soft reflection of that lamp,
a violet powder-coated shelf edge running under the counter top.
FRAMING: Eye-level frontal medium-wide, camera perfectly square to the counter, no tilt,
tripod locked, 50mm.
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels, UI
overlays, subtitles, extra limbs, distorted paws, malformed eyes, duplicated tails,
recognizable human faces, color grading, filter look, bloom, lens flare, oversaturation,
people, pets, wall clocks, computer monitors, digital displays, plants, posters.
```

**Frase de anexo — colar DEPOIS do prompt, no mesmo envio (ordem: anexos → prompt → frase):**

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

**Onde cai a copy:** eyebrow e H1 na faixa superior (parede roxa); os dois cards, A e B, sobre a faixa central. O balcão é a linha que separa os dois.

**Teste de aceitação:** regerar se a câmera não estiver perfeitamente frontal e nivelada — o time-lapse inteiro depende de a câmera nunca se mover, e uma perspectiva torta denuncia o corte. Regerar também se aparecer relógio, monitor ou qualquer painel: além de virar texto falso, um relógio contradiz o time-lapse.

**Salvar como:** `design/renders/02-dois-caminhos.png` — o clipe do Flow gerado a partir dele vai ao lado, como `design/renders/02-dois-caminhos.mp4`.

---

### 03 · `e-grave`

**Papel visual:** **still, não clipe.** O WOW desta seção é a ausência de movimento, e uma seção que declara isso não paga produção de Flow. A imagem é fundo: nove linhas de sinais de emergência em tipo grande vivem em cima dela, uma acesa por vez. É o beat 9 da página — no auge do medo, espetáculo é ofensivo.

**Proporção:** 16:9 full-bleed, servida como `.webp` com poster único.

**ANEXAR — 2 arquivos, no mesmo envio do prompt:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-santa-lucia-fundo-roxo.jpg` | o roxo em campo grande e em sombra: 87% do arquivo é `#603084` chapado, e dois terços deste quadro são parede roxa escura | desenhar o logo ou qualquer marca na parede do corredor |
| 2 | `assets-source/post-farmacia-completa-24h.jpg` | é o único asset da marca em que o **âmbar aparece como contorno fino e luminoso sobre roxo** — exatamente o papel das arandelas e do reflexo delas no piso; e a foto dentro do arco é fria, fechada e de baixo contraste, que é a faixa tonal do corredor | o arco, o contorno como moldura, o nível de exposição (o post é claro, o corredor é escuro) nem o assunto |

Sem terceiro anexo, e o motivo é o argumento da seção: faixa tonal estreita e parede vazia. Todo anexo extra chega trazendo mobiliário, e mobiliário aqui vira hotspot em cima das nove linhas de texto.

**Prompt completo:**

```
[STYLE ANCHOR verbatim]

SUBJECT: The empty corridor of a veterinary clinic at night, seen from the eye level of
somebody who has just walked in through the entrance.
COMPOSITION: One-point perspective with the vanishing point slightly right of center. The
left two thirds are an unbroken wall of deep violet paint held in shadow — that is where a
long list of DOM text will sit, so it must stay flat, quiet and free of any bright spot. The
whole frame lives inside a narrow tonal range; the single brightest point is the far end of
the corridor, and it is small.
DETAIL: Recessed amber wall sconces receding in perspective down the right-hand wall, a
polished cream floor holding a long soft reflection of them, flush closed doors with brushed
metal handles set into the violet wall.
FRAMING: Wide, eye level, camera square to the corridor axis, 50mm.
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels, UI
overlays, subtitles, extra limbs, distorted paws, malformed eyes, duplicated tails,
recognizable human faces, color grading, filter look, bloom, lens flare, oversaturation,
people, animals, gurneys, open doors, equipment carts, ceiling glare.
```

**Frase de anexo — colar DEPOIS do prompt, no mesmo envio (ordem: anexos → prompt → frase):**

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

**Onde cai a copy:** a lista inteira nos dois terços esquerdos, sobre a parede escura. O botão `Ligar (54) 3025-2223` em bloco de largura total, no rodapé da seção, quando todas as linhas acendem de uma vez.

**Teste de aceitação:** regerar se houver qualquer hotspot de luz onde o texto vai (a coreografia derruba as linhas inativas para 18% de opacidade — sobre um brilho, elas somem), ou se alguma porta estiver entreaberta mostrando interior: o corredor tem que ler como calmo e fechado, não como cena de correria.

**Salvar como:** `design/renders/03-e-grave.png` — só o still. Esta seção não tem clipe.

---

### 04 · `primeiros-minutos` — ◆ MAJOR

**Papel visual:** o plano-sequência pinado. Retoma **exatamente** do quadro onde o clipe do hero parou — a mão da câmera diante do vidro — a porta desliza e abre, e a câmera não corta mais até a luz da sala de exame acender. O scroll é a caminhada.

**Proporção:** 16:9 full-bleed.

**O problema de continuidade, e como ele se resolve.** O hero é foto real do prédio real; este quadro é gerado. Se o quadro gerado mostrar fachada, os dois nunca vão combinar e a emenda aparece. A solução é enquadrar **tão fechado no vidro que não sobra arquitetura nenhuma no quadro** — a continuidade é carregada pela luz âmbar e pelo reflexo da rua molhada, não pelo prédio. É por isso que o `EXCLUDE` deste bloco é o mais duro da página.

**ANEXAR — 2 arquivos, no mesmo envio do prompt:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-santa-lucia-fundo-roxo.jpg` | o âmbar `#FCB400` exato — nesta cena ele é a luz que vaza pelo vidro e o único ponto quente do quadro — e o roxo da quina do balcão entrevisto atrás | desenhar o logo, nem no vidro, nem em adesivo, nem refletido |
| 2 | `assets-source/post-farmacia-completa-24h.jpg` | a relação de foco que este quadro precisa: primeiro plano nítido recortado contra um roxo fora de foco atrás, sob luz fria de interior | o assunto (mão, luva, seringa), o arco, nem o enquadramento |

**Não anexar `assets-source/fachada-clinica-rua.webp` aqui**, por mais tentador que a continuidade com o hero faça parecer. Este bloco existe justamente para **não** ter arquitetura no quadro; a fachada anexada devolve batente, marquise e letreiro, e é assim que a emenda com a foto real fica visível. A continuidade é carregada pela luz âmbar e pelo asfalto molhado, não pelo prédio.

**Prompt completo:**

```
[STYLE ANCHOR verbatim]

SUBJECT: First-person point of view standing directly in front of the glass entrance door of
a veterinary clinic at night, close enough that only the glass, its frame and its handle are
in the frame.
COMPOSITION: The vertical edge of the door frame sits just left of center; the amber interior
light spilling through the glass fills the right half. A narrow strip of wet dark asphalt
occupies the bottom eighth. No DOM copy sits over this first frame — the five protocol steps
enter later as the camera advances — so keep the lower third an even mid-tone with no busy
detail.
DETAIL: A brushed metal vertical handle, the reflection of a wet street and a distant amber
streetlight held in the glass, and behind the glass an out-of-focus amber-lit reception
interior with a violet counter edge just readable through it.
FRAMING: Point of view, eye height, 50mm, framed tight enough that no building facade, no
wall beyond the door frame and no exterior architecture is visible.
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels, UI
overlays, subtitles, extra limbs, distorted paws, malformed eyes, duplicated tails,
recognizable human faces, color grading, filter look, bloom, lens flare, oversaturation,
storefront facade, building exterior, awning, street signs, house numbers, window decals,
any person reflected in the glass.
```

**Frase de anexo — colar DEPOIS do prompt, no mesmo envio (ordem: anexos → prompt → frase):**

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

**Teste de aceitação:** regerar se qualquer arquitetura além do batente aparecer — ela contradiz a foto real da fachada em que o hero termina, e o corte fica visível. Regerar também se houver pessoa refletida no vidro, ou se o interior estiver nítido: ele precisa estar desfocado, porque é para onde a câmera ainda vai.

**Salvar como:** `design/renders/04-primeiros-minutos.png` — é o **primeiro frame** que você vai subir no Google Flow. O clipe que voltar de lá salva ao lado, como `design/renders/04-primeiros-minutos.mp4`.

---

#### 04b · quadro final opcional

Se o Flow for usado no modo **first frame + last frame**, este é o último quadro. Interpolar entre dois quadros ancorados é o que mais aumenta a chance de o plano-sequência sair sem corte — e sem corte é requisito, não preferência: um corte dentro de clipe raspado lê como bug.

**ANEXAR — 2 arquivos, no mesmo envio do prompt:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-santa-lucia-fundo-roxo.jpg` | os mesmos hex do bloco 04 — é o que garante que o âmbar do primeiro quadro e o âmbar do último sejam o mesmo âmbar, sem o que a interpolação do Flow deriva de cor no meio do plano | desenhar o logo no armário, na luminária ou na parede |
| 2 | `assets-source/hero-veterinaria-com-gato.jpg` | a mesa de inox real sob luz clínica, com reflexo baixo e sem estouro no branco, e o roxo pintado em superfície de sala. É o asset que mais se parece com o que este prompt pede | pessoa, rosto, luva, animal ou enquadramento — o `EXCLUDE` deste bloco proíbe pessoa e animal |

Os dois quadros do plano-sequência (04 e 04b) compartilham o logo de propósito: são gerados em conversas separadas e precisam fechar na mesma cor.

```
[STYLE ANCHOR verbatim]

SUBJECT: The open doorway of a veterinary examination room at night, seen from the corridor,
with the room's light just coming on and a stainless examination table catching it.
COMPOSITION: The doorway occupies the center of the frame, the corridor walls falling into
violet shadow on both sides, so the eye lands in the middle and rests. Even mid-tones across
the lower third.
DETAIL: A stainless examination table with a cream mat on it, an amber-trimmed articulated
examination lamp folded above, a violet cabinet front out of focus at the room's far side.
FRAMING: Point of view, eye height, 50mm, one step short of the threshold.
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels, UI
overlays, subtitles, extra limbs, distorted paws, malformed eyes, duplicated tails,
recognizable human faces, color grading, filter look, bloom, lens flare, oversaturation,
people, animals, blood, surgical instruments laid out, monitors with displays.
```

**Frase de anexo — colar DEPOIS do prompt, no mesmo envio (ordem: anexos → prompt → frase):**

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

**Onde cai a copy:** os cinco passos entram um a um sobre o clipe, cada um no instante em que a câmera passa pelo lugar físico correspondente (porta · balcão · mãos recebendo a caixa · corredor · sala de exame). O par `Como chegar` + `Ligar (54) 3025-2223` fecha a seção.

**Teste de aceitação:** regerar se a mesa estiver com instrumental cirúrgico exposto ou qualquer sinal de procedimento em curso. A seção vende **protocolo**, não drama — e o design system proíbe encenar desfecho clínico.

**Salvar como:** `design/renders/04b-primeiros-minutos-final.png` — é o **último frame** que sobe no Flow junto com a 04. Ele não vira imagem da página: existe só para ancorar a interpolação. Se você não usar o modo *first frame + last frame*, este arquivo simplesmente não existe.

---

### 05 · `estrutura`

**Papel visual:** a única troca de registro visual da página. Sai fotografia, entra ilustração. Corte transversal do prédio em axonometria, as salas acendendo de baixo para cima conforme o scroll sobe. Quatro capítulos seguidos na mesma gramática fotográfica viram papel de parede; e uma axonométrica é mais legível em 375 px do que qualquer corredor filmado, além de entregar o que a foto não entrega: o prédio inteiro de uma vez.

**Proporção:** 4:3 (split — ilustração ao lado da lista de seis itens).

**Esta seção não recebe o STYLE ANCHOR fotográfico.** Uma âncora de 50mm f/2.0 aplicada a um vetor produz uma ilustração com desfoque falso e ruído de sensor. Como ela é usada **uma única vez na página inteira**, a exceção não quebra consistência nenhuma — a troca de registro *é* o ponto da seção. Em lugar dela, uma âncora própria, igualmente congelada:

```
ILLUSTRATION ANCHOR — section 05 only:

Flat vector architectural illustration, axonometric cross-section, clean uniform 1.5px line
work, flat color fills with no gradients and no texture, a single very soft ambient occlusion
where surfaces meet. Limited palette: cream #F7F4EF floors and ceilings, violet #603084 and
#482A78 exterior and structural walls, lilac #E4D2F0 interior partitions, amber #FCB400
reserved exclusively for equipment, fixtures and fittings. Even neutral ambient light
throughout with no glow, no light bloom and no lit-window effect baked into the artwork.
No photographic depth of field, no noise, no paper texture, no drop shadows.
```

**ANEXAR — 2 arquivos, no mesmo envio do prompt. São referências de _desenho_, não de fotografia — é a única seção da página em que isso muda:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-santa-lucia-fundo-roxo.jpg` | os hex, e mais do que os hex: as silhuetas de cão e gato são **vetor chapado** da marca — contorno limpo, preenchimento sólido, zero gradiente. É a gramática de desenho, não só a cor | desenhar o logo, o coração ou os bichos dentro da ilustração — a axonométrica não tem animal nenhum |
| 2 | `assets-source/post-banho-e-tosa.jpg` | os seis pictogramas circulares, o divisor tracejado em lilás `#E4D2F0` e o card branco: peso de traço uniforme, fill chapado, sem sombra projetada e sem gradiente. É o vocabulário vetorial que a axonométrica tem que falar para parecer da mesma marca | o assunto, o layout quadrado do post, o cão da foto, e sobretudo qualquer letra — o post é 40% tipografia |

**Nenhuma foto entra aqui.** Anexar `hero-veterinaria-com-gato.jpg` ou a fachada numa geração vetorial devolve profundidade de campo falsa, ruído de sensor e sombra projetada — os três itens que o `ILLUSTRATION ANCHOR` proíbe por escrito. E o teto fica em 2: numa geração de ilustração o excesso de referência não vira colagem de foto, vira o **layout quadrado do post** invadindo a composição axonométrica e comendo os 45% da esquerda.

**Prompt completo:**

```
[ILLUSTRATION ANCHOR verbatim]

SUBJECT: A three-storey veterinary hospital drawn as an axonometric cross-section with the
front wall cut away, showing seven distinct rooms stacked and side by side, each identifiable
purely by the equipment inside it.
COMPOSITION: The building block occupies the right 55% of the frame and is cropped by no
edge. The left 45% is flat empty cream — that is where the DOM list of six items sits, so
nothing may intrude on it, not a line, not a shadow, not a paw. Leave a clear 12% margin
above the roof for the section headline when the layout stacks on mobile.
DETAIL: Ground floor — a pharmacy with deep shelving of unlabelled boxes, and a laboratory
with benchtop analyzers, a centrifuge and a microscope. Middle floor — an imaging room with
an X-ray table and overhead arm plus a wheeled ultrasound cart, and a surgical suite with an
operating table, an anesthesia machine and a domed surgical lamp. Top floor — an intensive
care room with oxygen cages and monitor stands, a canine ward with a row of large kennels,
and a smaller feline ward with stacked cat condos that have round porthole doors. Amber shows
only on lamp heads, equipment trim and handrails.
FRAMING: Axonometric at roughly 30 degrees, viewed slightly from above, the whole building
inside the frame with air around it, 4:3.
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels, room
name plates, UI overlays, extra limbs, distorted paws, malformed eyes, people, animals,
gradients, glow, lit windows, drop shadows, paper texture, blueprint grid lines.
```

**Frase de anexo — colar DEPOIS do prompt, no mesmo envio (ordem: anexos → prompt → frase):**

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

**Duas instruções que parecem detalhe e não são:**

1. **Nada de luz assada na ilustração.** As salas precisam sair **apagadas e neutras** para o DOM acendê-las uma a uma com `stagger` de 0,10 s no easing `cubic-bezier(0.16, 1, 0.3, 1)`. Se o modelo entregar as janelas já brilhando, não há o que animar e o único movimento da seção morre.
2. **Cada sala tem que ser reconhecível sem legenda**, porque legenda é texto e texto é proibido. É por isso que o `DETAIL` nomeia o equipamento de cada uma: a silhueta do aparelho é o rótulo.

**Teste de aceitação:** regerar se a função de qualquer sala não for legível só pelo equipamento; se houver brilho, gradiente ou janela acesa assados na arte; se aparecer letra falsa em alguma caixa da farmácia; ou se qualquer elemento invadir os 45% da esquerda.

**Salvar como:** `design/renders/05-estrutura.png` — sem clipe. O `motion-prompts.md` cita esta seção como `05-estrutura.svg`: o PNG é o que sai do ChatGPT, e a vetorização para SVG é trabalho da Fase 10, dentro do repositório. Salve o PNG e siga.

> **Nota de escopo:** esta é uma ilustração de **capacidade instalada**, não uma planta baixa. A adjacência e a quantidade de andares são esquemáticas. Todos os sete ambientes desenhados constam em `facts.services` do design system — nenhum foi inventado.

---

### 06 · `quando-precisa-ficar` — PICO TÉCNICO

**Papel visual:** a única sequência de frames da página. Tomada única e **completamente imóvel** em que só o tempo passa: a luz da janela vai do preto das 2h ao azul das 6h ao dourado das 8h. É o clipe mais barato de extrair (câmera travada, só a luz muda) e o único impossível de substituir por still — a mudança ao longo do tempo **é** o conteúdo. O handoff de fundo roxo-noite → creme-dia da página inteira acontece dentro dele.

**Proporção:** 16:9 full-bleed. O still gerado é o quadro das 2h, o mais escuro.

**ANEXAR — 2 arquivos, no mesmo envio do prompt:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-santa-lucia-fundo-roxo.jpg` | roxo e âmbar exatos; nesta cena o âmbar é uma única luz de vigília baixa na parede e o roxo é a borda pintada do canil | desenhar o logo na parede, no canil ou no monitor |
| 2 | `assets-source/hero-veterinaria-com-gato.jpg` | tom de pelo natural e tom de inox sob luz clínica, sem grade de cor e sem estouro no branco. É o pelo que precisa continuar crível depois de ser reiluminado três vezes ao longo do clipe — preto, azul e dourado | pessoa, rosto, luva, animal acordado e encarando, nem enquadramento |

Nenhum post entra na 06, e a razão é medida: os quatro são alta-chave, quase brancos. Este é o quadro mais escuro da página, e referência clara empurra a exposição para cima — se a janela das 2h deixar de ser preta, o amanhecer não tem onde acontecer e a seção inteira deixa de existir.

**Prompt completo:**

```
[STYLE ANCHOR verbatim]

SUBJECT: A medium-sized short-haired dog asleep, curled on a folded fleece blanket inside an
open-fronted veterinary kennel, in a quiet ward in the middle of the night.
COMPOSITION: The kennel sits centered and slightly right, the camera dead still and square to
it. A tall narrow window fills the far left of the frame with a black street beyond it — the
entire light change of the clip happens in that window, so the left quarter stays free of any
object. The lower right third is an even dark surface with no detail, because three DOM
message cards will land there.
DETAIL: A stainless kennel frame with a violet powder-coated front edge, a cream fleece
blanket the dog is curled into, a vitals monitor on a stand behind and to the right showing
one thin green trace on a dim screen, and one small amber night light low on the wall.
FRAMING: Static wide, camera at kennel height, tripod locked, 50mm.
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels, UI
overlays, subtitles, extra limbs, distorted paws, malformed eyes, duplicated tails,
recognizable human faces, color grading, filter look, bloom, lens flare, oversaturation,
people, a second animal, wounds, blood, bandages, a cone collar, an open panting mouth, any
sign of distress, IV bags, syringes, digits or waveforms with numerals on the monitor screen.
```

**Frase de anexo — colar DEPOIS do prompt, no mesmo envio (ordem: anexos → prompt → frase):**

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

**Onde cai a copy:** headline e subheadline na faixa superior; os cinco itens à esquerda, abaixo da janela; os três boletins (`03h20`, `05h50`, `07h40`) entrando em pontos fixos do progresso, sobre a faixa escura inferior direita. **Nenhum CTA nesta seção** — pedir uma ação enquanto o visitante imagina o próprio animal internado transforma empatia em venda, e ele sente.

**Teste de aceitação — o mais rígido da página.** Regerar se:
- o cão parecer acordado, ofegante, ferido, enfaixado ou de colar elizabetano. **O clipe termina com ele ainda dormindo**, e um print desta tela não pode virar promessa de desfecho clínico;
- houver qualquer numeral no monitor (é texto gerado, e ainda inventa dado clínico);
- a janela não estiver completamente desobstruída — sem ela, o amanhecer não tem onde acontecer e a seção inteira deixa de existir;
- o enquadramento não sobreviver a ser reiluminado em dourado: o **último frame** é o que fica na tela sob `prefers-reduced-motion` e abaixo do limiar de conexão, então é ele, e não o primeiro, que precisa funcionar sozinho como still.

**Salvar como:** `design/renders/06-quando-precisa-ficar.png`, e o clipe do Flow como `design/renders/06-quando-precisa-ficar.mp4`.

---

### 07 · `ala-felina`

**Papel visual:** a primeira luz natural da página. A seção diz "são dois mundos separados" e o clipe faz o visitante **atravessar a separação com o próprio dedo**: travelling lateral que cruza a parede em match-cut, do lado barulhento para o lado quieto. O visitante não lê que as alas são separadas — ele as vê se separando.

**Proporção:** 16:9 full-bleed. O still é o quadro inicial, com a parede ainda dividindo o frame ao meio.

**ANEXAR — 2 arquivos, no mesmo envio do prompt:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-santa-lucia-fundo-roxo.jpg` | o roxo exato da moldura pintada do vidro fosco — nesta seção o roxo cabe num caixilho fino, e é onde erro de matiz mais aparece | desenhar o logo ou o coração na parede clara, que aqui é grande e vazia |
| 2 | `assets-source/post-exclusividade-felina.jpg` | pelo de gato em luz difusa alta sobre creme e lilás: é o tratamento felino que a marca **já publica**, e a 07 é a primeira luz natural da página | **a pose e a direção do olhar** — o gato do post encara a câmera com pupila aberta, que é exatamente o que o teste de aceitação desta seção manda regerar. Também não serve para o arco nem para o enquadramento |

Nada de segundo gato. Anexar também `hero-veterinaria-com-gato.jpg` daria duas fotos de gato, e com duas o modelo passa a fazer média das poses — além de trazer pessoa e mesa de inox, dois itens do `EXCLUDE` deste bloco.

**Prompt completo:**

```
[STYLE ANCHOR verbatim]

SUBJECT: A single interior wall photographed straight on: on its left half a frosted glass
panel with warm blurred movement behind it, and on its right half a cat sitting upright and
completely still on a cushion in daylight.
COMPOSITION: The vertical divide between the two halves falls at the exact center of the
frame. Both the glass panel and the cat sit in the lower two thirds. The upper third is
uninterrupted pale wall across the whole width — that is the negative space the eyebrow and
the headline sit on, and it must stay clean edge to edge.
DETAIL: Frosted glass set in a violet-painted frame with soft warm amber shapes moving behind
it and never resolving into an identifiable animal, a cream cushion on a low cream bench, and
soft window daylight coming from the right and raking across the pale wall.
FRAMING: Frontal medium-wide, camera square to the wall, at the eye level of a seated cat,
50mm.
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels, UI
overlays, subtitles, extra limbs, distorted paws, malformed eyes, duplicated tails,
recognizable human faces, color grading, filter look, bloom, lens flare, oversaturation,
any visible dog, cages, wire mesh, people, toys, food bowls.
```

**Frase de anexo — colar DEPOIS do prompt, no mesmo envio (ordem: anexos → prompt → frase):**

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

**Onde cai a copy:** eyebrow e headline no terço superior; os três itens e a linha de fechamento à direita, ao lado do gato; a faixa pill roxa de contorno âmbar (pets não convencionais) ao pé da seção.

**Teste de aceitação:** regerar se o gato estiver encarando a câmera com pupila dilatada, agachado ou com orelha para trás — o argumento inteiro da seção é **um gato calmo**, e um gato tenso desmente a copy na mesma tela em que ela é dita. Regerar também se o lado esquerdo resolver em um cachorro reconhecível: ele tem que permanecer forma quente e borrada, ou o match-cut perde a surpresa.

**Salvar como:** `design/renders/07-ala-felina.png`, e o clipe do Flow como `design/renders/07-ala-felina.mp4`.

---

### 08 · `rotina-e-estetica`

**Papel visual:** paga a dívida do arquétipo. É o destino do `É rotina` do hero e o ponto onde o público de rotina converte. Um plano contínuo em que o cão **entra molhado e sai seco** — e como o clipe é raspado pelo scroll, **rolar para cima molha o cachorro de novo.** Ninguém avisa; quem descobre, brinca.

**Proporção:** 16:9 full-bleed. O still é o quadro do meio do banho, com o cão ensaboado.

**ANEXAR — 3 arquivos, no mesmo envio do prompt. É o único bloco da página que usa o teto:**

| # | Arquivo | Serve de referência para | Não serve para |
|---|---|---|---|
| 1 | `assets-source/logo-santa-lucia-fundo-roxo.jpg` | roxo e âmbar exatos, aqui presentes na manga do jaleco, no avental e no acabamento do chuveirinho | desenhar o logo no azulejo, no avental ou na parede |
| 2 | `assets-source/post-banho-e-tosa.jpg` | o registro de banho que a marca já usa: alta-chave, superfície clara, espuma e bolhas suspensas no ar, cão calmo e à vontade. É o único asset que mostra como **esta** marca fotografa estética animal | o laço, a toalha lilás de estúdio, o fundo chapado e o card branco de itens — card é layout, e layout aqui volta como texto |
| 3 | `assets-source/hero-veterinaria-com-gato.jpg` | o jaleco roxo **real**: a trama do tecido, como o roxo se comporta na dobra da manga e o tom de pele do braço que sai dela. O `SUBJECT` deste bloco é um par de mãos em manga roxa, e este é o único arquivo que mostra uma | rosto, luva cirúrgica, mesa de inox ou qualquer sinal de ambiente clínico — a seção existe justamente para não parecer hospital |

Por que o terceiro entra aqui e em nenhum outro lugar: são três coisas distintas e nenhuma está nas outras duas — cor de marca, registro de banho e tecido de jaleco. Se o resultado voltar com o crop do post (cão de frente, centralizado, fundo chapado), corte o item 3 e regere; se persistir, fique só com o logo.

**Prompt completo:**

```
[STYLE ANCHOR verbatim]

SUBJECT: A medium-sized dog standing calmly in a grooming tub in morning light, mid-bath,
with a pair of hands in violet scrub sleeves lathering along its back.
COMPOSITION: The dog and the tub occupy the right two thirds. The left third is a clean pale
tiled wall held in soft focus — the headline and the sedation card sit there, so no suds, no
bubbles and no spray may cross into it. Keep the dog's head inside the central 60% of the
width so the mobile crop keeps it.
DETAIL: Pale cream subway tile catching window light, a handheld shower head in cream and
amber trim, thick white lather with a few soap bubbles suspended in the air on the right, and
the edge of a violet rubber apron entering at the right border of the frame.
FRAMING: Medium, very slightly above eye level, window daylight from the left, 50mm.
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels, UI
overlays, subtitles, extra limbs, distorted paws, malformed eyes, duplicated tails,
recognizable human faces, color grading, filter look, bloom, lens flare, oversaturation,
clinical equipment, cages, stainless medical instruments, monitors, restraint by the collar,
a frightened or cowering dog, shampoo bottles.
```

**Frase de anexo — colar DEPOIS do prompt, no mesmo envio (ordem: anexos → prompt → frase):**

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

**Onde cai a copy:** headline e subheadline à esquerda; o card `faixa-pill-roxa` do **banho com sedação** — o mais importante da seção — sobre o terço esquerdo, com a palavra em âmbar; as duas colunas de serviços abaixo, com divisores tracejados em lilás `#E4D2F0`. Foto de pet estática nesta seção recebe o tratamento `arco-organico` (máscara arredondada assimétrica com contorno âmbar deslocado atrás), que é o tratamento mais característico da marca.

**Teste de aceitação:** regerar se o cão parecer assustado, encolhido, ou preso pela coleira, e regerar se qualquer coisa clínica aparecer no quadro — mesa de inox, gaiola, monitor. Esta seção existe para **não** parecer hospital: é o único lugar da página onde a moldura de emergência precisa desaparecer por completo, e é isso que faz a frase "a clínica que existe para as três da manhã também é onde o seu pet toma banho" funcionar em vez de assustar.

**Salvar como:** `design/renders/08-rotina-e-estetica.png`, e o clipe do Flow como `design/renders/08-rotina-e-estetica.mp4`.

---

### 09 · `depoimentos`

> ## SEM IMAGEM — declarado no blueprint
> Nenhum still, nenhum clipe, nenhuma textura fotográfica.

A página foi cinemática por oito capítulos; aqui o visitante precisa **ler**. Vídeo atrás de texto de depoimento rouba a leitura, e o blueprint corta explicitamente a ideia da "luz do sol varrendo a parede vazia atrás dos depoimentos" como motion decorativo puro.

O que carrega a seção é DOM: fundo creme sólido `#F7F4EF`, cards brancos de radius 24px entrando pela assinatura de motion da marca — **revelação por máscara curva** (`clip-path` no formato do swoosh), nunca fade retangular, com stagger de 0,10 s. Entre os cards, o divisor `regua-ambar-coracao`. Custo de banda: zero.

---

### 10 · `faq`

> ## SEM IMAGEM — declarado no blueprint
> Nenhum still, nenhum clipe.

Fundo creme `#F7F4EF` com o motivo de **patas em lilás `#E4D2F0` a 10% de opacidade** e deriva lentíssima em `translateY` — textura de CSS, não animação, e não é asset gerado: é o motivo da marca, vetorial, servido como SVG. O único movimento funcional é o glifo de expandir, que é o **coração de contorno âmbar da marca** girando 45° na abertura.

---

### 11 · `contato` — fecho

**Papel visual:** dolly-out lento, o percurso inverso exato do hero. A câmera recua pela rua, o quarteirão vai escurecendo, e a janela âmbar da recepção continua acesa — no fim do recuo é a única luz visível na rua inteira. O clipe termina parado nesse quadro e fica.

> ## USAR FOTO REAL — não gerar
> - **Plano aberto, quarteirão inteiro:** `assets-source/fachada-clinica-rua.webp`
> - **Mobile 9:16:** `assets-source/fachada-clinica-vertical.png`

**ANEXAR: nada — mesma razão da 01.** Seção sem geração não tem anexo. O que existe aqui é graduação sobre a foto real e um recuo de câmera gerado pelo Flow a partir dela, e nos dois casos o arquivo entra como imagem de origem, não como referência de estilo.

**Tratamento:**
- Mesmo ponto de vista e mesma lente do hero, agora **em plano aberto, sem o crop do terço inferior** — o quarteirão inteiro entra: o telhado do vizinho à esquerda, a loja vermelha à direita, os fios cruzando o céu.
- Graduação noturna **mais profunda que a do hero**: as janelas vizinhas apagadas, o vermelho da loja da direita rebaixado, e a vitrine da clínica mantida como a única fonte de luz do quadro. É o pagamento do motivo visual que atravessou a página inteira.
- **Composição para a copy:** toda a copy do fecho é DOM em faixa creme abaixo da imagem, ou sobre o céu no terço superior. O telefone, o endereço e o horário **nunca dentro de pixel** — são texto selecionável, `tel:` e copiável, e é essa a ação de maior LTV da página (`Salvar contato`, `.vcf`).
- Alimentar o Flow com esta foto como primeiro frame. O recuo é gerado.

**Precificação declarada:** medium, 45% do Creative Budget — recuo de câmera na mesma fachada é um medium barato, e inflar essa curva era o ponto mais fácil de inflar do documento.

**Teste de aceitação:** refazer o crop se a vitrine não for o ponto mais claro do quadro depois da graduação, se algum rosto ou placa de carro ficar legível, ou se o plano ficar tão aberto que a fachada deixe de ser reconhecível — quem passa de carro na Jacob Luchesi precisa reconhecer o prédio.

**Salvar como:** nenhum still a salvar — mesma foto real da 01, em plano aberto. Suba `assets-source/fachada-clinica-rua.webp` no Flow e salve **o clipe que voltar** como `design/renders/11-contato.mp4`.

---

## 3 · Revisão do que voltar

Rejeitar e regerar sai mais barato do que consertar em CSS. Uma seção 80% na marca arrasta a página inteira mais do que uma seção sem imagem nenhuma.

- [ ] Ela senta ao lado da foto real da fachada sem parecer outra marca?
- [ ] O espaço negativo está **onde a linha `COMPOSITION` pediu**, e não em qualquer lugar? Se a legibilidade da copy exigir um scrim de opacidade alta, a foto foi desperdiçada e o enquadramento está errado.
- [ ] Alguma letra, número, logo ou rótulo inventado em qualquer canto do quadro? → regerar
- [ ] Patas, mãos, orelhas e olhos anatomicamente sãos em 100%? → é onde estes modelos falham
- [ ] A cor da marca está em **objeto** — jaleco, equipamento, parede pintada, luminária — ou virou lavagem sobre a imagem? Lavagem → regerar.
- [ ] Nenhum rosto humano reconhecível em nenhuma das oito imagens?
- [ ] O assunto e o espaço negativo cabem nos **60% centrais** (crop 9:16 do mobile)?
- [ ] Para 02 e 06: a câmera está perfeitamente nivelada e frontal? Todo o efeito depende de ela nunca se mover.
- [ ] Para 06: **o último frame funciona sozinho como still?** É ele que fica na tela sob `prefers-reduced-motion`, e é o que mais tempo passa visível na página inteira.
- [ ] Nenhum quadro encena desfecho clínico, ferimento, sofrimento animal ou promessa de recuperação?

## 4 · Pendências de asset

| # | Item | Impacto |
|---|---|---|
| 1 | **Foto noturna real da fachada**, da calçada oposta, com a vitrine acesa. Um celular, 22h, dois minutos. | Melhora hero e fecho de uma vez e elimina a graduação noturna sobre foto de dia — a única concessão visual do documento. |
| 2 | Confirmar se a **ala felina** tem porthole/condo (o `DETAIL` da 05 desenha assim). Se for baia comum, ajustar a ilustração. | Baixo — é ilustração esquemática, mas não vale desenhar equipamento que não existe. |
| 3 | Fotos reais de interior (recepção, internação, banho e tosa), se existirem. | Se existirem em qualidade utilizável, **substituem** os gerados de 02, 06 e 08. Foto real do lugar real sempre ganha de still gerado. |
