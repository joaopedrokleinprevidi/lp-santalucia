# Blueprint — Landing Page Santa Lúcia

> Documento final de direção. Substitui as três propostas anteriores.
> Fonte da verdade de fatos e cores: `design/design-system.json`.
> Nenhuma frase deste documento existe no site atual. Nenhum número aqui foi inventado.

---

## Decisão de direção

**Espinha dorsal: URGÊNCIA.** Venceu dois dos três juízes (Conversão 8,0 e Experiência 8,5) e perdeu o de Copy por meio ponto para a Jornada (8,0 × 8,5). A vitória não é de tom, é de arquitetura: é a única das três que trata os dois estados mentais como um **problema de roteamento**, com o garfo resolvido por peso de botão em vez de instrução escrita, e a única que projetou **duas rampas de Creative Budget** para dois públicos em vez de fingir que uma curva serve os dois.

O que foi enxertado, e de onde:

**Da JORNADA (vencedora de copy, 8,5)**
1. **O capítulo da internação, inteiro.** É o buraco confessado da Urgência — internação aparecia como bullet dentro de duas seções e nunca como capítulo. Entra como seção 06. O juiz de Experiência: *"é o melhor clipe individual dos três documentos, sem concorrência… responde a pergunta mais cara e mais mal respondida do mercado."* O juiz de Conversão validou o mecanismo (boletins com horário, número em vez de adjetivo).
2. **A regra de continuidade de câmera**, usada para consertar o pior erro da Urgência: *"a página entra pela porta duas vezes"* (Experiência). O clipe do hero agora **para na calçada** e a porta só abre uma vez na página inteira, no MAJOR da seção 04.
3. **Um CTA e nenhum secundário na seção de sinais.** *"Quem está aqui não escolhe entre dois canais, escolhe entre agir e não agir"* — o juiz de Conversão chamou de a decisão de CTA mais afiada dos três documentos.
4. **O telefone como texto em escala de display no hero**, não só dentro do botão. É o formato que as pessoas copiam.
5. Três linhas de copy: `Urgência não pega senha.`, `O exame não sai daqui. Nem a resposta.`, `Na dúvida, ligue. A pergunta é de graça.`

**Da CONFIANÇA (6,0 / 6,5 / 7,0 — perdeu, mas tinha as melhores peças soltas)**
6. **`Salvar contato` (.vcf)** como terciário no fecho. Os três juízes citaram: *"a única ação da página que converte quem não precisa da clínica hoje"*. Custo zero de motion, maior LTV do pacote.
7. **JSON-LD `VeterinaryCare`** com `openingHoursSpecification` 24/7 + **skip link `Ir direto para o telefone do plantão`** como primeiro elemento focável. Parte do tráfego converte antes de a página abrir.
8. **Disciplina técnica declarada por capítulo** (qual seção recebe sequência de frames, qual recebe `<video>` scrubbed, qual recebe still).
9. **Congelamento no último frame** sob `prefers-reduced-motion` — a versão estática conta o fim da história, não o começo.
10. Duas linhas: `É a diferença entre sedar em um pet shop e sedar em um hospital.` e `não uma baia comum com o nome de UTI`.

**Correções obrigatórias aplicadas (erros apontados nos três julgamentos)**
- **O hero foi reconstruído por orçamento de pixels.** A Urgência afirmava que o link de escape estava acima da dobra em 375 px; a soma dos elementos provava que não. Subheadline cortada de 159 → **91 caracteres**, faixa de credenciais empurrada para baixo da dobra, e o escape promovido de link terciário a **par segmentado `É agora` / `É rotina`** logo abaixo do eyebrow, alvos de 56 px, ambos visíveis em 375 × 667.
- **Hora fixa removida da H1.** `Três da manhã.` num eyebrow com relógio ao vivo se contradizia às 14h37.
- **Auto-plágio eliminado.** As frases recicladas entre seção e FAQ (`não é uma sala reservada…`, `não é sobreaviso`) aparecem agora **uma vez cada**, e o tique `atravessar a porta` — três vezes em posição de título — foi cortado inteiro.
- **Antíteses com travessão reduzidas a duas na página inteira.** A construção "não é X — é Y" é o carimbo de texto de IA em português e aparecia de quatro a cinco vezes em cada proposta.
- **`Ligar (54) 3025-2223`** substitui `Ligar agora` em todos os botões de telefone: elimina um passo de decisão e sobrevive a screenshot.
- **Registro estabilizado.** Sai `pelo qual`, `agenda de terceiro`, `o que está vindo`. Fica a voz do balcão.
- **Nada do site velho.** Nenhuma variação de `emitimos toda a documentação necessária`, `orientação da equipe para a melhor parada`, `12 razões`, `nossos diferenciais`.
- **Estacionamento nunca aparece como facilidade rotulada** (`unverified` no design system).
- **Desfecho clínico encenado foi cortado** do clipe da internação (o cão levantando a cabeça ao amanhecer). Ver §"O que ficou de fora".

---

## Experience Score alvo

### ★★★★☆ — com o capítulo 06 executado em padrão ★★★★★

A quinta estrela é recusada de propósito e a recusa tem um endereço: o visitante mais valioso desta página é alguém segurando o celular com uma mão, no 4G, dentro de um carro em movimento, de madrugada. Uma página flagship que demora quatro segundos para mostrar um telefone vale zero estrelas.

**O que a quinta estrela comprou de volta:**
- número de telefone no DOM antes de qualquer JavaScript executar
- primeira tela é poster estático; o clipe do hero só entra depois do LCP
- três seções (03, 09, 10) não têm clipe nenhum — decisão editorial, não economia
- FAQ legível com JS desligado (`<details>` nativo)

### Contagem de WOW

| Tipo | Onde | O que é |
|---|---|---|
| **1 MAJOR** | 04 `primeiros-minutos` | Plano-sequência pinado, sem corte, do carro à mesa de exame. O scroll é a caminhada. |
| **3 MEDIUM** | 02 `dois-caminhos` · 06 `quando-precisa-ficar` · 08 `rotina-e-estetica` | Time-lapse do balcão · a madrugada da internação (pico **técnico** da página) · o cão que entra molhado e sai seco |
| **6 SMALL** | 01, 03, 05, 07, 09, 11 | Dolly-in contido · coreografia tipográfica · axonométrica que acende sala a sala · travessia da parede · revelação por máscara curva · dolly-out final |
| **2 SILÊNCIOS** | 09, 10 | Zero clipe. Tipografia sustentando 14% do scroll. |

**Nota de direção sobre o pico duplo.** O MAJOR narrativo é a 04, porque derruba o medo de *todo* visitante de pronto-socorro ("vou ser atendido ou vou sentar numa cadeira de plástico?"). O pico **técnico** é a 06, porque é o único clipe da página que é impossível substituir por still e o mais barato de extrair (câmera imóvel, só a luz muda). Pico narrativo e pico técnico em capítulos diferentes é decisão, não indecisão: os 100% de budget vão para a 04, os 90% para a 06, e a única sequência de frames da página vai para a 06.

### Tipos de animação autorizados, por capítulo

| Técnica | Capítulos | Regra |
|---|---|---|
| **Sequência de frames** (canvas, pin + scrub) | **06, e só a 06** | Gated por `IntersectionObserver` + limiar de conexão. Abaixo do limiar, poster + os cinco itens em texto. |
| **`<video>` com `currentTime` mapeado ao scroll** | 01, 02, 04, 07, 08, 11 | Tomada única, movimento monotônico, sem corte. Corte dentro de clipe scrubbed lê como bug. |
| **Ilustração animada em DOM/SVG** | 05 | Sem vídeo. Salas acendendo por `stagger`. |
| **Still com poster + coreografia tipográfica** | 03 | Se a seção declara que o WOW é a ausência de movimento, ela não paga produção de clipe. |
| **Zero mídia** | 09, 10 | Declarado. |

### Orçamento de peso

| Item | Teto |
|---|---|
| LCP em 4G | **< 2,0 s** |
| CLS | **0** |
| Mídia acima da dobra | **≤ 400 KB** |
| Qualquer `<video>` individual | **≤ 900 KB** |
| Sequência de frames da 06 | ≤ 3,5 MB desktop · ≤ 2,0 MB mobile |
| **Total de mídia da página** | **≤ 12 MB desktop · ≤ 5 MB mobile** |
| Carregamento | `IntersectionObserver`, uma seção à frente. Nada de vídeo antes do primeiro scroll. |

