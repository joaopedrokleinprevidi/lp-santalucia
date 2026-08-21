---
name: prompt-animacao
description: Use when turning approved stills into Google Flow/Veo motion prompts for a scroll-driven page — clip length, one continuous move, no cut, no shake, no loop, safe area, and the playback technique each section lands on. Fase 8b, produz design/motion-prompts.md. Prompt de animacao, clipe gerado por IA, video de secao, escolha entre loop, scrub e canvas frames, comando ffmpeg de extracao preenchido e tabela de peso em MB.
argument-hint: [secao] [tecnica-de-destino]
allowed-tools: Read, Glob, Write
---

# Prompt de animação — Fase 8b

| | |
|---|---|
| **ENTRADA** | `design/image-prompts.md` (os stills já definidos, com o Style Anchor congelado no topo) · `design/creative-direction.json` (os `wow` de vídeo, `budget.mediaDesktopMB` e `mediaMobileMB` fixados na Fase 5) |
| **SAÍDA** | `design/motion-prompts.md` — um bloco por clipe, na mesma ordem dos stills, cada um declarando a **técnica pretendida** (loop · canvas frames · nenhuma) e o peso orçado |
| **ANTES** | `prompt-imagem` (Fase 8a). Sem still aprovado não existe primeiro frame, e sem primeiro frame o clipe não casa com a seção |
| **DEPOIS** | Fase 9 — o dev gera no Google Flow e salva em `design/renders/NN-secao.mp4`. Na Fase 10, `video-decisao` lê a técnica pretendida daqui e a ratifica contra o clipe que voltou; só então `video-encode` ou `video-to-website` implementam |

O still é o primeiro frame. O clipe precisa ser **raspável**, exigência mais dura que ficar
bonito: um clipe lindo com um corte no meio é inútil aqui e um clipe chato sem corte serve.

**Não repita o Style Anchor por escrito.** Ele é do `prompt-imagem` e vive no topo de
`design/image-prompts.md`. Cada bloco daqui manda copiar de lá, verbatim. Duas cópias divergem
na primeira correção, e aí o clipe deixa de casar com o próprio still.

**Sem acesso ao Google Flow, este arquivo não existe.** Declare isso, entregue só os stills e a
página usa parallax e ken burns em CSS — continua entregue.

## Passo 1 — Escolher a técnica de destino antes de escrever o prompt

A técnica muda o prompt, então ela é fixada antes de escrever — e na Fase 8b o clipe ainda não
existe, então quem fixa é esta skill. O alvo vai escrito no bloco: pedir um loop e receber um
clipe feito para scrub joga fora a rodada de geração inteira, que custa a parada 3 do dev.

Esta é a **técnica pretendida**, não a definitiva. Na Fase 10, `video-decisao` lê estes blocos,
aplica o teste honesto contra o clipe que de fato voltou e escreve `design/video-plan.md` —
único lugar onde a técnica vira decisão de implementação. A comparação de custo das três é de
[decision.md](../video-to-website/decision.md), que é a dona da tabela.

| Papel da seção | Técnica | O que isso exige do prompt |
|---|---|---|
| Ambiente, b-roll, textura sob a qual a pessoa lê | looping `<video>` | 8–10 s, movimento que não tem começo nem fim óbvios, câmera quase parada. O reinício do loop aparece: quanto menos deslocamento, menos ele incomoda |
| A mudança **é** o beat: transformação, revelação, dia→noite | canvas frame sequence | 6 s a 25 fps = 150 frames, o piso da técnica. Um movimento só, do primeiro ao último frame |
| Clipe longo (>30 s de movimento essencial), página só desktop | scrub `currentTime` | Legado. iOS Safari não busca frame-exato em vídeo comprimido e treme visivelmente |
| A seção já diz tudo no still | nenhuma | Não escreva bloco. Um clipe que não muda nada é peso puro |

