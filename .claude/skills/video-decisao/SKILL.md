---
name: "video-decisao"
description: "Use when a page has video and someone must decide whether it stays, what it says and which technique carries it — looping video, canvas frame sequence, or legacy currentTime scrub. Decide se o clipe fica ou vira foto, o papel narrativo (lugar, processo, transformacao, atmosfera), a tecnica por secao, loop ou once, e o fallback de scrub no iOS Safari. Keywords: video no scroll, background video, escolher tecnica de video, scroll-driven MP4."
argument-hint: "[chave-do-capitulo]"
---

# Video Decisão

| | |
|---|---|
| **ENTRADA** | `design/motion-prompts.md` (a técnica pretendida por clipe, fixada na Fase 8b) · os clipes que de fato voltaram, em `design/renders/*.mp4` ou `assets-source/*.mp4` · `design/landing-blueprint.md` (o papel de cada capítulo) · `design/creative-direction.json` (os `wow` de vídeo e `budget.mediaDesktopMB` / `mediaMobileMB` / `eagerMB`) |
| **SAÍDA** | `design/video-plan.md` — uma linha por capítulo: papel narrativo, teste honesto respondido, técnica, modo, duração-alvo, `priority` e skill dona |
| **ANTES** | `landing-motion-expert` (Fase 10c) roteia para cá assim que existe MP4 no projeto. O que já está fixado: os WOW de vídeo e o orçamento de mídia da Fase 7, e a técnica pretendida da Fase 8b |
| **DEPOIS** | `video-encode` para toda linha `<video>`; `video-to-website` para toda linha `canvas frames`; `gsap-scrolltrigger-expert` para a linha `scrub` do §6 |

Uma responsabilidade: decidir **se** o vídeo existe, **o que** ele diz e **qual técnica** o
carrega. Nenhum byte é transcodado aqui e nenhum componente é escrito aqui.

Com argumento, `[chave-do-capitulo]` é a chave do capítulo no blueprint (`hero`, `procedure`,
`consultation`) e a decisão é só dele. Sem argumento, percorra todos os capítulos que têm clipe.

## 1. O teste honesto

> Se esta seção fosse uma foto parada com uma boa legenda, o que se perderia?

| Resposta | Decisão |
|---|---|
| Nada | É foto. Corte o clipe do plano — ele custaria de 1 a 4 MiB para dizer o que um still já diz |
| Só a atmosfera: luz mudando, ar, gente passando ao fundo | `<video>` em loop |
| A mudança ao longo do tempo **é** o argumento | Sequência de frames em canvas |

Responda por escrito no plano, uma vez por capítulo. Vídeo que entra sem passar por esta
pergunta entrou por ser bonito, e é o primeiro a estourar o orçamento de mídia.

## 2. A escolha da técnica

A tabela de peso, precisão de quadro e custo de decode das três técnicas é de
[../video-to-website/decision.md](../video-to-website/decision.md). Ela é a dona: leia antes de
extrair ou transcodar qualquer coisa e não a copie para lugar nenhum. O que esta skill
acrescenta é a regra de escolha e o dono de cada saída.

| Técnica | Escolha quando | Modo | Quem implementa |
|---|---|---|---|
| `<video>` em loop | o padrão — a footage ambienta enquanto se lê por cima | `loop` | `video-encode` |
| `<video>` uma vez | o clipe tem começo e fim narrativos; rebobinar destrói o argumento | `once` | `video-encode` |
| Sequência de frames em canvas | o scroll precisa ser dono do playhead; uma por página, duas se forem distantes | — | `video-to-website` |
| `video.currentTime` scrubado | fallback declarado, nunca técnica primária — ver §5 | `scrub` | `video-decisao` + `gsap-scrolltrigger-expert` |

A maior parte do material publicado sobre scroll-driven video prescreve `currentTime` como
técnica principal. Está tecnicamente errado para qualquer página que abra no iPhone, e §5
mostra por quê.

## 3. O papel narrativo

Vídeo sem papel é peso. Antes da técnica, nomeie a função — e escreva o nome dela no plano.

| Papel | O que a footage precisa mostrar | Técnica | Modo | Duração útil | Copy por cima |
|---|---|---|---|---|---|
| Estabelecer lugar | o espaço real com gente dentro, câmera quase parada | `<video>` | `loop` | 6–12 s | sim, com scrim |
| Provar processo | mãos fazendo a coisa, plano fechado, sem corte | `<video>` | `once` | 8–15 s | pouca e curta |
| Mostrar transformação | antes e depois no mesmo enquadramento | canvas frames | — | 4–10 s de origem | não |
| Criar atmosfera | textura, luz, tecido, água; sem sujeito | `<video>` | `loop` | 4–8 s | sim, é o fundo |