### Acessibilidade

Um `<h1>`, no hero. `<section aria-labelledby>`. Ordem do DOM = ordem narrativa, nunca `order` de flexbox. Telefone, endereço e horário como texto selecionável, nunca dentro de imagem. Sob `prefers-reduced-motion` todo clipe congela **no último frame** e o protocolo da 04 vira `<ol>` numerada. Barra sticky recebe `inert` quando invisível e respeita `env(safe-area-inset-bottom)`.

---

## Como a página serve os dois visitantes

O problema não se resolve com tom. Resolve-se com roteamento. Cinco mecanismos, todos verificáveis em tela.

### 1. O par segmentado no hero, acima da dobra em 375 px

Logo abaixo do eyebrow, antes da H1 em ordem visual no mobile: dois alvos de 56 px, lado a lado.

`É agora` (sólido âmbar sobre roxo, rola 0 px — já é a seção certa) · `É rotina` (contorno, âncora para `#rotina-e-estetica`)

Um toque pula sete capítulos. O escape do público de rotina existe **no pixel 0**, que é onde o público de emergência já tinha o dele. Foi o ponto exato em que a proposta original mentiu para si mesma, e é o ponto em que este blueprint se compromete: **se o par segmentado não couber acima da dobra em 375 × 667, corta-se a faixa de credenciais e a linha de endereço, não o par.**

### 2. O canal é o garfo, e o peso dos botões inverte

| Estado | Canal | Onde é primário |
|---|---|---|
| Pânico | `tel:` — ninguém digita com a mão tremendo | Topo da página (01, 03) |
| Rotina | `wa.me` — assíncrono, confirma leitura, já está aberto no telefone | Fim da página (08, 11) |

No hero, `Ligar (54) 3025-2223` é sólido e `Falar no WhatsApp` é contorno. Na 11, invertido. O motivo é operacional: quem estava em pânico converteu na primeira tela e nunca chegou ao fim; quem chegou ao fim rolou onze capítulos, logo não está em emergência. **O visitante nunca precisa se declarar — o botão que a mão dele já ia pegar já o classificou.**

### 3. A quebra tonal, e ela é visível

Capítulos 01–06 vivem em roxo-noite (`#482A78` / `#2D1542`). O handoff de fundo acontece **dentro do clipe da 06**, junto com o amanhecer da janela da internação. De 07 em diante a página é creme `#F7F4EF`, luz de dia, âmbar solto. Quem chega na 08 pelo atalho do hero aterrissa num ambiente que não parece pronto-socorro. Mesma marca, outro clima. A moldura de emergência passa a ser lida como **capítulo**, não como a temperatura da marca inteira.

### 4. A frase que junta os dois públicos numa linha

Na 08: *"A clínica que existe para as três da manhã também é onde o seu pet toma banho."* Reenquadra retroativamente todo o arco de urgência como **benefício** para quem só queria marcar um banho, em vez de ruído que ele teve que rolar. E a bala que fecha: **banho com sedação, dentro de uma clínica com centro cirúrgico e UTI no mesmo endereço.** Só uma clínica 24h consegue oferecer isso.

### 5. A via expressa que existe fora da narrativa

| Posição | Elemento | Visível a partir de |
|---|---|---|
| Antes de qualquer pixel | JSON-LD `VeterinaryCare` com `telephone`, `address`, `openingHoursSpecification` 24/7 | SERP do Google |
| Primeiro elemento focável | Skip link `Ir direto para o telefone do plantão` | Um Tab |
| Header, todas as telas | Pill âmbar sobre roxo: `Plantão 24h · (54) 3025-2223` | Pixel 0, permanente |
| Hero | O número em escala `stat`, texto selecionável no DOM | Pixel 0, sem rolar |
| Barra sticky mobile (< 1024 px) | `Ligar` sólido · `WhatsApp` contorno · ícone de rota | **Pixel 0**, some quando `#contato` cruza 80% da viewport |

Sem JavaScript, sem CSS e sem mídia carregada, o telefone continua sendo o primeiro link clicável do documento, o FAQ continua abrindo e o endereço continua legível.

---

## Alocação do Creative Budget

```
ARCO 1 — EMERGÊNCIA (noite)
01 ███░░░░░░░  30%   hero contido de propósito
02 ████░░░░░░  45%   ◇ time-lapse do balcão: a tese da página numa imagem
03 ██░░░░░░░░  15%   still + coreografia tipográfica. O recuo é a ideia.
04 ██████████ 100%   ◆ MAJOR — plano-sequência pinado
05 █████░░░░░  50%   axonométrica ilustrada, troca de registro visual
06 █████████░  90%   ◇ pico TÉCNICO — a única sequência de frames da página

ARCO 2 — ROTINA + FECHO (dia → noite)
07 ██████░░░░  60%   travessia da parede, noite vira dia
08 ███████░░░  65%   ◇ molhado entra, seco sai, um plano só
09 ██░░░░░░░░  20%   revelação por máscara curva, sem clipe
10 █░░░░░░░░░  10%   silêncio deliberado
11 █████░░░░░  45%   dolly-out: a última janela acesa
```

| Trecho | % do budget | O que existe ali |
|---|---|---|
| 01 `agora` | 30% | Dolly-in lento até a calçada. A porta **não** abre. |
| 02 `dois-caminhos` | 45% | Câmera travada, time-lapse do mesmo balcão de 3h a 15h. Densidade de movimento baixíssima, densidade de ideia altíssima. |
| 03 `e-grave` | 15% | Nenhum clipe. Uma linha acesa por vez, o resto em 18% de opacidade. |
| 04 `primeiros-minutos` | **100%** | Plano-sequência único, pinado, scrubbed. Cinco passos entram quando a câmera passa pelo lugar físico correspondente. |
| 05 `estrutura` | 50% | Ilustração axonométrica do prédio em corte. As salas acendem de baixo para cima. Única troca de registro visual da página. |
| 06 `quando-precisa-ficar` | 90% | A única sequência de frames. Câmera imóvel, só o tempo passa. O handoff de fundo roxo→creme acontece dentro dela. |
| 07 `ala-felina` | 60% | Travelling lateral que atravessa a parede em match-cut. Primeira luz natural da página. |
| 08 `rotina-e-estetica` | 65% | Um plano contínuo: entra molhado, sai seco. Rolar para cima molha o cachorro de novo. |
| 09 `depoimentos` | 20% | Zero clipe. `clip-path` no formato do swoosh, stagger 0,10 s. |
| 10 `faq` | 10% | Zero clipe. O glifo de expandir é o coração da marca girando 45°. |
| 11 `contato` | 45% | Dolly-out pela rua. Recuo na mesma fachada é um medium, e está precificado como medium. |

**Duas deformações declaradas** em relação à progressão 20→40→60→80→100 do CLAUDE.md:

**O vale na 03.** É a seção de maior intenção da página e recebe o menor orçamento de todas. Motion sobre uma lista de sinais de emergência é motion competindo com a leitura que salva o animal. E se a própria seção declara que o WOW é a ausência de movimento, ela pede um still, não um clipe — pagar produção de Flow ali é pagar por nada.

**Duas cristas em vez de uma escalada.** Escalada monotônica coloca o pico no fim, onde o tutor em pânico jamais chega. A crista de emergência é a 04 (46% do scroll); a crista de rotina é a 08 (77%), que é quem de fato rola até lá. Duas plateias, dois cumes, um orçamento. O vale real está em 09–10, e ele existe para que os 45% da 11 soem como fechamento e não como eco.

---

## Seções

---

### 01 · `agora`

| Campo | Valor |
|---|---|
| **id** | `agora` |
| **Papel** | Responder em meio segundo à única pergunta que existe de madrugada: tem gente aí neste momento? E deixar o visitante calmo escapar sem se sentir expulso. |
| **Temperatura** | **8** — alta porque espelha quem chega. Não é 9: o pico pertence à 03. |
| **Scroll** | **11%** · `scroll={2.6}` · `scrollMobile={1.8}` |
| **WOW** | small (contenção deliberada) |
| **CTA** | 1 primário `tel:` + 1 secundário `wa.me` + par segmentado |