**Orçamento:** a página inteira ganha **uma** frame sequence, duas só se forem distantes. A
terceira transforma a landing num download e quem está no 4G fecha antes do primeiro frame.

## Passo 2 — Escrever o prompt de movimento

```
[STYLE ANCHOR copiado verbatim de design/image-prompts.md, uma linha]

MOTION: <um movimento contínuo, uma direção, sem corte>
CAMERA: <static | slow push in | slow pull back | gentle pan left-to-right>
SUBJECT ACTION: <pequena, natural, que se completa dentro do clipe>
PACE: slow, deliberate, 4–6 seconds
EXCLUDE: cuts, scene changes, camera shake, zoom snaps, text overlays, speed ramps, loops
```

O `EXCLUDE` de texto do still continua valendo aqui — letra num clipe é pior que letra num
still, porque ela se move e mascarar exigiria frame a frame.

### O que sobrevive à extração de frames

| Funciona | Falha |
|---|---|
| Push / pull lento de câmera | Qualquer corte — raspar através de um corte lê como bug |
| Pan de eixo único | Câmera na mão — o scrub amplifica micro-tremor em jitter |
| Pelo, vapor, tecido, água assentando | Movimento rápido de membro — deforma de frame a frame |
| Luz deslizando sobre uma superfície | Qualquer coisa com texto renderizado |
| Uma cabeça virando, uma vez, devagar | Loop — a ponta final do gerado quase nunca encontra a inicial |
| Rack de foco | Multidão, muitos assuntos em movimento |

### As restrições vêm da extração, não do gosto

- **4–6 segundos, e 6 quando o destino é canvas.** Na taxa nativa do Flow (25 fps), 6 s dá 150
  frames — o piso da técnica. Abaixo de 150 o movimento estroba; acima de 300 o payload deixa de
  valer o peso.
- **Um movimento.** O scroll mapeia linearmente para índice de frame. Dois movimentos no mesmo
  clipe produzem um tranco na emenda, e a câmera nunca inverte o sentido dentro do clipe.
- **Nunca peça loop.** A sequência descansa no último frame. Por isso **o último frame é
  escolhido como imagem**: ele fica mais tempo na tela, é o poster de `prefers-reduced-motion` e
  é o que aparece num screenshot. Escreva o `MOTION` de trás para frente — decida o quadro final
  primeiro.
- **Assunto na área segura.** `IMAGE_SCALE` 0.85 no canvas transforma os ~7% externos de cada
  borda em padding, e o crop 9:16 do mobile só garante os 60% centrais horizontais.
- **Peça lento.** Vídeo gerado não tem motion blur real; frame nítido em sequência estroba sob
  scrub rápido. Por isso `FRAME_SPEED` fica em 1.8, o extremo baixo da faixa de
  `video-to-website` — e é o prompt que precisa entregar movimento devagar o bastante para isso.
- **Nunca peça texto, nem "sutil".** Letra num clipe se move; mascarar exigiria frame a frame.

## Passo 3 — O comando ffmpeg, preenchido no bloco

O bloco entrega o comando já com o nome da seção, a fps e a largura dentro. `ffmpeg-static` e
`ffprobe-static` já estão em devDependencies: use `npx` ou `node scripts/…`. Nunca instale
ffmpeg global e nunca escreva caminho absoluto de binário num script — é a origem clássica do
pipeline que "funciona aqui".

**Conferir o que voltou do Flow, antes de extrair qualquer coisa:**

```bash
npx ffprobe -v error -select_streams v:0 \
  -show_entries stream=width,height,duration,r_frame_rate,nb_frames \
  -of csv=p=0 design/renders/04-estrutura.mp4
# esperado: 1920,1080,6.000000,25/1,150
```

Duração abaixo de 6 s ou fps abaixo de 25 já reprova o clipe para canvas: 5 s a 25 fps são 125
frames e estrobam. Regere no Flow — não interpole frame, o resultado é borrão fantasma.

