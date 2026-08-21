# Alt text, contraste sobre mídia e conexão lenta

Apoio do [SKILL.md](SKILL.md). Passos A6, A7 e A8 do protocolo.

## Alt text

Sete regras, cada uma com a razão.

1. **Alt responde "o que se perde se a imagem não carregar?"** — não "o que aparece na imagem".
   Uma foto está na página por um motivo; o alt carrega o motivo.
2. **Nunca comece com "Imagem de" / "Foto de"** — o leitor de tela já anuncia "imagem" antes.
   "Imagem: imagem de um cachorro" é o que a pessoa ouve.
3. **Teto de trabalho: 125 caracteres.** Passou disso, o ouvinte perde o fio. O excedente vira
   `<figcaption>`, que é texto normal e todo mundo lê.
4. **Nunca repita a legenda vizinha.** Se o texto ao lado já diz, o alt é `""`. Ouvir a mesma
   frase duas vezes seguidas soa como erro de página.
5. **Decorativa é `alt=""`, nunca `alt` ausente.** Sem o atributo, alguns leitores leem o nome do
   arquivo — e `journey-03.a1b2c3d4-640.webp` é a pior frase possível.
6. **Dentro de link ou botão, o alt é a função, não a cena.** O logo que leva à home é
   `alt="Beleza Completa — página inicial"`, não `alt="logotipo"`.
7. **Texto dentro da imagem entra no alt, literal.** Um letreiro, um preço, um selo. Se está
   escrito na foto e não está no alt, não existe para quem não vê.

Escreva em pt-BR, com ponto final — o ponto faz o leitor de tela pausar.

### Os casos deste tipo de projeto

| Cena | Inútil | Útil | Por quê |
|---|---|---|---|
| **Pet em atendimento** | `alt="cachorro"` · `alt="Foto de um cão no veterinário"` | `alt="Golden retriever deitado na mesa de exame enquanto a veterinária ausculta seu peito."` | A foto existe para provar cuidado. Sujeito + ação + relação é o que prova; a raça sozinha não prova nada |
| **Pet, retrato solto** | `alt="gato fofo"` | `alt="Gato cinza olhando para a câmera, colo de uma atendente ao fundo."` | Adjetivo de opinião ("fofo") não é informação. Onde o animal está diz mais do que como ele é |
| **Equipe** | `alt="equipe"` · `alt="nossa equipe profissional e acolhedora"` | `alt="As cinco profissionais da clínica, de jaleco branco, na recepção."` | Quantas pessoas, o que vestem, onde estão. Se a legenda visível já nomeia cada uma, o alt vira `""` |
| **Profissional em ação** | `alt="atendimento"` | `alt="Profissional da Beleza Completa avaliando o rosto de uma paciente."` | É o padrão que o `src/data/site.ts` já usa. Copie o formato |
| **Fachada** | `alt="fachada da clínica"` | `alt="Fachada da clínica na Rua Honório Hermeto, com letreiro iluminado e porta de vidro no nível da calçada."` | Foto de fachada existe para alguém **reconhecer o lugar da calçada**. O alt precisa carregar as pistas de reconhecimento — letreiro, cor, esquina, nível da rua — e não a palavra "fachada" |
| **Ambiente / sala** | `alt="recepção"` | `alt="Recepção da clínica Beleza Completa, com poltronas claras, iluminação suave e o letreiro da marca."` | Já em produção. Três atributos concretos bastam para a pessoa montar a cena |
| **Antes e depois** | `alt="antes e depois"` | `alt="Antes e depois de harmonização facial: à esquerda, sulcos marcados ao redor da boca; à direita, contorno suavizado."` | Sem dizer o que mudou, a imagem não comunica nada a quem não vê. O aviso de "resultados variam" é texto na página, nunca dentro do alt |
| **Poster de vídeo** | `alt="vídeo"` · `alt=""` | `alt="Aplicação de procedimento estético facial com técnica precisa."` | Sob reduced motion o vídeo **nunca é montado** e o poster é tudo que existe. Descreva o clipe, não o frame |
| **Passo numerado** | `alt="passo 3"` | `alt=""` + o número e o título como texto ao lado | O número já é texto; repetir é ruído |
| **Filete, textura, scrim** | qualquer texto | `alt=""` + `aria-hidden="true"` no contêiner | Não carrega informação. Anunciá-la só custa tempo |
| **Ícone dentro de botão com label** | `alt="seta"` | `aria-hidden="true"` | O botão já tem nome. A seta é reforço visual |