**Estabelecer lugar** é o papel do herói de uma LP de negócio local: prova que o lugar existe e
é assim. Loop, porque o visitante fica ali o tempo que quiser ler. A footage nunca é o assunto —
se alguém precisa parar de ler para olhar o vídeo, a footage está agitada demais.

**Provar processo** é a única função que justifica `once`. O clipe segura o último quadro e vira
fundo estático; um filme `once` já visto não rebobina quando o capítulo volta à tela.

**Mostrar transformação** é onde o scroll precisa ser dono do playhead — e é exatamente o caso
em que `currentTime` falha. Delegue para a sequência de frames ou reescreva a seção como
antes/depois em duas imagens. Não existe terceira saída boa.

**Criar atmosfera** é o único papel em que um clipe de 4 s serve. Abaixo disso o ciclo fica
perceptível mesmo com emenda perfeita.

## 4. O corte, antes de qualquer encode

- **Um vídeo, uma ideia.** Se são precisos dois planos para dizer a coisa, são dois capítulos.
  Um clipe com corte interno compete com a narrativa da página pelo mesmo momento de atenção.
- **Loop só é loop se o último quadro casar com o primeiro.** Corte num ponto em que a câmera
  esteja parada nos dois extremos. Nunca dê `loop` em footage com movimento direcional
  (travelling, zoom, pan): o retorno ao quadro 0 é um corte seco que aparece a cada ciclo, e o
  olho localiza a emenda na terceira volta.
