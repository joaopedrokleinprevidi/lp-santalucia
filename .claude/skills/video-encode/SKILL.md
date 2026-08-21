---
name: video-encode
description: Use when raw video must become web renditions and a working <video> element — ffmpeg transcode, CRF, faststart, poster frame, IntersectionObserver gating, autoplay rules, reduced-motion fallback. Transcodar video para web, gerar poster, montar o elemento video com muted playsInline loop, lazy load por gate, escolher rendition desktop ou mobile, peso por segundo. Keywords: ffmpeg, MP4, H.264, autoplay, poster, ChapterFilm, public/media.
argument-hint: [caminho-do-master] [chave-do-capitulo]
---

# Video Encode

| | |
|---|---|
| **ENTRADA** | `design/video-plan.md` (técnica, modo, alvo de duração e `priority` por capítulo) e os masters 1080p em `assets-source/*.mp4` |
| **SAÍDA** | `public/media/<chave>-{desktop,mobile}.<hash>.mp4` + poster responsivo, registrados em `src/generated/media.ts`; e o `<ChapterFilm>` montado no capítulo |
| **ANTES** | `video-decisao` — sem uma linha `<video>` no plano, esta skill não roda |
| **DEPOIS** | `gsap-scrolltrigger-expert` (consome o palco e o progresso do capítulo) e `audit-acessibilidade` (A4: scrim sobre o quadro mais claro; A5: reduced motion) |

Uma responsabilidade: transformar um master em bytes servíveis e num elemento `<video>` que
carrega tarde, toca só na tela e some sob reduced motion. **Nada de direção aqui** — se ainda
falta decidir se o clipe fica, volte para [video-decisao](../video-decisao/SKILL.md). Com
argumento, `[caminho-do-master]` é o arquivo 1080p em `assets-source/` e `[chave-do-capitulo]` é
a chave dele no array `VIDEOS` de `scripts/prepare-assets.mjs`.

## 1. O componente

A implementação provada está em `src/components/media/ChapterFilm.tsx`, reduzida ao que importa:

```tsx
interface ChapterFilmProps {
  film: ChapterVideo
  /** `loop` roda contínuo; `once` toca uma vez e segura o último quadro; `scrub`
   *  entrega o playhead ao hook de video-decisao. */
  mode: 'loop' | 'once' | 'scrub'
  posterAlt: string // descreve a footage para quem só vê o poster; obrigatório
  videoRef?: RefObject<HTMLVideoElement | null> // exposto para o modo `scrub`
  priority?: boolean
}

export function ChapterFilm({ film, mode, posterAlt, videoRef, priority = false }: ChapterFilmProps) {
  const containerRef = useRef<HTMLDivElement>(null)
  const internalRef = useRef<HTMLVideoElement>(null)
  const element = videoRef ?? internalRef

  const reducedMotion = useReducedMotion()
  const isDesktop = useIsDesktop()

  const [shouldLoad, setShouldLoad] = useState(priority)
  const [isOnScreen, setIsOnScreen] = useState(false)
  const [isReady, setIsReady] = useState(false)
  const [source, setSource] = useState<string | null>(null)

  /* A rendition é escolhida em JS — CSS não reduz um byte nem um ciclo de decode —
     e é congelada quando o gate abre (ver §7, "reavaliar no resize"). */
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

    // No `scrub` quem escreve o playhead é o hook de video-decisao; play() aqui
    // faria o decoder e o `currentTime` disputarem o mesmo valor.
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

`src` só entra no elemento depois do gate. Renderizar a tag sem `src` e adicionar depois é o que
impede o navegador de abrir conexão antes da hora — `preload="none"` sozinho não é garantia em
todos os engines.

`.chapter__media` é `position: absolute; inset: 0`, então carregar metadata nunca muda a altura
do capítulo: a altura vem de `--chapter-scroll-*`, em múltiplos de viewport. É isso que elimina
layout shift, não `aspect-ratio` no vídeo.

## 2. Os dois gates

São dois `IntersectionObserver` com margens diferentes porque resolvem problemas diferentes.

```tsx
/* Gate 1 — buscar o arquivo. Uma tela e meia de antecedência: num viewport de
   800 px isso são 1200 px, ~1 s a 1200 px/s de flick — tempo de o primeiro quadro
   chegar antes de o capítulo aparecer. Dispara uma vez e se desconecta. */
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

