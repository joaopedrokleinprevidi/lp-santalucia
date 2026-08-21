---
name: "motion-ui-expert"
description: "Use when building or refining component-level motion — button, card, input, link, menu, modal, toast, skeleton, cursor — and the states around them. Fase 10 — animacao de botao, card, formulario, menu mobile, modal, toast, skeleton, cursor; estado de hover, clique, foco, desabilitado, carregando, erro e sucesso; micro-interacao, feedback de formulario, spinner, botao magnetico, focus-visible. Nao serve para animacao dirigida por scroll."
argument-hint: "[componente] [estado]"
---

# Motion de componente — Fase 10

| | |
|---|---|
| **ENTRADA** | A lista de `small` por capítulo vinda de `design/creative-direction.json`; os tokens de duração e ease de `landing-motion-expert`; os componentes em `src/components/{ui,layout}/` |
| **SAÍDA** | Componentes com os estados implementados, os tokens de `:root` e `@theme` criados, e cada par `hover:` / `focus-visible:` fechado |
| **ANTES** | quem chama é `landing-motion-expert` (Fase 10c), depois de fixar os tokens, sempre que o alvo é botão, card, input, link, menu, modal, toast, skeleton ou cursor. Fora do trilho linear: sem uma rota da 10c, esta skill não tem fase própria |
| **DEPOIS** | `audit-responsivo` (Fase 11a — alvo de toque de 44px) e `audit-acessibilidade` (Fase 11b — `focus-visible` e estado sem cor) |

Scroll move a página. Isto move o componente. Tudo que responde a ponteiro, teclado, toque ou
mudança de estado interno pertence a esta skill; tudo que responde à posição do scroll não.

| A animação dispara por | Dono |
|---|---|
| hover, press, focus, disabled, loading, erro, sucesso | esta skill |
| abrir/fechar de menu, modal, toast, accordion, tab | esta skill |
| entrada de uma seção na viewport | `gsap-scrolltrigger-expert` |
| ordem e ritmo entre seções | `landing-motion-expert` |
| frame de vídeo preso ao scroll | `video-decisao` |

O CLAUDE.md já define o Motion ROI. Aqui a pergunta vira tabela: cada estado ou tem feedback
obrigatório, ou é decoração e pode ser cortado sem perda de informação. Código dos padrões —
tilt, menu, carregamento, skeleton, toast, clip-path, sublinhado, accordion, cursor, magnético,
tokens, par hover/foco e reduced motion — em [patterns.md](patterns.md).

## Números canônicos

As curvas são de `landing-motion-expert`, dona da linguagem de motion do projeto; aqui só se
escolhe qual delas cabe em cada interação. Deslocamento de controle usa `ease-out-quart` /
`power3.out`; painel que abre e fecha usa `ease-out-quint` / `power4.out`; `ease-out-expo` fica
com o reveal de copy, que é da skill de scroll.

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

Abaixo de ~100ms a transição termina antes de a atenção chegar: o olho lê uma troca instantânea e
a informação de "o que virou o quê" se perde — por isso o press pode ser curto (o dedo já sabe o
que fez) mas o hover não. Acima de ~400ms a espera vira lentidão. A exceção é deliberada: a
entrada de um modal chega a 350ms porque é troca de contexto e o olho precisa reancorar; nada em
hover pode. Saída sempre mais rápida que entrada — a entrada precisa ser notada, a saída precisa
sair do caminho.

