---
name: "copy-conversao"
description: "Use when writing the actual landing page copy — headline, eyebrow, body, button label, FAQ — and deciding how many CTAs and where. Fase 6b — tetos de caractere medidos, numero no lugar de adjetivo, estrategia de CTA com WhatsApp, FAQ ordenado pela objecao mais cara, e a proibicao de reaproveitar copy do site antigo ou frase generica do setor. Palavras-chave: copy, headline, eyebrow, texto da secao, CTA, WhatsApp, FAQ, conversao."
argument-hint: "[id-da-secao-ou-\x22tudo\x22]"
---

# Copy de conversão — Fase 6b

| | |
|---|---|
| **ENTRADA** | `design/estrutura.md` e `src/data/story.ts` (a ordem, o `question` e o `share` de cada seção); `design/pesquisa.md` (palavras literais do cliente, objeções em ordem de custo, frases banidas); `design/design-system.json` — `facts` é a única origem de número citável, `voice` fixa o tom, `unverified` lista o que não vira promessa; `design/briefing.json` — `business`, `services`, `doesNotOffer`, `differentiators`, `socialProof`, `pricing` para contato, oferta e prova; `scripts/check-banned-copy.mjs`, gravado por `niche-research`, é o portão de saída |
| **SAÍDA** | `design/landing-blueprint.md` com a copy completa, e a mesma copy em `src/data/site.ts` |
| **ANTES** | `estrutura-secoes` (Fase 6a) fixa quais seções existem e em que ordem |
| **DEPOIS** | `creative-direction-expert` (Fase 7) ratifica a direção sobre a estrutura e a copy prontas |

Esta skill escreve **as palavras**. Não muda a ordem das seções nem o `share` — se a copy não
couber, o problema é a copy, não a estrutura; volte à `estrutura-secoes` só quando a seção estiver
respondendo duas perguntas.

É a dona de dois conjuntos de números: os **tetos de caractere** e a **contagem de pontos de
conversão**. Nenhuma outra skill os redefine.

## O que já está decidido quando esta skill começa

`estrutura-secoes` entregou, por seção: `id`, `question` (a única pergunta que ela responde),
`beat` (0–10) e `share` do scroll. A copy responde exatamente aquela pergunta, no tom daquele
beat, dentro daquele espaço. Copy que responde outra pergunta destrói o arco inteiro e não é
detectável lendo a seção isolada.

## Tetos de caractere

Todo teto abaixo foi medido contra os tokens reais de `src/styles/index.css` e contra a copy real
de `src/data/site.ts`. São medidos contra a tipografia — não os aperte para caber mais texto.

| Elemento | Teto | Onde o teto vem |
|---|---|---|
| Headline do hero | **62 caracteres** | `--text-hero` = `clamp(2.125rem, 4.5vw, 4.5rem)`. A headline real tem 62 e ocupa 3 linhas no ceiling. A 4ª linha empurra o CTA para fora da dobra em 375px |
| Headline de capítulo | **56 caracteres** | `--text-chapter` = `clamp(1.875rem, 3.5vw, 3.25rem)` dentro de `max-w-5xl` |
| Eyebrow / label | **18 caracteres** | `--text-eyebrow` = `0.6875rem` com `letter-spacing: 0.24em`, mais a régua de 32px do componente `Eyebrow`. Os seis eyebrows reais têm 7 a 13 caracteres |
| Lead | **230 caracteres, 2 frases** | `--text-lead` em `max-w-xl`: 230 caracteres ≈ 4 linhas em desktop, 7 em 375px |
| Corpo de pilar (grade de 4) | **70 caracteres, 1 frase** | O maior real tem 67. Acima disso os quatro cards param de caber numa tela em 1280px |
| Corpo de etapa (lista vertical) | **130 caracteres, 2 frases** | A lista é lida uma linha por vez, não em paralelo. As cinco etapas reais vão de 82 a 125 |
| Rótulo de botão | **24 caracteres, 3 palavras** | `Action` usa `px-8` e `min-h-[56px]`. Passando disso o rótulo quebra em duas linhas em 375px |
| Pergunta de FAQ | **70 caracteres** | Tem que caber em uma linha ao lado do glifo de expandir |
| Resposta de FAQ | **400 caracteres** | Além disso a resposta vira artigo e ninguém termina |