**EYEBROW** *(26)*
`Plantão 24h · aberto agora`
→ ponto âmbar pulsando à esquerda. Sem relógio ao vivo: a H1 não cita hora, então nada precisa se sincronizar com nada.

**HEADLINE** *(44)*
**Seu pet passou mal. Tem gente acordada aqui.**

**SUBHEADLINE** *(91)*
Pronto-socorro veterinário 24 horas em Caxias do Sul. Urgência não precisa de hora marcada.

**CORPO**

Par segmentado, imediatamente abaixo do eyebrow, dois alvos de 56 px:
> `É agora` · `É rotina`

Telefone em escala `stat`, texto selecionável, também `tel:`:
> **(54) 3025-2223**

Endereço em uma linha, texto copiável:
> R. Jacob Luchesi, 3230 — Santa Lúcia, Caxias do Sul

Abaixo da dobra, tipo pequeno, baixo contraste:
> Desde 2015 · mais de 70.000 pets atendidos · 4,8 no Google, mais de 1.100 avaliações

**MICROCOPY DE CTA**
- Primário: `Ligar (54) 3025-2223` — `tel:+555430252223`, sólido âmbar sobre roxo
- Secundário: `Falar no WhatsApp` — contorno âmbar, `origin: hero`
- Link de texto, discreto: `Como chegar`

**O QUE SE MOVE**
*Imagem —* a fachada da clínica à noite, vista da calçada oposta. Asfalto molhado de garoa, letreiro roxo aceso, uma fatia de luz âmbar vazando por baixo da porta de vidro e caindo no chão. O resto do quarteirão apagado. Sem pessoas no quadro.
*Clipe (Flow) —* dolly-in lento e contínuo, da rua em direção à porta, velocidade constante, sem corte, câmera estável. **O clipe termina na calçada, com a mão da câmera parada diante do vidro. A porta não abre.**
*Por que —* é aqui que o pior erro da proposta original foi consertado. A página entrava pela porta duas vezes, e quando o MAJOR chegava aos 46% do scroll o gesto já tinha sido gasto aos 11%. Agora existe **uma** porta e ela abre **uma** vez, na 04. O hero cria a pergunta; ele não gasta a resposta.

---

### 02 · `dois-caminhos`

| Campo | Valor |
|---|---|
| **id** | `dois-caminhos` |
| **Papel** | Confirmar o visitante em pânico e resgatar o visitante calmo na mesma tela, sem que nenhum se sinta o público errado. O hero já roteou; esta seção tranquiliza. |
| **Temperatura** | **5** — queda de três pontos. Tensão sem alívio fecha aba. |
| **Scroll** | **6%** · `scroll={1.4}` · `scrollMobile={0.9}` |
| **WOW** | **medium** |
| **CTA** | 1 secundário `wa.me` no card de rotina |

**EYEBROW** *(26)*
`A mesma sala, dois motivos`

**HEADLINE** *(34)*
**A mesma recepção às 3h e às 15h.**

**SUBHEADLINE** *(129)*
Muda quem entra e muda o motivo. Quem tem pressa liga e vem. Quem está organizando a semana manda mensagem e escolhe o horário.

**CORPO — dois cards, pesos visuais diferentes**

**Card A — "É agora"** *(fundo roxo `#482A78`, contorno âmbar 3px, radius 999px no rodapé — `faixa-pill-contornada`)*
- Chegue direto. Não precisa agendar e não precisa avisar antes.
- Se puder, ligue no caminho: a equipe fica sabendo que vocês estão vindo.
- Tem equipe dentro da clínica neste momento, em qualquer hora do dia ou da noite.
- **(54) 3025-2223**

**Card B — "É rotina"** *(card branco, radius 24px, sombra difusa)*
- Banho, tosa, vacina, castração, consulta marcada e microchipagem.
- Manda mensagem e a gente encaixa no horário que couber na sua semana.
- Pet shop e farmácia veterinária no mesmo endereço.

**MICROCOPY DE CTA**
- Card B: `Agendar no WhatsApp` — contorno, `origin: rotina`
- Card A não tem botão: o número já está em texto dentro dele, em tipo grande. Dois botões idênticos lado a lado custam mais em decisão do que a própria ação.

**O QUE SE MOVE**
*Imagem —* o balcão da recepção, enquadramento fixo, frontal, uma única imagem-base. Plafon âmbar, vidro escuro atrás, ninguém na sala.
*Clipe (Flow) —* **time-lapse do mesmo balcão indo das 3h da manhã às 15h da tarde, num plano só.** Começa em luz âmbar de madrugada e termina em sol de janela, creme, movimento de pessoas passando em rastro. A luz é o que muda; o cenário é o mesmo. A câmera nunca se move.
*Por que —* é o conceito da página inteira dito em uma imagem: **é a mesma sala, muda o motivo pelo qual você entra nela.** O scroll controla a hora do dia. Movimento monotônico, perfeitamente reversível, e o clipe mais barato de produzir depois do da 06.
*Nota de continuidade —* esta seção está deliberadamente **fora** da continuidade de câmera 01 → 04 → 11. É outra gramática (câmera travada, tempo comprimido), e a diferença é o que impede a página de virar cinco planos iguais de alguém andando pela clínica.

---

### 03 · `e-grave`

| Campo | Valor |
|---|---|
| **id** | `e-grave` |
| **Papel** | Responder à pergunta que o tutor está digitando no Google em outra aba neste exato segundo. É o conteúdo de maior intenção da página. |
| **Temperatura** | **9 — o pico único da página.** Ele está olhando para o próprio animal enquanto lê. |
| **Scroll** | **10%** · `scroll={2.4}` · `scrollMobile={1.6}` |
| **WOW** | small — **por escolha, não por falta de orçamento** |
| **CTA** | **1 primário `tel:`, sem secundário** |

**EYEBROW** *(22)*
`Sinais que não esperam`

**HEADLINE** *(33)*
**Se você está vendo isto, é agora.**

**SUBHEADLINE** *(147)*
Nenhum destes sinais melhora sozinho até de manhã. Se um deles bate com o que você está vendo na sua frente, pegue o pet e venha para a clínica.

**CORPO — a lista, em tipo grande, uma linha acesa por vez**

- Respiração difícil, arrastada, ou de boca aberta parado no chão
- Língua ou gengiva arroxeada, ou muito pálida
- Barriga inchada e dura, com esforço para vomitar e nada sai
- Convulsão, tremor no corpo inteiro, ou desmaio que passou e voltou
- Não consegue urinar, principalmente gato macho
- Comeu o que não era comida: remédio de gente, chocolate, uva, osso, veneno de rato
- Sangramento que não estanca, atropelamento, queda de altura ou briga
- Trabalho de parto que passou de duas horas sem nascer filhote
- Não levanta, não come, não bebe e não responde ao próprio nome

**Fecho da seção, em destaque:**
> **Na dúvida, ligue. A pergunta é de graça.**

**Nota fina, obrigatória, em `small`:**
> Esta lista serve para você decidir procurar atendimento. Ela não é diagnóstico.

**MICROCOPY DE CTA**
- Primário, bloco de largura total: `Ligar (54) 3025-2223` — `tel:`, `origin: sinais`
- **Nenhum secundário.** Quem está aqui não escolhe entre dois canais; escolhe entre agir e não agir.

**O QUE SE MOVE**
*Imagem —* **still, não clipe.** O corredor da clínica visto de quem acabou de entrar: vazio, profundidade longa, luminárias âmbar, paredes roxo profundo. Gerada no GPT, servida como `.webp` com poster único.
*Movimento —* nenhum vídeo. A coreografia é **tipográfica**: conforme o scroll avança, a linha ativa fica em 100% de opacidade e todas as outras caem para 18%. Ler a lista é como ler com lanterna. Ao chegar na última linha, tudo acende de uma vez junto com o botão de ligar.
*Por que sem clipe —* a versão anterior alocava produção de Flow para um "push-in de 2% sem corte, sem partícula, sem brilho" e declarava no mesmo parágrafo que o WOW era a ausência de movimento. Era pagar produção por nada. Em beat 9, no auge do medo, espetáculo é ofensivo — e a economia aqui é o que financia a sequência de frames da 06.

