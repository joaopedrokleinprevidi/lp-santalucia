---
name: skill-builder
description: Use when creating a new skill, rewriting or auditing an existing one, splitting an oversized one, or deciding which skill owns a fact. Cria, audita e divide skills do Claude Code: frontmatter, description dentro do orcamento, contrato ENTRADA/SAIDA/ANTES/DEPOIS, SKILL.md de 150 a 300 linhas, arquivo de apoio e fronteira entre skills irmas. Palavras-chave: criar skill, auditar skill, dividir skill, description que nao dispara.
argument-hint: [nome-da-skill | caminho/para/SKILL.md]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(ls *), Bash(wc *)
---

# Skill Builder

| | |
|---|---|
| **ENTRADA** | O pedido (que decisão a skill nova detém) ou o caminho de um `SKILL.md` existente; `ls .claude/skills/`; o [CLAUDE.md](../../../CLAUDE.md) |
| **SAÍDA** | `.claude/skills/<nome>/SKILL.md` mais os arquivos de apoio dele — ou, no modo auditoria, o relatório com reprovações e veredito |
| **QUEM CHAMA** | O dev ou eu, fora do pipeline: quando falta uma decisão que nenhuma das 24 detém, quando um `SKILL.md` passa de 300 linhas, ou quando duas skills discordam sobre quem é dona de um número. Nunca uma fase — esta skill não roda dentro de um projeto de landing |
| **DEPOIS** | Nenhuma. A skill produzida entra no pipeline na camada dela e declara o próprio ENTRADA/SAÍDA/ANTES/DEPOIS — ver [fronteira por camada](#fronteira-por-camada) |

Uma skill é um SOP executável. Ela não existe para explicar um assunto — existe para que a mesma
tarefa produza o mesmo resultado na terça e na sexta, com um modelo diferente e um contexto
diferente.

O critério é **densidade operacional**: cada linha ou é um número, ou é código, ou é uma decisão
com causa. Prosa que não muda o output é peso morto que consome contexto.

Referência técnica de frontmatter, invocação, `allowed-tools`, `context: fork`, hooks e
troubleshooting: [reference.md](reference.md). Leia antes de configurar qualquer campo.

## Arquitetura: o conjunto é um algoritmo, não uma pilha

Cinco regras duras. Uma skill que viola qualquer uma delas é reescrita, não remendada.

| Regra | Número | Causa |
|---|---|---|
| Tamanho | `SKILL.md` entre **150 e 300 linhas** | Abaixo de 150 a skill não tem decisão própria e é um capítulo de outra. Acima de 300 o miolo perde peso na atenção do modelo e o detalhe deixa de ser lido |
| Responsabilidade | **uma** decisão por skill | Duas decisões na mesma skill não podem ser executadas em ordens diferentes, e é justamente a ordem que o pipeline precisa poder mudar |
| Contrato | **ENTRADA / SAÍDA / ANTES / DEPOIS** numa tabela no topo, antes de qualquer prosa | É o que faz a saída de uma skill ser a entrada da próxima. Sem isso o conjunto lê como documentação, não como algoritmo |
| Description | **≤450 caracteres** | Todas as descriptions somadas entram no contexto com teto de ~16.000 chars. Com ~24 skills, uma description longa **apaga a de outra skill** — e o corte é silencioso |
| Apoio | Passou de 300 linhas, o excedente vai para arquivo irmão referenciado | Detalhe consultado às vezes não pode custar contexto sempre |

O contrato é literalmente esta tabela, com estes quatro rótulos em maiúsculas:

```markdown
| | |
|---|---|
| **ENTRADA** | arquivos e dados que ela lê, com caminho real |
| **SAÍDA** | o arquivo ou artefato que ela produz, com caminho real |
| **ANTES** | a skill que roda antes, e o que ela já fixou |
| **DEPOIS** | a skill que roda depois, e o que ela vai consumir daqui |
```

**Skill fora do trilho linear troca `ANTES` por `QUEM CHAMA`.** Quatro das 24 não têm fase
própria — `skill-builder`, `video-to-website`, `gsap-scrolltrigger-expert`, `motion-ui-expert` —
porque são chamadas por roteamento, não por número de fase. Escrever `ANTES: Nenhuma` nelas
esconde a informação que importa: quem dispara, e sob qual condição. `QUEM CHAMA` nomeia a skill
roteadora e o gatilho.

`ANTES` ou `DEPOIS` vazio é resposta válida — escreva "Nenhuma" e o motivo. Vazio silencioso é
uma skill órfã que ninguém sabe quando chamar.

Um contrato só fecha se a **SAÍDA** de quem ela cita em `DEPOIS` aparece como **ENTRADA** de lá,
com o mesmo caminho literal. Duas falhas simétricas, as duas fatais: artefato que alguém lê e
ninguém escreve trava a fase na hora de rodar; artefato escrito que ninguém lê é trabalho que o
pipeline paga e joga fora. Confira nos dois sentidos antes de dar a skill por pronta.

Conhecimento que Claude precisa **sempre** é CLAUDE.md; **ao fazer X**, é skill; detalhe
consultado às vezes é arquivo de apoio; válido para um arquivo só é comentário no código — a
tabela completa está em [reference.md](reference.md#claudemd-ou-skill). Antes de criar, rode
`ls .claude/skills/` e leia a `description` das existentes: se uma cobre 70% do escopo,
**estenda** em vez de criar. Duas skills sobrepostas divergem em três meses e ninguém sabe qual
está certa.

---

## Modo 1 — criar skill nova

### Passo 0: infira antes de perguntar

Esta skill roda em sessões autônomas. Um interrogatório de 6 rodadas trava o trabalho. A regra:
**se dá para inferir do repositório ou do pedido, infira, execute, e declare a suposição no fim.**

| Pergunta | Onde inferir | Bloqueante? |
|---|---|---|
| O que a skill decide | Do pedido e do material de origem | Não |
| Nome | Do escopo — minúsculas, hífens, ≤64 chars, igual ao diretório | Não |
| Gatilho / palavras-chave | Do vocabulário do pedido + das descriptions das vizinhas | Não |
| ENTRADA / SAÍDA | Dos caminhos reais do repo — `design/`, `src/data/`, `public/`, `src/generated/` | Não |
| ANTES / DEPOIS | Da camada em que a decisão cai, na tabela de fronteira abaixo | Não |
| Passo a passo | Do código que faz a tarefa hoje (`src/`, `scripts/`) | Não |
| Guardrails | Dos anti-patterns já documentados nas irmãs e no CLAUDE.md | Não |

Só três coisas bloqueiam de verdade, porque nenhuma leitura de repositório resolve: **destino
externo com custo ou efeito irreversível** (chave, envio, deploy, geração paga); **escolha de
produto sem sinal no repo**, quando duas arquiteturas plausíveis levam a skills incompatíveis; e
**contradição entre o pedido e o CLAUDE.md** — pergunte qual vence, não escolha sozinho.

Fora disso, prossiga. Corrigir uma suposição errada custa uma edição; travar a sessão custa a
sessão inteira. Ao terminar, feche com um bloco `## Suposições` — uma linha por pergunta não
feita, cada uma nomeando a razão e o que fazer se a suposição estiver errada. Modelo em
[reference.md](reference.md#formato-do-bloco-de-suposições).

### Passo 1: tipo e frontmatter

| Tipo | O que faz | Frontmatter típico |
|---|---|---|
| **Tarefa** | Executa um fluxo com início e fim | `argument-hint`, às vezes `allowed-tools` |
| **Referência** | Injeta conhecimento que Claude aplica ao trabalho corrente | só `name` + `description` |
| **Fase do pipeline** | Detém uma decisão numerada e um artefato | `description` com o número da fase e o artefato |

Skill de referência **nunca** leva `context: fork` — o subagente recebe diretrizes sem tarefa e
devolve vazio.

- `name` — igual ao nome do diretório. Em skill de projeto o comando vem do **diretório**; o
  `name` é só o rótulo da listagem. Divergir não quebra `/nome-do-diretório` — cria duas verdades,
  com o menu exibindo um nome que ninguém consegue digitar.
- `description` — formato `Use when <gatilho em inglês>. <gatilho em português com as palavras
  que o usuário deste projeto realmente digita>.` Cite a fase e o artefato quando existirem.
  **≤450 caracteres, contados.** Sem prosa, sem repetir o corpo: é uma string de matching.
- `argument-hint` — se aceita argumento. Aparece no autocomplete do menu `/`.
- `allowed-tools` — se o escopo é restrito de verdade. Não use como segurança: ele **adiciona**
  auto-aprovações durante o turno que invoca a skill, e o grant expira na próxima mensagem. O
  campo que de fato *tira* uma ferramenta da mão do modelo é `disallowed-tools`.
- `disable-model-invocation: true` — só se a skill gasta dinheiro, envia algo ou é destrutiva.

Campo que você não precisa é campo que mente sobre o comportamento da skill. Omita.

### Passo 2: escreva com densidade

Ordem fixa de uma skill de tarefa: **contrato** no topo → **uma a três frases** dizendo qual
decisão é desta skill e qual não é, nomeando a dona da que não é → **não-negociáveis** em tabela
de thresholds → **fluxo** em passos numerados, cada um com o comando ou o código que executa, com
a acessibilidade dentro do passo em que ela acontece → **anti-patterns com causa** →
**verificação** com itens observáveis.

O padrão de "código real" deste repositório — o bloco TSX de referência e as quatro coisas que o
tornam real — está em [reference.md](reference.md#padrão-de-código-real). Escrever exemplo em
vanilla JS com GSAP via CDN é erro de fato aqui.

Regras de forma: sem formatação vertical (uma palavra por linha triplica o arquivo sem adicionar
informação); sem lista de marcas sem dizer **o que** copiar de cada uma; sem emoji decorativo; sem
repetir o CLAUDE.md; nunca invente assinatura de API — sem certeza, escreva o padrão sem fingir
precisão.

### Passo 3: arquivos de apoio

| Arquivo | Conteúdo | Exemplo neste projeto |
|---|---|---|
| `reference.md` | Tabelas de campo, API, troubleshooting | `skill-builder/reference.md` |
| `decision.md` | "Qual técnica usar" — tabela comparativa + orçamento | `video-to-website/decision.md` |
| `<tema>.md` | Um subsistema completo com código | `landing-motion-expert/patterns.md` |

Todo arquivo de apoio precisa de link markdown relativo no SKILL.md. Arquivo não referenciado
nunca carrega — ele não é opcional, é invisível.

### Passo 4: teste

Rode a [verificação](#verificação) do fim desta skill, e depois quatro testes de comportamento:
**linguagem natural** — 3 frases diferentes que deveriam disparar; não disparou, falta
palavra-chave. **Direto** — `/nome-da-skill` com argumento, confira se `$0` substitui. **Sem
argumento** — a skill precisa se comportar, não travar. **Orçamento** — `/context` mostra a
listagem já depois do corte e `/doctor` mostra os maiores contribuintes; description truncada é a
causa mais comum de skill que existe e não dispara.

---

## Densidade operacional

Oito testes, cada um com número. Reprovou em 3 ou mais: reescreva. Remendo em documento de
filosofia produz documento de filosofia com números soltos.

| Teste | Reprova quando | Aprova quando |
|---|---|---|
| **1. Número no lugar de adjetivo** | "tipografia grande", "premium", "fluido" | `clamp(3.5rem, 12vw, 11rem)`, `stagger: 0.12`, `≥800vh` |
| **2. Código real** | pseudocódigo, `// sua lógica aqui`, JS genérico com CDN | TSX que compila para React 19 + GSAP 3.15 + Lenis 1.3 + Tailwind 4 |
| **3. Tabela de decisão** | lista solta de opções | tabela com a coluna "quando usar" |
| **4. Anti-pattern com causa** | "nunca anime margin" | "nunca anime `margin` — dispara layout na main thread; use `transform`" |
| **5. Tamanho** | `wc -l` fora de 150–300 | dentro da faixa, excedente em apoio referenciado |
| **6. Contrato** | sem a tabela ENTRADA/SAÍDA/ANTES/DEPOIS no topo, ou com caminho que não existe | os quatro rótulos, com caminho real em ENTRADA e SAÍDA |
| **7. Description** | >450 chars, ou genérica ("ajuda com design") | ≤450, gatilho em inglês + palavras reais em português |
| **8. Verificação executável** | "garanta qualidade premium" | "rode `npm run build`", "375px: seções empilham, ≤550vh" |

Acessibilidade não é teste separado: reprova no teste 4 quando o guard de
`prefers-reduced-motion` aparece numa seção de rodapé em vez do passo em que roda.

---

## Modo 2 — auditar uma skill existente

Leia o arquivo inteiro antes de propor qualquer mudança. Nunca audite uma skill que você não leu.

**Camada 1 — estrutura (11 itens).** `name` == diretório; description ≤450 com palavras em PT e
EN; `argument-hint` se aceita argumento; `allowed-tools` se o escopo é restrito; `context: fork`
só em tarefa autocontida; `disable-model-invocation: true` se gera custo, envia ou destrói; nenhum
campo supérfluo; todo apoio referenciado por link relativo; nenhum órfão; todo caminho citado
existe (`ls` neles); nenhuma chave de API no corpo. **Camada 2 — densidade:** os oito testes
acima. **Camada 3 — fronteira:** não duplica outra skill nem o CLAUDE.md; onde encosta em outra,
aponta com link; a camada da skill é uma só.

O relatório sai no formato de [reference.md](reference.md#formato-do-relatório-de-auditoria):
título com a contagem de linhas, os três placares, as reprovações numeradas com teste, linha e
conserto, e o veredito.

**Dividir** é o veredito quando o contrato não fecha: a skill produz dois artefatos, ou a SAÍDA de
metade dela é a ENTRADA da outra metade. Ao dividir, cada lado fica **completo e autônomo** para a
sua responsabilidade, com ENTRADA/SAÍDA próprias; conteúdo que as duas precisam fica com uma e a
outra referencia. Apague o diretório antigo depois de migrar — órfão é a próxima contradição.

---

## Fronteira por camada

Não confie em lista fixa de skills: as divisões acontecem em paralelo e qualquer lista escrita
aqui envelhece. O diretório manda — `ls -1 .claude/skills/` e
`grep -h '^description:' .claude/skills/*/SKILL.md`. Toda skill deste projeto pertence a
**exatamente uma** destas camadas:

| Camada | Fases | Decide | Nunca decide |
|---|---|---|---|
| **Entrada** | 0–3 | o que existe de fato: briefing, inventário de asset lido um a um, DNA visual medido, lacunas e conflitos | o que a página diz ou parece |
| **Direção** | 4–7 | o que a página diz e em que ordem: pesquisa, Experience Score, orçamento, estrutura, copy | como qualquer coisa é implementada |
| **Mídia** | 8–9 | o que é gerado e como o byte chega: prompt, decisão de vídeo, encode, frame sequence | onde a mídia cai na narrativa — isso é Direção |
| **Design** | 10 | escala, espaçamento, grade, contraste, token, caráter, fonte, textura | movimento |
| **Motion** | 10 | linguagem de motion, scroll, componente | layout estático e ordem de seção |
| **Auditoria** | 11 | aprova ou reprova contra número medido | como consertar em detalhe — devolve para a dona |
| **Publicação** | 12 | repositório, deploy, domínio, cache, verificação pós-deploy | conteúdo |
| **Meta** | fora do pipeline | como as skills são escritas, auditadas e divididas | qualquer decisão de produto |

Cinco regras que resolvem toda disputa de escopo:

1. **Uma skill, uma camada.** Se ela decide em duas, ela é duas skills.
2. **Um artefato, uma dona** — quem *escreve* o arquivo. Quem lê, referencia; nunca copia.
3. **Dentro da camada, a fronteira é por artefato, não por assunto.** "Vídeo" não é fronteira;
   `design/video-plan.md` e `public/media/*.mp4` são.
4. **Camada posterior não reescreve artefato de camada anterior** — fato novo volta para o JSON de
   origem e a dona o escreve.
5. **Auditoria nunca é dona de número.** Ela mede contra o número de outra skill e cita a dona;
   portão com threshold próprio vira uma segunda verdade que ninguém consegue conciliar.

### O teste que pega a contradição entre irmãs

Quando duas skills citam o mesmo número, exatamente **uma** escreve o valor e a outra nomeia a
dona. Se as duas escrevem por extenso, divergem na primeira edição — e a que estiver errada
continuará parecendo autoritativa. Se as duas apontam uma para a outra, ninguém é dona e o número
não existe. Grepe cada threshold da skill nova antes de dar por pronta:

```bash
grep -rn "0\.68\|scrollMobile" .claude/skills/*/SKILL.md ../../CLAUDE.md   # troque pelo termo
```

Mais de um resultado que **escreve o valor** é reprovação de fronteira. A dona é a skill da camada
mais cedo que precisa decidir aquilo; nas outras o bloco vira um link de uma linha. Número que já
está no CLAUDE.md não se repete em skill nenhuma.

---

## Anti-patterns

- **`description` genérica** — não dispara quando precisa e dispara quando não precisa, porque o
  matching é feito sobre essa string e mais nada.
- **`description` acima de 450 chars** — não é só desperdício: o orçamento de ~16.000 é
  compartilhado, então a description longa faz o corte cair na skill vizinha, que some do radar
  sem nenhum erro visível.
- **SKILL.md acima de 300 linhas** — carrega inteiro ao ser invocada; come o orçamento e o miolo
  perde peso na atenção do modelo.
- **Skill sem contrato no topo** — vira documento de leitura. Ninguém sabe o que alimentá-la nem
  quem consome a saída, e a ordem do pipeline deixa de ser verificável.
- **Dividir uma skill jogando metade em cada arquivo** — as duas metades ficam incompletas e a
  execução precisa das duas, o que é o oposto do que a divisão pretendia.
- **Copiar trecho do CLAUDE.md** — ele já está no contexto. A cópia diverge no primeiro edit.
- **Arquivo de apoio sem link no SKILL.md** — não carrega sozinho. Sem o link, ele não existe.
- **Lista de opções sem "quando usar"** — o modelo escolhe o primeiro item, sempre, e o resultado
  vira loteria entre execuções.
- **Adjetivo como threshold** — "suave", "premium", "rápido" resolvem diferente a cada execução; a
  skill perde a única função que tinha, que é reprodutibilidade.
- **`name` diferente do diretório** — o comando continua saindo do diretório, então o menu exibe um
  rótulo que ninguém consegue digitar. O bug não aparece em teste; aparece quando outra pessoa
  procura a skill pelo nome exibido.

## Verificação

```bash
wc -l .claude/skills/*/SKILL.md | sort -n            # toda linha entre 150 e 300
node -e "const fs=require('fs');let t=0;for(const d of fs.readdirSync('.claude/skills')){const p='.claude/skills/'+d+'/SKILL.md';if(!fs.existsSync(p))continue;const m=fs.readFileSync(p,'utf8').match(/^description: (.*)\$/m);if(!m){console.log('SEM DESCRIPTION',d);continue}t+=m[1].length;if(m[1].length>450)console.log('ESTOUROU',d,m[1].length)}console.log('total',t,'de 16000')"
grep -L 'ENTRADA' .claude/skills/*/SKILL.md          # não pode listar nada
```

- [ ] `SKILL.md` entre 150 e 300 linhas
- [ ] Contrato ENTRADA/SAÍDA/ANTES/DEPOIS no topo, com caminho real nos dois primeiros
- [ ] `description` ≤450 chars, e o total do diretório dentro de 16.000
- [ ] `name` == nome do diretório; `ls` confirma
- [ ] Todo caminho citado no corpo existe
- [ ] Todo arquivo de apoio tem link markdown relativo, e nenhum órfão ficou para trás
- [ ] Todo threshold é um número, e o grep confirma que só esta skill o escreve
- [ ] Todo padrão implementável tem código TSX da stack real, e todo anti-pattern tem a causa
- [ ] Reduced motion aparece no passo em que roda, não no rodapé
- [ ] A skill dispara em 3 frases naturais diferentes
- [ ] Bloco `## Suposições` escrito, se algo foi inferido

Um exemplo mínimo e completo, comentado linha a linha: [exemplo.md](exemplo.md).
