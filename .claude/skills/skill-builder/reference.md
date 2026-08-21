# Referência técnica de skills

Referência completa de frontmatter, invocação, substituição de strings, `allowed-tools`,
padrões avançados e troubleshooting. Fonte: https://code.claude.com/docs/en/skills

O `SKILL.md` ao lado cobre *como decidir*. Este arquivo cobre *o que existe*.

---

## CLAUDE.md ou skill?

| | CLAUDE.md | Skill |
|---|---|---|
| **Quando carrega** | Toda conversa, sempre | Só quando invocada (`/nome` ou auto-detecção) |
| **Para que serve** | Regras do projeto, convenções, contexto permanente | Fluxo de uma tarefa específica |
| **Custo de contexto** | Sempre presente — mantenha enxuto | Só quando usada — mas mantenha abaixo de 500 linhas |
| **Exemplos** | "TypeScript em todos os arquivos", "rode os testes antes de commitar" | "Gerar resumo de PR", "extrair frames de vídeo", "auditar um capítulo" |

Regra: se Claude **sempre** precisa saber, é CLAUDE.md. Se precisa saber **ao fazer X**, é skill.

O CLAUDE.md continua valendo dentro da execução de uma skill. A skill é uma camada em cima,
nunca uma substituição. Por isso duplicar conteúdo do CLAUDE.md dentro de uma skill é puro
desperdício de contexto.

---

## Campos de frontmatter

Todos opcionais. Só `description` é recomendado em toda skill.

| Campo | Obrigatório | Tipo | Padrão | Descrição |
|-------|-------------|------|--------|-----------|
| `name` | Não | string | nome do diretório | **Só o rótulo exibido** na listagem. Em skill de projeto o comando `/nome` continua vindo do diretório. Minúsculas, números e hífens, máx. 64 chars. |
| `description` | Recomendado | string | 1º parágrafo do corpo | O que a skill faz e quando usar. É por aqui que Claude decide auto-invocar. `description` + `when_to_use` são truncados em **1.536 chars** na listagem. |
| `when_to_use` | Não | string | nenhum | Gatilhos e frases de exemplo. Anexado à `description` na listagem e conta no mesmo teto de 1.536. |
| `argument-hint` | Não | string | nenhum | Dica de autocomplete no menu `/`. Ex.: `[issue-number]`, `[filename] [format]` |
| `arguments` | Não | string ou lista | nenhum | Nomeia os argumentos posicionais para substituição `$nome`. `arguments: [issue, branch]` faz `$issue` = 1º, `$branch` = 2º. |
| `disable-model-invocation` | Não | boolean | `false` | Com `true`, só o usuário invoca. Remove a skill do contexto de Claude por completo. |
| `user-invocable` | Não | boolean | `true` | Com `false`, some do menu `/` **e** `/nome` deixa de funcionar. Só Claude invoca. |
| `allowed-tools` | Não | string ou lista | todas | Ferramentas liberadas sem prompt **durante o turno que invoca a skill**. O grant expira na próxima mensagem do usuário. |
| `disallowed-tools` | Não | string ou lista | nenhum | Ferramentas **retiradas** do pool enquanto a skill está ativa. Este é o campo que remove — `allowed-tools` só adiciona. |
| `model` | Não | string | herda | Mesmos valores de `/model`, ou `inherit`. Vale até o fim do turno, não é salvo. Com `context: fork`, define o modelo do subagente. |
| `effort` | Não | string | herda | `low`, `medium`, `high`, `xhigh`, `max`. Depende do modelo. |
| `context` | Não | string | nenhum | `fork` roda a skill num subagente isolado. |
| `agent` | Não | string | `general-purpose` | Tipo de subagente quando `context: fork`. `Explore`, `Plan`, `general-purpose`, ou agente de `.claude/agents/`. |
| `background` | Não | boolean | `true` | Só com `context: fork`. Padrão roda em background; `false` espera o resultado no mesmo turno. |
| `hooks` | Não | object | nenhum | Hooks registrados na invocação. **Continuam valendo pelo resto da sessão** — ver seção Hooks. |
| `paths` | Não | string ou lista | nenhum | Globs que limitam a auto-ativação. Ex.: `src/components/chapters/**` para uma skill de capítulo. |
| `shell` | Não | string | `bash` | Shell dos blocos `` !`comando` ``. `powershell` quando o ambiente é Windows sem Git Bash. |
| `metadata` | Não | map | nenhum | Dados livres para ferramentas próprias. Claude Code não age sobre eles. |
| `license` / `compatibility` | Não | string | nenhum | Campos da spec Agent Skills. Aceitos e ignorados pelo Claude Code. |

