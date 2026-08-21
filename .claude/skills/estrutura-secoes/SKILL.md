---
name: estrutura-secoes
description: Use when deciding a landing page section order, narrative archetype, emotional pacing and scroll budget per section. Fase 6a — escolhe o arquetipo (urgencia, transformacao, credencial, produto), ordena as secoes pela queda das objecoes, define o beat de cada uma e converte share em scroll/scrollMobile. Palavras-chave: estrutura da landing, ordem das secoes, storytelling, pacing, quanto scroll cada secao recebe, story map.
argument-hint: [tipo-de-negocio] [o-que-o-visitante-precisa-decidir]
---

# Estrutura de seções — Fase 6a

| | |
|---|---|
| **ENTRADA** | `design/briefing.json`, `design/design-system.json`, `design/pesquisa.md` (objeções em ordem de custo) e o bloco de orçamento da Fase 5: `depth` total e `experienceScore` |
| **SAÍDA** | `design/estrutura.md` — tabela do story map — e `src/data/story.ts`, o mesmo mapa como código |
| **ANTES** | `creative-direction-expert` (Fase 5) fixa `depth` total e Experience Score |
| **DEPOIS** | `copy-conversao` (Fase 6b) escreve a copy dentro desta estrutura |

Esta skill decide **a ordem em que as objeções caem** e **quanto scroll cada seção recebe**. Não
escreve uma linha de copy — isso é `copy-conversao`. Não decide como a seção parece
(`product-design-expert`) nem como ela se move (`landing-motion-expert`).

É a dona de dois números: o `share` de cada seção e a razão `scrollMobile / scroll`. Nenhuma
outra skill os escreve.

## Passo 0 — a pergunta que escolhe o arquétipo

> Qual é a coisa mais cara que o visitante tem medo de errar?

| Medo dominante | Arquétipo |
|---|---|
| Errar o **tempo** — não chegar, não conseguir atendimento agora | Urgência |
| Errar o **resultado no próprio corpo** — ficar artificial, se arrepender | Transformação |
| Errar **quem cuida** — cair num amador, perder dinheiro e tempo | Credencial |
| Errar a **ferramenta** — comprar algo que dá trabalho e não resolve | Produto |

Escolha **um**. Um arquétipo é uma promessa sobre a ordem em que as objeções são derrubadas;
misturar dois derruba nenhuma na hora certa.

| | Urgência | Transformação | Credencial | Produto |
|---|---|---|---|---|
| Tempo até a decisão | < 10 min | dias a semanas | dias | horas a dias |
| Prova que fecha | disponibilidade agora | antes/depois + processo | nome + registro | número de uso |
| Onde o pico cai | seção 03 | seção 05 | seção 02 | seção 03 |
| Exemplo típico | clínica veterinária 24h | Beleza Completa Barreiro | dentista, advogado | SaaS B2B |

Quando o projeto paga o custo, gere **três arquétipos concorrentes** e julgue cada um por
conversão, qualidade do pt-BR e experiência, depois sintetize a vencedora enxertando o melhor
das outras. Estrutura escolhida às cegas é a origem da maioria das landings medianas.

## Beat emocional

Beat aqui é a **temperatura emocional** da seção, de 0 a 10 — e o valor absoluto importa menos
que a diferença para a vizinha. Não confunda com o "beat" de `creative-direction-expert` e
`landing-motion-expert`, que é *uma coisa revelada dentro do capítulo*: unidade de motion, não de
emoção. No handoff diga sempre qual — **beat emocional** sai daqui, **beat revelado** sai de lá.

| Faixa | Estado do visitante |
|---|---|
| 0–2 | Lendo informação operacional: endereço, horário, forma de pagamento |
| 3–4 | Calmo, entendendo como funciona |
| 5–6 | Engajado, se projetando no resultado |
| 7–8 | Desejo ou tensão alta: o problema nomeado, o antes/depois, o depoimento |
| 9–10 | Ação iminente — só existe em Urgência |

