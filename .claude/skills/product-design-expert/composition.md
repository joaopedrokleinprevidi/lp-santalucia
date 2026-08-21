# Composição de seção — os cinco padrões

Todo bloco aqui usa os componentes reais do projeto: `Chapter` (seção alta com palco sticky de
100svh), `.beat` (camada absoluta que cruza fade sob controle de scroll), `.container-editorial`
(1280px + `padding-inline: clamp(24px, 5vw, 64px)`), `Scrim`, `Eyebrow`, `Words`, `Picture`.

**Contrato com o motion:** marque os alvos de animação com `data-*` (`data-title`, `data-lead`,
`data-pillar`), nunca com a classe de layout. Renomear uma classe de layout não pode quebrar uma
timeline, e é exatamente o que acontece quando `useChapterTimeline` seleciona `.max-w-2xl`.

## Matriz de sucessão

| Seção n | Sucessores válidos | Proibido |
|---|---|---|
| Centrado | Split, Borda, Full-bleed | Centrado |
| Split 5/7 | Split 7/5 (só com densidade diferente), Centrado, Borda, Sobreposto | Split na mesma direção |
| Borda | Centrado, Split, Full-bleed | Borda na mesma âncora |
| Full-bleed | Split, Borda, Centrado | Full-bleed |
| Sobreposto | Centrado, Split | Sobreposto |

Em 6 seções: ≥4 padrões distintos, nenhum mais de 2×. Vizinhas diferem em alinhamento **e** em
densidade.

```js
// Densidade da seção visível: % do viewport ocupado por tipo. Leve <25%, densa >45%.
const scope = document.querySelector('#consulta .beat')
const area = [...scope.querySelectorAll('h1,h2,h3,p,li,span,a')]
  .filter((el) => el.children.length === 0 && el.textContent?.trim())
  .reduce((sum, el) => {
    const r = el.getBoundingClientRect()
    return sum + r.width * r.height
  }, 0)
console.log(((area / (innerWidth * innerHeight)) * 100).toFixed(1) + '%')
```

## 1. Centrado

Hero, statement de uma frase, CTA final. Só quando o conteúdo é curto e não existe elemento
secundário disputando a linha do olhar.

```tsx
<div className="beat items-center justify-center" data-closing>
  <div className="container-editorial text-center">
    <Eyebrow className="justify-center text-rose-soft">{finale.eyebrow}</Eyebrow>

    <h2
      id="chapter-finale-title"
      data-title
      className="text-hero mx-auto mt-6 max-w-[18ch] font-display font-semibold text-balance text-white"
    >
      <Words text={finale.title} />
    </h2>

    <p data-lead className="text-lead mx-auto mt-8 max-w-[52ch] text-white/80">
      {finale.lead}
    </p>

    <Action data-cta className="mt-10" href={finale.cta.href}>
      {finale.cta.label}
    </Action>
  </div>
</div>
```

`max-w-[18ch]` no título e `max-w-[52ch]` no lead: sem os dois limites o texto centrado esparrama
até 1280px e cada linha começa num ponto diferente — o olho perde a âncora e a leitura para.

## 2. Split 5/7

Texto com prova visual em relação 1:1. A coluna 6 fica vazia de propósito: é o gutter da
composição, largo o bastante para o par ler como duas ideias ligadas, não como duas colunas de
tabela.

```tsx
<div className="beat items-center">
  <div className="container-editorial grid grid-cols-4 items-center gap-x-4 gap-y-10 sm:grid-cols-8 sm:gap-x-6 lg:grid-cols-12 xl:gap-x-8">
    <div className="col-span-4 sm:col-span-8 lg:col-span-5">
      <Eyebrow className="text-rose-soft">{clinic.eyebrow}</Eyebrow>
      <h2 data-title className="text-chapter mt-6 max-w-[20ch] font-display font-semibold text-balance">
        <Words text={clinic.title} />
      </h2>
      <p data-lead className="text-lead mt-6 max-w-[45ch] text-body-soft">
        {clinic.lead}
      </p>
    </div>

    <figure data-media className="col-span-4 sm:col-span-8 lg:col-span-6 lg:col-start-7">
      <Picture image={media.images.clinicRoom} className="rounded-card w-full" />
      <figcaption className="text-caption mt-3 text-body-soft">{clinic.caption}</figcaption>
    </figure>
  </div>
</div>
```

No mobile o split empilha. Decida explicitamente a ordem: mídia primeiro quando ela é a prova
(uma sala, um resultado), texto primeiro quando ela é ilustração. `order-first lg:order-none`
resolve sem duplicar markup.

## 3. Alinhado à borda