**Extrair (destino canvas frame sequence):**

```bash
mkdir -p public/frames/04-estrutura
npx ffmpeg -i design/renders/04-estrutura.mp4 \
  -vf "fps=25,scale=1920:-1:flags=lanczos" \
  -c:v libwebp -quality 78 \
  public/frames/04-estrutura/frame_%04d.webp

ls public/frames/04-estrutura | wc -l   # 150 — abaixo disso, estroba
du -sh public/frames/04-estrutura       # teto 8 MB; estourou, derrube a fps antes da qualidade
```

Segundo set a 1280 px só quando a seção está acima da dobra — um celular decodificando 300
frames de 1920 px é morto pelo sistema operacional.

**Congelar o último frame como imagem** (poster do reduced-motion e still de fallback):

```bash
npx ffmpeg -sseof -0.1 -i design/renders/04-estrutura.mp4 \
  -frames:v 1 -q:v 2 design/renders/04-estrutura-last.png
```

**Destino looping `<video>`:** não extraia frame nenhum. O transcode das duas renditions
(H.264 High, `yuv420p`, `-g 48`, `+faststart`, CRF 25 desktop / 27 mobile) é da skill
`video-encode` e roda dentro de `scripts/prepare-assets.mjs`. Aqui só se declara qual clipe vai
por esse caminho e com que duração.

## Passo 4 — Fechar a tabela-resumo de peso

O somatório vai no fim de `design/motion-prompts.md` e é portão de saída da Fase 8: ele tem que
caber no `budget.mediaDesktopMB` / `mediaMobileMB` fixados na Fase 5 — **8 MB e 3,5 MB** no
★★★★☆ padrão, 12 MB e 5 MB só num ★★★★★ orçado como tal.

Custo unitário para orçar antes de existir arquivo:

| Item | Desktop | Mobile | Fonte do número |
|---|---|---|---|
| Frame sequence, 150–300 frames WebP q78 @1920 | 2–8 MB | set separado @1280, ~45% do desktop | `decision.md` |
| Loop `<video>` H.264 | 0,31–0,44 MiB por segundo | 0,10–0,15 MiB/s | medido nos três filmes de referência; a dona é `video-encode` |
| Still full-bleed AVIF @1920 | ~0,2 MB | ~0,08 MB | estimativa de orçamento; o valor real sai do `prepare-assets.mjs` |

Reprovação por unidade: **0,45 MiB/s no desktop** e **0,15 MiB/s no mobile** para loop. Se um
clipe estoura, corte segundos antes de mexer em qualidade; se ainda estourar, suba o CRF em 2 —
não baixe a resolução, 720p esticado num palco de 1440 px é o que faz um filme parecer mole.

Verificação real, depois da Fase 10:

```bash
du -sh public/frames/* public/media/* | sort -h
```

## Passo 5 — Formato do bloco em `design/motion-prompts.md`

