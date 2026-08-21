---
name: motion-ui-expert
description: Use when building or refining component-level motion — button, card, input, link, menu, modal, toast, skeleton, cursor — and every state around them: hover, press, focus-visible, disabled, loading, error, success. Animação de botão, card, formulário, menu mobile, modal, toast, skeleton, cursor; estado de hover, clique, foco, desabilitado, carregando, erro e sucesso; micro-interação, feedback de formulário, spinner, transição de UI, botão magnético. Not for scroll-driven animation.
argument-hint: [componente] [estado]
---

# Motion de componente

Scroll move a página. Isto move o componente. Tudo que responde a ponteiro, teclado, toque ou
mudança de estado interno pertence a esta skill; tudo que responde à posição do scroll não.

| A animação dispara por | Dono |
|---|---|
| hover, press, focus, disabled, loading, erro, sucesso | esta skill |
| abrir/fechar de menu, modal, toast, accordion, tab | esta skill |
| entrada de uma seção na viewport | `gsap-scrolltrigger-expert` |
| ordem e ritmo entre seções | `landing-motion-expert` |
| frame de vídeo preso ao scroll | `scroll-video-director` |

O CLAUDE.md já define o Motion ROI ("por que isto existe?"). Aqui a pergunta vira tabela: cada
estado ou tem feedback obrigatório, ou é decoração e pode ser cortado sem perda de informação.

Código completo dos padrões (tilt, menu com stagger, botão em carregamento, skeleton, toast,
clip-path, sublinhado, accordion, cursor, botão magnético) em [patterns.md](patterns.md).

## Números canônicos

A tabela de curvas é de `landing-motion-expert` — ela é dona da linguagem de motion do projeto.
Aqui só se escolhe qual linha dela cabe em cada interação de componente. Deslocamento de
controle usa `ease-out-quart` / `power3.out`; painel que abre e fecha usa `ease-out-quint` /
`power4.out`; `ease-out-expo` fica com o reveal de copy, que é da skill de scroll.

| Interação | Duração | Ease em CSS | Ease em GSAP |
|---|---|---|---|
| hover — entrada | 150–200ms | `--ease-out-quart` | `power3.out` |
| hover — saída | 120–160ms (0,7–0,8× da entrada) | `ease-out` | `power2.out` |
| press (`pointerdown`) | 80–120ms | `ease-out` | `power2.out` |
| release | 140–180ms | `--ease-out-quart` | `power3.out` |
| anel de `:focus-visible` | 0ms | — | — |
| tooltip | 120ms, após 400–600ms de espera | `--ease-out-quint` | `power4.out` |
| dropdown / menu — entrada | 180–240ms | `--ease-out-quint` | `power4.out` |
| dropdown / menu — saída | 120–160ms | `ease-in` | `power2.in` |
| modal / sheet — entrada | 250–350ms | `--ease-out-quint` | `power4.out` |
| modal / sheet — saída | 180–220ms | `ease-in` | `power2.in` |
| toast — entrada | 200ms | `--ease-out-quint` | `power4.out` |
| toast — saída | 150ms | `ease-in` | `power2.in` |
| toast — permanência | 4000ms; 5500ms se oferece ação | — | — |
| skeleton shimmer | 1200–1600ms, loop | `linear` | `none` |
| spinner | 700–900ms por volta | `linear` | `none` |
| troca de rota / tab | 300ms saída, 400ms entrada | `--ease-out-quint` | `power4.out` |

Abaixo de ~100ms a transição termina antes de a atenção chegar: o olho lê uma troca
instantânea e a informação de "o que virou o quê" se perde — é por isso que o press pode ser
curto (o dedo já sabe o que fez) mas o hover não. Acima de ~400ms a espera deixa de ser lida
como resposta e passa a ser lida como lentidão do sistema. Toda micro-interação vive nessa
faixa. As exceções são deliberadas: a entrada de um modal chega a 350ms porque é troca de
contexto e o olho precisa reancorar; nada em hover pode.

