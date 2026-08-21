---
name: scroll-video-director
description: Use when a landing page has video and someone must decide whether it stays, which technique carries it and what it says — vídeo no scroll, background video, cinematic section, scroll-driven MP4, filme de capítulo, loop de fundo, autoplay, poster, transcodar vídeo para web, ffmpeg. Owns the <video> element itself — loop vs once, poster, IntersectionObserver gating, responsive renditions, ffmpeg encoding, reduced-motion fallback. Delegates frame-accurate scrubbing to video-to-website.
argument-hint: [caminho-do-master] [chave-do-capitulo]
---

# Scroll Video Director

Duas perguntas, nesta ordem: **este vídeo merece existir na página?** e **qual técnica o
carrega?** Código vem depois.

Com argumento: `[caminho-do-master]` é o arquivo 1080p em `assets-source/`, e
`[chave-do-capitulo]` é a chave que ele recebe no array `VIDEOS` de
`scripts/prepare-assets.mjs` (`hero`, `procedure`, `consultation`). Sem argumento, comece pela
seção 1.

Esta skill é dona do elemento `<video>`: modo de reprodução, poster, gating, renditions,
encoding, fallback. O scrub frame-a-frame **não** é dela. Quando a resposta for sequência de
frames em canvas, entregue para [video-to-website](../video-to-website/SKILL.md) e pare aqui.

## 1. A decisão de técnica vem primeiro

A tabela de custo/precisão das três técnicas está em
[../video-to-website/decision.md](../video-to-website/decision.md). Leia antes de extrair ou
transcodar qualquer coisa. O resumo operacional:

- **`<video>` em loop** — o padrão. Footage que ambienta enquanto o visitante lê por cima.
- **Sequência de frames em canvas** — quando a *mudança ao longo do tempo* é o argumento da
  seção. Uma por página, duas se forem distantes. Delegue.
- **`video.currentTime` scrubado** — fallback, não técnica primária. Ver a seção 6.

A maior parte do material sobre scroll-driven video prescreve `currentTime` como técnica
principal. Está errado: o Safari no iOS não busca quadro exato em vídeo comprimido, então o
scrub gagueja de forma que nenhum ajuste de JS conserta. Para scrub, a técnica é a sequência de
frames.

**Orçamento por página.** O que importa não é a soma dos arquivos, é o que baixa antes da
primeira interação: só o filme marcado `priority`. Nesta LP isso é **0,96 MiB no mobile e
3,06 MiB no desktop** — o herói e mais nada. Todo o resto entra por gate. Se um capítulo não
tem gate, ele não tem vídeo.

A faixa de 300 KB–2 MB que `decision.md` dá para "looping `<video>`" é a rendition **mobile**.
Um 1080p de 10 s a CRF 25 sai em 3–4 MiB e continua sendo a técnica certa, porque o desktop
recebe esse arquivo um por vez, sob gate, e nunca todos.

## 2. Direção narrativa — qual papel o vídeo tem

Vídeo sem papel é peso. Antes de escolher a técnica, nomeie a função:

| Papel | O que a footage precisa mostrar | Técnica | Modo | Duração útil | Copy por cima |
|---|---|---|---|---|---|
| Estabelecer lugar | o espaço real com gente dentro, câmera quase parada | `<video>` | `loop` | 6–12 s | sim, com scrim |
| Provar processo | mãos fazendo a coisa, plano fechado, sem corte | `<video>` | `once` | 8–15 s | pouca e curta |
| Mostrar transformação | antes e depois no mesmo enquadramento | canvas frames | — | 4–10 s de origem | não |
| Criar atmosfera | textura, luz, tecido, água; sem sujeito | `<video>` | `loop` | 4–8 s | sim, é o fundo |

**Estabelecer lugar** é o que o herói desta LP faz: prova que a clínica existe e é assim.
Loop, porque o visitante fica ali o tempo que quiser ler. A footage nunca é o assunto — se
alguém precisa parar de ler para olhar o vídeo, a footage está agitada demais.

**Provar processo** é a única função que justifica `once`. O clipe tem um começo e um fim
narrativos; rebobinar destrói o argumento. Ele segura o último quadro e vira fundo estático.

**Mostrar transformação** é onde o scroll precisa ser dono do playhead — e é exatamente o caso
em que `currentTime` falha. Delegue para a sequência de frames ou reescreva a seção como
antes/depois em duas imagens. Não existe terceira saída boa.

