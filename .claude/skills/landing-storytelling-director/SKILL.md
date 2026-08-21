---
name: landing-storytelling-director
description: Use when deciding what a landing page says and in which order — section order, narrative arc, pacing, scroll budget per section, CTA placement and FAQ ordering. Escolhe o arquétipo de estrutura, define a ordem das seções, o beat emocional de cada uma, quantos CTAs e onde, e as regras de copy (headline, eyebrow, corpo). Palavras-chave: estrutura da landing, ordem das seções, narrativa, storytelling, pacing, onde colocar o CTA, WhatsApp, headline, FAQ, section order, story arc, conversion copy.
argument-hint: [tipo-de-negocio] [o-que-o-visitante-precisa-decidir]
---

# Landing Storytelling Director

Decide **o que** o visitante experimenta e em que ordem. Não decide como se move
(`landing-motion-expert`) nem como parece (`product-design-expert`).

A entrega desta skill não é código de componente. É um **story map** em `src/data/story.ts`:
lista ordenada de seções, cada uma com a pergunta que responde, um beat emocional, uma fatia do
scroll e uma decisão de CTA. A copy que preenche essas seções vive em `src/data/site.ts`, fora
dos componentes, para que a narrativa possa ser lida de ponta a ponta como texto contínuo — que
é como ela é de fato consumida.

## Input

- Tipo de negócio e **o que o visitante precisa decidir** nesta página.
- Canal de conversão. Nos projetos deste workspace é **WhatsApp**, não formulário.
- As provas que existem de verdade: número de anos, registro profissional, fotos de resultado,
  depoimentos com nome. Prova inventada em landing de saúde é risco jurídico, não copy.
- O Experience Score já definido (ver CLAUDE.md).
- Se a lista de seções não vier pronta, leia `src/data/site.ts` e trabalhe a partir dela em vez
  de inventar capítulos.

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
| Pontos de conversão | 5–6 | 3–4 | 3–4 | 4–5 |
| Primeiro CTA | acima da dobra | não antes de 30% do scroll | acima da dobra | acima da dobra |
| Sticky mobile | desde o pixel 0 | a partir de 30% | a partir de 25% | a partir de 25% |
| Exemplo típico | clínica veterinária 24h | Beleza Completa Barreiro | dentista, advogado | SaaS B2B |

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

`Share` é a fatia do scroll total da página. `CTA` é `1` para primário, `2` para secundário,
`—` para nenhum. Detalhamento por seção — briefing de copy, o que quebra, variação mobile — em
[archetypes.md](archetypes.md).

### Urgência — serviço 24h, emergência, plantão

| # | Seção | Pergunta que responde | Beat | Share | CTA | Prova |
|---|---|---|---|---|---|---|
| 01 | Estado agora | "Vocês estão abertos neste momento?" | 7 | 12% | 1 + 2 | disponibilidade |
| 02 | Como chegar | "Onde é e quanto tempo eu levo?" | 6 | 8% | 2 | mapa, referência |
| 03 | Quando é emergência | "O que eu estou vendo é grave?" | **9** | 12% | 1 | lista de sinais |
| 04 | O que acontece ao chegar | "Vou ser atendido ou vou esperar?" | 5 | 16% | — | protocolo minuto a minuto |
| 05 | Estrutura | "Vocês têm o que o caso precisa?" | 4 | 12% | — | equipamento nomeado |
| 06 | Quem atende | "Tem profissional de verdade de plantão?" | 5 | 10% | — | nome + registro |
| 07 | Depoimentos de emergência | "Já resolveram um caso como o meu?" | 7 | 10% | — | testemunho com desfecho |
| 08 | FAQ | "Quanto custa? Preciso agendar?" | 3 | 10% | — | preço/faixa |
| 09 | Contato completo | "Como falo agora?" | 7 | 10% | 1 + 2 | endereço, telefone, mapa |

A 03 é o único pico: hero e fecho ficam em 7 justamente para que ela seja 9. A 04 cai quatro
pontos de propósito — tensão sem alívio fecha a aba. O CTA secundário aqui é `tel:`: em
emergência, ligar bate WhatsApp.

### Transformação — estética, resultado no corpo

