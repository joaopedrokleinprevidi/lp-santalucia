# Handoff

O que sai desta direção, para quem vai, e o que volta.

## Fronteiras

`landing-storytelling-director` decide **a ordem dos capítulos, o beat emocional e o `share` de
scroll de cada um**, e converte `share` em `scroll` / `scrollMobile` pela sua própria função
`scrollProp()`. Ela é a dona desses números.

A direção criativa entrega a essa skill, na passada 1, apenas **o `depth` total e a posição do
pico na curva** — não um `scroll` por capítulo. Na passada 2 ela lê o story map pronto, atribui a
`band` pela posição medida do meio de cada capítulo, e decide **quantos pontos e quais WOW cabem
ali**. Se um capítulo estourar o teto de pontos da sua band, corte pontos ou devolva o capítulo
para o storytelling repartir o `share` — nunca estique o `scroll` por conta própria.

A tabela de calibração no SKILL.md repete `scroll` e `scrollMobile` do story map só para as
colunas de ponto e faixa terem contra o que ser lidas. Divergiu? A do story map vence.

`landing-motion-expert` decide **a linguagem de motion** — duração, ease, distância de
deslocamento, stagger. A direção criativa decide **quantas coisas se movem e onde**. Um plano
que já escolheu `power3.out` invadiu o território errado.

`product-design-expert` decide **como a cena parece parada**. Se um capítulo não funciona
parado, nenhum orçamento de motion conserta — devolva para lá antes de alocar pontos.

## `design/creative-direction.json` — a página de referência, preenchida

```json
{
  "experienceScore": 5,
  "rationale": "Três filmes dirigidos, cinco princípios com copy própria e uma jornada de cinco passos sustentam um pico real; sem esse material o alvo seria 4.",
  "depth": 34.8,
  "budget": { "pointsTotal": 50, "mediaDesktopMB": 11.2, "mediaMobileMB": 3.5, "eagerMB": 3.2 },
  "chapters": [
    { "id": "inicio", "band": "abertura", "scroll": 4.6, "scrollMobile": 3.2, "points": 6,
      "entrance": "mask-lines-late", "silence": false,
      "roi": "Sem o atraso das linhas até 65%, a promessa chega antes de o filme estabelecer onde estamos." },
    { "id": "experiencia", "band": "escalada", "scroll": 3.2, "scrollMobile": 2.2, "points": 5,
      "entrance": "intro-exit-stagger", "silence": true,
      "roi": "Sem a saída do intro, os pilares competem com a headline que ainda está na tela." },
    { "id": "jornada", "band": "escalada", "scroll": 6, "scrollMobile": 4.2, "points": 11,
      "entrance": "rail-steps", "silence": false,
      "roi": "O movimento é a informação: cinco passos em ordem só são uma ordem se avançarem." },
    { "id": "consulta", "band": "plato", "scroll": 4, "scrollMobile": 2.8, "points": 6,
      "entrance": "media-push-in", "silence": false,
      "roi": "Sem a aproximação, a troca de cena com o capítulo anterior é um corte seco." },
    { "id": "cuidado", "band": "pico", "scroll": 4.8, "scrollMobile": 3.4, "points": 13,
      "entrance": "cycle-replace", "silence": false,
      "roi": "Uma grade deixaria o olho passar pelos cinco princípios de uma vez; o ciclo obriga a ordem de leitura." },
    { "id": "clinica", "band": "fecho", "scroll": 3.2, "scrollMobile": 2.4, "points": 5,
      "entrance": "media-pull-back", "silence": true,
      "roi": "Sem o recuo, o capítulo entra no mesmo volume do pico e o pico deixa de ser pico." },
    { "id": "agendar", "band": "fecho", "scroll": 2, "scrollMobile": 1.4, "points": 4,
      "entrance": "block-settle", "silence": false,
      "roi": "Sem o bloco único, o convite chega fatiado em stagger e o CTA parece mais uma seção." }
  ],
  "wow": [
    { "tier": "major", "chapters": ["cuidado"], "pattern": "pinned-chapter-storytelling",
      "cost": { "kb": 0, "loc": 110, "viewports": 4.8 },
      "fallback": "O stage deixa de ser sticky e os cinco princípios viram uma lista empilhada e visível, na mesma ordem.",
      "owner": "gsap-scrolltrigger-expert" },
    { "tier": "medium", "chapters": ["jornada"], "pattern": "rail-steps",
      "cost": { "kb": 0, "loc": 62, "viewports": 6 },
      "fallback": "O rail recebe flex-wrap e os cinco cartões empilham; thread e contador são ocultados.",
      "owner": "gsap-scrolltrigger-expert" },
    { "tier": "medium", "chapters": ["inicio", "consulta", "cuidado"], "pattern": "background-loop-film",
      "cost": { "kb": 11157, "loc": 38, "viewports": 0 },
      "fallback": "Nenhum elemento <video> é criado; cada capítulo carrega o poster responsivo no lugar.",
      "owner": "scroll-video-director" }
  ]
}
```

