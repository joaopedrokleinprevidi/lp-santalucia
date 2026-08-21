# Rede móvel real, WebView e economia de dados

Apoio do [SKILL.md](SKILL.md). Passos P2 e P4.

O público chega por WhatsApp e Instagram, quase sempre em Android de entrada, quase sempre em rede
móvel. O que isso muda, concretamente.

## Cada visita é fria

A WebView do Instagram tem cache próprio, isolado do Chrome. Nada do que o navegador do aparelho
já baixou ajuda: fonte, ícone, framework, tudo desce de novo. **O primeiro carregamento é o único
carregamento**, e é ele que o orçamento de 400 KB de caminho crítico protege.

Consequência prática: header de cache imutável melhora a segunda visita de quem volta pelo
navegador, e não melhora nada para quem chega pelo link. Otimize o primeiro carregamento primeiro.

## `saveData` existe e ninguém checa

Trate como reduced motion: o vídeo não é montado. `navigator.connection` não é padronizado e não
existe no Safari, então o tipo é declarado à mão e a ausência significa "sem restrição".

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

## A barra do app não retrai

Dentro da WebView do Instagram há uma barra fixa acima da página. `100vh` conta com uma retração
que nunca vem — `100svh` é o único valor honesto. É achado de `audit-responsivo` (R1), mas aparece
aqui porque o sintoma costuma ser confundido com lentidão: o CTA "não aparece" e a pessoa fecha.

## Autoplay pode ser recusado

Em WebView de iOS, `video.play()` rejeita mesmo com `muted` + `playsInline`. O `.catch()` precisa
deixar o poster como estado final aceitável, não como placeholder eterno — é o que `ChapterFilm`
faz com `.catch(() => setIsReady(true))`. Um vídeo que nunca inicia e um spinner eterno custam o
mesmo download e entregam coisas diferentes.

## A prévia do WhatsApp tem teto

`og:image` acima de ~300 KB não é renderizada e o link chega sem imagem. Verifique:

```bash
du -h public/media/og-image.jpg
```

Formato JPEG, sempre — AVIF e WebP não são renderizados de forma confiável pelos crawlers de
prévia. Dimensão 1200×630. Hoje o arquivo tem 63 KB.

## A aritmética do 4G lento

O throttling móvel padrão do Lighthouse entrega **1,6 Mbps de descida = 200 KB/s**, com 150 ms de
RTT e CPU 4× mais lenta. Traduzindo:

| Peso | Tempo só de transferência |
|---|---|
| 400 KB | 2,0s |
| 1 MB | 5,1s |
| 3,5 MB | 17,9s |
| 8 MB | 41,0s |

Por isso o orçamento por Experience Score não é um alvo de tempo: é um teto de dados. O tempo é
governado pelo caminho crítico, e o resto precisa chegar sob demanda ou não chegar.

O candidato a LCP aqui é o poster do hero, pré-carregado do `<head>` com `imagesrcset` em AVIF. Se
alguém trocar o LCP por um `<video>`, a métrica piora: o navegador não considera o primeiro quadro
tão cedo quanto uma imagem decodificada.

## CLS de origem invisível

Toda `<img>` com `width`/`height` do manifesto. O `Picture` deste projeto já passa os dois a
partir de `ResponsiveImage`; qualquer `<img>` escrita à mão fora dele é regressão.

O segundo produtor de CLS é a fonte: se a família final tem métrica diferente da fallback, o texto
reflui na troca. `size-adjust` e `ascent-override` na `@font-face` de fallback corrigem sem
esperar o download — e `font-display: swap` sem esse ajuste é exatamente o caso que gera o salto.