| # | Seção | Pergunta que responde | Beat | Share | CTA | Prova |
|---|---|---|---|---|---|---|
| 01 | Promessa | "Isto é para mim?" | 5 | 16% | — | — |
| 02 | A experiência | "Como é estar nesse lugar?" | 6 | 12% | — | ambiente, atendimento |
| 03 | O processo em etapas | "O que exatamente vai acontecer comigo?" | 5 | 20% | 1 | processo nomeado |
| 04 | A consulta | "Vão me ouvir ou me empurrar um pacote?" | 6 | 14% | — | consulta individual |
| 05 | O princípio | "Vou ficar artificial?" | **8** | 16% | — | antes/depois, resultado |
| 06 | A clínica | "É um lugar sério?" | 4 | 12% | 2 | fotos reais do espaço |
| 07 | Convite | "E agora?" | 7 | 10% | 1 + 2 | contato completo |

Nenhum CTA antes de 30% do scroll. O medo aqui não é de perder tempo, é de errar o rosto —
pedir agendamento antes de derrubar esse medo transforma o botão em objeto de recusa, e a
segunda aparição dele já carrega o "não".

O FAQ nesta estrutura vive **dentro** da seção 06, não como capítulo próprio: um capítulo de
FAQ entre a prova e o convite esfria o arco exatamente onde ele deveria fechar.

### Credencial — saúde, jurídico, ticket alto

| # | Seção | Pergunta que responde | Beat | Share | CTA | Prova |
|---|---|---|---|---|---|---|
| 01 | Afirmação com número | "Do que se trata e quem faz?" | 5 | 14% | 1 | número duro na headline |
| 02 | O risco de errar | "O que acontece se eu escolher mal?" | **8** | 12% | — | consequência concreta |
| 03 | O método | "Como vocês trabalham?" | 5 | 18% | — | etapas nomeadas |
| 04 | Quem é a pessoa | "Quem vai me atender?" | 6 | 14% | 2 | rosto, nome, registro |
| 05 | Casos | "Já resolveram como o meu?" | 7 | 12% | — | caso com desfecho |
| 06 | FAQ | "Quanto custa, quanto demora, é sigiloso?" | 3 | 12% | — | honorários |
| 07 | Contato | "Como começo?" | 6 | 10% | 1 | canal + horário |
| — | Rodapé com registro | — | 1 | 8% | — | registro, endereço, CNPJ |

O rosto da pessoa é a prova. Em serviço de credencial, um stock photo de "equipe sorrindo"
destrói mais confiança do que nenhuma foto.

### Produto — SaaS, self-serve

| # | Seção | Pergunta que responde | Beat | Share | CTA | Prova |
|---|---|---|---|---|---|---|
| 01 | O que é, em uma frase | "O que isso faz?" | 5 | 14% | 1 + 2 | — |
| 02 | Quem já usa | "Alguém sério confia nisso?" | 4 | 6% | — | logos, número de contas |
| 03 | O custo do problema | "Quanto isso está me custando hoje?" | **8** | 12% | — | número do desperdício |
| 04 | Como funciona em 3 passos | "Vai dar trabalho?" | 5 | 16% | 1 | tempo de setup |
| 05 | Features por resultado | "Resolve o meu caso específico?" | 4 | 18% | — | feature → consequência |
| 06 | Comparação | "Por que não a alternativa X?" | 6 | 10% | — | tabela honesta |
| 07 | Depoimento com métrica | "Quanto mudou para eles?" | 7 | 8% | — | número no depoimento |
| 08 | Preço + FAQ | "Quanto custa e como saio?" | 3 | 10% | 1 | tabela de preço |
| 09 | CTA final | — | 6 | 6% | 1 | — |

Único arquétipo em que prova social vem na seção 02, e o motivo é específico: o visitante compara
contra quarenta concorrentes idênticos, então "alguém sério confia nisso?" já está feita antes de
ele chegar. Em serviço local ela não está — ver anti-patterns.

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

- **Nenhuma seção abaixo de 6%.** Abaixo disso ela passa antes de ser lida e o visitante
  registra "havia algo ali" sem saber o quê.
- **Nenhuma seção acima de 22%.** Acima disso, num capítulo pinado, o visitante rola e a tela
  não muda o suficiente — ele conclui que a página travou e sai.
