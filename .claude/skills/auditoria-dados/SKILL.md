---
name: auditoria-dados
description: Use when the briefing, the asset inventory and the design system already exist and you must audit what is still missing or contradictory before any copy is written. Fase 3 do pipeline: monta os blocos bloqueia, importante e opcional com a pergunta exata em portugues de leigo e a suposicao padrao de cada uma, detecta conflito entre fontes, e lista o que nunca se inventa. Uma rodada unica. Produz design/lacunas.md.
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion, Bash(node *)
---

# Auditoria de dados — Fase 3

| | |
|---|---|
| **ENTRADA** | `design/briefing.json`, `design/inventario.json`, `design/design-system.json` |
| **SAÍDA** | `design/lacunas.md` — e os blocos `gaps`, `conflicts`, `assumptions` e `unverified` gravados em `briefing.json` |
| **ANTES** | `brand-dna-extractor` (Fase 2) |
| **DEPOIS** | `niche-research` (Fase 4), e daí em diante todo o pipeline |

Esta é a **parada do dev**. Ele responde uma vez e volta a não fazer nada até a revisão da copy.

Agora, e só agora, as perguntas existem — porque só agora se sabe o que os assets já responderam.
**Perguntar antes de ler é o erro mais caro do pipeline:** faz o dev ir atrás do cliente para buscar
um dado que estava na placa da fachada, e queima a única rodada que temos.

## A comparação

Percorra os três níveis de `briefing-cliente` (BLOQUEIA, PIORA, ENRIQUECE) e, campo por campo,
classifique:

| Estado do campo | Vai para |
|---|---|
| Tem `value` e `source` de asset ou doc | Nada. Está resolvido |
| Tem `value` com `source: "web:"` e é contato, horário ou preço | IMPORTANTE — pergunta de confirmação, porque `web:` é o dado que ninguém atualizou desde 2019 |
| `value: null` e está no Nível 1 | BLOQUEIA |
| `value: null` e está no Nível 2 | IMPORTANTE, com suposição adotada |
| `value: null` e está no Nível 3 | OPCIONAL, com suposição adotada |
| Duas fontes afirmam coisas incompatíveis | BLOQUEIA, como **conflito** — nunca como lacuna |

