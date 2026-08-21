# Números do projeto — tipografia, espaçamento e densidade de motion

Valores lidos de `src/styles/index.css`, `src/components/chapters/*.tsx`, `vite.config.ts` e
`scripts/prepare-assets.mjs`. São a base contra a qual os passos A2, A2b, A5 e A7 do
[SKILL.md](SKILL.md#protocolo-de-auditoria) medem. Se um `clamp()` mudar no diff, recalcule aqui
antes de auditar — um número errado nesta página vira uma aprovação errada no laudo.

## Tipografia por largura

Valores resolvidos dos `clamp()` do `@theme`, em CSS px:

| Token | declaração | @360 | @768 | @1440 | teto |
|---|---|---|---|---|---|
| `--text-hero` | `clamp(2.125rem, 4.5vw, 4.5rem)` | 34px | 34.6px | 64.8px | 72px a partir de 1600 |
| `--text-chapter` | `clamp(1.875rem, 3.5vw, 3.25rem)` | 30px | 30px | 50.4px | 52px a partir de 1486 |
| `--text-statement` | `clamp(1.625rem, 3.4vw, 3.25rem)` | 26px | 26.1px | 49px | 52px a partir de 1530 |
| `--text-lead` | `clamp(1rem, 1.15vw, 1.25rem)` | 16px | 16px | 16.6px | 20px a partir de 1739 |
| `--text-eyebrow` | `0.6875rem` | 11px | 11px | 11px | fixo |

O piso de 16px em `--text-lead` é de propósito: abaixo disso o Safari do iOS dá zoom ao focar um
campo, e o zoom desloca o layout inteiro. `--text-eyebrow` a 11px é rótulo, não texto — caixa
alta, `0.24em` de tracking — e **nunca carrega informação que não esteja repetida na copy**. Um
eyebrow com preço, endereço ou horário é BLOQUEIO em A2: 11px reprova como texto de leitura, e a
informação não existe em nenhum outro lugar da página.

## Espaçamento

`.container-editorial`: `max-width: 1280px`, `padding-inline: clamp(24px, 5vw, 64px)` — 24px a
360, 38.4px a 768, 64px de 1280 em diante.

24px é piso para texto **e** para alvos de toque. Um botão a 8px da borda de uma tela curva é
intocável com o polegar: a curvatura rejeita o toque antes de o navegador receber o evento. É a
mesma constante que aparece no A2b como distância mínima da borda.

## Alvos de toque que já existem no repo

| Componente | Medida | Situação |
|---|---|---|
| `Action` | `min-h-[56px]` | passa com folga |
| Hambúrguer e CTA da barra (`PageChrome`) | `size-11` / `min-h-11` = 44px | no piso exato |
| CTA do sheet (`PageChrome`) | `min-h-14` = 56px | passa |
| Trilho de capítulos (`PageChrome`) | `py-1`, abaixo de 44 | aceitável só porque existe apenas em `xl` com ponteiro fino, onde vale o piso 24 — abaixo de 1280 ele é `display: none` e o script do A2b nem o alcança |

## Densidade de motion

| | <768 | ≥768 |
|---|---|---|
| Orçamento de scroll do capítulo | `scrollMobile × 100svh`, hoje 1.4–4.2 | `scroll × 100svh`, hoje 2–6 |
| Beats simultâneos na stage | 1 | até 2 |
| Amplitude de translate/parallax | ≤ 40px | ≤ 120px |
| Vídeo | 1280×720 crf 27 (~1,0–1,3 MB) | 1920×1080 crf 25 (~3,2–4,3 MB) |
| Hover / magnético | não existe | só sob `(hover: hover) and (pointer: fine)` |

`.chapter` mede `(scroll + 1) × 100svh`: uma tela para a stage sticky mais o orçamento de rolagem.
Os sete capítulos hoje, e a razão mobile/desktop de cada um:

| Capítulo | `scroll` | `scrollMobile` | razão |
|---|---|---|---|
| `ChapterFinale` | 2 | 1.4 | 0,70 |
| `ChapterClinic` | 3.2 | 2.4 | 0,75 |
| `ChapterExperience` | 3.2 | 2.2 | 0,69 |
| `ChapterConsultation` | 4 | 2.8 | 0,70 |
| `ChapterHero` | 4.6 | 3.2 | 0,70 |
| `ChapterCare` | 4.8 | 3.4 | 0,71 |
| `ChapterJourney` | 6 | 4.2 | 0,70 |

A faixa observada é 0,69–0,75, dentro da faixa canônica **0,68–0,75** de
[landing-storytelling-director](../landing-storytelling-director/SKILL.md#pacing) — que é a dona
do número; esta página só o repete resolvido, para conferência. Fora dela o capítulo destoa: o
mesmo trecho de história custa três vezes mais dedo no celular do que roda de scroll no desktop.

## Adaptação de motion por breakpoint

*Como* adaptar pertence a `gsap-scrolltrigger-expert` (sticky em vez de pin, `gsap.matchMedia`,
`100svh` para texto e `100lvh` para mídia) e a `landing-motion-expert` (orçamento por seção).
Esta tabela é só o estado final que a auditoria confere, com o passo que o pega.

| Efeito no desktop | Abaixo de 768 precisa estar | Pego em | Porque |
|---|---|---|---|
| Stage sticky com 4 beats simultâneos | mesma stage, beats em série — um sai, o outro entra | A2 | 640px de altura não comportam título mais grade de quatro; eles se sobrepõem. É o `if (isMobile)` de `ChapterExperience` |
| Pin via `ScrollTrigger.pin` | `position: sticky` puro em CSS | A2 | o pin-spacer é medido em JS, e a barra de URL do Android muda a altura no meio da rolagem: o pin salta. Sticky não mede nada |
| Parallax ±120px | ≤ ±40px, ou removido | A2 | em 640px de altura, 120px de deslocamento tiram o elemento do enquadramento antes de ele ser lido |
| Scrub de N telas | 0,68 N a 0,75 N | A5 | o mesmo capítulo custa três vezes mais dedo no celular |
| Trilho horizontal preso ao scroll y | pilha vertical, ou `scroll-snap-type: x mandatory` nativo | A4 | no touch, arrastar para baixo para mover algo para o lado é um gesto brigando com o outro; e o card fora da tela nunca recebe foco |
| Cursor magnético (`useMagnetic`) | ausente, por `(pointer: fine)` | A2b | um tablet de 1024px tem largura de desktop e nenhum ponteiro |
| Canvas de 1920 com 300 frames | set de 1280, ou só o poster | A7 | 300 frames em largura total matam a aba no iOS |
| Cross-fade de 6 imagens | 3 | A7 | cada imagem extra é um download pago para ser visto por 400ms |
| `100vh` | `100svh` para texto, `100lvh` para mídia | A2 | com `100vh` a barra de URL cobre o CTA; com `100lvh` no texto, o mesmo. O projeto já separa os dois em `.chapter__stage` e `.chapter__media` |