- **A seção mais longa é sempre a que reduz mais medo** (método/processo), nunca o hero. O hero
  cria a pergunta; ele não precisa de scroll para isso.
- **Vizinhas nunca têm o mesmo beat.** Entre o pico e cada uma das suas vizinhas a diferença é
  **≥ 2**: um pico anunciado com 1 ponto de diferença não é lido como pico. Fora do pico, 1
  ponto basta — é o que separa duas seções calmas sem fingir tensão que não existe.
- **Depois do beat ≥ 8, a próxima cai ≥ 3.**
- **`scrollMobile` fica entre 0,68 e 0,75 × `scroll`**, capítulo a capítulo. É a faixa canônica
  do projeto — as outras skills a referenciam em vez de repeti-la — e na referência a razão medida
  vai de 0,69 a 0,75; fora dela o capítulo conta outra história no celular. Compare razão de prop
  com razão de prop: a das **alturas totais** (`1 + scroll`) é ~76% (26,6 vs 34,8 viewports),
  porque o `1` do palco sticky não encolhe.

### Convertendo share em `scroll` prop

`<Chapter scroll>` é medido em alturas de viewport **além** da primeira tela, então a altura
total de um capítulo é `1 + scroll`.

```ts
/**
 * Converte a fatia desejada do scroll total no valor da prop `scroll` de <Chapter>.
 * `total` = soma de (1 + scroll) de todos os capítulos da página.
 */
export const scrollProp = (share: number, total: number): number =>
  Number((share * total - 1).toFixed(1))

// scrollProp(0.20, 34.8) === 6.0 — o capítulo da jornada no projeto de referência.
```

`creative-direction-expert` orça capítulo pinado entre `scroll={2}` e `scroll={6}`, e é isso que
fecha o piso e o teto de `share` de verdade: com `total` = 34,8 o piso de 6% devolveria 1,1 e o
teto de 22% devolveria 6,7, os dois fora da faixa dele. Os limites efetivos nessa página são
**8,6% e 20,1%**, onde `agendar` (2,0) e `jornada` (6,0) estão. Caiu fora? Funda o capítulo ou
divida-o; nunca estique `scroll` além de 6. A faixa 2–6 vale só para capítulo pinado
(`<Chapter>`) — seção em fluxo normal (FAQ, contato, rodapé) ocupa a altura do próprio conteúdo,
e nela vale o piso de 6% e mais nada.

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

## Estratégia de CTA — WhatsApp primeiro

Ponto de conversão é **cada posição do scroll onde dá para agir**. O header conta um; a barra
sticky não conta em separado — ela é o mesmo pedido do header, reposicionado para o polegar.
Serviço local no Brasil quer **3 a 6**. Menos de 3 e quem decidiu na seção 04 rola até o fim
para agir. Mais de 6 e cada um vale menos: o visitante para de lê-los como convite e passa a
lê-los como mobília.

Posições, em ordem de obrigatoriedade:

1. **Header persistente** — sempre. É o único CTA que existe em todas as telas. Rótulo curto
   ("Agendar"), nunca a frase completa.
2. **Fim do capítulo de método/processo** — o ponto onde o medo caiu mais. Se a página tiver
   um só CTA no corpo, é este.
3. **Capítulo final** — primário + secundário, com contato completo ao lado.
4. Opcional: depois da prova (antes/depois, depoimento), quando o beat está em 7–8.
5. Opcional: fim do FAQ, apenas se o FAQ for capítulo próprio.

**Um primário por tela.** Dois botões do mesmo peso lado a lado custam mais em decisão do que a
própria ação, e o visitante adia. Use as variantes de `src/components/ui/Action.tsx`:
`solid`/`inverse` para o primário, `outline` para o secundário — a diferença de peso é o que
faz a escolha acontecer sem pensar.

| Arquétipo | Primário | Secundário |
|---|---|---|
| Urgência | WhatsApp (`wa.me`) | Telefone (`tel:`) — ligar bate mensagem em emergência |
| Transformação | WhatsApp | Navegação interna ("Conhecer a clínica") |
| Credencial | WhatsApp | Telefone ou e-mail com horário declarado |
| Produto | Signup | Demo / documentação |