---

### 04 · `primeiros-minutos`

| Campo | Valor |
|---|---|
| **id** | `primeiros-minutos` |
| **Papel** | Substituir medo por protocolo. A pergunta real não é "vocês são bons", é "vou ser atendido ou vou sentar numa cadeira de plástico e esperar?". |
| **Temperatura** | **5** — cai quatro pontos, exatamente onde deve cair. |
| **Scroll** | **14%** (a maior da página) · `scroll={3.4}` · `scrollMobile={2.4}` |
| **WOW** | **◆ MAJOR** |
| **CTA** | 1 primário (mapa) + 1 secundário (`tel:`) |

**EYEBROW** *(20)*
`Os primeiros minutos`

**HEADLINE** *(24)*
**Urgência não pega senha.**

**SUBHEADLINE** *(134)*
Quem chega mais grave é avaliado primeiro, mesmo tendo chegado depois. E quem está esperando fica sabendo por que a fila mudou de ordem.

**CORPO — o protocolo, cinco passos, sem tempo numérico inventado**

**1. Você chega**
Não precisa ter ligado, não precisa ter agendado, não precisa ser cliente. A recepção do plantão não fecha em nenhum dia do ano.

**2. O pet entra na frente do papel**
Alguém olha o seu animal antes de você preencher qualquer coisa. Cadastro depois.

**3. Triagem**
Sinais vitais, temperatura e nível de dor medidos logo na entrada. É essa medida que define a ordem, e não o relógio da chegada.

**4. Exame no mesmo prédio**
Sangue, raio-X e ultrassom saem aqui dentro. Você não sai daqui para buscar resultado em outro endereço e voltar.

**5. A conversa**
Alguém explica o que encontrou, o que vai fazer e o que ainda não dá para saber. Em português, sem termo que você tenha que pesquisar depois. Nada acontece sem a sua autorização, e o orçamento vem antes.

**MICROCOPY DE CTA**
- Primário: `Como chegar` — Google Maps
- Secundário: `Ligar (54) 3025-2223` — `tel:`, `origin: caminho`

> Aqui o medo está no ponto mais baixo e a confiança no mais alto. Mas a ação genuinamente útil neste ponto não é ligar de novo: é **pegar o carro**. Por isso o mapa é primário e o telefone é o gesto de quem já está dirigindo. Nenhum outro lugar da página repete esse par.

**O QUE SE MOVE — ◆ MAJOR**
*Imagem —* POV do tutor, noite, primeira pessoa, altura de olho.
*Clipe (Flow) —* **um plano-sequência único, sem nenhum corte**, retomando **exatamente** do quadro onde o clipe do hero parou: a mão da câmera diante do vidro. A porta desliza e abre — **a única vez na página inteira** — e a câmera continua: entrada → balcão da recepção → as mãos recebendo a caixa de transporte → corredor → a luz da sala de exame acendendo no fim do percurso. Câmera fluida, levemente na mão, velocidade constante. Práticas âmbar por dentro, noite roxa nas janelas.
*Mecânica —* capítulo **pinado**, clipe raspado frame a frame pelo scroll. Cada um dos cinco passos entra em texto no momento exato em que a câmera passa pelo lugar físico correspondente. **O scroll é a caminhada.** O visitante termina a seção tendo sensorialmente feito o trajeto, e a decisão de sair de casa fica menor.
*Requisito de produção —* movimento ininterrupto do começo ao fim. Qualquer corte quebra o scrub e o efeito morre. Nenhum rosto reconhecível: mãos, jaleco, costas, silhueta fora de foco.

---

### 05 · `estrutura`

| Campo | Valor |
|---|---|
| **id** | `estrutura` |
| **Papel** | Provar que o prédio dá conta do caso, sem adjetivo: equipamento nomeado e a consequência de cada um. |
| **Temperatura** | **3** — a mais fria do arco de emergência. É leitura de informação. |
| **Scroll** | **9%** · `scroll={2.2}` · `scrollMobile={1.5}` |
| **WOW** | small |
| **CTA** | nenhum |

**EYEBROW** *(20)*
`Diagnóstico no local`

**HEADLINE** *(38)*
**O exame não sai daqui. Nem a resposta.**

**SUBHEADLINE** *(145)*
Laboratório, raio-X e ultrassom ficam no mesmo prédio em que o seu pet está. A amostra não viaja, não entra em fila de coleta e não volta em dois dias.

**CORPO — sempre estrutura → consequência**

- **Laboratório interno próprio** — hemograma e bioquímico processados aqui. O resultado sai em minutos, na mesma madrugada.
- **Raio-X** — trauma, obstrução e corpo estranho aparecem antes de a suspeita virar espera.
- **Ultrassonografia** — o abdome é olhado por dentro sem abrir nada e sem depender de agenda de outra clínica.
- **Centro cirúrgico e anestesiologia** — se o caso virar cirurgia, ela acontece no mesmo prédio, na mesma noite, com quem já examinou.
- **UTI veterinária** — suporte e monitoramento contínuo, não uma baia comum com o nome de UTI.
- **Farmácia veterinária 24h** — o remédio prescrito às 4h da manhã é dispensado às 4h da manhã, no balcão da frente.

**MICROCOPY DE CTA**
Nenhum. É a única seção longa sem pedido — o visitante precisa de um trecho para só absorver.

**O QUE SE MOVE**
*Imagem —* **troca deliberada de registro visual: sai a fotografia, entra ilustração.** Corte transversal do prédio em axonometria quente, na paleta da marca — paredes `#603084`, luz `#FCB400`, chão `#F7F4EF` — com as salas rotuladas: laboratório, imagem, centro cirúrgico, UTI, internação canina, internação felina, farmácia.
*Movimento —* sem vídeo. As salas **acendem uma a uma**, de baixo para cima, conforme o scroll sobe, com `stagger` de 0,10 s e o easing `cubic-bezier(0.16, 1, 0.3, 1)`. Parallax sutil entre os andares. Patas em lilás `#E4D2F0` a 10% derivando muito lentamente atrás do corte.
*Por que o registro muda —* quatro capítulos seguidos na mesma gramática fotográfica viram papel de parede. Esta é a única troca de registro da página, é mais legível em 375 px do que qualquer corredor filmado, e entrega o que a foto não entrega: ver o prédio inteiro de uma vez.

---

### 06 · `quando-precisa-ficar`

| Campo | Valor |
|---|---|
| **id** | `quando-precisa-ficar` |
| **Papel** | Responder a pergunta que nenhum site de clínica responde bem: *e se ele tiver que ficar aí sozinho?* A resposta não é a estrutura, é o boletim. |
| **Temperatura** | **7** — alta por peso, não por alarme. |
| **Scroll** | **11%** · `scroll={2.6}` · `scrollMobile={1.8}` |
| **WOW** | **medium na hierarquia narrativa · PICO TÉCNICO da página** |
| **CTA** | **nenhum, deliberadamente** |

**EYEBROW** *(20)*
`Quando precisa ficar`

**HEADLINE** *(39)*
**Ele dorme aqui. Você acompanha de casa.**

**SUBHEADLINE** *(151)*
Ninguém interna e vai embora tranquilo. Por isso a internação manda boletim no seu WhatsApp durante o plantão, sem você precisar ligar para perguntar.

**CORPO**

- **Internação monitorada** — medicação, controle de dor e sinais vitais acompanhados durante todo o plantão.
- **UTI veterinária** — para o caso que não pode ficar sem vigilância até de manhã.
- **Centro cirúrgico no mesmo endereço** — se precisar operar de madrugada, ninguém é transferido para outro lugar.
- **Alas separadas por espécie** — cães de um lado, gatos do outro, também de madrugada.
- **Boletim no WhatsApp** — o que foi feito e como ele está sendo acompanhado, com foto ou vídeo quando faz sentido.

**Os três boletins que entram sobre o clipe** *(texto de processo, nunca de prognóstico)*:
> `03h20` — Medicação da madrugada aplicada. Ele está monitorado.
> `05h50` — Acesso trocado, hidratação seguindo. Sem intercorrência até agora.
> `07h40` — Foto dele agora. A equipe do dia já assumiu o acompanhamento.

