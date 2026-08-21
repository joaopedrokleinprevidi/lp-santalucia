---
name: niche-research
description: Use when a landing page needs market grounding before any copy is written — the client's site, Instagram, Google reviews and 3 to 5 local competitors. Fase 4, produz design/pesquisa.md: as frases do setor proibidas na copy, a brecha, as palavras literais das avaliacoes, as objecoes em ordem de custo e o que a regulacao do nicho exige. Palavras-chave: pesquisa de mercado, concorrente, avaliacao do Google, objecao, brecha, FAQ real.
argument-hint: [nome-do-cliente] [cidade] [segmento]
allowed-tools: WebSearch, WebFetch, Read, Glob, Grep, Write, AskUserQuestion, Bash(node *)
---

# Pesquisa de nicho — Fase 4

| | |
|---|---|
| **ENTRADA** | `design/design-system.json` (`brand`, `facts.services`, `facts.claims`, `unverified`), `design/briefing.json` e `design/lacunas.md`; do usuário: URL do site, @ das redes, link do Google Maps |
| **SAÍDA** | `design/pesquisa.md` com seis seções de título fixo, e `scripts/check-banned-copy.mjs` gravado no projeto |
| **ANTES** | `auditoria-dados` (Fase 3) — as lacunas já perguntadas, para não repetir pergunta |
| **DEPOIS** | `creative-direction-expert` (Fase 5) e `estrutura-secoes` (Fase 6a) |

Uma responsabilidade: descobrir **o que a página não pode falar** e entregar a brecha por onde
ela vai falar outra coisa. Nenhuma linha de copy é escrita aqui — isso é `copy-conversao`.

Pesquisar depois de escrever não é a mesma coisa em outra ordem: a copy já nasceu com o ângulo
genérico e a pesquisa vira justificativa dele.

Fato novo descoberto aqui **volta para `design-system.json`** (em `facts` com a fonte, ou em
`unverified`). Não vira dado solto em `pesquisa.md`. Uma fonte da verdade só continua sendo uma
se ninguém abrir a segunda.

## Entrada — uma rodada de perguntas, nunca duas

O dev é leigo e o trabalho dele é trazer dado e colar prompt. Pingar pergunta a pergunta gasta o
tempo dele e o seu. Leia `design/lacunas.md` antes de montar o bloco: item já respondido lá
**não se pergunta de novo**.

Envie de uma vez, com a suposição em itálico ao lado de cada item, e a instrução de que "não
tem" é resposta válida:

| # | O que peço | Suposição se não vier |
|---|---|---|
| 1 | Endereço do site atual | Assumo que não existe site e pulo a frente 1 |
| 2 | @ do Instagram e do Facebook | Busco por nome + cidade e uso o que for público |
| 3 | Os 5 posts com mais interações dos últimos 90 dias, print ou link (Instagram → Painel profissional → Conteúdo → ordenar por interações) | Uso os posts que já estão em `assets-source/` e declaro a amostra como parcial |
| 4 | Link do perfil no Google Maps (app → Compartilhar → Copiar link) | Peço as 10 avaliações mais recentes coladas como texto |
| 5 | Os 3 concorrentes que o cliente cita de cabeça | Pego os 5 mais avaliados do mesmo serviço num raio de 5 km |
| 6 | Faixa de preço praticada e a lista do que ele **não** faz | A página não cita preço e evita serviço que eu não veja num asset |
| 7 | Registro profissional / responsável técnico (CRMV, CRO, CREF, CRECI…) | A página não exibe registro — e alguns nichos não podem publicar sem ele |

Os itens 3 e 4 estão no bloco em vez de na sua lista de buscas porque **não existem sem login**:
alcance, salvamentos e engajamento vivem no painel do dono, a maioria dos perfis não renderiza
por fetch anônimo, e a lista de avaliações do Maps é montada por JavaScript.

## Escopo: passada completa ou curta

| Condição | Passada |
|---|---|
| Cliente sem diferencial escrito, ou nicho com regulação (saúde, jurídico, veterinária, alimentação, financeiro) | **Completa** — as 5 frentes, 3 a 5 concorrentes, 25 a 40 buscas/fetches |
| Diferencial já documentado **e** `facts.claims` com ≥2 números verificáveis **e** nicho sem conselho profissional | **Curta** — frentes 3 e 4 apenas, 2 concorrentes, ≤8 buscas |
| Reposicionamento, cidade nova, ou "não sei o que me diferencia" | **Completa, sem corte** |

Nunca zero. Duas buscas — as avaliações do cliente e a home de dois concorrentes — são o piso
absoluto: "palavras do cliente" e "o que todo mundo diz" são as duas únicas seções que não podem
ser deduzidas. Ou você leu, ou você inventou.