Saída sempre mais rápida que entrada. A entrada precisa ser notada; a saída precisa sair do
caminho.

**Stagger** não tem tabela própria aqui: use a tabela por contagem de itens de
`landing-motion-expert` (2–3 itens → 0,10–0,14s; 4–8 → 0,06–0,08s; 9–20 → 0,03–0,04s), com o
teto de `stagger × contagem ≤ 0,9s`. Um menu de seis capítulos cai na faixa de 0,06s.

### Tokens

`src/styles/index.css` já tem `--ease-out-expo` e `--ease-out-quint` no bloco `@theme`. Falta a
curva de deslocamento, e ela vai no `@theme` porque `--ease-*` é um dos 20 namespaces do
Tailwind 4 — a variável passa a gerar o utilitário `ease-out-quart`:

```css
@theme {
  --ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
  /* Erro de formulário. Sem token, o hex acaba escrito em cinco arquivos diferentes. */
  --color-danger: #b3261e;
}
```

As durações **não** vão no `@theme`. Não existe namespace de `transition-duration` no Tailwind 4,
então `--dur-*` não geraria utilitário nenhum — e o `@theme` descarta do output as variáveis que
não viram utilitário nem aparecem no source escaneado. A documentação do Tailwind é explícita:
`:root` é o lugar de variável que não tem utilitário correspondente.

```css
/* Fora do @theme — sobrevive ao tree-shaking sem depender do scanner. */
:root {
  --dur-press: 90ms;
  --dur-hover: 180ms;
  --dur-hover-out: 140ms;
  --dur-panel: 220ms;
  --dur-modal: 300ms;
  --dur-modal-out: 200ms;
  --dur-toast: 200ms;
}
```

Consuma pela forma canônica de variável do Tailwind 4 — `duration-(--dur-hover)`, que é o
mesmo que `duration-[var(--dur-hover)]` mas é o que o plugin do editor aceita sem aviso — ou
direto no CSS de componente. Nunca escreva o número solto em dois lugares.