**MICROCOPY DE CTA**
Nenhum. É o único capítulo de alta temperatura sem botão. Pedir uma ação no ponto em que o visitante está imaginando o próprio animal internado transforma empatia em venda, e ele sente. O silêncio aqui é o que dá crédito ao CTA da 08.

**O QUE SE MOVE — PICO TÉCNICO**
*Imagem —* sala de internação à noite. Um cão dormindo enrolado num cobertor dentro de uma baia, monitor com linha verde ao fundo, uma janela alta à esquerda com a rua escura do lado de fora. Luz baixa, roxo `#2D1542` dominante.
*Clipe (Flow) —* **uma tomada única e completamente imóvel em que só o tempo passa.** A luz da janela vai de preto de 2h para azul de 6h para dourado de 8h. O monitor pulsa. Nada mais se move no quadro.
*Mecânica —* **esta é a única seção da página autorizada a usar sequência de frames** com `pin` + `scrub` em canvas. É o clipe mais barato de extrair (câmera travada, só a luz muda) e o único impossível de substituir por still: a mudança ao longo do tempo **é** o conteúdo. O scroll é o relógio — rolar é a madrugada passando. Os três boletins entram em pontos fixos do progresso.
*Handoff de fundo —* a transição roxo-noite → creme-dia da página inteira acontece **dentro** deste capítulo, junto com o amanhecer. O capítulo 07 já nasce em outro dia.
*Corte de risco —* o plano **termina com o cão ainda dormindo**. Ele não levanta a cabeça, não olha para a porta, não acorda. Ver §"O que ficou de fora".
*Gate obrigatório —* só inicializa acima de um limiar de conexão e viewport. Abaixo dele, e sob `prefers-reduced-motion`, entrega o **último frame** (a janela dourada) mais os cinco itens em texto e os três boletins estáticos. Nenhuma informação se perde.

---

### 07 · `ala-felina`

| Campo | Valor |
|---|---|
| **id** | `ala-felina` |
| **Papel** | O diferencial estrutural mais difícil de copiar. Para o tutor de gato, esta seção sozinha decide a clínica. |
| **Temperatura** | **5** — o assunto dela é silêncio, e a página já amanheceu. |
| **Scroll** | **8%** · `scroll={1.9}` · `scrollMobile={1.3}` |
| **WOW** | small |
| **CTA** | 1 secundário `wa.me` |

**EYEBROW** *(10)*
`Ala felina`

**HEADLINE** *(50)*
**Cão não escuta gato, gato não sente cheiro de cão.**

**SUBHEADLINE** *(133)*
Recepção, consultório e internação exclusivos para felinos. Da porta de entrada até a alta, o seu gato não cruza com nenhum cachorro.

**CORPO**

- **Recepção só de gato** — a espera acontece em outra sala, sem focinho estranho encostando na caixa de transporte.
- **Consultório só de gato** — mesa, material e ambiente usados só por felinos. Metade do estresse de um gato em clínica é olfativo.
- **Internação só de gato** — a recuperação acontece longe do latido e do movimento da ala canina.

**Linha de fechamento, em destaque:**
> **Gato estressado esconde sintoma. Tirar o cachorro do caminho não é conforto — é parte do exame.**

**Faixa pill roxa com contorno âmbar, ao pé do capítulo:**
> **Pet não convencional também tem plantão 24h aqui.** Se o seu não é cão nem gato, manda mensagem antes de vir para a equipe orientar o transporte.

**MICROCOPY DE CTA**
- Secundário: `Agendar na ala felina` — `wa.me`, `origin: felina`, mensagem pré-preenchida: *"Olá! Tenho um gato e queria agendar na ala felina."*

**O QUE SE MOVE**
*Imagem —* uma parede, enquadramento frontal. À esquerda, através de vidro jateado, silhuetas em movimento e desordem quente: o lado canino. À direita, um gato sentado imóvel numa almofada, luz de janela, silêncio absoluto.
*Clipe (Flow) —* travelling lateral que **atravessa a parede** num match-cut contínuo, do lado barulhento para o lado quieto. Conforme a câmera cruza, o lado esquerdo desfoca e dessatura, o lado direito ganha nitidez e claridade.
*Por que —* a seção diz "são dois mundos separados" e o clipe faz o visitante atravessar a separação com o próprio dedo. O movimento **é** o argumento, e o visitante não lê que as alas são separadas: ele as vê se separando.

---

### 08 · `rotina-e-estetica`

| Campo | Valor |
|---|---|
| **id** | `rotina-e-estetica` |
| **Papel** | Pagar integralmente a dívida do arquétipo. É o destino do `É rotina` do hero e o ponto onde o público de rotina converte. |
| **Temperatura** | **3** — leve, doméstica, de sábado de manhã. O contraste com a 03 é o argumento. |
| **Scroll** | **10%** · `scroll={2.4}` · `scrollMobile={1.6}` |
| **WOW** | **medium** |
| **CTA** | 1 primário `wa.me` |

**EYEBROW** *(17)*
`Rotina e estética`

**HEADLINE** *(33)*
**Banho de sábado, vacina de terça.**

**SUBHEADLINE** *(131)*
A clínica que existe para as três da manhã também é onde o seu pet toma banho. O que muda é o motivo da visita, e só isso.

**CORPO**

**Card em destaque, o mais importante da seção — `faixa-pill-roxa`, palavra em âmbar:**
> **Banho com sedação.** Para o pet que simplesmente não tolera o procedimento acordado: idoso, com dor, ou que entra em pânico. Feito com avaliação veterinária antes e com médico veterinário no prédio do começo ao fim. **É a diferença entre sedar em um pet shop e sedar em um hospital.**

**Duas colunas, divisores tracejados em lilás `#E4D2F0`:**

*Estética*
- Banho
- Tosa
- Corte de unha
- Matização
- Hidratação
- Hidratação de coxins

*Rotina clínica*
- Consulta agendada e retorno
- Vacinação
- Castração
- Microchipagem
- Atestados e laudos
- Pet shop e farmácia veterinária no mesmo endereço

**MICROCOPY DE CTA**
- Primário: `Agendar no WhatsApp` — `origin: rotina`, mensagem pré-preenchida: *"Olá! Vim pelo site e queria agendar banho, tosa ou consulta de rotina."*

**O QUE SE MOVE — MEDIUM**
*Imagem —* a sala de banho e tosa em luz de manhã, azulejo claro, espuma, bolhas suspensas, um cão de porte médio dentro da banheira. Creme e âmbar dominando, roxo reduzido a acento. Nada de clínico no quadro. Mãos apenas, sem rosto.
*Clipe (Flow) —* **um plano contínuo em que o cão entra molhado e sai seco.** Sem corte: água escorrendo, espuma subindo, o secador levantando o pelo, o pelo assentando limpo e cheio. Bolhas subindo durante toda a passagem.
*A delícia escondida —* como o clipe é raspado pelo scroll, **rolar para cima molha o cachorro de novo.** Ninguém avisa. Quem descobre, brinca, e o tempo na página sobe.
*Complemento em DOM —* os chips de serviço em marquee horizontal lento, com a velocidade modulada por `ScrollTrigger.velocity`. Foto de pet, quando estática nesta seção, recebe o tratamento `arco-organico` — máscara arredondada assimétrica com contorno âmbar deslocado atrás.

---

### 09 · `depoimentos`

| Campo | Valor |
|---|---|
| **id** | `depoimentos` |
| **Papel** | Responder "já resolveram um caso como o meu?" com a voz de outra pessoa. Vem tarde de propósito: em serviço local, depoimento antes do método é enfeite e é pulado. |
| **Temperatura** | **6** |
| **Scroll** | **7%** · `scroll={1.7}` · `scrollMobile={1.1}` |
| **WOW** | small |
| **CTA** | nenhum |

**EYEBROW** *(13)*
`4,8 no Google`

**HEADLINE** *(43)*
**Mais de 1.100 pessoas já contaram como foi.**

**SUBHEADLINE** *(126)*
Nenhuma linha abaixo foi escrita por nós. São avaliações públicas de tutores de Caxias, copiadas do jeito exato que eles deixaram.

**CORPO — avaliações reais, nesta ordem exata**

> "Equipe extremamente humana, atenciosa e profissional em um dos momentos mais difíceis da minha vida. Gratidão imensa por todo o cuidado e dedicação."
> — **Paproski**