Em Urgência o hero é a exceção mais apertada: primário e secundário precisam ficar visíveis **sem
rolar** em 375px, o que trava a headline em ~48 caracteres, não 62.

## Número em vez de adjetivo

Toda afirmação de capacidade vira número **quando o número existe em `design-system.json.facts`
com `source`, ou em `briefing.json` num campo com `source` e `confirmed: true`**. Número que só
existe na cabeça de quem escreve é promessa falsa sobre negócio real. Campo `assumed: true` e item
de `unverified` não viram número na página — viram pergunta em `design/lacunas.md`.

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
funciona porque a contagem é o alívio. "12 anos de experiência" como headline do hero não funciona
porque o visitante ainda não pediu credencial — ele ainda está descobrindo do que a página trata.

## Eyebrow

O eyebrow diz **onde você está**. A headline diz **o que acontece aqui**. Se os dois dizem a mesma
coisa, um dos dois é inútil.

- 1 a 3 palavras, ≤ 18 caracteres, sem verbo, sem ponto final.
- É nome de capítulo: "A consulta", não "Entenda como funciona a consulta".
- Numeração (`03 / Sua jornada`) só se a página inteira tiver progressão declarada. Numerar quatro
  seções de nove faz o visitante procurar as outras cinco.
- Em Urgência, e só ali, o eyebrow pode carregar estado em vez de rótulo: "Aberto agora",
  "Plantão 24h". É o único caso em que ele informa.

## O que a copy não pode reaproveitar

- **Nada do site antigo do cliente**, nem parafraseado. O site antigo é o motivo de existir um
  novo; reciclar a frase recicla o problema, e o cliente reconhece na primeira leitura.
- **Nenhuma frase da lista banida de `design/pesquisa.md`.** Essa lista é o que os 3 a 5
  concorrentes da mesma cidade já dizem, com contagem. Repetir uma delas coloca o cliente na
  mesma prateleira que a pesquisa custou para mapear. "Excelência no atendimento", "profissionais
  altamente qualificados", "sua satisfação é nossa prioridade" nunca sobrevivem à checagem.
- **Nenhuma promessa que caia em `unverified`** do `design-system.json` ou em "não pode prometer"
  da pesquisa. Em nicho regulado (saúde, jurídico) isso é risco jurídico, não estilo.

As palavras que **entram** vêm das citações literais de avaliação em `pesquisa.md`. O visitante
descreve o próprio problema com o vocabulário dele, não com o do setor.

```bash
node scripts/check-banned-copy.mjs src/data/site.ts   # precisa sair 0
```

## Estratégia de CTA — WhatsApp primeiro

Ponto de conversão é **cada posição do scroll onde dá para agir**. O header conta um; a barra
sticky não conta em separado — ela é o mesmo pedido do header, reposicionado para o polegar.
Serviço local no Brasil quer **3 a 6**. Menos de 3 e quem decidiu na seção 04 rola até o fim para
agir. Mais de 6 e cada um vale menos: o visitante para de lê-los como convite e passa a lê-los
como mobília.

| Arquétipo | Pontos | Primeiro CTA | Sticky mobile a partir de | Primário | Secundário |
|---|---|---|---|---|---|
| Urgência | 5–6 | acima da dobra | pixel 0 | WhatsApp (`wa.me`) | Telefone (`tel:`) — ligar bate mensagem em emergência |
| Transformação | 3–4 | não antes de 30% do scroll | 30% | WhatsApp | Navegação interna ("Conhecer a clínica") |
| Credencial | 3–4 | acima da dobra | 25% | WhatsApp | Telefone ou e-mail com horário declarado |
| Produto | 4–5 | acima da dobra | 25% | Signup | Demo / documentação |

Em Transformação o medo não é perder tempo, é errar o rosto — pedir agendamento antes de derrubar
esse medo transforma o botão em objeto de recusa, e a segunda aparição dele já carrega o "não".