### Mensagem pré-preenchida por origem

Sem formulário não existe campo escondido, então a origem tem que viajar dentro da própria
mensagem. Isso é a única analítica de funil que uma landing de WhatsApp consegue — e faz o
atendente responder certo na primeira volta.

```ts
// src/data/cta.ts
import { clinic } from './site'

// Uma fonte só para o número. Duplicar o literal aqui garante que um dia os dois
// divergem e metade dos botões passa a discar a linha antiga.
const WHATSAPP_NUMBER = clinic.phoneE164.replace('+', '') // wa.me rejeita o sinal

export type CtaOrigin = 'header' | 'hero' | 'method' | 'proof' | 'faq' | 'finale' | 'sticky'

const INTENT: Record<CtaOrigin, string> = {
  header: 'Olá! Gostaria de agendar uma avaliação.',
  hero: 'Olá! Vim pelo site e gostaria de agendar uma avaliação.',
  method: 'Olá! Li sobre as etapas do atendimento e quero agendar a avaliação.',
  proof: 'Olá! Vi os resultados no site e queria saber se serve para o meu caso.',
  faq: 'Olá! Fiquei com uma dúvida sobre o atendimento.',
  finale: 'Olá! Gostaria de agendar minha avaliação.',
  sticky: 'Olá! Gostaria de falar com alguém agora.',
}

export const whatsappHref = (origin: CtaOrigin): string =>
  `https://wa.me/${WHATSAPP_NUMBER}?text=${encodeURIComponent(INTENT[origin])}`