> "As meninas sempre muito queridas e atenciosas, desde a recepção até o atendimento. Fizeram tudo o que foi possível para a minha gatinha Suzi. Somente agradecer por todo cuidado e carinho."
> — **Marcelle Scheinpflug**

> "Meu pequeno foi super bem atendido e voltou para casa recuperado. Atendimento com atenção, carinho e profissionalismo."
> — **Jane Rodrigues**

> "Clínica maravilhosa! Estrutura ótima, colaboradores educados e atenciosos, sempre com explicações claras e muito carinho no atendimento. Recomendo imensamente!"
> — **Rafaela Pereira Prigol**

> "Adorei o atendimento. Extremamente humano e gentil. A estrutura do pet é fenomenal."
> — **Arthur Lucena**

*Decisão editorial declarada:* o depoimento do Paproski fala de um dos momentos mais difíceis da vida dele, provavelmente uma perda. Ele fica, e fica em primeiro. Uma clínica de emergência que só mostra finais felizes está mentindo, e o tutor em pânico reconhece a mentira. Manter esse depoimento é o que dá credibilidade aos outros quatro, e ele não promete desfecho clínico nenhum.

Selo ao lado da lista: **4,8 / 5 no Google · mais de 1.100 avaliações**

**MICROCOPY DE CTA**
Nenhum.

**O QUE SE MOVE**
**Nenhum clipe. Esta seção é tipografia, e a declaração é explícita.** A página foi cinemática por oito capítulos; aqui o visitante precisa **ler**, e vídeo atrás de texto de depoimento rouba a leitura.
*Movimento —* fundo creme sólido `#F7F4EF`, cards brancos radius 24px entrando pela assinatura de motion da marca: **revelação por máscara curva** (`clip-path` no formato do swoosh), nunca fade retangular. Stagger de 0,10 s. Entre os cards, o divisor `regua-ambar-coracao`. Parallax sutil de velocidade entre a coluna de cards e o fundo.

---

### 10 · `faq`

| Campo | Valor |
|---|---|
| **id** | `faq` |
| **Papel** | Derrubar as objeções operacionais que travam a mensagem no WhatsApp. Não é conteúdo, é desobstrução. |
| **Temperatura** | **2** — o momento mais frio da página, de propósito, logo antes do fecho quente. |
| **Scroll** | **7%** · `scroll={1.7}` · `scrollMobile={1.1}` |
| **WOW** | nenhum |
| **CTA** | nenhum |

**EYEBROW** *(17)*
`Perguntas diretas`

**HEADLINE** *(31)*
**O que a recepção mais responde.**

**SUBHEADLINE** *(95)*
Se a sua dúvida não estiver aqui, manda no WhatsApp. Alguém responde, inclusive de madrugada.

**CORPO — sete perguntas em `<details>` nativo, ordenadas por custo de abandono, com JSON-LD `FAQPage`**

**1. Vocês estão abertos agora?**
Sim. 24 horas, todos os dias do ano, feriado e madrugada de domingo incluídos. Não é sobreaviso: existe equipe dentro do prédio na hora em que você ligar.

**2. Quanto custa e como se paga?**
O valor da consulta de plantão é informado por telefone antes de você sair de casa. Exames, internação e procedimentos são orçados à parte e só acontecem depois da sua autorização. Aceitamos Pix, cartão de crédito e débito, e dinheiro. O atendimento é particular: você sai com nota e com o relatório do que foi feito, que é o que o seu plano pede para reembolsar.

**3. Preciso agendar ou posso chegar direto?**
Em urgência, chegue direto e não precisa avisar. Para consulta de rotina, banho, tosa, vacina e castração, manda mensagem no WhatsApp: você escolhe o horário e espera menos.

**4. Onde fica e onde eu paro o carro?**
R. Jacob Luchesi, 3230, bairro Santa Lúcia, em Caxias do Sul. A parada é na via em frente à clínica. Se estiver com o pet no colo, encoste na porta e entre — o carro se resolve depois.

**5. Meu gato vai esperar junto com os cachorros?**
Não. Recepção, consultório e internação felinos são separados dos caninos, do começo ao fim do atendimento.

**6. Meu pet ficou internado. Como eu fico sabendo dele?**
Pelo WhatsApp, ao longo do plantão: o que foi feito, como ele está sendo acompanhado e foto ou vídeo quando faz sentido.

**7. Atendem outros animais além de cão e gato?**
Sim, inclusive no plantão 24h. Manda a espécie do seu pet no WhatsApp antes de sair de casa para a equipe confirmar e orientar o transporte.

**MICROCOPY DE CTA**
Nenhum. A seção de contato vem imediatamente depois e um pedido aqui roubaria força dela. A barra sticky continua na tela.

**O QUE SE MOVE**
**Nenhum clipe e nenhuma imagem. Esta seção é tipografia, e a declaração é explícita.** Fundo creme `#F7F4EF` com o motivo de patas em lilás `#E4D2F0` a 10% de opacidade e deriva lentíssima em `translateY` — textura, não animação. Único movimento funcional: o glifo de expandir, que é o **coração de contorno âmbar da marca**, girando 45° na abertura com `cubic-bezier(0.16, 1, 0.3, 1)` em 0,2 s, e a altura do `<details>`. Régua tracejada em lilás entre as perguntas. Custo de banda: zero.

---

### 11 · `contato`

| Campo | Valor |
|---|---|
| **id** | `contato` |
| **Papel** | Fechar entregando o único ativo que a página precisa transferir para a memória do visitante: o telefone. E pagar o motivo visual que atravessou a página: a janela âmbar acesa. |
| **Temperatura** | **8** — volta ao calor da abertura, mas por confiança em vez de medo. |
| **Scroll** | **7%** · `scroll={1.7}` · `scrollMobile={1.1}` |
| **WOW** | small |
| **CTA** | 1 primário `wa.me` + 1 secundário `tel:` + 1 terciário `.vcf` |

**EYEBROW** *(12)*
`Aberto agora`

**HEADLINE** *(37)*
**Guarde este número antes de precisar.**

**SUBHEADLINE** *(135)*
R. Jacob Luchesi, 3230 — Santa Lúcia, Caxias do Sul. Aberto 24 horas, todos os dias. Ligue se for agora; mande mensagem se der para esperar.

**CORPO — tudo em texto real no DOM, nada dentro de imagem**

- **Telefone**, escala `stat`, texto selecionável e `tel:` — **(54) 3025-2223**
- **Endereço** — R. Jacob Luchesi, 3230 — Santa Lúcia, Caxias do Sul/RS · CEP 95032-000
- **Horário** — 24 horas · todos os dias · inclusive feriados
- **Farmácia veterinária aberta 24h**
- Mapa estático com `loading="lazy"` + link `Abrir no Google Maps`
- Faixa de credenciais — Desde 2015 · mais de 70.000 pets atendidos · 4,8 no Google, mais de 1.100 avaliações
- Letra miúda — Atendimento particular.

**MICROCOPY DE CTA**
- Primário: `Falar no WhatsApp` — sólido, `origin: fecho`
- Secundário: `Ligar (54) 3025-2223` — contorno, `tel:`
- **Terciário, link de texto:** `Salvar contato` — arquivo `.vcf`

> **Inversão deliberada em relação ao hero.** Quem rolou onze capítulos não está em pânico; quem estava converteu na primeira tela. É o mesmo par de canais com os pesos trocados, e isso fecha o princípio do garfo.
> **O `Salvar contato` é a única ação da página que converte quem não precisa da clínica hoje.** Numa clínica 24h, o cliente de emergência é adquirido meses antes da emergência. Custa zero de orçamento de motion e é o maior LTV do documento.

**O QUE SE MOVE**
*Imagem —* a mesma fachada do hero, mesmo ponto de vista e mesma lente, agora em plano aberto com o quarteirão inteiro no enquadramento.
*Clipe (Flow) —* **dolly-out lento**, o percurso inverso exato do hero. A câmera recua pela rua, o quarteirão vai ficando escuro, as janelas vizinhas apagadas, e a janela âmbar da recepção continua acesa. No fim do recuo é a única luz visível na rua inteira. O clipe termina parado nesse quadro e fica.
*Por que e por quanto —* o hero levou o visitante até a porta; o fecho o coloca do lado de fora e mostra o que ele agora sabe que existe. Fecha o arco visual sem repetir o hero, que é o anti-pattern mais comum de seção final. **E está precificado como medium (45%), não como pico:** recuo de câmera na mesma fachada é um medium barato, e inflar essa curva era o ponto mais fácil de inflar da proposta original.