- **Se a copy é o ponto, a footage perde.** Texto branco de 16 px sobre um quadro com highlight
  em movimento exige `0.62` de ink sobre o pixel mais claro que ele cobre; a tabela por
  luminância e o script que a mede são de
  [audit-acessibilidade](../audit-acessibilidade/alt-e-scrim.md#texto-sobre-vídeo), que é dona do
  número. Se o quadro não permite o contraste, o corte está errado — não escureça a página
  inteira para salvar um clipe.
- **Corte antes de comprimir.** Tirar 4 s de um clipe de 14 s economiza mais bytes do que
  qualquer ajuste de CRF, e a seção quase sempre fica melhor.

## 5. Orçamento: o que baixa antes da primeira interação

O que importa não é a soma dos arquivos da pasta, é o que a página busca antes do primeiro
scroll: só o filme marcado `priority`. Todo o resto entra por gate de `IntersectionObserver`
(`video-encode` §2). **Se um capítulo não tem gate, ele não tem vídeo.**

Na LP de referência isso são **0,96 MiB no mobile e 3,06 MiB no desktop** — o herói e mais nada.
Esse é o número que precisa caber em `budget.eagerMB`; a soma de todas as renditions cabe em
`budget.mediaDesktopMB` / `mediaMobileMB` (★★★★☆ padrão: 8 MB e 3,5 MB).

A faixa de 300 KB–2 MB que `decision.md` dá para looping `<video>` é a rendition **mobile**. Um
1080p de 10 s a CRF 25 sai em 3–4 MiB e continua sendo a técnica certa, porque no desktop esse
arquivo desce um por vez, sob gate, e nunca todos juntos.

Se o plano estoura o orçamento, corte um capítulo de vídeo inteiro. Reduzir a qualidade dos
três para caber entrega três filmes ruins em vez de dois bons.

## 6. Fallback declarado: scrub por `currentTime`

**Só use quando** o clipe é longo demais para extrair frames (>30 s de movimento essencial)
**e** a seção é desktop. Em qualquer outro caso a resposta é sequência de frames ou loop.

**A limitação.** O Safari no iOS não faz seek de quadro exato em vídeo comprimido. Um
`currentTime` arbitrário resolve para o keyframe anterior, e a imagem só atualiza quando o
decoder alcança o quadro pedido — com GOP de 48 quadros isso é até 1,6 s de imagem parada
enquanto o dedo continua andando. O sintoma é gagueira, não lentidão, e não há correção do lado
do JS. É por isso que a sequência de frames em canvas existe.

Duas consequências que precisam de decisão explícita **antes** de escrever o hook:

- **`mode="scrub"` não degrada sozinho.** Onde o `matchMedia` não casa (todo touch, e com ele o
  iOS), ninguém escreve `currentTime` e ninguém chama `play()`: o capítulo congela no primeiro
  quadro. Quem monta escolhe o modo pelo ponteiro —
  `useMediaQuery('(pointer: fine)') ? 'scrub' : 'loop'` — ou a seção inteira é outra coisa no
  mobile.
- **Com `preload="metadata"` o primeiro quadro só chega no primeiro seek.** Até lá o poster cobre
  o palco, o que é correto, mas um capítulo scrubado que ninguém rola nunca mostra vídeo.

Monte o `<ChapterFilm mode="scrub" videoRef={ref}>` de `video-encode` e passe o mesmo ref para o
hook. O componente não chama `play()` nesse modo; o hook fica dono do `currentTime`.

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

      // O matchMedia reverte o ScrollTrigger sozinho; o rAF pendente não. Sem
      // isto, `flush` roda depois do unmount e escreve em elemento morto.
      return () => cancelAnimationFrame(raf)
    })

    return () => mm.revert()
  }, [scope, video])
}
```

Os dois gates são necessários por motivos diferentes: `raf` limita a um seek por quadro de tela
(o `onUpdate` do ScrollTrigger dispara mais que isso durante um flick) e `STEP` descarta o seek
que não mudaria pixel nenhum — sem ele o decoder recebe pedidos sub-quadro e fica ocupado sem
produzir imagem. `gsap.matchMedia(node)` faz aqui o papel do `gsap.context()`: escopa e reverte
tudo que criou, o que impede o Strict Mode do React 19 de registrar o ScrollTrigger duas vezes.

Nunca chame `play()` num vídeo scrubado — o playhead do decoder e o `currentTime` que você
escreve disputam o mesmo valor e a imagem oscila. Nunca dê `loop` nele: o retorno a 0 inverte a
direção percebida do scroll.

## 7. O plano

`design/video-plan.md` é a saída desta skill e a entrada das duas seguintes. Uma tabela basta:

```md
| Capítulo | Papel | Teste honesto | Técnica | Modo | Alvo | priority | Dona |
|---|---|---|---|---|---|---|---|
| hero | estabelecer lugar | perde a sala viva | `<video>` | loop | 10 s | sim | video-encode |
| procedure | provar processo | perde o gesto contínuo | `<video>` | once | 10 s | não | video-encode |
| results | mostrar transformação | perde a mudança | canvas frames | — | 6 s | não | video-to-website |
| depoimentos | — | nada se perde | still | — | — | — | (sem vídeo) |
```

Toda linha `canvas frames` sai daqui e entra em `video-to-website` com a duração de origem; toda
linha `<video>` entra em `video-encode` com modo, alvo de duração e `priority`.

## 8. Anti-patterns

- **Vídeo sem papel nomeado** — vira peso que ninguém consegue defender no corte de orçamento, e
  é sempre o primeiro a ser cortado depois, quando já custou uma rodada de geração.
- **`loop` em footage com movimento direcional** — o retorno ao quadro 0 é um corte seco visível
  a cada ciclo. Se a câmera anda, o modo é `once` ou o corte está errado.
- **Frame sequence em toda seção** — cada uma custa de 2 a 8 MB. A terceira transforma a landing
  num download e quem está no 4G fecha antes do primeiro frame.
- **`currentTime` como técnica primária** — gagueira garantida no iOS, sem correção possível em
  JS. É fallback desktop, e só sob a condição de §6.
- **Dois filmes tocando na mesma tela** — dois decoders concorrentes derrubam o frame rate dos
  dois. Se a composição pede dois, um deles é imagem.
- **Texto essencial dentro do quadro** — não é selecionável, não é traduzível, some sob
  `object-fit: cover` em 9:16 e não existe para busca. Toda copy é DOM.
- **Faixa de áudio** — som que começa sozinho é motivo de fechar a aba, e som com controle é mais
  um controle para operar. `video-encode` remove a faixa com `-an`.

## 9. Checklist da decisão

- [ ] Todo capítulo com clipe tem o teste honesto respondido por escrito no plano.
- [ ] Todo capítulo com vídeo tem um dos quatro papéis nomeado, não "ambiente".
- [ ] Nenhuma linha `mostrar transformação` saiu com técnica `<video>`.
- [ ] No máximo uma sequência de frames na página; duas só se estiverem a mais de dois capítulos
      de distância.
- [ ] Toda linha `once` tem começo e fim narrativos; toda linha `loop` foi assistida por três
      ciclos sem que a emenda fosse localizável.
- [ ] Um único capítulo com `priority: sim`, e a rendition mobile dele cabe em `budget.eagerMB`.
- [ ] A soma das renditions planejadas cabe em `budget.mediaDesktopMB` / `mediaMobileMB`.
- [ ] Toda linha tem skill dona escrita; nenhuma linha ficou sem destino.