Uma página tem **exatamente um** beat ≥ 8. Dois picos e nenhum é pico; nenhum e a página é uma
sequência de informações.

## Os quatro arquétipos

`Share` é a fatia do scroll total da página. A decisão de quais dessas seções recebem CTA é de
`copy-conversao`, não daqui. Detalhamento por seção — o que a seção precisa entregar, o que
quebra quando ela é montada errada, variação mobile — em [archetypes.md](archetypes.md).

### Urgência — serviço 24h, emergência, plantão

| # | Seção | Pergunta que responde | Beat | Share | Prova |
|---|---|---|---|---|---|
| 01 | Estado agora | "Vocês estão abertos neste momento?" | 7 | 12% | disponibilidade |
| 02 | Como chegar | "Onde é e quanto tempo eu levo?" | 6 | 8% | mapa, referência |
| 03 | Quando é emergência | "O que eu estou vendo é grave?" | **9** | 12% | lista de sinais |
| 04 | O que acontece ao chegar | "Vou ser atendido ou vou esperar?" | 5 | 16% | protocolo minuto a minuto |
| 05 | Estrutura | "Vocês têm o que o caso precisa?" | 4 | 12% | equipamento nomeado |
| 06 | Quem atende | "Tem profissional de verdade de plantão?" | 5 | 10% | nome + registro |
| 07 | Depoimentos de emergência | "Já resolveram um caso como o meu?" | 7 | 10% | testemunho com desfecho |
| 08 | FAQ | "Quanto custa? Preciso agendar?" | 3 | 10% | preço/faixa |
| 09 | Contato completo | "Como falo agora?" | 7 | 10% | endereço, telefone, mapa |

A 03 é o único pico: hero e fecho ficam em 7 justamente para que ela seja 9. A 04 cai quatro
pontos de propósito — tensão sem alívio fecha a aba.

### Transformação — estética, resultado no corpo

| # | Seção | Pergunta que responde | Beat | Share | Prova |
|---|---|---|---|---|---|
| 01 | Promessa | "Isto é para mim?" | 5 | 16% | — |
| 02 | A experiência | "Como é estar nesse lugar?" | 6 | 12% | ambiente, atendimento |
| 03 | O processo em etapas | "O que exatamente vai acontecer comigo?" | 5 | 20% | processo nomeado |
| 04 | A consulta | "Vão me ouvir ou me empurrar um pacote?" | 6 | 14% | consulta individual |
| 05 | O princípio | "Vou ficar artificial?" | **8** | 16% | antes/depois, resultado |
| 06 | A clínica | "É um lugar sério?" | 4 | 12% | fotos reais do espaço |
| 07 | Convite | "E agora?" | 7 | 10% | contato completo |

O FAQ nesta estrutura vive **dentro** da seção 06, não como capítulo próprio: um capítulo de FAQ
entre a prova e o convite esfria o arco exatamente onde ele deveria fechar.

### Credencial — saúde, jurídico, ticket alto

| # | Seção | Pergunta que responde | Beat | Share | Prova |
|---|---|---|---|---|---|
| 01 | Afirmação com número | "Do que se trata e quem faz?" | 5 | 14% | número duro na headline |
| 02 | O risco de errar | "O que acontece se eu escolher mal?" | **8** | 12% | consequência concreta |
| 03 | O método | "Como vocês trabalham?" | 5 | 18% | etapas nomeadas |
| 04 | Quem é a pessoa | "Quem vai me atender?" | 6 | 14% | rosto, nome, registro |
| 05 | Casos | "Já resolveram como o meu?" | 7 | 12% | caso com desfecho |
| 06 | FAQ | "Quanto custa, quanto demora, é sigiloso?" | 3 | 12% | honorários |
| 07 | Contato | "Como começo?" | 6 | 10% | canal + horário |
| — | Rodapé com registro | — | 1 | 8% | registro, endereço, CNPJ |

