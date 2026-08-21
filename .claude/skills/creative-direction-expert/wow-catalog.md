# Catálogo de WOW moments

Custo declarado antes de implementar, para poder ser cobrado depois. `kb` é o que o momento
adiciona ao download; `loc` é o tamanho do hook ou da timeline que o implementa; `viewports` é
o quanto ele consome do orçamento de scroll do capítulo.

`kb` é o custo **do mecanismo**, não do capítulo. `pinned-chapter-storytelling` custa 0 KB porque
o pin é CSS; se o capítulo pinado também tem um filme de fundo, esse filme é uma **entrada
separada** de `background-loop-film`, com o seu próprio KB e a sua própria vaga de medium. Somar
os dois numa entrada só esconde qual dos dois cortar quando o orçamento estourar.

Os snippets existem para o custo em LOC ser conferível. Quem escreve é o especialista dono. `q`
é o helper de `src/lib/gsap.ts` (`q(scope, seletor)` → `HTMLElement[]` dentro do capítulo).

## Major — na faixa de pico (65–85%)

Quantos cabem é a linha do Score na rubrica do SKILL.md (★★★★☆ leva 1; ★★★★★ leva 1–2), não um
número fixo deste catálogo.

| Padrão | kb | loc | viewports | Dono | Fallback obrigatório |
|---|---|---|---|---|---|
| `canvas-frame-sequence` | 2 000–8 000 | 120–160 | 4–6 | `video-to-website` | Poster do frame 1 em `<Picture>`; capítulo vira seção estática empilhada |
| `scrubbed-currentTime` | 1 000–4 000 | 50–80 | 4–6 | `scroll-video-director` | Poster; o `<video>` nunca é criado |
| `pinned-chapter-storytelling` | 0 | 60–110 | 4–6 | `gsap-scrolltrigger-expert` | `.chapter { height: auto }` — os beats voltam ao fluxo normal, um abaixo do outro |
| `horizontal-chapter` | 0 | 70–120 | 5–6 | `gsap-scrolltrigger-expert` | Rail com `flex-wrap: wrap`; os itens empilham e todos ficam alcançáveis |

**Quando vale:** quando a mudança ao longo do tempo **é** o conteúdo — uma transformação, um
procedimento, uma sequência que só faz sentido em ordem. **Quando não vale:** quando a mesma
informação cabe numa foto com legenda. Faça o teste honesto: se o capítulo fosse uma imagem
estática bem legendada, o que se perde? Se a resposta for "atmosfera", use loop de vídeo (medium).

`canvas-frame-sequence` e `scrubbed-currentTime` são mutuamente exclusivos na mesma página, e a
escolha entre eles não é sua: **`canvas-frame-sequence` é a técnica primária de scrub;
`scrubbed-currentTime` é o fallback.** Um `currentTime` arbitrário resolve para o keyframe
anterior, então o scrub gagueja de um jeito que nenhum ajuste de JS conserta. Aloque a faixa e
o KB no plano; quem decide qual das duas é `scroll-video-director`, pela árvore em
`video-to-website/decision.md`.

## Medium — 2 (★★★★☆) ou 2–3 (★★★★★), separados do major por ao menos um capítulo

Os `kb` abaixo são **por ocorrência**. Um device que se repete é uma entrada só no array `wow`,
mas com o `kb` somado: três filmes de fundo de 3–4 MB são uma entrada de ~11 000 KB, não três de
3 500. E é essa entrada única que pode encostar no capítulo do major sem ferir a regra de
separação — o que a regra proíbe são dois momentos **distintos** colados.

| Padrão | kb | loc | viewports | Dono | Fallback obrigatório |
|---|---|---|---|---|---|
| `rail-steps` | 0 | 45–70 | 3–5 | `gsap-scrolltrigger-expert` | Cartões empilham em coluna; thread e contador somem (`[data-progress-row] { display: none }`) |
| `cycle-replace` | 0 | 30–50 | 3–5 | `gsap-scrolltrigger-expert` | Os N estados viram uma lista visível de uma vez |
| `card-stacking` | 0 | 40–60 | 3–4 | `gsap-scrolltrigger-expert` | Cartões em grade normal |
| `background-loop-film` | 300–1 500 | 20–40 | 0 | `scroll-video-director` | `<Picture>` do poster; o `<video>` nunca é criado |
| `background-color-handoff` | 0 | 15–25 | 1–2 | `gsap-scrolltrigger-expert` | Cada capítulo com a sua cor final, sem transição |
| `veil-close` | 0 | 10–20 | 1 | `gsap-scrolltrigger-expert` | Veil na opacidade final |

**Quando vale:** quando o conteúdo já tem estrutura — uma sequência ordenada de 4–7 itens, itens
comparáveis, dois estados tonais — e o motion só torna essa estrutura legível. **Quando não
vale:** quando o motion precisa criar a estrutura que o conteúdo não tem. Cinco cartões que não
são uma sequência não viram uma sequência por deslizarem.

## Small — cabem dentro de um beat que já existe, nunca criam beat próprio