Não adicione campo só porque existe. Cada campo presente é uma afirmação sobre o comportamento
da skill; um campo errado é pior que campo ausente.

**Exportar para fora do Claude Code:** upload na claude.ai, Skills API e `package_skill.py`
aceitam apenas `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`.
Qualquer outro campo — `argument-hint` inclusive — faz o empacotamento falhar com erro duro, não
com um aviso. Skills que só rodam neste repositório não têm essa restrição.

---

## Matriz de controle de invocação

Como `disable-model-invocation` e `user-invocable` interagem:

| Frontmatter | Usuário invoca? | Claude invoca? | Quando entra no contexto |
|-------------|-----------------|----------------|--------------------------|
| (padrão — ambos omitidos) | Sim | Sim | Description sempre no contexto. Corpo carrega ao invocar. |
| `disable-model-invocation: true` | Sim | Não | Description **fora** do contexto. Corpo carrega quando o usuário invoca. |
| `user-invocable: false` | Não | Sim | Description sempre no contexto. Corpo carrega quando Claude invoca. |

Duas sutilezas que costumam morder:

- `user-invocable: false` esconde do menu `/` **e** faz `/nome` não rodar, mas não bloqueia a
  ferramenta Skill. Para bloquear invocação programática, use `disable-model-invocation: true`.
- `disable-model-invocation: true` é a restrição mais forte: tira a skill do contexto, bloqueia o
  acesso via ferramenta Skill, impede o pré-carregamento em subagentes e impede que uma tarefa
  agendada dispare a skill.

---

## Substituição de strings

Placeholders disponíveis no corpo da skill:

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `$ARGUMENTS` | Todos os argumentos passados na invocação | `/fix-issue 123` → `$ARGUMENTS` = `123` |
| `$ARGUMENTS[N]` | Argumento específico, índice base 0 | `$ARGUMENTS[0]` = primeiro argumento |
| `$N` | Atalho para `$ARGUMENTS[N]` | `$0` = primeiro, `$1` = segundo |
| `$nome` | Argumento nomeado, declarado no campo `arguments` | `arguments: [issue, branch]` → `$issue`, `$branch` |
| `${CLAUDE_SESSION_ID}` | ID da sessão atual | Útil para logs e arquivos por sessão |

**Fallback automático:** se `$ARGUMENTS` não aparecer em lugar nenhum do corpo, os argumentos
são anexados como `ARGUMENTS: <valor>` no fim. Funciona, mas o posicionamento é cego — prefira
colocar o placeholder onde ele importa.

Exemplo com argumentos posicionais:

```yaml
---
name: migrate-component
description: Migrate a component from one framework to another
argument-hint: [component] [from-framework] [to-framework]
---

Migre o componente $0 de $1 para $2.
Preserve todo o comportamento e os testes existentes.
```

`/migrate-component SearchBar React Vue` substitui `$0` por `SearchBar`, `$1` por `React`,
`$2` por `Vue`.

Três detalhes que decidem se a skill funciona sem argumento:

- **Aspas agrupam.** O corte é estilo shell: `/minha-skill "hero completo" segundo` faz
  `$0` = `hero completo`. Sem aspas, um valor com espaço se parte em dois.
- **Placeholder sem argumento fica literal.** `$2` com um argumento só permanece como o texto
  `$2` no prompt. Argumento nomeado ausente vira string vazia. Por isso toda skill precisa de uma
  linha dizendo o que fazer quando o argumento não veio.
- **`\$1.00`** escapa um `$` literal antes de dígito ou de `ARGUMENTS`.

---

## Onde a skill mora

O local determina quem consegue usá-la:

| Local | Caminho | Vale para | Prioridade |
|-------|---------|-----------|------------|
| Enterprise | Managed settings | Toda a organização | Máxima |
| Pessoal | `~/.claude/skills/<nome>/SKILL.md` | Todos os seus projetos | Alta |
| Projeto | `.claude/skills/<nome>/SKILL.md` | Só este projeto | Média |
| Plugin | `<plugin>/skills/<nome>/SKILL.md` | Onde o plugin estiver ativo | Mínima |