### Produto — SaaS, self-serve

| # | Seção | Pergunta que responde | Beat | Share | Prova |
|---|---|---|---|---|---|
| 01 | O que é, em uma frase | "O que isso faz?" | 5 | 14% | — |
| 02 | Quem já usa | "Alguém sério confia nisso?" | 4 | 6% | logos, número de contas |
| 03 | O custo do problema | "Quanto isso está me custando hoje?" | **8** | 12% | número do desperdício |
| 04 | Como funciona em 3 passos | "Vai dar trabalho?" | 5 | 16% | tempo de setup |
| 05 | Features por resultado | "Resolve o meu caso específico?" | 4 | 18% | feature → consequência |
| 06 | Comparação | "Por que não a alternativa X?" | 6 | 10% | tabela honesta |
| 07 | Depoimento com métrica | "Quanto mudou para eles?" | 7 | 8% | número no depoimento |
| 08 | Preço + FAQ | "Quanto custa e como saio?" | 3 | 10% | tabela de preço |
| 09 | CTA final | — | 6 | 6% | — |

Único arquétipo em que prova social vem na seção 02 — ver anti-patterns.

## Pacing

Tetos e pisos por papel de seção, independentes do arquétipo:

| Papel | Share mínimo | Share máximo | Beat típico |
|---|---|---|---|
| Hero | 12% | 18% | 5–7 |
| Tensão / problema | 8% | 13% | 7–9 |
| Método / processo | 14% | 22% | 4–6 |
| Prova argumentada (antes/depois, caso, depoimento) | 8% | 17% | 6–8 |
| Confirmação social (logos, contagem) — só em Produto | 6% | 8% | 3–5 |
| Credencial / equipe | 8% | 14% | 4–6 |
| Ambiente / estrutura | 6% | 12% | 3–6 |
| FAQ | 8% | 12% | 2–4 |
| Convite final | 6% | 12% | 6–8 |

Regras que se verificam:

- **Nenhuma seção abaixo de 6%.** Abaixo disso ela passa antes de ser lida e o visitante registra
  "havia algo ali" sem saber o quê.
- **Nenhuma seção acima de 22%.** Acima disso, num capítulo pinado, o visitante rola e a tela não
  muda o suficiente — ele conclui que a página travou e sai.
- **A seção mais longa é sempre a que reduz mais medo** (método/processo), nunca o hero. O hero
  cria a pergunta; ele não precisa de scroll para isso.
- **Vizinhas nunca têm o mesmo beat.** Entre o pico e cada uma das vizinhas a diferença é **≥ 2**:
  um pico anunciado com 1 ponto de diferença não é lido como pico. Fora do pico, 1 ponto basta.
- **Depois do beat ≥ 8, a próxima cai ≥ 3.**

## Convertendo share em `scroll`

`<Chapter scroll>` é medido em alturas de viewport **além** da primeira tela, então a altura total
de um capítulo é `1 + scroll`.

```ts
/**
 * Converte a fatia desejada do scroll total no valor da prop `scroll` de <Chapter>.
 * `total` = soma de (1 + scroll) de todos os capítulos da página.
 */
export const scrollProp = (share: number, total: number): number =>
  Number((share * total - 1).toFixed(1))

// scrollProp(0.20, 34.8) === 6.0 — o capítulo da jornada no projeto de referência.
```

**`scrollMobile` fica entre 0,68 e 0,75 × `scroll`**, capítulo a capítulo. É a faixa canônica do
projeto — as outras skills a referenciam em vez de repeti-la — e na referência medida a razão vai
de 0,69 a 0,75; fora dela o capítulo conta outra história no celular. Compare razão de prop com
razão de prop: a das **alturas totais** (`1 + scroll`) é ~76% (26,6 vs 34,8 viewports), porque o
`1` do palco sticky não encolhe.