Também acrescente a animação que o `Submit` usa ([patterns.md § 3](patterns.md#3-botão-em-carregamento)). O `@keyframes` vai
**dentro** do `@theme`: é assim que o Tailwind 4 associa o keyframe ao utilitário `animate-*` e
o emite junto com ele.

```css
@theme {
  --animate-spinner: spinner 800ms linear infinite;

  @keyframes spinner {
    to { rotate: 360deg; }
  }
}
```

## Tabela de estados

Cada linha é uma mudança visual observável. "Obrigatório" significa: sem isso o usuário não
sabe o que aconteceu, e a coluna sobrevive a `prefers-reduced-motion` como troca instantânea.

### Botão (`src/components/ui/Action.tsx`)

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| rest | `bg-ink`, `shadow-card`, sem transform | — | — |
| hover | `bg-ink-soft`, `shadow-lift`, `-translate-y-px`, magnético ≤5px | 180ms `--ease-out-quart` | sim — cor basta |
| hover out | volta ao rest | 140ms `ease-out` | sim |
| active | `scale(0.98)`, sombra volta a `shadow-card` | 90ms `ease-out` | sim |
| focus-visible | outline 2px `rose-soft` offset 3px **+ o mesmo lift do hover** | 0ms outline / 180ms lift | sim |
| disabled | `opacity .45`, `cursor-not-allowed`, sem hover, sem magnético | 0ms | sim |
| loading | largura travada, label a `opacity .35`, spinner 16px sobreposto, `aria-busy` | 120ms fade | sim |

O ícone que acompanha o label anda 4px no eixo x (`group-hover:translate-x-1`) em 300ms — é
camada de cima, não é obrigatório, e some sob reduced-motion. `Action.tsx` já faz isso, mas
com `group-hover:` sozinho: falta o `group-focus-visible:translate-x-1` que dá o mesmo sinal a
quem chega pelo teclado.

Cuidado com a ordem de quem escreve `transform`: se o controle usa `useMagnetic`, o GSAP grava
a propriedade `transform` inline e o `active:scale-[0.98]` do Tailwind deixa de ter efeito.
Nesses controles o press vem da opção `press` do hook, não da classe.

### Card

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| rest | borda `--color-line`, `shadow-card`, mídia `scale(1)` | — | — |
| hover | `translateY(-4px)`, `shadow-lift`, borda `ink/25`, mídia `scale(1.03)` | 220ms quart; mídia 600ms | sim — borda ou sombra basta |
| tilt | `rotationX/rotationY` ≤5°, `transformPerspective: 900` | quickTo 0.5s `power3.out` | não |
| active | `scale(0.995)`, sombra de volta a `shadow-card` | 90ms | não |
| focus-within | idêntico ao hover, mais o outline no link interno | 0 / 220ms | sim |
| loading | skeleton com a altura exata do card final | shimmer 1400ms | sim |
| indisponível | `opacity .5`, `aria-disabled` no link, hover desligado | 0ms | sim |

Card clicável inteiro: o alvo é o link do título com `::after { position: absolute; inset: 0 }`,
nunca um `onClick` na `<div>` — a div não recebe foco, não abre em nova aba e não aparece no
menu de contexto.

### Input

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| rest | borda `--color-line` | — | — |
| hover | borda `ink/25` | 150ms | não |
| focus-visible | borda `rose`, ring 3px `rose-soft/35`, label sobe 12px e vai a `0.6875rem` | 160ms quart | sim |
| preenchido | label permanece no alto | — | sim |
| inválido | borda `--color-danger`, ícone + texto entram com fade e 2px de subida | 180ms | sim |
| shake do inválido | 3 ciclos, ≤6px, ≤240ms — só acompanhando a mensagem | 240ms | não |
| válido | check desenha em 200ms — só quando a validação é assíncrona | 200ms quint | não |
| disabled | `opacity .5`, sem transição de borda | 0ms | sim |

Erro nunca é comunicado só por cor: cor **+** ícone **+** texto. Vermelho/verde é indistinguível
para cerca de 8% dos homens, e a borda sozinha não diz o que corrigir. O texto do erro só chega
ao leitor de tela se estiver amarrado: `aria-invalid="true"` no campo e `aria-describedby`
apontando para o `id` da mensagem. Sem isso o campo é anunciado como "inválido" e nada mais.

### Link inline

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| rest | `background-size: 0% 1px`, posição `100% 100%` | — | — |
| hover | `background-size: 100% 1px`, posição `0 100%` | 220ms quart | sim |
| hover out | volta a `0%` recolhendo pela direita | 220ms | não |
| focus-visible | sublinhado a 100% **imediato** + outline | 0ms | sim |
| visitado | mudança de cor, sem transição | 0ms | não |

O sublinhado que cresce da esquerda e recolhe pela direita é a única propriedade animada
(`background-size`); a posição troca sem transição. Código em
[patterns.md § 7](patterns.md#7-sublinhado-que-cresce).

### Item de menu / rail (`src/components/layout/PageChrome.tsx`)

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| rest | traço 16px, label `opacity 0` | — | — |
| hover | traço 24px, label `opacity .7` | 300ms | sim |
| focus-visible | idêntico ao hover + outline | 0ms outline | sim |
| atual | traço 32px em `--color-gilt`, label `opacity 1`, `aria-current` | 300ms | sim |

Estado atual do rail em `PageChrome.tsx`: o **label** emparelha `group-hover:opacity-70` com
`group-focus-visible:opacity-70`, mas o **traço** não — `group-hover:w-6`, `group-hover:bg-ink/60`
e `group-hover:bg-white/60` estão sozinhos. É o único par correto em todo o `src/`
(`rg -c "focus-visible:" src` devolve 1). Quando tocar no rail, complete o traço; o label é o
modelo, não a média.

### Sheet / modal

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| abrindo | backdrop `autoAlpha 0→1` linear; painel `autoAlpha 0→1` + `translateY(16→0)`; itens com stagger 60ms | 300ms quint | sim |
| aberto | `Escape` fecha, `Tab` circula dentro do sheet, Lenis parado via `stop()` | — | sim |
| fechando | `autoAlpha 1→0` + `translateY(0→8)`, itens saem `from: 'end'` | 200ms `power2.in` | sim |
| fechado | foco devolvido ao gatilho, `inert` no sheet, `visibility: hidden` | 0ms | sim |

Estado atual do `PageChrome`: o sheet usa `hidden={!menuOpen}`, o que remove o elemento do
render no mesmo frame em que a animação de saída deveria começar — a saída nunca é vista.
A correção é uma timeline pausada com `reverse()`, em
[patterns.md § 2](patterns.md#2-menu-com-stagger-com-saída-de-verdade).

### Toast

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| entrada | `translateY(12→0)` + `opacity 0→1` | 200ms quint | sim |
| permanência | 4000ms; 5500ms com ação; timer pausa em hover e em foco | — | sim |
| saída | `opacity → 0` + `translateY(0→6)` | 150ms | não |
| empilhado | máximo 3 visíveis; o quarto descarta o mais antigo | 200ms | não |
| a11y | `role="status"` (já implica `aria-live="polite"`); erro usa `role="alert"` | — | sim |

Toast nunca move o foco. `aria-live` anuncia sem tirar o teclado de onde ele está. O container
com `role="status"` tem de existir no DOM **antes** do texto aparecer: uma live region que monta
já preenchida não é anunciada por vários leitores de tela.

## Motion que informa vs motion que decora

| Situação | Feedback | Obrigatório | Sob reduced-motion vira |
|---|---|---|---|
| submit disparado | `aria-busy` + `disabled` em ≤100ms; o spinner só depois de 200ms | sim | troca instantânea de estado |
| espera acima de 1s | skeleton ou barra determinada | sim | skeleton estático, sem shimmer |
| erro de validação | cor + ícone + texto | sim | tudo aparece de uma vez |
| sucesso de envio | toast ou confirmação inline, ≥2s visível | sim | aparece sem deslocamento |
| item removido de lista | fade 150ms antes de colapsar o espaço | sim | some direto |
| foco por teclado | anel visível | sim | idêntico (já é instantâneo) |
| hover em elemento clicável | qualquer mudança perceptível | sim | mudança de cor, sem transform |
| desabilitado | opacidade + cursor | sim | idêntico |
| stagger de entrada | — | não | cai por completo |
| botão magnético | — | não | não roda |
| tilt de card | — | não | não roda |
| cursor customizado | — | não | não renderiza |
| clip-path de preenchimento | — | não | vira troca de cor |
| shimmer, marquee, aurora | — | não | `animation: none` |

Regra de corte: se o feedback é obrigatório, ele não pode depender de movimento. Movimento é a
camada de cima. Se a linha só existe na camada de cima, ela é opcional por definição.

## Focus-visible é requisito

Toda declaração de `:hover` tem uma linha irmã. Não há hover sem equivalente de teclado, exceto
os três efeitos que dependem da posição do ponteiro (magnético, tilt, cursor) — e para esses o
equivalente é o estado estático, não uma simulação do ponteiro.

```css
/* src/styles/index.css já define o :focus-visible global (rose-soft, offset 3px).
   Isto é o par hover/teclado no nível do componente. */
@layer components {
  .control {
    transition:
      background-color var(--dur-hover) var(--ease-out-quart),
      box-shadow var(--dur-hover) var(--ease-out-quart),
      transform var(--dur-hover) var(--ease-out-quart);
  }

  /* Em CSS escrito à mão, o hover precisa da media query. Sem ela, o estado
     gruda depois do tap em telas de toque. */
  @media (hover: hover) and (pointer: fine) {
    .control:hover {
      box-shadow: var(--shadow-lift);
      transform: translateY(-1px);
    }
  }

  .control:focus-visible {
    box-shadow: var(--shadow-lift);
    transform: translateY(-1px);
  }

  .control:active {
    transform: scale(0.98);
    transition-duration: var(--dur-press);
  }
}
```

Em Tailwind 4 o variant `hover:` já compila dentro de `@media (hover: hover)`; a media query
manual só é necessária em CSS próprio. O par continua obrigatório:

```
hover:shadow-lift  hover:-translate-y-px
focus-visible:shadow-lift  focus-visible:-translate-y-px
active:scale-[0.98]  active:duration-(--dur-press)
disabled:opacity-45 disabled:shadow-card disabled:translate-y-0 disabled:cursor-not-allowed
```

Verificação — os dois conjuntos devem ser iguais, tirando magnético/tilt/cursor. O `(?:^|[\s"'`])`
existe porque `rg -o "hover:"` casaria dentro de `group-hover:` e poluiria a lista; o `-r '$1'`
devolve só o utilitário, sem `sed` (que não existe no PowerShell):

```bash
rg -o "(?:^|[\s\"'\`])hover:([a-z0-9:_-]+(?:\[[^\]]*\])?)" -r '$1' src -I | sort -u
rg -o "(?:^|[\s\"'\`])focus-visible:([a-z0-9:_-]+(?:\[[^\]]*\])?)" -r '$1' src -I | sort -u
```

Repita trocando `hover:` por `group-hover:` e `focus-visible:` por `group-focus-visible:` — os
pares dentro de `group` quebram com a mesma frequência e não aparecem na primeira comparação.

## Botão magnético

O `src/hooks/useMagnetic.ts` de hoje tem dois defeitos: chama `getBoundingClientRect()` dentro
de `pointermove` — style + layout síncronos a cada evento, e um mouse de 1000 Hz emite vários
eventos por frame — e não limpa o `transform` no desmonte, então um botão que sai da tela
inclinado volta inclinado. A versão corrigida mede em `pointerenter`, coalesce em um
`requestAnimationFrame` e faz `clearProps` no cleanup: [patterns.md § 10](patterns.md#10-botão-magnético).

Ela troca a assinatura de `useMagnetic(strength)` para `useMagnetic({ strength, press })`.
`Action.tsx` chama sem argumento e não quebra; qualquer chamada com número posicional precisa
virar objeto na mesma edição.

## Carregamento

O spinner não substitui o label: se substituir, a caixa encolhe, o layout salta e o ponteiro
cai fora do alvo — o clique seguinte vai para o elemento de baixo. O label desce a `opacity .35`
e continua ocupando o espaço; o spinner entra sobreposto em `position: absolute`.

| Espera prevista | O que mostrar |
|---|---|
| < 200ms | nada — um spinner que pisca é pior que a espera |
| 200ms – 1s | spinner no próprio controle acionado, com 200ms de atraso antes de aparecer |
| 1s – 5s | skeleton com a forma e a altura do conteúdo final |
| > 5s | progresso determinado + texto do que está acontecendo |

Componente `Submit` completo em [patterns.md § 3](patterns.md#3-botão-em-carregamento); skeleton
e shimmer em [§ 4](patterns.md#4-skeleton).

## prefers-reduced-motion por componente

Deslocamento sai. Opacidade e cor ficam: nenhuma das duas move nada na tela, e são elas que
carregam a informação de mudança de estado.

| Componente | Vira instantâneo | Permanece |
|---|---|---|
| botão | `translateY`, `scale`, magnético, ícone que anda | cor de fundo, sombra, anel de foco |
| card | lift, tilt, `scale` da mídia | borda, sombra, outline |
| input | shake, subida do label | cor da borda, ícone e texto do erro |
| link | crescimento do sublinhado | sublinhado presente desde o rest |
| menu | stagger, `translateY`, `timeScale` | cross-fade de 150ms |
| modal | `translateY`, escala | cross-fade de 150ms do backdrop e do painel |
| toast | entrada e saída deslizando | fade de 150ms; permanência inalterada |
| skeleton | shimmer | bloco cinza estático |
| spinner | — | continua girando a 1200ms via `[data-motion-safe='spin']` |
| cursor | tudo | cursor nativo do sistema |

O bloco global em `src/styles/index.css` zera `transition-duration`, `animation-duration` e
`animation-iteration-count` com `!important` em `*`, `*::before` e `*::after`. É um piso seguro,
mas leva junto duas coisas que a tabela acima quer manter: os cross-fades de 150ms **e o
spinner** — com `animation-iteration-count: 1` e duração de 0.001ms ele para no primeiro frame,
e o único sinal de progresso indeterminado desaparece. Marque explicitamente quem sobrevive:

```css
@media (prefers-reduced-motion: reduce) {
  /* ...o bloco global de `*` que já existe, com os !important... */

  /* Opacidade e cor não deslocam nada, então podem continuar transicionando.
     Especificidade 0,1,0 vence o `*` do bloco global independente da ordem. */
  [data-motion-safe='fade'],
  [data-motion-safe='fade'] * {
    transition-property: opacity, background-color, border-color, color !important;
    transition-duration: 150ms !important;
  }

  /* O giro é o único jeito de dizer "ainda estou processando" sem barra de
     progresso. Mais lento para reduzir o estímulo, nunca parado. */
  [data-motion-safe='spin'] {
    animation-duration: 1200ms !important;
    animation-iteration-count: infinite !important;
  }
}
```

Em JS, `useReducedMotion()` (`src/hooks/useMediaQuery.ts`) já é reativo via
`useSyncExternalStore` — use-o em vez de ler `matchMedia` uma única vez, ou o visitante que
liga a preferência no meio da sessão continua vendo a animação. Em timelines GSAP, prefira
`gsap.matchMedia()`, que reverte sozinho quando a consulta deixa de bater.

## Anti-patterns

- **`transition: all`** — o navegador interpola toda propriedade que mudar, inclusive as que
  você nunca quis animar. Duas consequências: uma troca de classe que mexe em `width`,
  `padding` ou `grid-template` vira layout por frame; e propriedades que não interpolam
  (`backdrop-filter: none → blur(14px)`) travam a transição inteira. Foi exatamente isso que
  atrasou a barra do `PageChrome` até a lista de propriedades ser explicitada. Liste sempre —
  e o rail do mesmo arquivo ainda tem duas ocorrências vivas (`transition-all duration-300`,
  linhas 208 e 219): ali o que muda é `width`, ou seja, layout por frame.
- **`:hover` sem `@media (hover: hover)` em CSS próprio** — em tela de toque o estado gruda
  depois do tap e só sai quando outro elemento é tocado; o botão fica permanentemente aceso.
- **Spinner no lugar do label** — a caixa encolhe, o layout salta e o ponteiro sai do alvo.
- **Animar `height: auto`** — `auto` não é número; sem valor inicial e final numéricos não há
  interpolação e o painel salta. Use `grid-template-rows: 0fr → 1fr`.
- **`will-change` fixo no CSS** — cada elemento marcado ganha uma camada de composição
  permanente. Com dezenas de cards isso consome memória de GPU e o texto pode ser rasterizado
  antes da escala final (borrado no Safari). Aplique na entrada do hover, remova no
  `transitionend`. Acima de ~20 elementos simultâneos, não use.
- **Animar `box-shadow` em lista longa** — cada frame repinta o blur inteiro. Ponha a sombra em
  um `::after` e anime a `opacity` dele: opacidade é composta, sombra é repintada.
- **`back` / `elastic` em controle clicável** — o overshoot move o alvo depois que o usuário já
  mirou. Reserve para elementos que ninguém precisa acertar.
- **Menu com `opacity: 0` ainda no fluxo** — continua clicável e continua na ordem de
  tabulação; o teclado entra num menu invisível. Use `visibility: hidden` (o `autoAlpha` do
  GSAP faz isso), `inert` ou `hidden`.
- **`hidden` no mesmo frame do fechamento** — o elemento sai do render antes da transição
  rodar, então a saída nunca aparece. Guarde o estado `closing` e esconda no `transitionend`.
- **`getBoundingClientRect()` dentro de `pointermove`** — força style + layout síncronos por
  evento. Meça em `pointerenter`, invalide em `scroll` e `resize`.
- **Toast que move o foco** — o leitor de tela perde o ponto de leitura e o teclado é jogado
  para fora do formulário. `aria-live` anuncia sem roubar o foco.
- **Erro só por cor** — invisível para daltonismo vermelho-verde. Cor + ícone + texto.
- **Resposta de hover acima de 250ms** — o ponteiro costuma atravessar o elemento antes do fim;
  parece atraso, não suavidade. A regra vale para o que **confirma** o hover: cor, sombra,
  borda, transform do controle. Uma camada secundária que ninguém está esperando pode ser mais
  lenta — o zoom da mídia dentro do card a 600ms, o varrer do `clip-path` a 260ms — porque a
  confirmação já chegou e ela só continua acontecendo por trás.
- **Shake sem teto** — repetição rápida é gatilho vestibular. Máximo 3 ciclos, ≤6px, ≤240ms, e
  sempre acompanhando a mensagem de erro.
- **`cursor: none` global** — some o cursor do sistema e, com ele, o tamanho e o contraste que o
  usuário configurou na acessibilidade do SO. Restrinja a `[data-cursor]` sobre superfícies de
  mídia; campos de texto mantêm o I-beam.
- **Uma transição diferente por componente** — quando cada botão tem sua curva, o conjunto lê
  como colagem. Todas as durações saem dos tokens; nenhuma é escrita inline.

## Checklist

Rode antes de considerar a interface pronta.

- [ ] `rg -n "transition-all|transition: all" src` retorna vazio (hoje devolve `PageChrome.tsx`
      208 e 219 — é a linha de base a zerar)
- [ ] Os conjuntos `hover:` e `focus-visible:` batem, tirando magnético/tilt/cursor (hoje
      `rg -c "focus-visible:" src` devolve 1 contra 21 de `hover:`)
- [ ] `Tab` percorre botão, card, item de menu e modal com anel visível em todo estado que tem
      hover
- [ ] Submit entra em `aria-busy` em ≤100ms e a largura do botão não muda
- [ ] DevTools → Rendering → *Emulate prefers-reduced-motion*: nada desloca; cor, opacidade e
      anel de foco continuam; **e o spinner continua girando** — se parou, falta o
      `[data-motion-safe='spin']`
- [ ] Modo touch (ou aparelho real): nenhum estado de hover fica preso depois do tap
- [ ] Menu fechado não recebe `Tab` — confira `visibility`/`inert`, não só `opacity`
- [ ] Menu e modal animam a **saída**, não só a entrada
- [ ] Performance panel com 4× de throttle: hover e press acima de 50fps, sem faixa roxa
      (layout) na trilha
- [ ] DevTools → Rendering → *Emulate vision deficiencies* → Achromatopsia: os erros de
      formulário continuam legíveis
- [ ] `rg -n "will-change" src/styles src/components` — cada ocorrência é justificada e
      contável
- [ ] Toast anuncia sem mover o foco; o timer pausa em hover e em foco