**Stagger** vem da tabela por contagem de itens de `landing-motion-expert` (2–3 itens →
0,10–0,14s; 4–8 → 0,06–0,08s; 9–20 → 0,03–0,04s), com o teto de `stagger × contagem ≤ 0,9s`.
**Tokens**: `--ease-out-quart` e `--color-danger` no `@theme`; as durações `--dur-*` no `:root`,
porque Tailwind 4 não tem namespace de `transition-duration` e o `@theme` descartaria a variável.
Blocos prontos em [patterns.md § 11](patterns.md#11-tokens-do-projeto).

## Tabela de estados

Cada linha é uma mudança visual observável. "Obrigatório" significa: sem isso o usuário não sabe
o que aconteceu, e a coluna sobrevive a `prefers-reduced-motion` como troca instantânea.

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

O ícone que anda 4px no eixo x (`group-hover:translate-x-1`, 300ms) é camada de cima, não
obrigatória. `Action.tsx` já faz isso com `group-hover:` sozinho: falta o
`group-focus-visible:translate-x-1` que dá o mesmo sinal a quem chega pelo teclado. E cuidado com
quem escreve `transform`: se o controle usa `useMagnetic`, o GSAP grava a propriedade inline e o
`active:scale-[0.98]` deixa de ter efeito — nesses controles o press vem da opção `press` do
hook, não da classe.

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
nunca um `onClick` na `<div>` — a div não recebe foco, não abre em nova aba e não aparece no menu
de contexto.

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
para cerca de 8% dos homens, e a borda sozinha não diz o que corrigir. O texto só chega ao leitor
de tela amarrado por `aria-invalid="true"` no campo e `aria-describedby` apontando para o `id` da
mensagem — sem isso o campo é anunciado como "inválido" e nada mais.

### Link inline

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| rest | `background-size: 0% 1px`, posição `100% 100%` | — | — |
| hover | `background-size: 100% 1px`, posição `0 100%` | 220ms quart | sim |
| hover out | volta a `0%` recolhendo pela direita | 220ms | não |
| focus-visible | sublinhado a 100% **imediato** + outline | 0ms | sim |
| visitado | mudança de cor, sem transição | 0ms | não |

O sublinhado que cresce da esquerda e recolhe pela direita anima uma propriedade só
(`background-size`); a posição troca sem transição. [patterns.md § 7](patterns.md#7-sublinhado-que-cresce).

### Item de menu / rail (`src/components/layout/PageChrome.tsx`)

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| rest | traço 16px, label `opacity 0` | — | — |
| hover | traço 24px, label `opacity .7` | 300ms | sim |
| focus-visible | idêntico ao hover + outline | 0ms outline | sim |
| atual | traço 32px em `--color-gilt`, label `opacity 1`, `aria-current` | 300ms | sim |

Estado atual do rail: o **label** emparelha `group-hover:opacity-70` com
`group-focus-visible:opacity-70`, mas o **traço** não — `group-hover:w-6` e os dois
`group-hover:bg-*` estão sozinhos. É o único par correto em todo o `src/`
(`rg -c "focus-visible:" src` devolve 1); o label é o modelo, não a média.

### Sheet / modal

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| abrindo | backdrop `autoAlpha 0→1` linear; painel `autoAlpha 0→1` + `translateY(16→0)`; itens com stagger 60ms | 300ms quint | sim |
| aberto | `Escape` fecha, `Tab` circula dentro do sheet, Lenis parado via `stop()` | — | sim |
| fechando | `autoAlpha 1→0` + `translateY(0→8)`, itens saem `from: 'end'` | 200ms `power2.in` | sim |
| fechado | foco devolvido ao gatilho, `inert` no sheet, `visibility: hidden` | 0ms | sim |

Estado atual do `PageChrome`: o sheet usa `hidden={!menuOpen}`, que remove o elemento do render
no mesmo frame em que a saída deveria começar — a saída nunca é vista. A correção é uma timeline
pausada com `reverse()`: [patterns.md § 2](patterns.md#2-menu-com-stagger-com-saída-de-verdade).

### Toast

| Estado | O que muda | Duração / ease | Obrigatório |
|---|---|---|---|
| entrada | `translateY(12→0)` + `opacity 0→1` | 200ms quint | sim |
| permanência | 4000ms; 5500ms com ação; timer pausa em hover e em foco | — | sim |
| saída | `opacity → 0` + `translateY(0→6)` | 150ms | não |
| empilhado | máximo 3 visíveis; o quarto descarta o mais antigo | 200ms | não |
| a11y | `role="status"` (já implica `aria-live="polite"`); erro usa `role="alert"` | — | sim |

Toast nunca move o foco, e o container com `role="status"` tem de existir no DOM **antes** do
texto aparecer: live region que monta já preenchida não é anunciada por vários leitores de tela.

## Motion que informa vs motion que decora

| Situação | Feedback | Obrigatório | Sob reduced-motion vira |
|---|---|---|---|
| submit disparado | `aria-busy` + `disabled` em ≤100ms; spinner só depois de 200ms | sim | troca instantânea de estado |
| espera acima de 1s | skeleton ou barra determinada | sim | skeleton estático, sem shimmer |
| erro de validação | cor + ícone + texto | sim | tudo aparece de uma vez |
| sucesso de envio | toast ou confirmação inline, ≥2s visível | sim | aparece sem deslocamento |
| item removido de lista | fade 150ms antes de colapsar o espaço | sim | some direto |
| foco por teclado | anel visível | sim | idêntico (já é instantâneo) |
| hover em elemento clicável | qualquer mudança perceptível | sim | mudança de cor, sem transform |
| desabilitado | opacidade + cursor | sim | idêntico |
| stagger de entrada; magnético, tilt e cursor; clip-path; shimmer, marquee, aurora | — | não | não roda, e nada entra no lugar |

Regra de corte: se o feedback é obrigatório, ele não pode depender de movimento. Movimento é a
camada de cima; o que só existe nela é opcional por definição.

## Focus-visible é requisito

Toda declaração de `:hover` tem uma linha irmã. Não há hover sem equivalente de teclado, exceto
os três efeitos que dependem da posição do ponteiro (magnético, tilt, cursor) — e para esses o
equivalente é o estado estático, não uma simulação do ponteiro. O par em CSS próprio e em
utilitário: [patterns.md § 12](patterns.md#12-par-hover--focus-visible-em-css-próprio).

Os dois conjuntos de utilitários devem ser iguais, tirando magnético/tilt/cursor. Os dois `rg`
que os extraem, e a repetição obrigatória para `group-hover:` / `group-focus-visible:`, estão em
[patterns.md § 12](patterns.md#12-par-hover--focus-visible-em-css-próprio).

`src/hooks/useMagnetic.ts` é o caso que quebra esse par por outro motivo: ele tem dois defeitos
de performance e de cleanup, e a versão corrigida troca a assinatura para
`useMagnetic({ strength, press })` — diagnóstico e código em
[patterns.md § 10](patterns.md#10-botão-magnético).

## Carregamento

O spinner não substitui o label: a caixa encolhe, o layout salta e o clique seguinte vai para o
elemento de baixo. O label desce a `opacity .35` e continua ocupando o espaço; o spinner entra
sobreposto em `position: absolute`. Componente `Submit` em
[patterns.md § 3](patterns.md#3-botão-em-carregamento); skeleton em [§ 4](patterns.md#4-skeleton).

| Espera prevista | O que mostrar |
|---|---|
| < 200ms | nada — um spinner que pisca é pior que a espera |
| 200ms – 1s | spinner no próprio controle acionado, com 200ms de atraso antes de aparecer |
| 1s – 5s | skeleton com a forma e a altura do conteúdo final |
| > 5s | progresso determinado + texto do que está acontecendo |

## prefers-reduced-motion por componente

Deslocamento sai. Opacidade e cor ficam — não movem nada na tela e são elas que carregam a
informação de mudança de estado.

O bloco global de `*` com `!important` derruba junto os cross-fades e o spinner. Quem sobrevive
precisa ser marcado explicitamente, e `useReducedMotion()` é reativo. Os dois blocos, e a tabela
do que vira instantâneo e do que permanece em cada um dos dez componentes, em
[patterns.md § 13](patterns.md#13-prefers-reduced-motion--o-que-sobrevive).

## Anti-patterns

- **`transition: all`** — interpola toda propriedade que mudar. Uma troca de classe que mexe em
  `width`, `padding` ou `grid-template` vira layout por frame, e propriedades que não interpolam
  (`backdrop-filter: none → blur(14px)`) travam a transição inteira. Foi isso que atrasou a barra
  do `PageChrome`; o rail do mesmo arquivo ainda tem duas ocorrências vivas (linhas 208 e 219),
  onde o que muda é `width`.
- **`:hover` sem `@media (hover: hover)` em CSS próprio** — em tela de toque o estado gruda depois
  do tap e só sai quando outro elemento é tocado.
- **Animar `height: auto`** — `auto` não é número; sem valor inicial e final numéricos não há
  interpolação e o painel salta. Use `grid-template-rows: 0fr → 1fr`.
- **`will-change` fixo no CSS** — cada elemento marcado ganha camada de composição permanente. Com
  dezenas de cards isso consome memória de GPU e o texto pode ser rasterizado antes da escala
  final. Aplique na entrada do hover, remova no `transitionend`; acima de ~20 simultâneos, não use.
- **Animar `box-shadow` em lista longa** — cada frame repinta o blur inteiro. Ponha a sombra num
  `::after` e anime a `opacity` dele: opacidade é composta, sombra é repintada.
- **`back` / `elastic` em controle clicável** — o overshoot move o alvo depois que o usuário mirou.
- **Menu com `opacity: 0` ainda no fluxo** — continua clicável e na ordem de tabulação; o teclado
  entra num menu invisível. Use `visibility: hidden` (o `autoAlpha` do GSAP faz isso) ou `inert`.
- **`hidden` no mesmo frame do fechamento** — o elemento sai do render antes de a transição rodar,
  e a saída nunca aparece. Guarde o estado `closing` e esconda no `transitionend`.
- **`getBoundingClientRect()` dentro de `pointermove`** — força style + layout síncronos por
  evento. Meça em `pointerenter`, invalide em `scroll` e `resize`.
- **Toast que move o foco** — o leitor de tela perde o ponto de leitura e o teclado é jogado para
  fora do formulário. `aria-live` anuncia sem roubar o foco.
- **Erro só por cor** — invisível para daltonismo vermelho-verde. Cor + ícone + texto.
- **Resposta de hover acima de 250ms** — o ponteiro atravessa o elemento antes do fim; parece
  atraso, não suavidade. Vale para o que **confirma** o hover: cor, sombra, borda, transform do
  controle. Camada secundária pode ser mais lenta — o zoom da mídia a 600ms, o `clip-path` a
  260ms — porque a confirmação já chegou e ela só continua acontecendo por trás.
- **Shake sem teto** — repetição rápida é gatilho vestibular. Máximo 3 ciclos, ≤6px, ≤240ms, e
  sempre acompanhando a mensagem de erro.
- **`cursor: none` global** — some o cursor do sistema e, com ele, o tamanho e o contraste que o
  usuário configurou na acessibilidade do SO. Restrinja a `[data-cursor]` sobre mídia.
- **Uma transição diferente por componente** — quando cada botão tem sua curva, o conjunto lê como
  colagem. Todas as durações saem dos tokens; nenhuma é escrita inline.

## Checklist

- [ ] `rg -n "transition-all|transition: all" src` retorna vazio (hoje devolve `PageChrome.tsx`
      208 e 219 — é a linha de base a zerar)
- [ ] Os conjuntos `hover:` e `focus-visible:` batem, tirando magnético/tilt/cursor (hoje
      `rg -c "focus-visible:" src` devolve 1 contra 21 de `hover:`)
- [ ] `rg -n "will-change" src/styles src/components` — cada ocorrência justificada e contável
- [ ] `Tab` percorre botão, card, item de menu e modal com anel visível em todo estado que tem hover
- [ ] Submit entra em `aria-busy` em ≤100ms e a largura do botão não muda
- [ ] DevTools → Rendering → *Emulate prefers-reduced-motion*: nada desloca; cor, opacidade e anel
      de foco continuam; **e o spinner continua girando** — se parou, falta `[data-motion-safe='spin']`
- [ ] Modo touch (ou aparelho real): nenhum estado de hover fica preso depois do tap
- [ ] Menu fechado não recebe `Tab` (confira `visibility`/`inert`, não só `opacity`), e menu e
      modal animam a **saída**, não só a entrada
- [ ] Performance panel com 4× de throttle: hover e press acima de 50fps, sem faixa roxa (layout)
- [ ] Rendering → *Emulate vision deficiencies* → Achromatopsia: os erros continuam legíveis
- [ ] Toast anuncia sem mover o foco; o timer pausa em hover e em foco