---

## Mapa de scroll e beat

| # | id | Pergunta que responde | Beat | Share | `scroll` | `scrollMobile` | CTA |
|---|---|---|---|---|---|---|---|
| 01 | `agora` | Tem gente aí neste momento? | **8** | 11% | 2.6 | 1.8 | 1 + 2 + garfo |
| 02 | `dois-caminhos` | Eu sou qual dos dois? | 5 | 6% | 1.4 | 0.9 | 2 |
| 03 | `e-grave` | O que estou vendo é grave? | **9** | 10% | 2.4 | 1.6 | **1, sozinho** |
| 04 | `primeiros-minutos` | Vou ser atendido ou vou esperar? | 5 | **14%** | 3.4 | 2.4 | 1 + 2 |
| 05 | `estrutura` | Vocês têm o que o caso precisa? | 3 | 9% | 2.2 | 1.5 | — |
| 06 | `quando-precisa-ficar` | E se ele tiver que ficar aí sozinho? | 7 | 11% | 2.6 | 1.8 | — |
| 07 | `ala-felina` | E se o meu for gato? | 5 | 8% | 1.9 | 1.3 | 2 |
| 08 | `rotina-e-estetica` | Isto serve para o meu dia normal? | 3 | 10% | 2.4 | 1.6 | 1 |
| 09 | `depoimentos` | Já resolveram um caso como o meu? | 6 | 7% | 1.7 | 1.1 | — |
| 10 | `faq` | Quanto custa e preciso agendar? | 2 | 7% | 1.7 | 1.1 | — |
| 11 | `contato` | Como eu falo agora? | **8** | 7% | 1.7 | 1.1 | 1 + 2 + 3 |

**Total:** 100% · **35,0 alturas de viewport** no desktop · **27,2** no mobile (78%).

**Curva de beat:** `8 · 5 · 9 · 5 · 3 · 7 · 5 · 3 · 6 · 2 · 8`

Verificações:
- Nenhum share abaixo de 6% nem acima de 22% ✓
- Nenhuma vizinha a menos de 2 de distância no beat ✓
- Depois do beat 9, a seguinte cai 4 ✓
- A seção mais longa (04, 14%) é a que derruba o medo mais caro, não o hero ✓
- **Um único 9.** As pontas ficam em 8 porque em arquétipo de Urgência a página abre e fecha em temperatura de ação. Existe um pico narrativo e apenas um.
- Pontos de conversão em botão no corpo: **6** (01, 02, 03, 04, 08, 11) + header + sticky ✓ dentro de 3–6
- Um primário por tela em todas as onze ✓

---

## Mapa de CTA

| Posição | Texto | Destino | Peso |
|---|---|---|---|
| Skip link (1º focável) | `Ir direto para o telefone do plantão` | `#telefone-plantao` | acessibilidade |
| Header, todas as telas | `Plantão 24h · (54) 3025-2223` | `tel:+555430252223` | pill persistente |
| Header, ≥1024 px | `WhatsApp` | `wa.me` · `origin: header` | contorno |
| Sticky mobile, slot 1 | `Ligar` | `tel:+555430252223` | **sólido** |
| Sticky mobile, slot 2 | `WhatsApp` | `wa.me` · `origin: sticky` | contorno |
| Sticky mobile, slot 3 | ícone de rota | Google Maps | ícone |
| 01 · garfo | `É agora` | permanece na seção | **sólido** |
| 01 · garfo | `É rotina` | `#rotina-e-estetica` | contorno |
| 01 · principal | `Ligar (54) 3025-2223` | `tel:` | **primário** |
| 01 · principal | `Falar no WhatsApp` | `wa.me` · `origin: hero` | secundário |
| 01 · texto | `Como chegar` | Google Maps | link |
| 02 · card B | `Agendar no WhatsApp` | `wa.me` · `origin: rotina` | secundário |
| 03 · fecho | `Ligar (54) 3025-2223` | `tel:` · `origin: sinais` | **primário, sem par** |
| 04 · fecho | `Como chegar` | Google Maps | **primário** |
| 04 · fecho | `Ligar (54) 3025-2223` | `tel:` · `origin: caminho` | secundário |
| 07 · fecho | `Agendar na ala felina` | `wa.me` · `origin: felina` | secundário |
| 08 · fecho | `Agendar no WhatsApp` | `wa.me` · `origin: rotina` | **primário** |
| 11 · fecho | `Falar no WhatsApp` | `wa.me` · `origin: fecho` | **primário** |
| 11 · fecho | `Ligar (54) 3025-2223` | `tel:` | secundário |
| 11 · fecho | `Salvar contato` | download `.vcf` | terciário, link |
| Rodapé | telefone, endereço, horário | texto + `tel:` | — |

**Mensagens pré-preenchidas por origem** — é a única analítica de funil que uma landing de WhatsApp consegue, e faz o atendente responder certo na primeira volta:

```ts
export type CtaOrigin = 'header' | 'hero' | 'rotina' | 'caminho' | 'felina' | 'fecho' | 'sticky'

const INTENT: Record<CtaOrigin, string> = {
  header:  'Olá! Preciso falar com a clínica.',
  hero:    'Olá! Vim pelo site e preciso de atendimento.',
  rotina:  'Olá! Vim pelo site e queria agendar banho, tosa ou consulta de rotina.',
  caminho: 'Olá! Estou indo aí com meu pet agora.',
  felina:  'Olá! Tenho um gato e queria agendar na ala felina.',
  fecho:   'Olá! Vim pelo site e gostaria de falar com a equipe.',
  sticky:  'Olá! Preciso falar com alguém agora.',
}
```

**Regra da barra sticky:** visível desde o pixel 0 (é arquétipo de Urgência), some quando `#contato` cruza 80% da viewport, recebe `inert` quando invisível, respeita `env(safe-area-inset-bottom)`, nunca sobre campo de input. Três slots e não dois porque em plantão 24h a rota é ação operacional, não conversão concorrente.

---

## O que ficou de fora e por quê

Boas ideias que foram examinadas e cortadas. Registradas aqui para não voltarem em uma próxima rodada.

| Ideia | Origem | Por que saiu |
|---|---|---|
| **O cão levantando a cabeça e olhando para a porta no fim do clipe da internação** | Jornada, cap. 06 | É desfecho clínico encenado, e o design system proíbe prometer desfecho. Um print dessa tela vira promessa. O clipe termina com o cão dormindo. |
| **Mensagem de WhatsApp "queria adiantar a triagem"** | Jornada, cap. 03 | Não se faz triagem por WhatsApp. Cria a pior primeira interação possível: o tutor manda, ninguém tria, ele chega achando que já estava na fila. |
| **Clipe do tórax de um cão respirando "um pouco rápido demais"** | Jornada, cap. 02 | Imagem manipuladora e **sem valor de scrub**: o usuário rola e nada muda, o que ensina, no momento exato em que ele está aprendendo a interação, que rolar não controla a página. |
| **`Falar no WhatsApp` como primário do hero** | Confiança, cap. 01 | Às 2h da manhã ninguém digita. O próprio documento admitia três seções adiante que "ligar bate mensagem em emergência" — e corrigia a 46% do scroll, onde o visitante já foi embora. |
| **Abrir a página com credencial: "Setenta mil vezes alguém confiou aqui"** | Confiança, cap. 02 | Nó sintático em português, e escrever 70.000 por extenso contraria o princípio de número da própria marca. O número entra na faixa de credenciais do hero, não na H1. |
| **Seção de contadores animados (11 anos / 70.000 / 4,8 / 24-7)** | Confiança, cap. 02 | Um capítulo inteiro para dados que cabem em uma faixa de uma linha no hero e outra no fecho. Comprava 8% do scroll e não derrubava nenhum medo. |
| **A luz do sol varrendo a parede vazia atrás dos depoimentos** | Confiança, cap. 08 | Motion decorativo puro. Se fosse uma still com legenda, nada se perderia — e o CLAUDE.md manda remover. |
| **O corredor que acende sala por sala em vídeo** | Confiança, cap. 04 | É a mesma ideia da axonométrica da seção 05. Ficou a **ilustrada**: mais legível em 375 px, mais barata, e é a única troca de registro visual da página. |
| **Clipe na seção de sinais (03)** | Urgência, cap. 03 | A seção declarava que o WOW era a ausência de movimento e mesmo assim pedia produção de Flow. Virou still com poster; a economia financia a sequência de frames da 06. |
| **A porta abrindo no fim do clipe do hero** | Urgência, cap. 01 | A página entrava pela porta duas vezes e queimava o gesto do MAJOR aos 11% do scroll. A porta abre uma vez só, na 04. |
| **100% do Creative Budget no dolly-out final** | Urgência, cap. 10 | Recuo de câmera na mesma fachada é medium, não pico. Reprecificado em 45%. |
| **Seção de equipe com nomes e CRMV** | — | Não temos os dados. Sem eles, nenhum rosto humano reconhecível pode ser gerado como profissional da clínica. Todos os briefings de imagem especificam mãos, jaleco, costas ou silhueta fora de foco. |
| **Segunda landing para tráfego pago de "banho e tosa"** | Urgência, §6 | Não é corte, é recomendação registrada: se houver verba de mídia paga separada para estética, não mande esse tráfego para esta página. Esta LP é excelente para o pânico e boa para a rotina. |