Nomes iguais em níveis diferentes: o de maior prioridade vence — enterprise sobrepõe pessoal,
pessoal sobrepõe projeto. Uma skill de projeto também sobrepõe uma skill embutida de mesmo nome,
**mas não os apelidos dela**: um `code-review` em `.claude/skills/` substitui `/code-review`, e
`/review` continua chamando a embutida. Skills de plugin usam o namespace `plugin-name:skill-name`
e por isso nunca colidem.

**Monorepo:** ao editar arquivos em subdiretórios, Claude Code descobre automaticamente skills
em `.claude/skills/` aninhados (ex.: `packages/frontend/.claude/skills/`).

**Diretórios extras:** skills dentro de `.claude/skills/` em diretórios passados com `--add-dir`
carregam automaticamente, com detecção de mudança ao vivo.

---

## `allowed-tools`

Restringe quais ferramentas rodam sem prompt de permissão enquanto a skill está ativa. As
permissões globais continuam valendo por cima.

Sintaxe básica — nomes separados por vírgula:

```yaml
allowed-tools: Read, Grep, Glob
```

Essas rodam livres. Qualquer outra pede permissão.

Padrões por ferramenta, com glob:

```yaml
allowed-tools: Bash(git *), Bash(npm test), Read
```

O glob entre parênteses filtra os argumentos:

| Padrão | O que libera |
|--------|--------------|
| `Bash` | Qualquer comando bash (sem restrição) |
| `Bash(git *)` | Qualquer comando começando com `git` (`git status`, `git diff`) |
| `Bash(npm test)` | Exatamente `npm test` |
| `Bash(python scripts/*)` | Python em qualquer arquivo de `scripts/` |
| `Read` | Ler qualquer arquivo (sem restrição de argumento) |

Combinações comuns:

```yaml
# Skill de leitura pura — não modifica arquivo
allowed-tools: Read, Grep, Glob

# Só comandos específicos
allowed-tools: Bash(git status), Bash(git diff), Read

# Só ferramentas de web
allowed-tools: WebSearch, WebFetch

# Lê arquivos e roda um script específico
allowed-tools: Read, Glob, Bash(node scripts/prepare-assets.mjs *)
```

`allowed-tools` **adiciona** auto-aprovações às permissões existentes, e só durante o turno que
invoca a skill — o grant expira na próxima mensagem do usuário. Não remove permissão já concedida
globalmente: não use como mecanismo de segurança, use como redução de fricção.

Para **remover** uma ferramenta do pool enquanto a skill roda, o campo é `disallowed-tools`:

```yaml
# Loop autônomo que nunca deve parar para perguntar
allowed-tools: Read, Glob, Bash(npm run typecheck)
disallowed-tools: AskUserQuestion
```

---

## Injeção dinâmica de contexto

A sintaxe `` !`comando` `` roda um comando de shell **antes** de o conteúdo da skill chegar em
Claude. A saída substitui o placeholder inline.

```yaml
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Contexto do PR
- Diff: !`gh pr diff`
- Comentários: !`gh pr view --comments`
- Arquivos alterados: !`gh pr diff --name-only`

## Tarefa
Resuma este pull request...
```

Ordem de execução:

1. Cada `` !`comando` `` executa imediatamente, antes de Claude ver qualquer coisa.
2. A saída substitui o placeholder.
3. Claude recebe o prompt já renderizado, com os dados dentro.

É pré-processamento. Claude vê só o resultado final, nunca os comandos.

Bons usos: status do git, branch, diff, resposta de API ao vivo, variáveis de ambiente,
listagem de arquivos, metadados do projeto.

Cuidado: a saída entra inteira no prompt. `` !`ls -R` `` num repositório grande queima milhares
de tokens antes de a skill começar. Filtre no comando, não depois.

---

## `context: fork` — rodar num subagente

`context: fork` roda a skill num contexto isolado. O corpo da skill vira o prompt da tarefa do
subagente. O subagente **não** tem acesso ao histórico da conversa.

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Pesquise $ARGUMENTS a fundo:

1. Encontre arquivos relevantes com Glob e Grep
2. Leia e analise o código
3. Resuma os achados com referências de arquivo específicas
```

**Use `context: fork` quando:**

- A tarefa é autocontida e não precisa do histórico da conversa.
- A skill produz saída verbosa que você quer fora do contexto principal.
- Você quer forçar restrição de ferramenta pelo tipo de agente.
- A tarefa é cara e um modelo mais barato resolve.

**Não use quando:**

- A skill contém diretrizes ("use estas convenções de API") sem tarefa concreta. O subagente
  recebe diretrizes e nenhum prompt acionável, e devolve nada útil.
- A skill precisa de ida e volta com o usuário.
- A skill depende de contexto da conversa atual.

Opções de `agent`:

| Agente | Ferramentas | Carrega CLAUDE.md? | Melhor para |
|--------|-------------|--------------------|-------------|
| `Explore` | Leitura | **Não** | Pesquisa, descoberta de arquivo, busca em código |
| `Plan` | Leitura | **Não** | Pesquisa para planejamento |
| `general-purpose` | Todas | Sim | Tarefas multi-etapa complexas |
| custom | conforme definido | Sim | Agentes de `.claude/agents/` |

`Explore` e `Plan` pulam CLAUDE.md e git status de propósito, para manter o contexto pequeno.
Neste projeto isso é decisivo: o pipeline de especialistas, o Experience Score e o Creative Budget
moram no CLAUDE.md. Uma skill com `agent: Explore` enxerga só o próprio SKILL.md — use para
levantar fatos, nunca para produzir trabalho que precisa obedecer às regras do projeto.

Por padrão o subagente roda em **background** e o resultado chega depois; `background: false`
espera no mesmo turno.

Skills e subagentes se combinam nas duas direções:

| Abordagem | System prompt | Tarefa | Também carrega |
|-----------|---------------|--------|----------------|
| Skill com `context: fork` | Do tipo de agente | Conteúdo do SKILL.md | CLAUDE.md — exceto com `Explore` ou `Plan` |
| Subagente com campo `skills` | Corpo markdown do subagente | Mensagem de delegação de Claude | Skills pré-carregadas + CLAUDE.md |

---

## Arquivos de apoio

Uma skill pode ter vários arquivos no seu diretório. Mantenha o SKILL.md abaixo de 500 linhas e
mova referência detalhada para arquivos irmãos.

```
minha-skill/
  SKILL.md              # Instruções principais (obrigatório, <500 linhas)
  reference.md          # Referência detalhada
  decision.md           # Tabela de decisão / quando usar o quê
  scripts/
    helper.mjs          # Script utilitário
```

Referencie-os do SKILL.md com link markdown relativo para que Claude saiba que existem. Arquivos
de apoio **não** carregam automaticamente — só quando Claude precisa. Um arquivo de apoio nunca
referenciado é um arquivo morto.

---

## Hooks dentro da skill

Skills podem declarar hooks de ciclo de vida no frontmatter. **Todos os eventos de hook são
suportados.**

Atenção ao tempo de vida, que é a pegadinha desta seção: um hook declarado numa skill é
registrado na invocação e **continua rodando pelo resto da sessão**, inclusive em turnos que não
têm nada a ver com a skill. Não é escopado à execução. Para rodar uma vez só, marque
`once: true` no hook.

(Hook declarado num **subagente** é que tem escopo curto: sai quando o subagente termina, e um
`Stop` ali vira `SubagentStop`. Skill não faz essa conversão.)

```yaml
---
name: safe-editor
description: Edit files with automatic linting
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      once: true
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

O comando do hook recebe JSON via stdin com o contexto da ferramenta. Exit code `0` permite —
o stdout vai para o log de debug, exceto em `UserPromptSubmit`, `UserPromptExpansion` e
`SessionStart`, onde vira contexto visível. Exit code `2` bloqueia a ação, e a mensagem de bloqueio
é o texto do stderr (ou a razão do JSON, se houver).

Como o registro sobrevive à skill, um `PreToolUse` com `matcher: "Bash"` declarado aqui passa a
inspecionar **todo** comando bash da sessão. Se essa não é a intenção, use `once: true` ou
declare o hook no `settings.json` do projeto, onde o escopo é explícito.

---

## Scripts embutidos (saída visual)

Skills podem trazer scripts que geram saída visual (HTML, imagens, gráficos).

```yaml
---
name: codebase-visualizer
description: Generate an interactive tree visualization of your codebase
allowed-tools: Bash(node scripts/*)
---

Rode o script de visualização:

  node .claude/skills/codebase-visualizer/scripts/visualize.mjs .
```