| Padrão | kb | loc | viewports | Dono | Fallback obrigatório |
|---|---|---|---|---|---|
| `mask-lines-reveal` | 0 | 90–130, uma vez | 0 | `gsap-scrolltrigger-expert` | `.word { overflow: visible }` e opacidade 1 |
| `media-push-in` / `media-pull-back` | 0 | 8–15 | 0 | `gsap-scrolltrigger-expert` | Mídia no `scale` final, sem transform |
| `counter` | 0 | 15–25 | 0 | `motion-ui-expert` | Número final impresso |
| `magnetic-cta` | 0 | 25–35 (hook já existe) | 0 | `motion-ui-expert` | Botão parado; hover só muda cor |
| `hover-lift` | 0 | 1–3 | 0 | `motion-ui-expert` | Sem transform; `:focus-visible` continua marcando |
| `scroll-cue` | 0 | 10–15 | 0 | `motion-ui-expert` | Cue estático ou removido |
| `marquee` | 0 | 8–12 (CSS) | 0 | `motion-ui-expert` | `animation: none` — o texto para onde está |
| `curtain-lift` | 0 | 18–25 | 0 | `motion-ui-expert` | Curtain removida no primeiro frame |
| `eyebrow-slide` | 0 | 3–5 | 0 | `gsap-scrolltrigger-expert` | Eyebrow no lugar |
| `stagger-group` | 0 | 5–10 | 0 | `gsap-scrolltrigger-expert` | Grupo visível de uma vez |
| `aurora-drift` | 0 | 12–20 (CSS) | 0 | `frontend-design` | Gradiente parado |
| `image-parallax` | 0 | 6–12 | 0 | `gsap-scrolltrigger-expert` | Imagem sem transform |

`mask-lines-reveal` é o único small com custo de LOC alto, e é pago **uma vez**: o hook
`useChapterReveals` já existe e serve a página inteira. Por capítulo, o custo é uma entrada no
array `REVEALS`.

**Quando vale:** quando o beat já existe e o small é o acabamento dele. **Quando não vale:**
quando você precisou alongar o `scroll` do capítulo para caber o small — isso o promove a
medium sem entregar o valor de um medium.

Um capítulo de silêncio recebe **exatamente um** small. Dois já é ruído no lugar onde o
silêncio era a decisão.

## Três mecanismos, medidos no repositório de referência

Para conferir os números de LOC das linhas acima.

**`rail-steps`** — `ChapterJourney`. O rail avança em passos, não em glide: 28% de cada beat é
viagem, 72% é o cartão parado sendo lido.

```ts
const TRAVEL_SHARE = 0.28
const beat = (RAIL_END - RAIL_START) / Math.max(cards.length - 1, 1)

for (let i = 0; i < cards.length - 1; i++) {
  const at = RAIL_START + i * beat
  tl.to('[data-rail]', {
    x: () => -(cards[i + 1].offsetLeft - cards[0].offsetLeft),
    duration: beat * TRAVEL_SHARE,
    ease: 'power2.inOut',
  }, at + beat * (1 - TRAVEL_SHARE))
}
```

Custo real: 5 cartões cronometrados + thread + contador + 2 reveals + eyebrow = 11 pontos,
6 viewports desktop.

**`cycle-replace`** — `ChapterCare`. Um princípio por vez ocupa o centro do frame. Uma grade
deixaria o olho passar por todos de uma vez; o ciclo obriga a ordem.

```ts
q(scope, '[data-principle]').forEach((principle, i) => {
  const at = CYCLE_START + i * CYCLE_BEAT
  tl.fromTo(principle, { autoAlpha: 0 }, { autoAlpha: 1, duration: 0.01 }, at)
    .to(principle, { autoAlpha: 0, y: -22, duration: 0.025 }, at + CYCLE_BEAT - 0.03)
})
```

`CYCLE_BEAT = 0.1` em unidades de história. A conta que converte isso em scroll real é
`unit × scroll × altura_do_viewport`: com `scroll={4.8}` num viewport de 900px, 0.1 são **432px,
quase metade de uma tela** por princípio — tempo de ler a passo de caminhada em vez de piscar
dentro de um flick. (O comentário no topo de `ChapterCare.tsx` diz "a third of a screen"; a
conta diz metade. A conta vence.) Abaixo de ~300px de scroll o beat some dentro de um flick de
trackpad — é por isso que `scrollMobile` não pode ser cortado junto com a unidade.

**`media-push-in` vs `media-pull-back`** — `ChapterConsultation` e `ChapterClinic`. Mesmo
mecanismo, sinal oposto, dois capítulos de distância entre eles.

```ts
// consulta — a cena se aproxima
tl.fromTo('[data-film] video, [data-film] img', { scale: 1 }, { scale: 1.05, duration: 1 }, 0)
// clinica — a cena recua e assenta
tl.fromTo('[data-room] img', { scale: 1.08 }, { scale: 1, duration: 0.7 }, 0)
```

8 a 15 linhas cada, zero KB, zero viewports. É o melhor retorno por linha do catálogo — e por
isso mesmo é o mais fácil de usar demais: acima de duas ocorrências na página, vira o tique da
página, não um recurso.

## Custos que não aparecem na tabela

- **Um major atrasa o resto do projeto**, não só o download. Trate `viewports` e `loc` como
  piso, e reserve uma passada extra de `responsive-e-acessibility` só para ele.
- **Cada rendição de vídeo dobra o trabalho de encoding** e a superfície de bug (desktop e
  mobile divergindo). `scripts/prepare-assets.mjs` já gera as duas — o custo é revisar as duas.
- **Todo padrão com `N` itens** cresce em pontos junto com o conteúdo. Um rail que nasce com 5
  cartões e vira 9 saiu da faixa da sua band sem ninguém mexer numa linha de animação.