**Tiques de redação banidos nesta página:** `atravessar a porta` em título (aparecia três vezes), a construção `não é X — é Y` acima de duas ocorrências no documento inteiro (usada em 05 e 07, e em nenhum outro lugar), exclamação em série, `seu melhor amigo`, `anjinho`, `peludo`, e qualquer variação das frases do site atual.

---

## Pendências com o cliente

Puxadas do campo `unverified` do `design-system.json`.

| # | Item | Impacto | Bloqueia publicação? |
|---|---|---|---|
| 1 | **Número do WhatsApp.** Não existe em nenhum asset. É o canal de conversão principal desta LP e há **7 CTAs** apontando para ele. | Crítico | **SIM** |
| 2 | **Pets não convencionais 24h.** Conflito declarado: o FAQ do site atual diz "foco principal em cães e gatos"; o post de `/assets-source` anuncia atendimento 24h para não convencionais. A copy foi escrita a favor do post, defensivamente ("confirme a espécie antes de sair de casa"). Se estiver errado, saem a faixa da seção 07 e a pergunta 7 do FAQ. | Alto — diferencial forte se confirmado, promessa falsa se não | **SIM** |
| 3 | **Faixa de preço da consulta de plantão.** A resposta 2 do FAQ é a melhor possível sem dado, e ainda assim é evasiva. Um "a partir de R$ X" converte mais do que a melhor redação evasiva, mesmo quando o valor é alto. | Alto em conversão | Não |
| 4 | **Compromisso de processo: "valor informado por telefone antes de você sair de casa" e "orçamento antes de qualquer autorização".** É promessa operacional, não fato extraído de asset. Precisa de confirmação por escrito. | Médio | **SIM** para essas duas linhas |
| 5 | **Estacionamento.** O site diz "vagas próximas", a foto da fachada mostra vagas na rua. **Nenhuma linha desta página promete estacionamento próprio, e nenhuma usa o rótulo "Estacionamento" numa lista de facilidades.** Confirmar se pode citar as vagas na via. | Baixo | Não |
| 6 | **Nomes e CRMV da equipe clínica.** Sem eles, nenhuma seção de equipe e nenhum rosto gerado. Se surgirem, uma 12ª seção "Quem está de plantão" (beat 5, ~8%) entra entre a 05 e a 06 e os shares se redistribuem proporcionalmente. | Médio | Não |
| 7 | **Handles de Instagram, Facebook, YouTube e LinkedIn.** Muito tráfego chega do Instagram e deveria poder voltar. Rodapé. | Baixo | Não |
| 8 | **Fontes originais da marca.** Playfair Display + Plus Jakarta Sans são substitutas de classe, não as fontes do manual. Confirmar se existe manual de marca. | Baixo | Não |
| 9 | **Uso das avaliações do Google com nome** na seção 09. São públicas, mas vale registrar a autorização. | Baixo | Não |
| 10 | **"Desde 2015" está usado em toda a página no lugar de "10+ anos".** A data não envelhece o site; a contagem envelhece. Confirmar preferência. | Baixo | Não |

**Deliberadamente não prometido em nenhuma linha desta página:** estacionamento próprio, convênio ou plano de saúde pet, telebusca, telessaúde, nome ou CRMV de qualquer profissional, quantidade de veterinários na equipe, especialidades não listadas nos assets, tempo de espera numérico e qualquer desfecho clínico.

---

## Handoff

```ts
// src/data/story.ts — escrito antes de qualquer componente existir.
export const story = [
  { id: 'agora',                question: 'Tem gente aí neste momento?',          beat: 8, share: 0.11, cta: 'both',    proof: 'availability' },
  { id: 'dois-caminhos',        question: 'Eu sou qual dos dois?',                beat: 5, share: 0.06, cta: 'secondary' },
  { id: 'e-grave',              question: 'O que eu estou vendo é grave?',        beat: 9, share: 0.10, cta: 'primary' },
  { id: 'primeiros-minutos',    question: 'Vou ser atendido ou vou esperar?',     beat: 5, share: 0.14, cta: 'both' },
  { id: 'estrutura',            question: 'Vocês têm o que o caso precisa?',      beat: 3, share: 0.09, cta: 'none',    proof: 'credential' },
  { id: 'quando-precisa-ficar', question: 'E se ele tiver que ficar aí sozinho?', beat: 7, share: 0.11, cta: 'none' },
  { id: 'ala-felina',           question: 'E se o meu for gato?',                 beat: 5, share: 0.08, cta: 'secondary' },
  { id: 'rotina-e-estetica',    question: 'Isto serve para o meu dia normal?',    beat: 3, share: 0.10, cta: 'primary' },
  { id: 'depoimentos',          question: 'Já resolveram um caso como o meu?',    beat: 6, share: 0.07, cta: 'none',    proof: 'testimonial' },
  { id: 'faq',                  question: 'Quanto custa e preciso agendar?',      beat: 2, share: 0.07, cta: 'none' },
  { id: 'contato',              question: 'Como eu falo agora?',                  beat: 8, share: 0.07, cta: 'both',    proof: 'availability' },
] as const
```

**Para `product-design-expert`:** a quebra tonal noite (01–06) → dia (07–09) → neutro (10) → noite (11) é decisão narrativa, não estética, e não pode ser alterada sem refazer o mapa. O handoff de fundo roxo→creme acontece **dentro** da seção 06. Cada troca de fundo usa a `curva-swoosh` da marca, nunca linha reta. As seções 09 e 10 são as únicas sem mídia e precisam de tratamento tipográfico que sustente sozinho 14% do scroll somados.

**Para `landing-motion-expert`:** beat ≤ 3 recebe **uma** revelação por tela (05, 08, 10). A única sequência de frames autorizada é a da 06. Se algum capítulo pedir mais scroll do que o `share` permite, a resposta é **cortar a seção, não esticá-la** — ela está respondendo duas perguntas.

**Para `ai-visual-prompt-director`:** três registros visuais distintos, e a distinção é intencional. Fotográfico noturno: 01, 02, 03, 04, 06, 11. Ilustrado axonométrico: 05. Fotográfico diurno: 07, 08. Sem imagem: 09, 10. Paleta obrigatória: 01–06 roxo `#603084` / `#482A78` / `#2D1542` dominante; 07–11 creme `#F7F4EF` dominante com âmbar `#FCB400` liberado como acento, respeitando a proporção 60/30/10 do design system.

**Para o Flow, regras que todos os seis clipes precisam obedecer ou o scrub trava:** uma tomada contínua por clipe; movimento lento e monotônico, a câmera nunca inverte o sentido dentro do mesmo clipe; iluminação estável exceto na 02 e na 06, onde a mudança de luz **é** o conteúdo; assunto centralizado o suficiente para sobreviver ao crop 9:16 do mobile; 8 a 12 segundos de origem, extraídos para 90–140 frames, com a 06 podendo passar disso.