/* Gate 2 — decodificar. Mesmo formato, `rootMargin: '10% 0px'`, sem `disconnect()`:
   fica vivo e só alimenta `setIsOnScreen`, que o efeito de play/pause consome.
   Margem curta porque decode é o custo contínuo — um capítulo fora da tela
   decodificando rouba main thread do ScrollTrigger. */
```

Só o capítulo marcado `priority` no plano nasce com `shouldLoad = true`. Todos os outros esperam
o gate 1 — é isso que mantém o peso da primeira interação em uma rendition, não a soma da pasta.

## 3. Os atributos obrigatórios

| Atributo | Por que é obrigatório |
|---|---|
| `muted` (atributo **e** propriedade) | Chrome e Safari só liberam autoplay para vídeo mudo. Sem os dois, `play()` rejeita com `NotAllowedError` e o capítulo fica no poster para sempre |
| `playsInline` | Sem ele o iOS abre o player nativo em tela cheia no primeiro `play()`, e o capítulo inteiro some atrás dele. `muted` e `playsInline` juntos são a condição de autoplay: um sem o outro não vale |
| `preload="metadata"` | `auto` baixa o arquivo inteiro; `none` adia a duração, de que o fallback de scrub precisa. `metadata` são alguns KB |
| `disablePictureInPicture` | Sem ele, Safari e Chrome oferecem PiP num vídeo decorativo. Um fundo virando janela flutuante é um bug visível |
| `aria-hidden="true"` + `tabIndex={-1}` | Footage decorativa não deve ser anunciada como "vídeo" nem receber foco: não há controle nenhum ali para operar. Se a footage carrega informação, não esconda — use `once` e descreva o conteúdo em texto DOM ao lado |
| poster via `<Picture>`, não pelo atributo `poster` | O atributo aceita uma URL única: sem AVIF, sem `srcset`, sem LQIP e **sem `alt`**. O `<img>` sobreposto com crossfade de 700 ms dá tudo isso |

## 4. Encoding

Duas renditions por filme, ambas masterizadas do 1080p original, geradas em
`scripts/prepare-assets.mjs` com o binário de `ffmpeg-static` que já está em devDependencies.
Nunca instale ffmpeg global e nunca escreva caminho absoluto de binário num script.

```js
await ffmpeg([
  '-i', input,
  '-an',                                                  // sem faixa de áudio
  '-vf', `scale=${r.width}:${r.height}:flags=lanczos,hqdn3d=${r.denoise}`,
  '-c:v', 'libx264', '-profile:v', 'high', '-preset', 'slower',
  '-crf', String(r.crf),
  '-pix_fmt', 'yuv420p',
  '-g', '48',
  '-movflags', '+faststart',
  dest,                                                   // `${key}-${suffix}.${hash}.mp4`
])
```

| Rendition | Escala | CRF | `hqdn3d` | Serve para |
|---|---|---|---|---|
| desktop | 1920×1080 | 25 | `1:1:4:4` | `(min-width: 768px)` |
| mobile | 1280×720 | 27 | `1.5:1.5:6:6` | resto |

- **`-movflags +faststart`** — move o átomo `moov` para o início. Sem isso o índice fica no fim do
  arquivo e o navegador baixa quase tudo antes do primeiro quadro. O sintoma é poster preso por
  segundos num vídeo que já está 80% baixado.
- **`-pix_fmt yuv420p`** — 4:2:0 é o único chroma que todo decoder de hardware aceita. 4:2:2 e
  4:4:4 saem do encoder sem erro e não abrem em lugar nenhum.
- **`-g 48`** — keyframe a cada 48 quadros (1,6–2 s). GOP curto é o que torna o reinício do loop
  limpo e o seek tolerável. GOP longo economiza ~5% e transforma qualquer seek em salto.
- **`-preset slower`** — o encode roda uma vez no build; CPU ali não custa nada. Rende ~8% a menos
  de arquivo que `medium` no mesmo CRF.
- **`-an`** — vídeo mudo com faixa de áudio ainda carrega os bytes dela e ainda faz o navegador
  instanciar o pipeline de áudio.
- **`hqdn3d`** — os masters carregam grão de sensor que o x264 gasta muitos bits preservando e que
  ninguém vê em movimento. Medido contra a fonte: custa 0,0013 de SSIM e economiza 14%.
- **Hash do master no nome** — sem ele um re-encode reutiliza a URL antiga e quem já visitou fica
  com o corte anterior até o cache expirar. É assim que uma correção de qualidade não chega em
  quem viu o problema.

**Por que H.264 e não VP9/AV1.** H.264 High/yuv420p é o único perfil com decode em hardware
garantido em todo aparelho que abre esta página, iPhone antigo e Android de entrada incluídos.
VP9 tem suporte irregular no Safari e AV1 só decodifica em hardware em chips recentes; quando o
decode cai para software, o custo vai para a main thread — a mesma que roda o ScrollTrigger, e o
sintoma não é "vídeo pesado", é "scroll travando". O ganho seria ~30% de bitrate: em 3 MB,
900 KB, que não se troca por risco de scroll a 40 fps. Única exceção é canal alpha, que o H.264
não tem — aí é HEVC-alpha para Safari e WebM/VP9-alpha para o resto, via `<source>`. É caro;
prefira redesenhar a seção para não precisar de alpha.

**Tamanho-alvo.** O custo medido é de **0,31 a 0,44 MiB por segundo no desktop** e **0,10 a
0,15 MiB/s no mobile** — quanto mais movimento no quadro, mais alto dentro da faixa.

| Filme | Duração | desktop | mobile | desktop MiB/s |
|---|---|---|---|---|
| `hero` (câmera quase parada) | 10 s | 3,06 MiB | 0,96 MiB | 0,31 |
| `procedure` (mãos em primeiro plano) | 10 s | 4,09 MiB | 1,23 MiB | 0,41 |
| `consultation` | 8 s | 3,49 MiB | 1,18 MiB | 0,44 |

O teto que vale como reprovação é **0,45 MiB/s no desktop** e **0,15 MiB/s no mobile** (MiB
porque é o que `ls -l` conta: 3.213.394 bytes = 3,06 MiB).

Se um clipe estoura, corte segundos antes de mexer em qualidade. Se ainda estourar, suba o CRF
em 2 — **não** baixe a resolução: 720p esticado num palco de 1440 px foi exatamente o que fez o
filme de abertura deste projeto parecer mole. O encoder nunca foi o problema; os pixels que
faltavam eram.

## 5. Poster

```js
await ffmpeg(['-ss', String(video.poster), '-i', input, '-frames:v', '1', posterRaw])
```

Depois passa pelo mesmo pipeline responsivo das imagens (AVIF + WebP + LQIP inline). Escolha um
timestamp em que o quadro pareça uma fotografia: nada de motion blur no meio de um gesto. Os
valores usados aqui são `0.35`, `3.2` e `0.6` — nenhum é 0, porque o primeiro quadro costuma ser
o pior. O crossfade de 700 ms perdoa diferença pequena entre poster e quadro 0; se o poster for
de um momento visivelmente outro, mude o timestamp em vez de alongar o crossfade.

## 6. Reduced motion

Sob `prefers-reduced-motion: reduce` o elemento `<video>` **não é criado**. Não é pausado, não é
escondido: não existe. Pausar depois já baixou os bytes, e um `<video autoplay loop>` pausado já
deixou a política de autoplay rodar. O ramo de saída antecipada está no componente de §1, antes
de qualquer JSX de vídeo, e o poster responsivo carrega a cena inteira — é a única leitura que
aquele visitante vai ter dela, e por isso `posterAlt` é prop obrigatória. "Vídeo do capítulo" não
é alt; "Aplicação de um procedimento estético facial na sala de atendimento" é. O protocolo de
auditoria é de [audit-acessibilidade](../audit-acessibilidade/reduced-motion.md) §4.

## 7. Anti-patterns

- **`autoPlay` como atributo** — o navegador baixa e toca assim que o elemento entra no DOM,
  antes de o capítulo estar perto. Três capítulos assim são três decoders vivos na primeira
  pintura. Use o gate e chame `play()`.
- **`preload="auto"`** — baixa o arquivo inteiro e disputa o pool de conexões com as fontes e o
  poster do herói, adiando o LCP.
- **Servir a rendition desktop no telefone e resolver com CSS** — `width: 100%` não reduz um byte
  nem um ciclo de decode. Escolha o `src` em JS.
- **Reavaliar a rendition no resize** — trocar o `src` de um vídeo em reprodução recarrega o
  elemento e mostra preto, porque o poster já sumiu no crossfade. Congele no gate.
- **`filter: blur()` ou `mix-blend-mode` sobre `<video>`** — força o compositor a reler a textura
  do decoder a cada quadro; é a diferença entre 60 e 25 fps num Android médio. Para escurecer,
  sobreponha um `<div>` de cor sólida (é o que o componente `Scrim` faz).
- **Animar `width`/`height` do `<video>`** — cada quadro remonta o layout e o pipeline de escala
  reinicia. Anime `transform: scale()`.
- **Poster genérico de banco** — em rede lenta ele fica na tela os segundos inteiros até o
  primeiro quadro decodificar, e sob reduced motion é a única imagem que existe. É um quadro da
  footage real ou não é poster.
- **Caminho absoluto de binário no script** (`C:\Users\...\ffmpeg.exe`) — quebra em qualquer
  outra máquina e no CI. `ffmpeg-static` exporta o caminho.

## 8. Checklist

Rode com `npm run dev` e o DevTools aberto.

- [ ] Aba nova, Network filtrado em `Media`: só o filme `priority` aparece antes do 1º scroll, e
      a requisição do capítulo seguinte **começa antes** de ele aparecer na tela.
- [ ] Console, com um capítulo de filme fora da tela:
      `[...document.querySelectorAll('video')].map(v => [v.paused, v.muted, v.playsInline])` —
      todo elemento fora da tela vem `paused: true`, e todos vêm `muted: true, playsInline: true`.
- [ ] Rendering → Emulate CSS `prefers-reduced-motion: reduce`, recarregar:
      `document.querySelectorAll('video').length === 0` e o poster visível em cada capítulo.
- [ ] Emulação de iPhone + throttle "Slow 4G": poster em menos de 1 s, troca poster→vídeo sem
      flash branco, e nenhum player nativo em tela cheia ao tocar.
- [ ] Pausar no quadro mais claro e medir o contraste do corpo de texto por cima: ≥ 4.5:1.
- [ ] Performance, 6× CPU throttle, rolando um capítulo com filme: nenhuma long task acima de
      50 ms atribuída a decode de vídeo.
- [ ] `ls -l public/media/*.mp4` dividido pela duração em `src/generated/media.ts`: nenhum desktop
      acima de 0,45 MiB/s (472.000 bytes/s), nenhum mobile acima de 0,15 MiB/s (157.000 bytes/s).
- [ ] Tab do topo ao rodapé: o foco nunca para num `<video>`.
- [ ] Redimensionar cruzando 768 px: o capítulo não muda de altura e não aparece uma segunda
      requisição de `.mp4` para o mesmo capítulo.

O ScrollTrigger que dá progresso a qualquer coisa aqui é do `gsap-scrolltrigger-expert`: esta
skill entrega o elemento e o palco, não o número que o move.