```

Toda âncora de WhatsApp leva `target="_blank" rel="noopener noreferrer"` e um `aria-label` que
diz o canal — "Agendar avaliação pelo WhatsApp". O rótulo visível sozinho não informa que a
página vai ser trocada por outro app.

### Sticky: quando vale e quando irrita

| Situação | Sticky? | Causa |
|---|---|---|
| Urgência, mobile | Sim, desde o pixel 0 | A decisão pode ser tomada na primeira frase; esconder o botão custa a conversão inteira |
| Transformação, mobile | Só depois de 30% do scroll | Antes disso o visitante ainda não quer nada; a barra come 72px de tela durante justamente a leitura que criaria a vontade |
| Credencial / Produto, mobile | Depois de 25% | O visitante ainda está avaliando; a barra vira ruído sobre a avaliação |
| Qualquer arquétipo, ≥ 1024px | Não | O header já está sempre visível. Uma segunda barra pede a mesma coisa duas vezes |
| Capítulo final na tela | Some | Dois pedidos idênticos na mesma tela cancelam-se |
| Sobre campo de input | Nunca | O teclado mobile sobe e a barra tapa o campo que está sendo preenchido |

O gate é um único `ScrollTrigger` sobre o documento, dentro de `gsap.context()`, que liga
`self.progress >= revealAt` com "o capítulo final ainda não entrou na tela". O hook
`useStickyCta` e a barra — com `inert` quando invisível, `env(safe-area-inset-bottom)` e
`motion-reduce` — estão em [cta-and-faq.md](cta-and-faq.md).

## Regras de copy

Todo teto abaixo foi medido contra os tokens reais de `src/styles/index.css` e contra a copy
real de `src/data/site.ts`.

| Elemento | Teto | Onde o teto vem |
|---|---|---|
| Headline do hero | **62 caracteres** | `--text-hero` = `clamp(2.125rem, 4.5vw, 4.5rem)`. A headline real do projeto tem 62 e ocupa 3 linhas no ceiling. A 4ª linha empurra o CTA para fora da dobra em 375px |
| Headline de capítulo | **56 caracteres** | `--text-chapter` = `clamp(1.875rem, 3.5vw, 3.25rem)` dentro de `max-w-5xl` |
| Eyebrow / label | **18 caracteres** | `--text-eyebrow` = `0.6875rem` com `letter-spacing: 0.24em`, mais a régua de 32px do componente `Eyebrow`. Os seis eyebrows reais têm 7 a 13 caracteres |
| Lead | **230 caracteres, 2 frases** | `--text-lead` em `max-w-xl`: 230 caracteres ≈ 4 linhas em desktop, 7 em 375px |
| Corpo de pilar (grade de 4) | **70 caracteres, 1 frase** | O maior real tem 67. Acima disso os quatro cards param de caber numa tela em 1280px |
| Corpo de etapa (lista vertical) | **130 caracteres, 2 frases** | A lista é lida uma linha por vez, não em paralelo. As cinco etapas reais vão de 82 a 125 |
| Rótulo de botão | **24 caracteres, 3 palavras** | `Action` usa `px-8` e `min-h-[56px]`. Passando disso o rótulo quebra em duas linhas em 375px |
| Pergunta de FAQ | **70 caracteres** | Tem que caber em uma linha ao lado do glifo de expandir |
| Resposta de FAQ | **400 caracteres** | Além disso a resposta vira artigo e ninguém termina |

### Número em vez de adjetivo

Toda afirmação de capacidade vira número **quando o número existe e é verificável**.

| Adjetivo | Número |
|---|---|
| "Atendimento rápido" | "Resposta no WhatsApp em até 10 minutos, das 9h às 18h" |
| "Anos de experiência" | "Desde 2016" — a data não envelhece o site; a contagem envelhece |
| "Equipe qualificada" | "Dra. Nome, CRO-MG 00000" |
| "Muitos clientes" | "1.240 avaliações desde 2016" |
| "Tecnologia moderna" | O nome do equipamento, ou nada |

Quando **não** usar número: quando ele é pequeno e a comparação com o concorrente é o assunto
("3 profissionais"). Nesse caso use o número que é forte — anos, casos, tempo de resposta — ou
nenhum. Nunca aproxime para cima.

Número na headline só quando ele **é** a promessa. "Seu atendimento acontece em cinco etapas"
funciona porque a contagem é o alívio. "12 anos de experiência" como headline do hero não
funciona porque o visitante ainda não pediu credencial — ele ainda está descobrindo do que a
página trata.

### Eyebrow

O eyebrow diz **onde você está**. A headline diz **o que acontece aqui**. Se os dois dizem a
mesma coisa, um dos dois é inútil.

- 1 a 3 palavras, ≤ 18 caracteres, sem verbo, sem ponto final.
- É nome de capítulo: "A consulta", não "Entenda como funciona a consulta".
- Numeração (`03 / Sua jornada`) só se a página inteira tiver progressão declarada. Numerar
  quatro seções de nove faz o visitante procurar as outras cinco.
- Em Urgência, e só ali, o eyebrow pode carregar estado em vez de rótulo: "Aberto agora",
  "Plantão 24h". É o único caso em que ele informa.

## FAQ que converte

A primeira pergunta é a **objeção mais cara** — aquela que, sem resposta, faz a aba fechar.
Não a mais fácil de responder, não a que a empresa prefere.

Método:

1. Liste tudo que o atendente responde no WhatsApp **antes** de conseguir agendar. Essa lista
   é o FAQ real. Não invente perguntas.
2. Ordene por custo de abandono: quanto vale o visitante que vai embora por falta daquela
   resposta.
3. A pergunta de preço vai em 1º ou 2º, **sempre**.
4. Máximo 7 perguntas. A 8ª é conteúdo de blog.

| Posição | Urgência | Transformação | Credencial | Produto |
|---|---|---|---|---|
| 1 | "Vocês estão abertos agora?" | "Quanto custa?" | "Como vocês cobram?" | "Quanto custa?" |
| 2 | "Quanto custa a emergência?" | "Fica artificial?" | "Quanto tempo leva?" | "Migra meus dados?" |
| 3 | "Preciso agendar ou posso chegar?" | "Dói? Quanto tempo de recuperação?" | "Quem vai me atender?" | "Preciso de time técnico?" |
| 4 | "Onde fica? Tem estacionamento?" | "Quanto tempo dura o resultado?" | "Atende o meu caso?" | "E se eu quiser sair?" |
| 5 | "Aceitam qual pagamento?" | "E se eu não gostar?" | "É sigiloso?" | "Tem plano gratuito?" |

Dentro de cada resposta: **a primeira frase responde**, o resto justifica. Nunca o contrário —
quem lê FAQ está com pressa e abandona no meio da justificativa se a resposta ainda não veio.

Marcação: `<details>` nativo. O teclado já funciona, o `<summary>` já anuncia o próprio estado,
e **a resposta continua no DOM com o item fechado**: Ctrl+F acha, o leitor de tela acha, o
Googlebot acha ao renderizar. Atrás de `{open && …}` ela não existe enquanto está fechada. Estes
projetos são SPA client-rendered, então o que vale é o DOM renderizado — nenhuma das duas versões
está no HTML servido. Componente e JSON-LD de `FAQPage` em [cta-and-faq.md](cta-and-faq.md).

## Anti-patterns

- **Prova social na seção 02 de uma landing de serviço local.** Um depoimento é resposta a uma
  pergunta ("posso confiar?") que o visitante ainda não fez — ele ainda está em "do que se
  trata?". Sem a pergunta, o depoimento lê como enfeite e é pulado, e queima a posição em que
  ele teria peso. Exceção: prova de *disponibilidade* em Urgência ("3 veterinários de plantão
  agora") não é prova social, é informação operacional, e pertence à seção 02.
- **Copiar a ordem Hero → Prova → Problema → Solução de template SaaS.** Ela foi desenhada para
  software self-serve, onde a prova é o diferencial contra quarenta concorrentes idênticos. Em
  clínica local o diferencial é a pessoa, e a prova só ganha peso depois do método.
- **Hero que explica tudo.** O hero é varrido, não lido. Cinco parágrafos ali não são lidos mais
  rápido, são lidos menos — e quem conclui que já viu o argumento não rola.
- **Preço apenas "sob consulta".** O visitante não adia a pergunta, adia a conversa. Uma faixa
  ("a partir de R$ X", "avaliação sem custo") converte melhor que silêncio mesmo quando é alta:
  filtra quem não fecharia e tranquiliza quem fecharia.
- **FAQ com perguntas que ninguém faz.** "Por que escolher a Beleza Completa?" é copy de vendas
  fantasiada de resposta. O visitante reconhece e para de ler — inclusive as reais, que estão
  logo abaixo.
- **Feature sem consequência.** "Tecnologia moderna" não responde nada. Sem a frase seguinte
  ("equipamentos que oferecem mais precisão, conforto e segurança") o item ocupa espaço e não
  move ninguém. São sempre três: feature → por que importa → resultado.
- **Capítulo final que repete o hero.** Se a última seção diz o mesmo que a primeira, a página
  inteira foi um parêntese. O fechamento é a única seção com contato completo: endereço,
  horário, telefone, WhatsApp.
- **Uma seção respondendo duas perguntas.** O visitante retém uma ideia por tela; a segunda
  apaga a primeira e nenhuma fica.
- **Formulário como conversão principal.** No Brasil, em serviço local, o formulário devolve o
  controle para a empresa ("entraremos em contato") e o visitante perde a garantia de resposta.
  O WhatsApp é assíncrono, tem confirmação de leitura e já está aberto no telefone dele. Use
  formulário só quando houver necessidade legal de registro.
- **Seção que só existe para ter a seção.** "Sobre nós", "Nossos valores", "Missão e visão" não
  respondem pergunta nenhuma do visitante. Cada uma custa share que sai do capítulo que derruba
  medo — e o que sobra é uma página mais longa que convence menos.

## Acessibilidade — dentro do fluxo, não no rodapé

A estrutura narrativa *é* a estrutura de acessibilidade. Elas não são duas tarefas.

- **A ordem narrativa é a ordem do DOM.** Nunca reordene com `order` do flexbox: Tab e leitor
  de tela seguem o DOM, e uma história contada fora de ordem é outra história.
- Cada capítulo é `<section aria-labelledby>` apontando para a própria headline (ver
  `src/components/ui/Chapter.tsx`). Seção sem nome acessível não aparece na lista de landmarks,
  e quem navega por landmarks não consegue pular para ela.
- Um `<h1>` por página, no hero. Capítulos usam `<h2>`. FAQ dentro de um capítulo usa `<h3>`.
  Nível pulado quebra a navegação por cabeçalhos, que é como se lê uma landing longa sem ver.
- Endereço, horário e telefone existem como texto no DOM — nunca só dentro de imagem ou de
  frame de vídeo. É a informação que mais é copiada da página.
- Sob `prefers-reduced-motion` a página vira seções empilhadas em ordem (o CSS já faz isso).
  Verifique que a **história** continua legível nessa forma: é ela que o Google indexa.
- O CTA sticky invisível recebe `inert` — não basta `opacity: 0`, ou o Tab cai num botão que
  ninguém vê.

## Story map — o artefato que sai daqui

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
  readonly cta: 'primary' | 'secondary' | 'both' | 'none'
  readonly proof?: 'availability' | 'number' | 'face' | 'testimonial' | 'credential' | 'before-after'
}

export const story: readonly StorySection[] = [
  { id: 'inicio', question: 'Isto é para mim?', beat: 5, share: 0.161, cta: 'none' },
  { id: 'experiencia', question: 'Como é estar nesse lugar?', beat: 6, share: 0.121, cta: 'none' },
  { id: 'jornada', question: 'O que vai acontecer comigo?', beat: 5, share: 0.201, cta: 'primary' },
  // …
]
```