Três leituras que este JSON deixa explícitas, e que são as que mais escapam:

- **O major custa 0 KB.** O pin é CSS (`.chapter` + `.chapter__stage`). Os 4,28 MB do filme de
  `cuidado` estão na entrada de `background-loop-film`, junto com os outros dois — é lá que se
  corta peso, e separá-los é o que torna possível cortar um sem perder o outro.
- **O ciclo de princípios não é uma quarta entrada.** Ele é o mecanismo do major; se fosse uma
  entrada própria, os 4,8 viewports de `cuidado` apareceriam contados duas vezes.
- **Os três filmes são uma vaga de medium, não três.** Na terceira ocorrência o filme de fundo
  virou a textura da página; o visitante não o registra como um terceiro momento. Contar três
  estouraria a faixa de medium do ★★★★★ (2–3) sem que nada tivesse mudado na página.

Soma: 1 major + 2 medium, e `cost.kb` do array = 11 157 KB = `budget.mediaDesktopMB` (11,2).
Todo MB tem dono.

Os `small` não entram no array `wow` — eles são atribuídos ao especialista dono no momento da
delegação. Mas a rubrica **conta** smalls (★★★★★ pede 8–14), então liste-os por capítulo antes
de delegar, mesmo que fora do JSON: é o item que passa despercebido com mais frequência, porque
cada um isolado é barato demais para chamar atenção.

## Tabela de delegação

Delegue na ordem do CLAUDE.md. Passe **o recorte do JSON que é do especialista**, não o arquivo
inteiro.

| Especialista | Recebe deste plano | Precisa devolver |
|---|---|---|
| `landing-storytelling-director` | Da passada 1: `experienceScore`, `depth` total e em que posição da curva o pico cai | Ordem final, copy por capítulo, `share` → `scroll` / `scrollMobile`, posição dos CTAs |
| `product-design-expert` | `chapters[].band` e `points` | Composição de cada capítulo funcionando **parada**, escala tipográfica, contraste medido |
| `landing-motion-expert` | O plano inteiro | Linguagem de motion (duração, ease, stagger, distância) e o roteamento para os especialistas abaixo |
| `gsap-scrolltrigger-expert` | `chapters[].scroll`, `scrollMobile`, `entrance`, e os `wow` cujo `owner` é ele | Timelines por capítulo dentro do orçamento de viewports declarado |
| `scroll-video-director` | Os `wow` de vídeo, `budget.mediaDesktopMB`, `mediaMobileMB`, `eagerMB` | Renditions transcodadas, poster, gating por `IntersectionObserver` |
| `video-to-website` | Apenas um `wow` de `pattern: "canvas-frame-sequence"` | Sequência de frames dentro do orçamento de KB declarado |
| `motion-ui-expert` | A lista de smalls por capítulo | Estados de componente, sem tocar em scroll |
| `responsive-e-acessibility` | Todo `fallback` do array `wow` | Confirmação de que cada fallback existe no código e foi visto rodando |

## O que cobrar de volta

A direção não termina no handoff. Depois da implementação, três números precisam bater com o
JSON — e é a mesma direção que cobra:

1. `sum(1 + scroll)` no código bate com `depth` (±1 viewport).
2. Os pontos contados no código bate com `chapters[].points` (±2 por capítulo). Um capítulo que
   cresceu 4 pontos durante a implementação saiu da sua band sem ninguém decidir isso.
3. O peso das renditions bate com `budget`. Meça, não pergunte — `node`, porque `-printf` é
   extensão do GNU find e não existe no `find` do Windows:

```bash
node -e "
const fs = require('fs'), p = 'public/media';
const rows = fs.readdirSync(p).filter(f => f.endsWith('.mp4'))
  .map(f => [fs.statSync(p + '/' + f).size, f]).sort((a, b) => b[0] - a[0]);
for (const [s, f] of rows) console.log((s / 1e6).toFixed(2).padStart(6), 'MB', f);"
```

Divergência não é falha de quem implementou — é uma decisão criativa tomada por acidente. Ou o
plano é corrigido e volta a valer, ou o excesso é cortado. As duas saídas servem; deixar os dois
números diferentes é a única que não serve.