Também vira pergunta: todo `inventario.json.assets` com `usable: false` cujo `reason` seja
resolvível pelo cliente (logo em baixa, foto com marca d'água, HEIC), e toda entrada de
`inventario.json.contradictions`.

Grave o resultado em `briefing.json.gaps` com as três chaves `blocking`, `important` e `optional`, e
escreva `design/lacunas.md` com a redação completa.

## Conflito não é lacuna

Uma **lacuna** é ausência: ninguém afirmou nada, então adoto a leitura mais conservadora, declaro a
suposição e sigo. Um **conflito** é presença dupla e incompatível: qualquer escolha que eu faça
publica uma afirmação que outra fonte do próprio cliente nega. Não existe leitura conservadora — só
existe escolher em quem apostar.

Cinco tipos aparecem quase sempre:

| Tipo | Exemplo real |
|---|---|
| Contradição direta | O post anuncia "especialistas em pets não convencionais, 24 h"; o FAQ do site diz "foco principal em cães e gatos, consulte-nos para outras espécies" |
| Aritmética que não fecha | O site diz "10+ anos"; a fundação registrada é 2015, o que dá 11 em 2026 |
| Contato divergente | O telefone da placa não é o telefone do Google Business |
| Horário divergente | O post antigo diz "seg a sáb"; o Google diz "24 horas, todos os dias" |
| Promessa de infraestrutura | O site promete estacionamento; a foto da fachada mostra vaga na calçada pública |

O caso dos pets não convencionais mostra o custo de cada lado. Publicar o post: um tutor de coelho
dirige até lá às três da manhã e não é atendido. Publicar o FAQ: o cliente perde o diferencial mais
forte que tem, porque a página nega um serviço que ele presta. Uma linha de pergunta resolve;
nenhuma das duas apostas se conserta depois de publicada.

**Regra:** a fonte mais recente e mais específica vira a **hipótese registrada**, nunca a decisão.

```json
{ "subject": "atendimento a pets não convencionais",
  "sourceA": "asset:post-atendimento-24h-pets-nao-convencionais.jpg",
  "claimA": "Especialistas em pets não convencionais, atendimento 24h",
  "sourceB": "doc:conteudo-textual-site.md",
  "claimB": "Foco principal em cães e gatos; consulte-nos para outras espécies",
  "hypothesis": "O post é mais recente e mais específico; a hipótese é que o FAQ está desatualizado.",
  "cost": "Se a página afirmar e for falso, um tutor dirige até lá de madrugada e não é atendido.",
  "resolved": false }
```

`resolved: false` **não impede o avanço de fase — impede a publicação.** O pipeline continua; o
`vercel --prod` não. Carregue a pendência visível até a Fase 12.

A pergunta de conflito tem quatro partes e nenhuma é dispensável: as duas fontes citadas
literalmente, a pergunta direta, o custo concreto de errar, e o que acontece enquanto não há
resposta. Sem a terceira parte, o dev trata como detalhe e não repassa. Modelo pronto em
[lacunas-e-perguntas.md](lacunas-e-perguntas.md#conflitos).

## Dado que nunca se inventa

Nenhum destes entra em `facts` sem `source`. Não existe telefone aproximado.

| Dado | O que acontece se estiver errado |
|---|---|
| Telefone | Toca na casa de um estranho. A LP não converte e ninguém descobre por semanas |
| WhatsApp | O botão principal da página abre conversa com desconhecido |
| Endereço e CEP | O mapa e o JSON-LD mandam o cliente para a quadra errada |
| CNPJ | Informação societária falsa no rodapé de um negócio real |
| Registro profissional (CRMV, CRO, CREA, OAB…) | Número inventado aparenta exercício irregular. Quem fiscaliza é o conselho, e a multa é do cliente |
| Preço | Preço publicado é oferta. O cliente é obrigado a honrar ou a desmentir a própria página |
| Horário | "24 h" onde não é 24 h coloca gente na porta às três da manhã |
| Nome de pessoa | Nome e foto sem autorização é uso indevido de imagem |
| Nota e nº de avaliações | Número inflado é publicidade enganosa, e se verifica em um clique |

Sem `source`, o dado vai para `unverified` e vira pergunta do bloco BLOQUEIA. Nunca para `facts`.

Nestes nove, a opção de saída da pergunta existe, mas o efeito dela é **adiar a publicação**, não
adotar um valor. A descrição precisa dizer isso com estas palavras: `"Sem esse dado, o site não pode
ir ao ar."`

## A rodada única

Quatro regras:

1. **Junte tudo antes de perguntar qualquer coisa.** Os três blocos, numa rodada. Pergunta pingada
   obriga o dev a incomodar o cliente cinco vezes, e na terceira ele para de responder.
2. **Toda pergunta vem com o padrão que será adotado se não houver resposta.** "Não sei" tem que ser
   resposta válida e sem custo — o dev é leigo e frequentemente não sabe mesmo.
3. **Nunca pergunte o que está num asset.** Se está na placa, você já tem.
4. **Português de leigo.** "Qual o número do WhatsApp comercial, com DDD?" — não "confirmar o
   endpoint de conversão".

| Não escreva | Escreva |
|---|---|
| "Validar o `openingHoursSpecification`" | "Abre no feriado? Em qual feriado fecha?" |
| "Assets em resolução insuficiente" | "O logo que você mandou está pequeno. Existe o arquivo original, do designer? Termina em `.svg`, `.ai` ou `.pdf`" |
| "Definir a estratégia de prova social" | "Quantas avaliações a empresa tem no Google e qual a nota?" |

A redação exata de cada pergunta, por campo, com a coluna "se não responder" que vira
`assumptions`: [lacunas-e-perguntas.md](lacunas-e-perguntas.md#banco-de-perguntas). Copie a coluna
literalmente — ela já foi escrita para leigo.

### Sessão interativa

Use `AskUserQuestion`. Ela aceita até 4 perguntas por chamada, `header` de até 12 caracteres e 2 a 4
opções. Agrupe por tema e dispare as chamadas **em sequência, sem trabalho entre elas** — do ponto
de vista do dev é uma rodada só.

| Chamada | Header | Tema |
|---|---|---|
| 1 | `Contato`, `WhatsApp`, `Endereço`, `Horário` | os dados que travam a publicação |
| 2 | `Serviços`, `Não faz`, `Conflito`, `Preço` | o que a página pode e não pode prometer |
| 3 | `Domínio`, `Redes`, `Google`, `Jurídico` | SEO, prova social, CNPJ e registro |
| 4 | `Assets`, `Vídeo`, `Vetos`, `Referência` | material e gosto |

Em toda pergunta, uma das opções é a saída: `"Não sei — decida por mim"`, com a descrição dizendo
qual suposição será adotada. Pergunta sem essa saída trava o pipeline num dev que não tem a resposta.

### Sessão não interativa

Não pergunte. Três coisas, nesta ordem: escreva o bloco completo em `design/lacunas.md` com as
mesmas perguntas; adote a suposição de cada campo, marque `assumed: true` no `briefing.json` e
registre em `assumptions` com `reason` e `reversible`; siga. Nada trava, exceto a publicação — que
continua bloqueada por qualquer `conflicts[].resolved: false` ou por dado ausente da tabela acima.

Parar um pipeline não interativo à espera de resposta é o pior dos dois mundos: não pergunta e não
entrega.

| Suposição permitida | Suposição proibida |
|---|---|
| Tom de voz, ordem das seções, paleta de apoio, proporção de imagem | Qualquer linha da tabela "nunca se inventa" |
| Quantidade de seções, arquétipo de estrutura | Serviço que a empresa oferece |
| Que o horário do Google é o vigente, **se for a única fonte** | Que o horário do Google é o vigente, **se um asset disser outro** — isso é conflito |

## O artefato: `design/lacunas.md`

Três blocos. Cada item traz a pergunta exata, o motivo em uma frase e a suposição adotada enquanto
não há resposta. É o arquivo que o dev leva para a conversa com o cliente, então não pode ter jargão
nenhum.

```markdown
# Lacunas — Santa Lúcia

Gerado em 2026-08-20, depois de ler 9 assets e 2 documentos.
São 12 perguntas, todas de uma vez. Responda o que souber; o resto tem um padrão adotado.

## BLOQUEIA — sem isso a página não vai ao ar

### 1. WhatsApp
**Pergunta:** Qual o número do WhatsApp que a empresa atende, com DDD? É o mesmo do fixo?
**Por que:** É o botão principal da página. Sem ele, o CTA não existe.
**Enquanto não responder:** o botão fica desativado e a publicação está travada.

### 2. Conflito — pets não convencionais
**Pergunta:** <bloco de conflito completo: as duas fontes, a pergunta, o custo, o que acontece>
**Fontes:** `post-atendimento-24h-pets-nao-convencionais.jpg` × `docs/conteudo-textual-site.md`
**Enquanto não responder:** a página não menciona o serviço.

## IMPORTANTE — a página fica pior sem isso

### 3. Domínio
**Pergunta:** A empresa já tem endereço de site comprado?
**Por que:** define o endereço final, o link que aparece no WhatsApp e o cadastro no Google.
**Suposição adotada:** `santa-lucia.vercel.app`. Reversível, com custo de reindexação.

## OPCIONAL — melhora, não trava

### 9. Vídeos
**Pergunta:** Tem algum vídeo do local, mesmo curto?
**Por que:** vídeo real vira a seção mais forte da página.
**Suposição adotada:** página só com imagens.

---

## Suposições adotadas (resumo)

| Campo | Adotei | Reversível? |
|---|---|---|
| domínio | `.vercel.app` | sim, com custo de reindexação |
| preço | sem preço; CTA "falar no WhatsApp" | sim |
```

`lacunas.md` some quando o bloco BLOQUEIA esvazia. Enquanto houver conflito com `resolved: false`, a
publicação está travada mesmo que tudo o mais esteja pronto.

## Portão

- [ ] `node -e "JSON.parse(require('fs').readFileSync('design/briefing.json','utf8'))"` sai limpo
- [ ] `design/lacunas.md` existe, com os três blocos, e nenhuma frase técnica dentro dele
- [ ] Todo item de `gaps.blocking` tem pergunta correspondente escrita em `lacunas.md`
- [ ] Toda pergunta tem a linha "enquanto não responder" ou "suposição adotada" preenchida
- [ ] Nenhum dado da tabela "nunca se inventa" está em `facts` sem `source`
- [ ] Todo conflito está em `conflicts` com `hypothesis`, `cost` e `resolved: false`
- [ ] Todo `assumed: true` no briefing tem entrada correspondente em `assumptions`
- [ ] Bloco BLOQUEIA vazio, **ou** as perguntas já foram feitas em uma rodada só

Verificação dos conflitos abertos, que a Fase 12 vai repetir:

```bash
node -e "const b=JSON.parse(require('fs').readFileSync('design/briefing.json','utf8'));const c=(b.conflicts||[]).filter(x=>!x.resolved);console.log(c.length?'aberto: '+c.map(x=>x.subject).join(', '):'conflitos ok')"
```

## Anti-patterns

- **Perguntar antes de ler os assets** — queima a única rodada com coisas que estavam na placa da
  fachada, e ensina o dev que responder não adianta.
- **Pingar pergunta a pergunta** — cada rodada é um telefonema do dev para o cliente. Na terceira, o
  cliente para de atender.
- **Resolver conflito por conta própria** — escolher a fonte "mais provável" publica uma afirmação
  que uma fonte do próprio cliente nega. É o único caso em que perguntar é obrigatório.
- **Promover inferência a fato** — no `briefing.json` a diferença é uma chave; na página publicada é
  uma promessa falsa sobre um negócio real.
- **Pergunta sem opção de saída** — um dev que não sabe a resposta trava o pipeline inteiro num
  campo opcional.
- **Escrever a pergunta em jargão** — o dev repassa a pergunta ao cliente. Se ele não entende, ele
  não pergunta, e a lacuna volta na Fase 12.
- **Travar um pipeline não interativo esperando resposta** — declare a suposição, marque
  `reversible`, siga, e deixe a pergunta escrita.
- **Tratar conflito como lacuna** — lacuna tem leitura conservadora; conflito não tem nenhuma. Uma
  suposição em cima de conflito é uma aposta silenciosa.

## Handoff

`design/lacunas.md` é lido pelo dev. `briefing.json.gaps`, `conflicts` e `unverified` são lidos por
`niche-research` (que pode fechar uma lacuna lendo o Google Business e devolve o fato ao briefing),
`copy-conversao` (nada que esteja em `unverified` ou em conflito aberto vira copy),
`prompt-imagem` (não gera imagem que afirme fato não confirmado) e por `publicar-lp`, que trata
`resolved: false` como bloqueio de deploy.

Fato novo descoberto em qualquer fase **volta** para o JSON de origem, nunca vira dado solto no
artefato da fase.