## Handoff

- `creative-direction-expert` entrega, na passada 1, o `depth` total e a posição do pico; esta
  skill devolve ordem final, `share`, `beat` e **os `scroll` / `scrollMobile` já convertidos** por
  `scrollProp()` sobre aquele `depth` — **é dona desses dois números**. A direção criativa ratifica
  na passada 2 (band, pontos, WOW) e, se um capítulo estourar o teto de pontos da band, devolve
  para cá repartir o `share`; nunca estica o `scroll`. Duas skills escrevendo `scroll` produzem
  duas páginas de altura diferente, e nenhuma das duas erra sozinha.
- `product-design-expert` recebe `question` e `proof` — eles definem o layout da seção.
- `landing-motion-expert` recebe `beat`. Regra de tradução: beat ≤ 4 recebe **uma** revelação
  por tela; beat ≥ 7 é onde o WOW major pode ser gasto.
- Se o motion pedir mais scroll do que o `share` permite, a resposta é cortar a seção, não
  esticá-la. Uma seção que precisa de 25% do scroll está respondendo duas perguntas.

## Verificação

Rode com a página aberta em `npm run dev`:

```js
// Shares reais medidos do DOM — compare com o story map. Rode duas vezes: numa janela
// larga e numa de 375px, porque a altura do capítulo troca de `--chapter-scroll-desktop`
// para `--chapter-scroll-mobile` e os dois conjuntos de share são orçados separadamente.
const main = document.querySelector('main')
console.table(
  [...main.querySelectorAll(':scope > section[id]')].map((s) => ({
    id: s.id,
    share: +((s.offsetHeight / main.offsetHeight) * 100).toFixed(1),
    title: s.querySelector('h1, h2')?.textContent?.trim().slice(0, 48),
  })),
)
```