**Criar atmosfera** é o único papel em que um clipe de 4 s serve. Abaixo disso o ciclo fica
perceptível mesmo com emenda perfeita.

### O corte, antes do encode

- **Um vídeo, uma ideia.** Se são precisos dois planos para dizer a coisa, são dois capítulos.
  Um clipe com corte interno compete com a narrativa da página pelo mesmo momento de atenção.
- **Loop só é loop se o último quadro casar com o primeiro.** Corte num ponto em que a câmera
  esteja parada nos dois extremos. Nunca dê `loop` em footage com movimento direcional
  (travelling, zoom, pan): o retorno ao quadro 0 é um corte seco que aparece a cada ciclo, e o
  olho localiza a emenda na terceira volta.
- **Se a copy é o ponto, a footage perde.** Sobre um frame com highlight em movimento, texto
  branco de 16px exige `0.62` de ink sobre o pixel mais claro que ele cobre — a tabela por
  luminância e o script que a mede são de
  [responsive-e-acessibility](../responsive-e-acessibility/alt-e-contraste.md#texto-sobre-vídeo),
  que é dono do número. Use o componente `Scrim`, não `filter`: `filter` força o compositor a
  reler a textura do decoder a cada quadro.
- **Corte antes de comprimir.** Tirar 4 s de um clipe de 14 s economiza mais bytes do que
  qualquer ajuste de CRF, e a seção quase sempre fica melhor.

## 3. O componente `<video>`

A implementação provada está em `src/components/media/ChapterFilm.tsx`. A estrutura, reduzida
ao que importa:

```tsx
import { useEffect, useRef, useState, type RefObject } from 'react'
import type { ChapterVideo } from '../../generated/media'
import { useIsDesktop, useReducedMotion } from '../../hooks/useMediaQuery'
import { Picture } from './Picture'

interface ChapterFilmProps {
  film: ChapterVideo
  /** `loop` roda contínuo atrás do capítulo; `once` toca uma vez e segura o
   *  último quadro; `scrub` entrega o playhead ao scroll (ver a seção 6). */
  mode: 'loop' | 'once' | 'scrub'
  /** Descreve a footage para quem só vai ver o poster. Obrigatório. */
  posterAlt: string
  /** Exposto para o modo `scrub`, que precisa escrever `currentTime`. */
  videoRef?: RefObject<HTMLVideoElement | null>
  priority?: boolean
}

export function ChapterFilm({
  film,
  mode,
  posterAlt,
  videoRef,
  priority = false,
}: ChapterFilmProps) {
  const containerRef = useRef<HTMLDivElement>(null)
  const internalRef = useRef<HTMLVideoElement>(null)
  const element = videoRef ?? internalRef

  const reducedMotion = useReducedMotion()
  const isDesktop = useIsDesktop()

  const [shouldLoad, setShouldLoad] = useState(priority)
  const [isOnScreen, setIsOnScreen] = useState(false)
  const [isReady, setIsReady] = useState(false)
  const [source, setSource] = useState<string | null>(null)

  /* Gate 1 — buscar o arquivo. Uma tela e meia de antecedência: num viewport
     de 800 px isso são 1200 px, ~1 s a 1200 px/s de flick — tempo de o
     primeiro quadro chegar antes de o capítulo aparecer. */
  useEffect(() => {
    if (shouldLoad || reducedMotion) return
    const node = containerRef.current
    if (!node) return
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (!entry.isIntersecting) return
        setShouldLoad(true)
        observer.disconnect()
      },
      { rootMargin: '150% 0px' },
    )
    observer.observe(node)
    return () => observer.disconnect()
  }, [shouldLoad, reducedMotion])

  /* Gate 2 — decodificar. Margem curta: decode é o custo contínuo, e um capítulo
     fora da tela decodificando rouba main thread do ScrollTrigger. */
  useEffect(() => {
    const node = containerRef.current
    if (!node || reducedMotion) return
    const observer = new IntersectionObserver(([entry]) => setIsOnScreen(entry.isIntersecting), {
      rootMargin: '10% 0px',
    })
    observer.observe(node)
    return () => observer.disconnect()
  }, [reducedMotion])

  /* A rendition é escolhida em JS — CSS não reduz um byte nem um ciclo de
     decode — e é congelada quando o gate abre. Reavaliar num resize que cruza
     768 px trocaria o `src` de um vídeo já tocando: o elemento recarrega e
     mostra preto, porque o poster já desapareceu. */
  useEffect(() => {
    if (!shouldLoad || source) return
    setSource(isDesktop ? film.desktop : film.mobile)
  }, [shouldLoad, source, isDesktop, film])

  useEffect(() => {
    const video = element.current
    if (!video || !source) return

    // Propriedade, não só atributo: alguns engines avaliam a política de autoplay
    // antes de o atributo refletir, e aí play() é rejeitado sem motivo aparente.
    video.muted = true

    // No modo `scrub` quem escreve o playhead é o hook da seção 6. Chamar
    // play() aqui faria o decoder e o `currentTime` disputarem o mesmo valor.
    if (mode === 'scrub') return

    if (!isOnScreen) {
      video.pause()
      return
    }
    // Um filme `once` já visto não rebobina ao voltar — reiniciar vira tique.
    if (mode === 'once' && video.ended) return
    video.play().catch(() => setIsReady(true))
  }, [element, isOnScreen, mode, source])

  if (reducedMotion) {
    return (
      <div ref={containerRef} className="chapter__media">
        <Picture image={film.poster} alt={posterAlt} sizes="100vw" priority={priority} />
      </div>
    )
  }

  return (
    <div ref={containerRef} className="chapter__media">
      <Picture
        image={film.poster}
        alt={posterAlt}
        sizes="100vw"
        priority={priority}
        className="absolute inset-0 transition-opacity duration-700 ease-out-expo"
        style={{ opacity: isReady ? 0 : 1 }}
      />
      <video
        ref={element}
        preload={source ? 'metadata' : 'none'}
        {...(source ? { src: source } : {})}
        muted
        playsInline
        loop={mode === 'loop'}
        disablePictureInPicture
        aria-hidden="true"
        tabIndex={-1}
        onCanPlay={() => setIsReady(true)}
      />
    </div>
  )
}
```

`src` só entra no elemento depois do gate. Renderizar a tag sem `src` e adicionar depois é o
que impede o navegador de abrir conexão antes da hora — `preload="none"` sozinho não é
garantia em todos os engines.

| Atributo | Por que é obrigatório |
|---|---|
| `muted` (atributo **e** propriedade) | Chrome e Safari só liberam autoplay para vídeo mudo. Sem os dois, `play()` rejeita com `NotAllowedError` e o capítulo fica no poster. |
| `playsInline` | Sem ele o iOS abre o player nativo em tela cheia no primeiro `play()`, e o capítulo inteiro some atrás dele. |
| `preload="metadata"` | `auto` baixa o arquivo inteiro; `none` adia a duração, de que o fallback de scrub precisa. `metadata` são alguns KB. |
| `disablePictureInPicture` | Sem ele, Safari e Chrome oferecem PiP no vídeo decorativo. Um fundo virando janela flutuante é um bug visível. |
| `aria-hidden="true"` + `tabIndex={-1}` | Footage decorativa não deve ser anunciada como "vídeo" nem receber foco — não há controle nenhum ali para operar. Se a footage carrega informação, não esconda: use `once` e descreva o conteúdo em texto DOM ao lado. |
| poster via `<Picture>`, não pelo atributo `poster` | O atributo aceita uma URL única: sem AVIF, sem `srcset`, sem LQIP e **sem `alt`**. O `<img>` sobreposto com crossfade de 700 ms dá tudo isso. |

O `.chapter__media` é `position: absolute; inset: 0`, então carregar metadata nunca muda a
altura do capítulo — a altura vem de `--chapter-scroll-*`, em múltiplos de viewport. Isso é o
que elimina layout shift, não `aspect-ratio` no vídeo.

## 4. Encoding

Duas renditions por filme, ambas masterizadas do 1080p original, geradas em
`scripts/prepare-assets.mjs` com o binário do `ffmpeg-static` que já está em devDependencies.
Nunca instale ffmpeg global e nunca escreva caminho absoluto de binário num script.

```js
await ffmpeg([
  '-i', input,
  '-an',                                                  // sem faixa de áudio
  '-vf', `scale=${r.width}:${r.height}:flags=lanczos,hqdn3d=${r.denoise}`,
  '-c:v', 'libx264',
  '-profile:v', 'high',
  '-preset', 'slower',
  '-crf', String(r.crf),
  '-pix_fmt', 'yuv420p',
  '-g', '48',
  '-movflags', '+faststart',
  dest,
])
```

| Rendition | Escala | CRF | `hqdn3d` | Serve para |
|---|---|---|---|---|
| desktop | 1920×1080 | 25 | `1:1:4:4` | `(min-width: 768px)` |
| mobile | 1280×720 | 27 | `1.5:1.5:6:6` | resto |

Por que cada flag:

- **`-movflags +faststart`** — move o átomo `moov` para o início. Sem isso o índice fica no fim
  do arquivo e o navegador precisa baixar quase tudo antes do primeiro quadro. O sintoma é
  poster preso por segundos num vídeo que já está 80% baixado.
- **`-pix_fmt yuv420p`** — 4:2:0 é o único chroma que todo decoder de hardware aceita. 4:2:2 e
  4:4:4 saem do encoder sem erro e não abrem em lugar nenhum.
- **`-g 48`** — keyframe a cada 48 quadros (1,6–2 s). GOP curto é o que torna o reinício do
  loop limpo e o fallback de seek tolerável. GOP longo economiza ~5% e transforma qualquer
  seek em salto.
- **`-preset slower`** — o encode roda uma vez no build; tempo de CPU ali não custa nada. Rende
  ~8% a menos de arquivo que `medium` no mesmo CRF.
- **`-an`** — vídeo mudo com faixa de áudio ainda carrega os bytes dela e ainda faz o navegador
  instanciar o pipeline de áudio.
- **`hqdn3d`** — os masters carregam grão de sensor que o x264 gasta muitos bits preservando e
  que ninguém vê em movimento. Medido contra a fonte: custa 0,0013 de SSIM e economiza 14%.
- **Hash do master no nome do arquivo** — sem isso um re-encode reutiliza a URL antiga, e quem
  já visitou continua com o corte anterior até o cache expirar. É assim que uma correção de
  qualidade não chega em quem viu o problema.

**Por que H.264 e não VP9/AV1.** H.264 High/yuv420p é o único perfil com decode em hardware
garantido em todo aparelho que abre esta página, incluindo iPhone antigo e Android de entrada.
VP9 tem suporte irregular no Safari e AV1 só decodifica em hardware em chips recentes. Quando o decode cai para
software, o custo vai para a main thread — a mesma que roda o ScrollTrigger. O sintoma não é
"vídeo pesado", é "scroll travando". O ganho seria ~30% de bitrate: em 3 MB, 900 KB. Não se
troca 900 KB por risco de scroll a 40 fps. Única exceção real: canal alpha, que o H.264 não
tem — aí é HEVC-alpha para Safari e WebM/VP9-alpha para o resto, via `<source>`. É caro;
prefira redesenhar a seção para não precisar de alpha.

**Tamanho-alvo.** Nesta configuração o custo real medido nos três filmes é de **0,31 a
0,44 MiB por segundo no desktop** e **0,10 a 0,15 MiB/s no mobile**. Quanto mais movimento no
quadro, mais alto dentro da faixa — `procedure` e `consultation` têm mãos em primeiro plano,
`hero` tem câmera quase parada.

| Filme | Duração | desktop | mobile | desktop MiB/s |
|---|---|---|---|---|
| `hero` | 10 s | 3,06 MiB | 0,96 MiB | 0,31 |
| `procedure` | 10 s | 4,09 MiB | 1,23 MiB | 0,41 |
| `consultation` | 8 s | 3,49 MiB | 1,18 MiB | 0,44 |

Os números são MiB porque é isso que `ls -l` conta (3.213.394 bytes = 3,06 MiB). O teto que
vale como reprovação é **0,45 MiB/s no desktop** e **0,15 MiB/s no mobile**.

Se um clipe estoura, corte segundos antes de mexer em qualidade. Se ainda estourar, suba o CRF
em 2 — **não** baixe a resolução: 720p esticado num palco de 1440 px foi exatamente o que fez o
filme de abertura deste projeto parecer mole. O encoder nunca foi o problema; os pixels que
faltavam eram.

### Poster

```js
await ffmpeg(['-ss', String(video.poster), '-i', input, '-frames:v', '1', posterRaw])
```

Depois passa pelo mesmo pipeline responsivo das imagens (AVIF + WebP + LQIP inline). Escolha um
timestamp em que o quadro pareça uma fotografia: nada de motion blur no meio de um gesto. Os
valores usados aqui são `0.35`, `3.2` e `0.6` — nenhum deles é 0, porque o primeiro quadro de
um clipe costuma ser o pior. O crossfade de 700 ms perdoa uma diferença pequena entre poster e
quadro 0; se o poster for de um momento visivelmente outro, mude o timestamp em vez de alongar
o crossfade.

## 5. Reduced motion

Sob `prefers-reduced-motion: reduce` o elemento `<video>` **não é criado**. Não é pausado, não é
escondido: não existe. Pausar depois já baixou os bytes, e um `<video autoplay loop>` pausado
ainda deixou a política de autoplay rodar. O poster responsivo carrega a cena inteira, com
`alt` descrevendo a footage — é a única leitura que aquele visitante vai ter dela.

```tsx
if (reducedMotion) {
  return (
    <div ref={containerRef} className="chapter__media">
      <Picture image={film.poster} alt={posterAlt} sizes="100vw" priority={priority} />
    </div>
  )
}
```

O `posterAlt` é obrigatório na prop justamente por isso. "Vídeo do capítulo" não é alt;
"Aplicação de um procedimento estético facial na sala de atendimento" é.

## 6. Fallback: scrub por `currentTime`

**Só use quando** o clipe é longo demais para extrair frames (>30 s de movimento essencial) *e*
a seção é desktop. Em qualquer outro caso a resposta é sequência de frames ou loop.

**A limitação, declarada.** O Safari no iOS não faz seek de quadro exato em vídeo comprimido.
Um `currentTime` arbitrário resolve para o keyframe anterior e a imagem só atualiza quando o
decoder alcança o quadro pedido — com GOP de 48 quadros isso é até 1,6 s de imagem parada
enquanto o dedo continua andando. O sintoma é gagueira, não lentidão, e não há correção do lado
do JS. É por isso que a sequência de frames em canvas existe.

Monte o `ChapterFilm` com `mode="scrub"` e passe o mesmo ref para os dois — o componente não
chama `play()` nesse modo, e o hook fica dono do `currentTime`. Duas consequências que precisam
de decisão explícita antes de escrever isto:

- **`mode="scrub"` não degrada sozinho.** Onde o `matchMedia` não casa (todo touch, e com ele o
  iOS), ninguém escreve `currentTime` e ninguém chama `play()`: o capítulo fica no primeiro
  quadro para sempre. Quem monta decide o modo pelo ponteiro — `useMediaQuery('(pointer: fine)')
  ? 'scrub' : 'loop'` — ou a seção inteira é outra coisa no mobile.
- **Com `preload="metadata"` o primeiro quadro só chega no primeiro seek.** Até lá o poster
  cobre o palco, o que é o comportamento correto, mas significa que um capítulo scrubado que
  ninguém rola nunca mostra vídeo.

```tsx
import { useLayoutEffect, type RefObject } from 'react'
import { gsap, ScrollTrigger } from '../lib/gsap' // registro único do plugin

export function useScrubbedFilm(
  video: RefObject<HTMLVideoElement | null>,
  scope: RefObject<HTMLElement | null>,
) {
  useLayoutEffect(() => {
    const el = video.current
    const node = scope.current
    if (!el || !node) return

    const mm = gsap.matchMedia(node)

    // `pointer: fine` exclui touch, e com ele o iOS, onde o seek gagueja.
    // Onde não casa, este hook não faz nada — quem escolhe o fallback é
    // o componente pai, não este arquivo.
    mm.add('(prefers-reduced-motion: no-preference) and (pointer: fine)', () => {
      const STEP = 1 / 30 // um quadro a 30 fps; abaixo disso o seek não muda pixel nenhum
      let target = 0
      let raf = 0

      const flush = () => {
        raf = 0
        if (Math.abs(el.currentTime - target) < STEP) return
        el.currentTime = target
      }

      ScrollTrigger.create({
        trigger: node,
        start: 'top top',
        end: 'bottom bottom',
        scrub: true,
        onUpdate: (self) => {
          if (!el.duration) return // metadata ainda não chegou
          target = self.progress * el.duration
          if (raf) return // no máximo um seek por quadro de tela
          raf = requestAnimationFrame(flush)
        },
      })

      // O matchMedia reverte o ScrollTrigger acima sozinho; o rAF pendente não.
      // Sem isto, `flush` roda depois do unmount e escreve em elemento morto.
      return () => cancelAnimationFrame(raf)
    })

    return () => mm.revert()
  }, [scope, video])
}
```

Os dois gates existem por motivos diferentes e ambos são necessários: `raf` limita a um seek
por quadro de tela (o `onUpdate` do ScrollTrigger dispara mais que isso durante um flick), e
`STEP` descarta o seek que não mudaria pixel nenhum. Sem o segundo, o decoder recebe pedidos
sub-quadro e fica ocupado sem produzir imagem.

`gsap.matchMedia(node)` faz aqui o papel do `gsap.context()`: escopa os seletores e reverte
tudo que criou. `mm.revert()` no cleanup é o que impede o Strict Mode do React 19 de registrar
o ScrollTrigger duas vezes na segunda montagem.

Nunca chame `play()` num vídeo scrubado: o playhead do decoder e o `currentTime` que você
escreve disputam o mesmo valor e a imagem oscila. E nunca dê `loop` nele — o retorno a 0
inverte a direção percebida do scroll.

## 7. Anti-patterns

- **`autoPlay` como atributo** — o navegador baixa e toca assim que o elemento entra no DOM,
  antes de o capítulo estar perto. Três capítulos assim são três decoders vivos na primeira
  pintura. Use o gate e chame `play()`.
- **`preload="auto"`** — baixa o arquivo inteiro e disputa o pool de conexões com as fontes e o
  poster do herói, adiando o LCP.
- **Servir a rendition desktop no telefone e resolver com CSS** — `width: 100%` não reduz um
  byte nem um ciclo de decode. Escolha o `src` em JS.
- **`filter: blur()` ou `mix-blend-mode` sobre `<video>`** — força o compositor a ler de volta a
  textura do decoder a cada quadro; é a diferença entre 60 e 25 fps num Android médio. Para
  escurecer, sobreponha um `<div>` de cor sólida (é o que `Scrim` faz).
- **Animar `width`/`height` do `<video>`** — cada quadro remonta o layout e o pipeline de escala
  reinicia. Anime `transform: scale()`.
- **Texto essencial dentro do quadro** — não é selecionável, não é traduzível, some sob
  `object-fit: cover` em 9:16 e não existe para busca.
- **Áudio** — nenhum vídeo desta página tem faixa de áudio (`-an`). Som que começa sozinho é
  motivo de fechar a aba, e som com controle é um controle a mais para operar.
- **Dois filmes tocando na mesma tela** — dois decoders concorrentes derrubam o frame rate dos
  dois. Se a composição pede dois, um deles é imagem.
- **Poster genérico de banco** — em rede lenta o poster fica na tela os segundos inteiros até o
  primeiro quadro decodificar, e sob `prefers-reduced-motion` ele é a única imagem que existe.
  É um quadro da footage real ou não é poster.

## 8. Checklist de verificação

Rode com `npm run dev` e o DevTools aberto.

- [ ] Aba nova, Network filtrado em `Media`: só o filme com `priority` aparece antes do
      primeiro scroll.
- [ ] Rolando até o capítulo seguinte, a requisição do filme dele **começa antes** de ele
      aparecer na tela.
- [ ] Console, com um capítulo de filme fora da tela:
      `[...document.querySelectorAll('video')].map(v => [v.paused, v.muted, v.playsInline])` —
      todo elemento fora da tela vem `paused: true`, e todos vêm `muted: true, playsInline: true`.
- [ ] Rendering → Emulate CSS `prefers-reduced-motion: reduce`, recarregar:
      `document.querySelectorAll('video').length === 0` e o poster está visível em cada capítulo.
- [ ] Emulação de iPhone + throttle "Slow 4G": poster em menos de 1 s, e a troca poster→vídeo
      acontece sem flash branco.
- [ ] Loop: assistir três ciclos completos sem conseguir localizar a emenda.
- [ ] Pausar o vídeo no quadro mais claro e medir o contraste do corpo de texto por cima:
      ≥ 4.5:1.
- [ ] Performance, 6× CPU throttle, rolando um capítulo com filme: nenhuma long task acima de
      50 ms atribuída a decode de vídeo.
- [ ] `ls -l public/media/*.mp4` — dividido pela duração em `src/generated/media.ts`, nenhum
      desktop passa de 0,45 MiB/s (472.000 bytes/s) nem nenhum mobile de 0,15 MiB/s
      (157.000 bytes/s).
- [ ] Tab do topo ao rodapé: o foco nunca para num `<video>`.
- [ ] Redimensionar a janela cruzando 768 px: o capítulo não muda de altura, e no Network não
      aparece uma segunda requisição de `.mp4` para o mesmo capítulo — a rendition é congelada
      no gate, então o vídeo em curso não recarrega.

O ScrollTrigger que dá progresso para qualquer coisa aqui é responsabilidade do
`gsap-scrolltrigger-expert`; esta skill só entrega o elemento de mídia e o número que ele
consome.