```markdown
### 04 · `estrutura` — canvas frame sequence

**Still de origem:** `design/renders/04-estrutura.png` (bloco 04 de `image-prompts.md`)
**Salvar o clipe em:** `design/renders/04-estrutura.mp4`  ·  **6 s · 25 fps · 1920×1080**

**Prompt (Google Flow, anexando o still como primeiro frame):**

    [STYLE ANCHOR copiado de design/image-prompts.md]

    MOTION: The camera pushes in very slowly toward the stainless table, one direction only,
    from medium-wide to medium. Nothing else in the room moves.
    CAMERA: slow push in, locked axis, no tilt, no roll.
    SUBJECT ACTION: The amber lamp reflection on the tiled wall slides a few centimetres as
    the camera advances. No person enters the frame.
    PACE: slow, deliberate, 6 seconds.
    EXCLUDE: cuts, scene changes, camera shake, zoom snaps, text overlays, letters, numbers,
    signage, labels, speed ramps, loops, people entering frame.

**Quadro final pretendido:** mesa centralizada, parede roxa ocupando a metade superior — é este
quadro que fica na tela no fim do scroll e é o poster do `prefers-reduced-motion`.

**Conferir e extrair:**

    npx ffprobe -v error -select_streams v:0 -show_entries \
      stream=width,height,duration,r_frame_rate,nb_frames -of csv=p=0 \
      design/renders/04-estrutura.mp4        # esperado 1920,1080,6.000000,25/1,150

    mkdir -p public/frames/04-estrutura
    npx ffmpeg -i design/renders/04-estrutura.mp4 \
      -vf "fps=25,scale=1920:-1:flags=lanczos" -c:v libwebp -quality 78 \
      public/frames/04-estrutura/frame_%04d.webp

**Peso orçado:** 150 frames ≈ 4,5 MB desktop · set @1280 ≈ 2,0 MB mobile

**Teste de aceitação — regere se:** houver qualquer corte ou mudança de cena; a câmera tremer;
a velocidade mudar no meio; o clipe fechar em loop no ponto de partida; aparecer letra ou
rótulo; o assunto sair dos 60% centrais; o último frame não funcionar como fotografia parada.
```

Campos obrigatórios: still de origem, técnica de destino, caminho de salvamento, duração e fps,
prompt completo, quadro final pretendido, comando de conferência e extração, peso orçado, teste
de aceitação. Faltando qualquer um, o dev improvisa na Fase 9.

## Passo 6 — Revisar o clipe que voltou

- [ ] `ffprobe` bate com o pedido: duração, fps e `nb_frames` esperados
- [ ] Nenhum corte, nenhuma mudança de cena — passe uma vez inteira olhando só para isso
- [ ] Nenhum tremor de câmera; o scrub multiplica micro-tremor em jitter
- [ ] Uma direção só, sem inversão e sem rampa de velocidade
- [ ] Nenhuma letra, número, rótulo ou mostrador em qualquer frame
- [ ] O assunto fica nos 60% centrais horizontais e fora dos 7% de borda em **todo** o clipe
- [ ] O último frame funciona como imagem parada — é ele que fica mais tempo na tela
- [ ] Nenhuma anatomia quebrada durante o movimento (pata, orelha, dedo) a 100% de zoom
- [ ] A tabela-resumo de peso fecha dentro de `mediaDesktopMB` / `mediaMobileMB`
- [ ] No máximo uma frame sequence na página, duas só se distantes

## Anti-patterns

- **Pedir loop** — a ponta final do gerado quase nunca encontra a inicial, e a emenda vira um
  salto visível bem no ponto onde o scroll para.
- **Dois movimentos no mesmo clipe** — o scroll mapeia linear para índice de frame; a emenda
  vira tranco. Se a seção precisa de dois beats, são dois capítulos.
- **Movimento rápido ou complexo** — deforma na geração e estroba no scrub, porque não há motion
  blur real para preencher entre frames nítidos.
- **Frame sequence em toda seção** — 2 a 8 MB cada; a terceira transforma a landing num download.
- **Copiar o Style Anchor para dentro deste arquivo** — duas cópias divergem e o clipe deixa de
  casar com o próprio still. Referencie `design/image-prompts.md`.
- **Animar uma imagem abstrata** — forma sem geometria estável vira morph psicodélico sob scrub;
  o portão de assunto concreto do `prompt-imagem` existe também por causa disto.
- **Interpolar frames para chegar aos 150** — o resultado é borrão fantasma. Regere com 6 s.
- **Instalar ffmpeg global ou fixar caminho de binário** — o script passa a funcionar só na
  máquina de quem escreveu, e o build da Vercel nem roda esse pipeline.
- **Escolher a técnica depois de gerar** — o prompt de um loop e o de uma frame sequence são
  diferentes; decidir depois joga fora a rodada de geração.