`creative-direction-expert` orça capítulo pinado entre `scroll={2}` e `scroll={6}`, e é isso que
fecha o piso e o teto de `share` de verdade: com `total` = 34,8 o piso de 6% devolveria 1,1 e o
teto de 22% devolveria 6,7, os dois fora da faixa dele. Os limites efetivos nessa página são
**8,6% e 20,1%**. Caiu fora? Funda o capítulo ou divida-o; nunca estique `scroll` além de 6. A
faixa 2–6 vale só para capítulo pinado — seção em fluxo normal (FAQ, contato, rodapé) ocupa a
altura do próprio conteúdo, e nela vale o piso de 6% e mais nada.

### Referência medida — Beleza Completa (Transformação, 7 capítulos)

| Capítulo | `scroll` | Altura | Share desktop | `scrollMobile` | Share mobile |
|---|---|---|---|---|---|
| 01 Início | 4.6 | 5.6 | 16.1% | 3.2 | 15.8% |
| 02 A experiência | 3.2 | 4.2 | 12.1% | 2.2 | 12.0% |
| 03 Sua jornada | 6.0 | 7.0 | **20.1%** | 4.2 | 19.5% |
| 04 A consulta | 4.0 | 5.0 | 14.4% | 2.8 | 14.3% |
| 05 O cuidado | 4.8 | 5.8 | 16.7% | 3.4 | 16.5% |
| 06 A clínica | 3.2 | 4.2 | 12.1% | 2.4 | 12.8% |
| 07 Agendar | 2.0 | 3.0 | 8.6% | 1.4 | 9.0% |
| | | **34.8** | 100% | | 100% |

## O artefato

```ts
// src/data/story.ts — escrito antes de qualquer componente existir.
export interface StorySection {
  readonly id: string
  /** A única pergunta que esta seção responde. Duas perguntas = duas seções. */
  readonly question: string
  /** Temperatura emocional, 0–10. Vizinhas nunca empatam; o pico difere delas em ≥ 2. */
  readonly beat: number
  /** Fatia do scroll, 0–1: 0.06 a 0.22 e, num capítulo pinado, com `scroll` entre 2 e 6. */
  readonly share: number
  readonly scroll: number
  /** 0,68 a 0,75 × scroll. */
  readonly scrollMobile: number
  /** Preenchido por `copy-conversao`, que é a dona da estratégia de CTA. */
  readonly cta: 'primary' | 'secondary' | 'both' | 'none'
  readonly proof?: 'availability' | 'number' | 'face' | 'testimonial' | 'credential' | 'before-after'
}

export const story: readonly StorySection[] = [
  { id: 'inicio', question: 'Isto é para mim?', beat: 5, share: 0.161, scroll: 4.6, scrollMobile: 3.2, cta: 'none' },
  { id: 'jornada', question: 'O que vai acontecer comigo?', beat: 5, share: 0.201, scroll: 6, scrollMobile: 4.2, cta: 'none' },
  // …
]
```

`design/estrutura.md` é a mesma lista em tabela, para o dev revisar sem abrir código: id, seção,
pergunta, beat, share, `scroll`, `scrollMobile`, prova.

## A ordem narrativa é a ordem de acessibilidade

Não são duas tarefas. **A ordem narrativa é a ordem do DOM** — nunca reordene com `order` do
flexbox: Tab e leitor de tela seguem o DOM, e uma história contada fora de ordem é outra história.
Cada capítulo é `<section aria-labelledby>` apontando para a própria headline; seção sem nome
acessível não aparece na lista de landmarks. Um `<h1>` por página, no hero; capítulos em `<h2>`;
FAQ em `<h3>`. Sob `prefers-reduced-motion` a página vira seções empilhadas nesta ordem — e é essa
forma que o Google indexa, então verifique que a história continua legível nela.

## Anti-patterns