"O nicho é óbvio" não é critério de pulo. O óbvio é exatamente o que todo mundo do setor está
dizendo agora — ou seja, o conteúdo da lista de proibições.

## As cinco frentes

Queries literais, comportamento real de cada fonte e os fallbacks estão em
[fontes.md](fontes.md). Aqui fica o método e o que se extrai.

### 1. O site atual — extrair fato, nunca redação

Três `WebFetch` bastam: home, serviços, contato.

| Extraia | Descarte |
|---|---|
| Serviços, preços publicados, horários, endereço, registro, nomes, ano de fundação | Toda e qualquer frase pronta |
| Onde estão os CTAs e qual canal eles usam | O tom, a headline, a ordem das seções |
| O que o cliente promete em público (vira `unverified` se não bater com os assets) | "Sobre nós", "missão e visão" |

**Ler é para saber o fato. A redação é lixo tóxico.** Melhorar a frase do site velho mantém o
ângulo do site velho — e o site velho é o motivo de existir este projeto.

Sinais de desatualização valem registro, porque cada um é uma objeção que a página nova derruba
de graça: copyright anterior ao ano passado, telefone sem o nono dígito, aviso de pandemia,
tabela de preço com data antiga, link de rede quebrado, ausência de WhatsApp, "em breve".

### 2. Instagram e Facebook — a voz real e o FAQ escondido

Quatro perguntas, nesta ordem:

1. **Qual serviço ele mais anuncia?** Conte posts por serviço nos últimos 30. O mais anunciado
   paga a conta, e costuma não ser o que o site coloca em primeiro lugar.
2. **Qual post engajou mais e por quê?** O formato importa mais que o assunto: antes/depois,
   preço declarado, bastidor, resposta a dúvida, humor. O formato campeão é pista de qual seção
   merece o WOW.
3. **Qual é a voz real dele?** Emoji ou não, "você" ou "vocês", apelido do cliente. Se divergir de
   `voice` no `design-system.json`, o JSON continua mandando nos tokens e a pesquisa registra a
   divergência com exemplo — voz de post fala com quem já segue.
4. **O que perguntam nos comentários?** É o FAQ real, escrito por quem tinha a dúvida. Pergunta
   repetida ≥3 vezes entra no FAQ obrigatoriamente.

### 3. Avaliações no Google — as palavras que o visitante já usa

Leia as 15 mais recentes e **todas** as de 1 e 2 estrelas dos últimos 12 meses. Registre nota,
total e quantas negativas.

| O que você lê | O que vira |
|---|---|
| Elogio, com as palavras do avaliador | Copy: o substantivo dele entra na headline ou no corpo |
| Reclamação | Objeção a derrubar na página, com resposta no FAQ |
| Pergunta ou confusão ("achei que fosse…") | Pergunta de FAQ, literal |
| Elogio ao mesmo detalhe por ≥3 pessoas | Candidato a brecha — diferencial que o cliente nem sabe que tem |

**Cite literal**, com o erro de digitação e o regionalismo: o valor da citação é ser a palavra
que o visitante já usa, e parafrasear devolve a frase para o nosso vocabulário. Mínimo de **8
citações, sendo ≥2 negativas**. Sem negativa, você coletou marketing, não pesquisa.

Nome completo e foto do avaliador **não entram** — depoimento identificado exige autorização
escrita, registrada por `briefing-cliente` em
`briefing.json.socialProof.testimonials[].authorized`. O texto público, sem identificação, entra.

Reclamação nunca vira silêncio. "Reclamaram do preço, então não falamos de preço" deixa a objeção
mais cara sem resposta, e o visitante vai perguntar no WhatsApp do concorrente.

### 4. Concorrentes — 3 a 5, mesma cidade

Menos de 3 e não dá para dizer "todo mundo diz": dois iguais é coincidência. Mais de 5 e o sexto
repete as mesmas afirmações — você gasta contexto e o resultado não muda.

Escolha os que o cliente citou (item 5) mais os mais avaliados do mesmo serviço num raio de 5 km.
Máximo 3 fetches por concorrente: home, uma página de serviço, bio do Instagram. Uma linha por
concorrente, com a **data em que foi lido** — preço e horário mudam em semanas, e uma tabela sem
data não se sabe se ainda vale.

"Promessa principal" é o `<h1>` da home mais o rótulo do primeiro CTA; sem site, a bio do
Instagram. Ela é **evidência**, não texto reutilizável: só aparece na tabela e na lista de banidas.