Copy sobre filme. O tipo se ancora numa margem e deixa o centro do quadro para o assunto — o
padrão de `ChapterConsultation`, onde a footage tem rostos.

```tsx
<ChapterFilm film={media.videos.consultation} mode="loop" posterAlt="…" />
<Scrim variant="bottom" />

<div className="beat items-end pb-[12svh]" data-intro>
  <div className="container-editorial">
    <div className="max-w-[42ch]">
      <div data-eyebrow>
        <Eyebrow className="text-rose-soft">{consultation.eyebrow}</Eyebrow>
      </div>
      <h2
        id="chapter-consulta-title"
        data-title
        className="text-shadow-cinema text-chapter mt-6 font-semibold text-balance text-white"
      >
        <Words text={consultation.title} />
      </h2>
      <p data-lead className="text-lead mt-6 max-w-[48ch] text-white/80">
        {consultation.lead}
      </p>
    </div>
  </div>
</div>
```

O `Scrim` é obrigatório e direcional: um gradiente que escurece só a borda ancorada. Scrim de
tela inteira apaga a imagem que justificava o vídeo. `pb-[12svh]` mantém o texto acima da chrome
do navegador móvel — `svh` e não `vh`, porque `vh` mede o viewport *grande* e dentro de um palco
de `100svh` daria um respiro maior que o pedido justamente com a barra de URL na tela. O palco
mede `svh` e a mídia mede `lvh` exatamente por isso.

## 4. Full-bleed

Uma peça de mídia que é o argumento, ou o marquee de `--text-display`. Fora de um palco sticky, o
escape da margem:

```tsx
<div className="relative left-1/2 w-screen -translate-x-1/2 overflow-hidden">
  <div className="marquee-track flex w-max gap-[6vw]" aria-hidden="true">
    <span className="text-display font-display leading-none whitespace-nowrap uppercase">
      {marquee.text}
    </span>
    <span className="text-display font-display leading-none whitespace-nowrap uppercase">
      {marquee.text}
    </span>
  </div>
</div>
```

`w-screen` é `100vw` e inclui a largura da barra de rolagem no desktop — sem `overflow-x: clip`
no `body` (o projeto já tem) isso vira ~15px de rolagem horizontal. O texto é duplicado para que
a volta do loop nunca mostre vazio, e o wrapper leva `aria-hidden="true"`: um leitor de tela
anunciando a mesma frase duas vezes é ruído, não ênfase.

## 5. Sobreposto

Número atrás do título, mídia cruzando o tipo, passagem entre capítulos. Empilhe na mesma célula
de grade em vez de usar margem negativa — margem negativa muda a altura do fluxo e o ScrollTrigger
remede tudo no primeiro resize.

```tsx
<div className="beat items-center">
  <div className="container-editorial grid grid-cols-1 grid-rows-1">
    <span
      aria-hidden="true"
      className="text-display col-start-1 row-start-1 -translate-x-[0.08em] font-display leading-none text-rose/12 select-none"
    >
      {String(index).padStart(2, '0')}
    </span>

    <div className="col-start-1 row-start-1 self-end pl-[12%]">
      <h2 data-title className="text-chapter max-w-[20ch] font-display font-semibold text-balance">
        <Words text={chapter.title} />
      </h2>
    </div>
  </div>
</div>
```

`pl-[12%]` fica dentro da faixa de 8–15%: abaixo de 8% lê como desalinhamento acidental, acima de
20% o título perde legibilidade sobre a forma de trás. O número decorativo é `aria-hidden` e
`select-none`; a 12% de opacidade ele não precisa passar contraste porque não carrega informação.

## Alinhamento óptico

```tsx
{/* Aspas de abertura penduradas: a haste da aspa não conta como início de linha. */}
<blockquote className="text-statement -ml-[0.42em] font-display">“{quote}”</blockquote>

{/* Letra redonda no início de uma manchete: overshoot de 1–3px para parecer alinhada. */}
<h2 className="text-chapter -ml-[0.02em]">Odontologia…</h2>
```

Alinhamento geométrico exato de aspas, círculos e letras redondas parece torto — o olho compara
área, não coordenada. Corrija só o primeiro caractere e só quando ele for redondo ou pontuação.

## Colapso no mobile, por padrão

| Padrão | 360–639px |
|---|---|
| Centrado | mantém; título cai para 12–16ch por linha |
| Split | empilha, ordem decidida; gutter vira `gap-y-10` (40px) |
| Borda | mantém a âncora inferior, `pb-[12svh]`, texto até 45ch |
| Full-bleed | mantém; marquee vai ao teto da faixa (15vw) para não passar de 3 palavras por tela |
| Sobreposto | reduz a sobreposição para 6–8% ou vira empilhado — 15% em 360px cobre o título |