- **Prova social na seção 02 de uma landing de serviço local.** Um depoimento responde uma
  pergunta ("posso confiar?") que o visitante ainda não fez — ele ainda está em "do que se trata?".
  Sem a pergunta, o depoimento lê como enfeite, é pulado, e queima a posição em que teria peso.
  Exceção: prova de *disponibilidade* em Urgência ("3 veterinários de plantão agora") não é prova
  social, é informação operacional, e pertence à seção 02.
- **Copiar a ordem Hero → Prova → Problema → Solução de template SaaS.** Ela foi desenhada para
  software self-serve, onde a prova é o diferencial contra quarenta concorrentes idênticos. Em
  clínica local o diferencial é a pessoa, e a prova só ganha peso depois do método.
- **Uma seção respondendo duas perguntas.** O visitante retém uma ideia por tela; a segunda apaga
  a primeira e nenhuma fica. Se a seção precisa de mais de 22% do scroll, ela são duas.
- **Seção que só existe para ter a seção.** "Sobre nós", "Nossos valores", "Missão e visão" não
  respondem pergunta nenhuma do visitante. Cada uma custa share que sai do capítulo que derruba
  medo, e o resultado é uma página mais longa que convence menos.
- **Capítulo final que repete o hero.** Se a última seção diz o mesmo que a primeira, a página
  inteira foi um parêntese.
- **Esticar `scroll` além de 6 para caber a narrativa.** O capítulo pinado passa a ficar parado na
  tela por mais de seis viewports e o visitante conclui que a página travou. A correção é dividir
  o capítulo ou cortar a seção.
- **Dois picos.** Com dois beats ≥ 8 o visitante recalibra a escala para cima, nenhum dos dois é
  lido como pico e a página vira barulho constante.

## Handoff

- `creative-direction-expert` entregou na Fase 5 o `depth` total e a posição do pico; esta skill
  devolve ordem final, `share`, `beat` e os `scroll` / `scrollMobile` já convertidos. Ele ratifica
  na Fase 7 (band, pontos, WOW) e, se um capítulo estourar o teto de pontos da band, devolve para
  cá repartir o `share` — nunca estica o `scroll`. Duas skills escrevendo `scroll` produzem duas
  páginas de altura diferente, e nenhuma das duas erra sozinha.
- `copy-conversao` recebe `question`, `beat` e `share`: a pergunta define o que a copy tem que
  responder, o share define quanto texto cabe.
- `product-design-expert` recebe `question` e `proof` — eles definem o layout da seção.
- `landing-motion-expert` recebe `beat`: beat ≤ 4 recebe **uma** revelação por tela; beat ≥ 7 é
  onde o WOW major pode ser gasto.

## Verificação

```js
// Shares reais medidos do DOM — compare com o story map. Rode duas vezes, numa janela larga e
// numa de 375px: a altura troca de `--chapter-scroll-desktop` para `--chapter-scroll-mobile`.
const main = document.querySelector('main')
console.table(
  [...main.querySelectorAll(':scope > section[id]')].map((s) => ({
    id: s.id,
    share: +((s.offsetHeight / main.offsetHeight) * 100).toFixed(1),
    title: s.querySelector('h1, h2')?.textContent?.trim().slice(0, 48),
  })),
)
```

- [ ] Soma dos `share` = 1; nenhum abaixo de 6% nem acima de 22%
- [ ] `scrollMobile` entre 0,68 e 0,75 × `scroll` em todo capítulo
- [ ] Nenhum `scroll` de capítulo pinado fora da faixa 2–6
- [ ] A coluna `title` lida de cima a baixo forma uma narrativa, não uma lista de assuntos
- [ ] Nenhum par de vizinhas com o mesmo `beat`; o pico difere das duas vizinhas em ≥ 2
- [ ] Exatamente um beat ≥ 8 na página, e a seção seguinte cai ≥ 3
- [ ] Cada seção responde exatamente uma pergunta, escrita em `question`
- [ ] Com `prefers-reduced-motion: reduce`: a ordem das seções ainda conta a história