## Texto sobre vídeo

A armadilha: o contraste é medido contra o **pixel mais claro que o texto pode cobrir**, em
qualquer frame e em qualquer recorte de viewport. Um verificador que amostra um screenshot
aprova uma página que reprova três segundos depois. O poster é o pior lugar possível para medir
— ele foi escolhido a dedo justamente por ser bonito.

### Quanto scrim

Opacidade mínima de `--color-ink` (#32151e) sobre o pixel mais claro que o texto cobre:

| Pixel do frame | Texto branco 16px (4.5:1) | Texto branco ≥24px (3:1) |
|---|---|---|
| #ffffff | **0.62** | 0.48 |
| #cccccc | 0.50 | 0.35 |
| #808080 | 0.14 | dispensa scrim (3.9:1) |

Traduzido para os gradientes do `Scrim.tsx`, assumindo o pior caso (frame branco):

| variant | Faixa onde o scrim ≥ 0.62 | Consequência |
|---|---|---|
| `left` / `right` | primeiros ~40% da largura (o stop cai de 0.78 a 30% para 0.32 a 62%; 0.62 fica em 41% da linha do gradiente, ≈40% da largura com a inclinação de 10°) | copy de 16px precisa terminar antes de 40% do quadro |
| `bottom` | ~32% inferiores da altura (0.92 a 0%, 0.6 a 34%; 0.62 fica em 32%) | copy de 16px vive na faixa de baixo, não no meio |
| `center` | **nenhuma** — o centro é o ponto mais claro (0.50 → 3.3:1) | é scrim de **headline**: `--text-statement` (piso 26px) passa com 3:1; um lead de 16px reprova. Correção: subir o stop central de `0.5` para `0.66`, que dá 5.4:1 |

### O procedimento

```ts
const SRGB = (c: number) => (c <= 0.04045 ? c / 12.92 : ((c + 0.055) / 1.055) ** 2.4)
const lum = (r: number, g: number, b: number) =>
  0.2126 * SRGB(r / 255) + 0.7152 * SRGB(g / 255) + 0.0722 * SRGB(b / 255)

/** Contraste de texto branco sobre uma cor. */
const onWhiteText = (r: number, g: number, b: number) => 1.05 / (lum(r, g, b) + 0.05)

/** Menor alpha de ink que leva texto branco ao alvo sobre o pixel [r,g,b]. */
function scrimAlpha([r, g, b]: [number, number, number], target = 4.5): number {
  const ink: [number, number, number] = [0x32, 0x15, 0x1e]
  for (let a = 0; a <= 1; a += 0.01) {
    const [mr, mg, mb] = [r, g, b].map((c, i) => a * ink[i] + (1 - a) * c)
    if (onWhiteText(mr, mg, mb) >= target) return Number(a.toFixed(2))
  }
  return 1
}

/** Pixel mais claro do vídeo sob o retângulo de um bloco de texto. */
function brightestUnder(textEl: HTMLElement, video: HTMLVideoElement): [number, number, number] {
  const t = textEl.getBoundingClientRect()
  const v = video.getBoundingClientRect()
  const canvas = document.createElement('canvas')
  canvas.width = Math.max(1, Math.round(t.width / 4))
  canvas.height = Math.max(1, Math.round(t.height / 4))
  const ctx = canvas.getContext('2d', { willReadFrequently: true })!

  // object-fit: cover — o vídeo é maior que a caixa em um dos eixos.
  const scale = Math.max(v.width / video.videoWidth, v.height / video.videoHeight)
  const sx = (t.left - (v.left + (v.width - video.videoWidth * scale) / 2)) / scale
  const sy = (t.top - (v.top + (v.height - video.videoHeight * scale) / 2)) / scale
  ctx.drawImage(video, sx, sy, t.width / scale, t.height / scale, 0, 0, canvas.width, canvas.height)

  const { data } = ctx.getImageData(0, 0, canvas.width, canvas.height)
  let best: [number, number, number] = [0, 0, 0]
  let bestL = -1
  for (let i = 0; i < data.length; i += 4) {
    const L = lum(data[i], data[i + 1], data[i + 2])
    if (L > bestL) { bestL = L; best = [data[i], data[i + 1], data[i + 2]] }
  }
  return best
}
```

Rode `brightestUnder` em pelo menos cinco posições do clipe (`video.currentTime = d * k / 4`,
`k = 0..4`), em 390×844 e em 1440×900 — o recorte de `object-fit: cover` muda com a proporção, e
a região sob o texto muda com ele. Pegue o pior resultado, compare com `scrimAlpha` e confira
contra a opacidade real do gradiente naquela posição.

**Reprova se** o alpha necessário no pior frame é maior que o alpha aplicado ali.

Se o scrim necessário fica tão escuro que apaga a fotografia, o problema não é o scrim: é a copy
estar no lugar errado do quadro. Mova o texto para onde a imagem já é escura.

## Conexão lenta e WebViews

O público chega por WhatsApp e Instagram, quase sempre em Android de entrada, quase sempre em
rede móvel. O que isso muda, concretamente:

- **Cada visita é fria.** A WebView do Instagram tem cache próprio, isolado do Chrome. Nada do
  que o navegador do aparelho já baixou ajuda. O primeiro carregamento é o único carregamento.
- **`saveData` existe e ninguém checa.** Trate como reduced motion — o vídeo não é montado.
  `navigator.connection` não é padronizado e não existe no Safari, então o tipo é declarado à
  mão e a ausência significa "sem restrição":

  ```ts
  type NetInfo = { saveData?: boolean; effectiveType?: string }

  /** Lido uma vez por montagem. `connection` só emite `change`, e nem sempre —
      não vale um `useSyncExternalStore` como o de `useReducedMotion`. */
  export function useDataSaver(): boolean {
    return useMemo(() => {
      const c = (navigator as Navigator & { connection?: NetInfo }).connection
      return Boolean(c?.saveData) || /^(slow-)?2g$/.test(c?.effectiveType ?? '')
    }, [])
  }
  ```

  Em `ChapterFilm`, o mesmo caminho do poster: `if (reducedMotion || dataSaver) return <Picture …>`.
  1,0–1,3 MB por capítulo é o custo real do vídeo mobile deste projeto; três capítulos são 3,5 MB
  que alguém paga por megabyte.
- **A barra do app não retrai.** Dentro da WebView do Instagram há uma barra fixa acima da
  página. `100vh` conta com a retração que nunca vem — `100svh` é o único valor honesto.
- **Autoplay pode ser recusado.** Em WebView de iOS, `video.play()` rejeita mesmo com `muted` +
  `playsInline`. O `.catch()` precisa deixar o poster como estado final aceitável, não como
  placeholder eterno — é o que `ChapterFilm` faz com `.catch(() => setIsReady(true))`.
- **A prévia do WhatsApp tem teto.** `og:image` acima de ~300 KB não é renderizada e o link chega
  sem imagem. Verifique: `du -h public/media/og-image.jpg`. Hoje: 64 KB.
- **Orçamento de LCP.** Alvo ≤ 2.5s sob o throttling móvel que o Lighthouse aplica por padrão
  (150 ms RTT, 1,6 Mbps de descida, CPU 4×). Meça sempre no build de `npm run preview`: o dev
  server do Vite não tem bundle nem minificação, e o número que ele dá não existe em produção.
  O candidato a LCP aqui é o poster do hero, pré-carregado do `<head>` com `imagesrcset` em AVIF.
  Se alguém trocar o LCP por um `<video>`, a métrica piora: o navegador não considera o primeiro
  quadro de vídeo tão cedo quanto uma imagem decodificada.
- **CLS.** Toda `<img>` com `width`/`height` do manifesto. O `Picture` deste projeto já passa os
  dois a partir de `ResponsiveImage`; qualquer `<img>` escrita à mão fora dele é regressão.