A skill orquestra; o script faz o trabalho pesado. Serve para grafo de dependências, relatório
de cobertura, docs de API, qualquer saída visual. Neste workspace, scripts em skill devem usar
Node (`.mjs`) — é a runtime já instalada — e nunca hardcodar caminho de binário de máquina.

---

## Extended thinking (ultrathink)

A palavra `ultrathink` em qualquer ponto do conteúdo da skill ativa raciocínio estendido para
aquela execução. Pode estar num comentário, num título ou no meio do texto.

Use em: análise que pesa múltiplos fatores, decisão de arquitetura com trade-off, depuração onde
a causa raiz importa. Não use em skill de execução mecânica — só encarece.

---

## Permissões e controle de acesso

Três formas de controlar quais skills Claude pode invocar:

**1. Desabilitar todas** negando a ferramenta Skill em `/permissions`:

```
# regras de deny:
Skill
```

**2. Permitir ou negar skills específicas** por regra de permissão:

```
# permitir só estas
Skill(commit)
Skill(review-pr *)

# negar específicas
Skill(deploy *)
```

Sintaxe: `Skill(nome)` para match exato, `Skill(nome *)` para prefixo com qualquer argumento.

**3. Esconder skill individual** com `disable-model-invocation: true` no frontmatter dela.

---

## Troubleshooting

### A skill não dispara

1. **Confira a `description`** — ela tem as palavras que alguém realmente diria? É por ela que
   Claude decide carregar.
2. **Verifique visibilidade** — pergunte "quais skills estão disponíveis?" para confirmar.
3. **Reformule o pedido** — tente um texto mais próximo da description.
4. **Invoque direto** — `/nome-da-skill` confirma se a skill funciona de todo.
5. **Cheque `disable-model-invocation`** — com `true`, Claude não auto-invoca; só `/nome` funciona.

### A skill dispara demais

1. Estreite a `description` — condições de gatilho mais específicas.
2. Adicione `disable-model-invocation: true` se só a invocação manual faz sentido.

### Claude não enxerga todas as skills

A listagem sempre contém **todos os nomes**; o que é cortado são as descriptions. O orçamento da
listagem é ~1% da janela de contexto. Quando estoura, Claude Code descarta a description das
skills que você menos invoca — o nome fica, o gatilho some, e a skill deixa de ser auto-detectada
sem nenhum erro visível. Cada entrada (`description` + `when_to_use`) também é truncada em 1.536
caracteres, independentemente do orçamento.

- Diagnóstico: `/doctor` estima o custo da listagem e aponta os maiores contribuintes. A linha
  Skills do `/context` mostra o tamanho **já depois** do corte.
- Correção na fonte: description curta, caso de uso principal na frente — o corte é por fim de
  string, então o que está no começo sobrevive.
- Correção por configuração: `skillListingBudgetFraction` (ex.: `0.02` = 2%),
  `skillListingMaxDescChars`, ou a env `SLASH_COMMAND_TOOL_CHAR_BUDGET` com um número fixo de
  caracteres. Em `skillOverrides`, marcar uma skill como `"name-only"` libera espaço para as outras.

Com 13 skills no projeto, esse orçamento é real e não teórico: uma description de 400 caracteres
consome o espaço de duas outras.

### Skill com `context: fork` devolve nada útil

A skill provavelmente contém diretrizes sem tarefa concreta. Subagente precisa de prompt
acionável, não de material de referência. Adicione instrução explícita: "Faça X, depois devolva Y."

### Argumentos não substituem

- Confirme que `$ARGUMENTS`, `$N` ou `$ARGUMENTS[N]` aparece no conteúdo.
- Se nenhum aparece, os argumentos são anexados como `ARGUMENTS: <valor>` no fim (fallback).
- Argumentos são separados por espaço, com aspas agrupando — sem aspas, um valor com espaço se
  parte em dois.
- Placeholder posicional sem argumento correspondente fica literal no prompt (`$2` continua `$2`).

---

## Documentação relacionada

- Skills: https://code.claude.com/docs/en/skills
- Subagentes: https://code.claude.com/docs/en/sub-agents
- Hooks: https://code.claude.com/docs/en/hooks
- Plugins: https://code.claude.com/docs/en/plugins
- Memória (CLAUDE.md): https://code.claude.com/docs/en/memory
- Permissões: https://code.claude.com/docs/en/permissions