Posições, em ordem de obrigatoriedade:

1. **Header persistente** — sempre. É o único CTA que existe em todas as telas. Rótulo curto
   ("Agendar"), nunca a frase completa.
2. **Fim do capítulo de método/processo** — o ponto onde o medo caiu mais. Se a página tiver um só
   CTA no corpo, é este.
3. **Capítulo final** — primário + secundário, com contato completo ao lado.
4. Opcional: depois da prova (antes/depois, depoimento), quando o beat está em 7–8.
5. Opcional: fim do FAQ, apenas se o FAQ for capítulo próprio.

**Um primário por tela.** Dois botões do mesmo peso lado a lado custam mais em decisão do que a
própria ação, e o visitante adia. Use as variantes de `src/components/ui/Action.tsx`:
`solid`/`inverse` para o primário, `outline` para o secundário — a diferença de peso é o que faz a
escolha acontecer sem pensar.

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

Toda âncora de WhatsApp leva `target="_blank" rel="noopener noreferrer"` e um `aria-label` que diz
o canal — "Agendar avaliação pelo WhatsApp". O rótulo visível sozinho não informa que a página vai
ser trocada por outro app.

Onde a barra sticky irrita mais do que ajuda, o hook `useStickyCta`, a barra com `inert` e
`env(safe-area-inset-bottom)`, e o pareamento de variante por fundo estão em
[cta-and-faq.md](cta-and-faq.md). Em ≥ 1024px não existe sticky: o header já está sempre visível e
uma segunda barra pede a mesma coisa duas vezes.

## FAQ que converte

A primeira pergunta é a **objeção mais cara** — aquela que, sem resposta, faz a aba fechar. Não a
mais fácil de responder, não a que a empresa prefere.

1. Liste tudo que o atendente responde no WhatsApp **antes** de conseguir agendar. Essa lista é o
   FAQ real, e `design/pesquisa.md` já a traz classificada em fatal / adiadora / residual. Não
   invente perguntas.
2. Ordene por custo de abandono: quanto vale o visitante que vai embora por falta da resposta.
3. A pergunta de preço vai em 1º ou 2º, **sempre**.
4. Máximo 7 perguntas. A 8ª é conteúdo de blog.

| Posição | Urgência | Transformação | Credencial | Produto |
|---|---|---|---|---|
| 1 | "Vocês estão abertos agora?" | "Quanto custa?" | "Como vocês cobram?" | "Quanto custa?" |
| 2 | "Quanto custa a emergência?" | "Fica artificial?" | "Quanto tempo leva?" | "Migra meus dados?" |
| 3 | "Preciso agendar ou posso chegar?" | "Dói? Quanto tempo de recuperação?" | "Quem vai me atender?" | "Preciso de time técnico?" |
| 4 | "Onde fica? Tem estacionamento?" | "Quanto tempo dura o resultado?" | "Atende o meu caso?" | "E se eu quiser sair?" |
| 5 | "Aceitam qual pagamento?" | "E se eu não gostar?" | "É sigiloso?" | "Tem plano gratuito?" |

Dentro de cada resposta: **a primeira frase responde**, o resto justifica. Nunca o contrário — quem
lê FAQ está com pressa e abandona no meio da justificativa se a resposta ainda não veio.

Marcação: `<details>` nativo. O teclado já funciona, o `<summary>` já anuncia o próprio estado, e
**a resposta continua no DOM com o item fechado**: Ctrl+F acha, o leitor de tela acha, o Googlebot
acha ao renderizar. Atrás de `{open && …}` ela não existe enquanto está fechada. Componente,
`name="faq"` e o JSON-LD de `FAQPage` em [cta-and-faq.md](cta-and-faq.md).

## Anti-patterns

- **Hero que explica tudo.** O hero é varrido, não lido. Cinco parágrafos ali não são lidos mais
  rápido, são lidos menos — e quem conclui que já viu o argumento não rola.