Depois, a contagem que produz a seção mais útil do arquivo — o procedimento de normalização está
em [fontes.md](fontes.md#contagem-de-afirmações). **Afirmação de valor que aparece em ≥3 de 5 vai
para "O que todo mundo diz" e está proibida na nossa copy.** Fato operacional (horário, endereço,
forma de pagamento) não entra na contagem: só promessa.

Registre também o **baseline visual** em três linhas: todos usam foto de banco? todos a mesma
paleta do setor? todos abrem com carrossel? Surpresa é desvio de uma média, e a média está aqui.

### 5. Exigências do nicho

Tabela por nicho, com a busca de confirmação de cada norma, em [nichos.md](nichos.md). Regras que
valem para todos:

- **Publicidade obriga.** O que a página promete vira obrigação contratual (CDC, arts. 30 e 35).
  Preço anunciado é preço a cumprir; prazo anunciado é prazo a cumprir.
- **Superlativo exige prova.** "O melhor da cidade" sem substanciação é peça derrubável pelo
  CONAR. Número verificável ou nada.
- **Conselho profissional muda o que pode ser dito**, não só o que deve ser exibido: saúde não
  promete desfecho, advocacia não promete resultado nem capta clientela, veterinária precisa de
  responsável técnico registrado, alimentação precisa de alergênico declarado.
- Norma muda. Confirme a resolução vigente e registre a data da consulta. Onde houver dúvida a
  saída é `unverified`, não uma aposta.

## Objeções: classificar antes de ordenar

"Ordem do mais caro para o mais barato" precisa de uma definição de caro. Custo = o valor do
visitante que vai embora por falta daquela resposta.

| Classe | Definição | Onde ela morre na página |
|---|---|---|
| **Fatal** | Sem resposta o visitante fecha a aba e não volta: preço fora da faixa, "não atende meu caso", distância, horário incompatível | Antes de 40% do scroll, ou posição 1 do FAQ |
| **Adiadora** | Ele não fecha, adia — e adiado é perdido: como agenda, precisa marcar, tem estacionamento, quanto demora | FAQ, posições 2 a 5 |
| **Residual** | Só aparece em quem já decidiu: forma de pagamento, o que levar, onde estacionar | Seção de contato ou rodapé |

Escreva 5 a 7 objeções, cada uma com a fonte (avaliação negativa, comentário, ausência na
concorrência) e a classe. A ordem delas **é** a ordem de derrubada que `estrutura-secoes` usa
para sequenciar seções e FAQ.

## A brecha: três testes

Brecha é o que ninguém no mercado local está dizendo e este cliente pode dizer. Só é válida se
passar nos três:

1. **Ninguém diz** — não aparece na home nem na bio de nenhum dos 3 a 5 concorrentes.
2. **O cliente prova** — existe fato correspondente em `design-system.json.facts`, com fonte.
   Falhou aqui, vira `unverified` e **não pode virar headline**.
3. **Responde a uma objeção** da lista acima. Brecha que não derruba medo é curiosidade.

| Tipo de brecha | Como se prova | Exemplo de forma |
|---|---|---|
| Operacional | horário, tempo de resposta medido, raio de atendimento | "Resposta no WhatsApp em até 10 minutos, das 9h às 18h" |
| De prova | número, registro, anos, volume | "Desde 2016", "Dra. Nome, CRO-MG 00000" |
| De escopo | atende um caso que os outros recusam | o caso, nomeado |
| De método | uma etapa do processo que os outros não nomeiam | a etapa, nomeada |
| De risco | política clara de retorno, reagendamento, garantia | a política, escrita |

Um adjetivo nunca é brecha. "Atendimento humanizado" qualquer concorrente publica na terça-feira
sem mudar nada no negócio — e três deles já publicaram.

## O artefato: design/pesquisa.md

Seis seções, **com estes títulos exatos**, porque as skills seguintes fazem `Grep` por eles e o
script anti-plágio corta o arquivo por `## O que todo mundo diz`:

```
## Concorrentes          tabela: nome | promessa (H1 literal) | preço | faz melhor | não faz | lido em
## O que todo mundo diz  bullets "<frase literal>" — aparece em <n>/<total>. É lista de PROIBIÇÃO
## A brecha              o que ninguém diz + o fato que prova + a objeção que derruba + os 3 testes
## Palavras do cliente   "<citação literal>" — <n>★, <mês/ano> · uso: headline | corpo | FAQ
## Objeções              1. <objeção> — fatal|adiadora|residual · fonte: <onde> · derrubar em: <seção>
## Exigências do nicho    tabela + subseção "### Não pode prometer"
```

Cabeçalho na primeira linha:
`<!-- Passada: completa|curta · Período: <datas> · Fontes lidas: <n> -->`.

Modelo completo, com uma linha de exemplo preenchida em cada tabela, em [template.md](template.md).

## Anti-plágio

Pesquisar concorrente serve para saber o que **não** repetir. Copiar frase ou estrutura é o
caminho mais curto para a landing genérica que o CLAUDE.md proíbe — e é pior que não pesquisar,
porque produz uma cópia com convicção.

- **Frase** entra só na tabela de concorrentes e na lista de banidas. Nunca em outro lugar.
- **Estrutura** não se copia nem do primeiro colocado: o ranking dele mede idade de domínio e
  links, não ordem de seção. Você importa a forma sem a causa.
- **Se 4 de 5 usam Hero → Serviços → Sobre → Contato**, essa ordem é exatamente o que os torna
  intercambiáveis. É motivo para não usar.

O gate é um script porque roda duas vezes — aqui como linha de base, e no portão da Fase 6 contra
a copy final. Grave-o em `scripts/check-banned-copy.mjs`: código completo e o porquê do limiar de
4 palavras em [anti-plagio.md](anti-plagio.md).

## Handoff

| Saída | Quem consome | O que vira lá |
|---|---|---|
| A brecha | `estrutura-secoes` → `copy-conversao` | o ângulo do hero e a headline |
| Objeções, na ordem e com a classe | `estrutura-secoes` | ordem das seções e do FAQ; fatal define o que vem antes de 40% |
| Palavras do cliente | `copy-conversao` | vocabulário da copy e perguntas do FAQ, literais |
| Preço do setor | `copy-conversao` | a faixa que responde a pergunta 1 do FAQ |
| O que todo mundo diz | todos, e o portão da Fase 6 | lista de proibição verificada por script |
| Baseline visual do setor | `creative-direction-expert` | o desvio dessa média = onde gastar o WOW major |
| Formato de post campeão | `creative-direction-expert` | pista de qual capítulo merece o momento memorável |
| Exigências do nicho | `copy-conversao` + `product-design-expert` | o que a página tem que exibir e onde |
| Não pode prometer | `copy-conversao` | restrição de copy, checada no portão |
| Fato novo descoberto | `brand-dna-extractor` | volta para `design-system.json` em `facts` ou `unverified` |

A pesquisa não é copiada para dentro do blueprint. É referenciada — uma objeção reclassificada
aqui propaga sozinha em vez de virar duas versões do mercado.

## Anti-patterns

- **Melhorar a frase do site antigo** — a frase melhorada carrega o ângulo original, então a
  página nova nasce com o posicionamento que já não converteu. Extraia fato; escreva do zero.
- **Copiar a estrutura do concorrente melhor colocado** — o ranking vem de domínio antigo e
  backlinks, não da ordem das seções.
- **Pesquisar 12 concorrentes** — depois do quinto as afirmações se repetem, o custo em contexto
  sobe e a lista de banidas não muda.
- **Parafrasear a avaliação** — o valor da citação é ser a palavra que o cliente usou. Reescrita,
  ela vira a nossa palavra de novo e a pesquisa não trouxe nada.
- **Transformar reclamação em tabu** — a objeção mais cara fica sem resposta e migra inteira para
  o WhatsApp do concorrente.
- **Declarar brecha sem o fato que a prova** — promessa que o cliente não sustenta é risco
  jurídico e cancelamento na primeira semana. Sem prova, é `unverified`.
- **Tabela de concorrente sem data** — seis meses depois ninguém sabe se a coluna ainda vale, e a
  decisão foi tomada em cima dela.
- **Nome e foto de avaliador na página** — depoimento identificado exige autorização escrita.
- **Terminar sem nenhuma frase banida** — significa que você leu os concorrentes procurando
  inspiração em vez de repetição. Refaça a contagem.

## Verificação

```bash
node scripts/check-banned-copy.mjs   # linha de base: lista ≥3 frases (o portão real é na Fase 6)
node -e "const t=require('fs').readFileSync('design/pesquisa.md','utf8');['Concorrentes','O que todo mundo diz','A brecha','Palavras do cliente','Objeções','Exigências do nicho'].forEach(h=>{if(!t.includes('## '+h))throw new Error('falta a seção: '+h)});console.log('seções ok')"
```

- [ ] `design/pesquisa.md` existe com os seis títulos exatos
- [ ] 3 a 5 concorrentes, cada linha com a data de leitura e o H1 literal
- [ ] Lista de banidas com ≥3 frases, cada uma com a contagem `<n>/<total>`
- [ ] ≥8 citações literais de avaliação, sendo ≥2 negativas, sem nome de avaliador
- [ ] A brecha traz os três testes marcados e aponta o fato de `facts` que a prova
- [ ] 5 a 7 objeções, cada uma classificada em fatal / adiadora / residual, com fonte
- [ ] Exigências do nicho com fonte (conselho, lei ou expectativa) e data de consulta
- [ ] "Não pode prometer" preenchido — mesmo em nicho sem conselho, o CDC entra
- [ ] Baseline visual do setor em três linhas, para a direção criativa
- [ ] Todo fato novo devolvido a `design-system.json` em `facts` ou `unverified`
- [ ] Nenhuma frase de concorrente aparece fora da tabela e da lista de banidas