- [ ] Nenhum `share` abaixo de 6% nem acima de 22%
- [ ] A coluna `title` lida de cima a baixo forma uma narrativa, não uma lista de assuntos
- [ ] Nenhum par de vizinhas com o mesmo `beat`; o pico difere das duas vizinhas em ≥ 2
- [ ] Exatamente um beat ≥ 8 na página, e a seção seguinte cai ≥ 3
- [ ] Headline do hero ≤ 62 caracteres; todo rótulo de botão ≤ 24
- [ ] Entre 3 e 6 pontos de conversão renderizados; um primário por tela
- [ ] Toda âncora `wa.me` carrega `?text=` com a origem embutida
- [ ] Primeira pergunta do FAQ é preço ou disponibilidade
- [ ] Com todo `<details>` fechado, Ctrl+F acha o texto de qualquer resposta do FAQ
- [ ] Com JavaScript desligado, o `<noscript>` de `index.html` ainda dá nome, endereço e
      telefone clicável — é tudo que resta de uma SPA nessa condição
- [ ] Em 375px: o primeiro CTA aparece sem rolar (Urgência) ou só após 30% (Transformação)
- [ ] Com `prefers-reduced-motion: reduce`: a ordem das seções ainda conta a história
- [ ] Tab do topo ao rodapé nunca para num controle invisível