- **Preço apenas "sob consulta".** O visitante não adia a pergunta, adia a conversa. Uma faixa
  ("a partir de R$ X", "avaliação sem custo") converte melhor que silêncio mesmo quando é alta:
  filtra quem não fecharia e tranquiliza quem fecharia.
- **FAQ com perguntas que ninguém faz.** "Por que escolher a Beleza Completa?" é copy de vendas
  fantasiada de resposta. O visitante reconhece e para de ler — inclusive as reais, logo abaixo.
- **Feature sem consequência.** "Tecnologia moderna" não responde nada. Sem a frase seguinte
  ("equipamentos que oferecem mais precisão, conforto e segurança") o item ocupa espaço e não move
  ninguém. São sempre três: feature → por que importa → resultado.
- **Formulário como conversão principal.** No Brasil, em serviço local, o formulário devolve o
  controle para a empresa ("entraremos em contato") e o visitante perde a garantia de resposta. O
  WhatsApp é assíncrono, tem confirmação de leitura e já está aberto no telefone dele. Use
  formulário só quando houver necessidade legal de registro.
- **Rótulo de botão genérico.** "Saiba mais" e "Clique aqui" não dizem o que acontece depois do
  clique, e são exatamente o que o leitor de tela anuncia fora de contexto ao listar os links.
- **Mesma origem em todos os `wa.me`.** Se toda mensagem chega igual, o atendente recomeça a
  conversa do zero e o único sinal de funil que a página tinha desaparece.
- **Copy escrita antes da estrutura.** Sem `question` definido, o texto vira descrição da empresa
  em vez de resposta à dúvida, e a seção passa a caber em qualquer posição — que é o sintoma de
  que ela não pertence a nenhuma.

## O artefato — `design/landing-blueprint.md`

Um bloco por seção, na ordem de `story.ts`, com todo campo já dentro do teto:

```
## 03 · jornada — beat 5 · share 20,1% · scroll 6 / scrollMobile 4.2
Pergunta: O que exatamente vai acontecer comigo?
EYEBROW  (≤18): Sua jornada
HEADLINE (≤56): Seu atendimento acontece em cinco etapas
LEAD    (≤230): …
CORPO           5 etapas, ≤130 cada
CTA             primário · origem `method` · "Agendar avaliação"
PROVA           processo nomeado — fonte: design-system.json.facts.claims[2]
```

O dev revisa este arquivo, não o `site.ts`. É a Parada do Dev da Fase 6: mudar uma headline aqui
custa cinco minutos; mudar depois da Fase 9 custa uma rodada inteira de geração de imagem, porque
a composição de cada render foi desenhada em volta daquele texto.

## Verificação

```bash
node scripts/check-banned-copy.mjs src/data/site.ts    # sai 0
node -e "const s=require('fs').readFileSync('src/data/site.ts','utf8');const m=s.match(/headline:\s*'([^']*)'/g)||[];m.forEach(h=>console.log(h.length-12,h))"
```

- [ ] Toda headline de hero ≤ 62 caracteres (≤ 48 em Urgência), de capítulo ≤ 56
- [ ] Todo eyebrow ≤ 18 caracteres, sem verbo e sem ponto final
- [ ] Todo rótulo de botão ≤ 24 caracteres e 3 palavras, e nenhum é "Saiba mais"
- [ ] Nenhuma frase banida de `pesquisa.md` na copy; nada reaproveitado do site antigo
- [ ] Todo número citado existe em `design-system.json.facts` com fonte
- [ ] Entre 3 e 6 pontos de conversão renderizados; um primário por tela
- [ ] Toda âncora `wa.me` carrega `?text=` com uma origem distinta embutida
- [ ] Em 375px o primeiro CTA aparece sem rolar (Urgência) ou só após 30% (Transformação)
- [ ] Primeira pergunta do FAQ é preço ou disponibilidade; no máximo 7 perguntas
- [ ] Com todo `<details>` fechado, Ctrl+F acha o texto de qualquer resposta
- [ ] Endereço, horário e telefone existem como texto no DOM, nunca só dentro de imagem
- [ ] Com JavaScript desligado, o `<noscript>` ainda dá nome, endereço e telefone clicável
