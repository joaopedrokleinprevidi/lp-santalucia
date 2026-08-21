# Credenciais e acessos

Nesta linha de produção o deploy é **automático**: o repositório é criado, o projeto é ligado na
Vercel e o site é publicado sem você digitar comando nenhum. Seu papel nas credenciais é um só —
aprovar o login no navegador quando a ferramenta abrir a janela.

Este arquivo diz o que precisa estar destravado, em que fase cada coisa trava se faltar, e como
destravar.

## Verificação rápida — roda na Fase 0, antes de qualquer trabalho

```bash
node --version      # precisa v20 ou superior
git --version
gh auth status      # GitHub
claude mcp list     # procure a linha do servidor vercel
vercel whoami       # só se for usar a CLI em vez do MCP
```

Saída de uma máquina pronta:

```
v22.23.1
git version 2.51.0.windows.1
github.com
  ✓ Logged in to github.com account joaopedrokleinprevidi (keyring)
  - Token scopes: 'gist', 'read:org', 'repo', 'workflow'
plugin:vercel:vercel: https://mcp.vercel.com (HTTP) - ✔ Connected
```

O escopo `repo` é o que importa no GitHub — sem ele o `gh` autentica mas não cria repositório.

---

## Tabela mestra

| Credencial | O que destrava | Bloqueia o pipeline? | Trava na fase | Como conseguir |
|---|---|---|---|---|
| Node ≥20 + git | build, pipeline de imagem, transcode de vídeo | Sim | 10 — implementação | [nodejs.org](https://nodejs.org) (LTS) e [git-scm.com](https://git-scm.com) |
| GitHub via `gh auth login` | criação do repositório `lp-<slug>` e o push | Não antes da 12 | 12 — publicar | Conta grátis + `gh auth login` |
| **Vercel via MCP** (preferido) | criar projeto, publicar, ler log de build e erro de runtime, consultar domínio | Não antes da 12 | 12 — publicar | `claude mcp add` + aprovar no navegador |
| Vercel via CLI (alternativa) | o mesmo, digitado à mão | Não antes da 12 | 12 — publicar | `npm i -g vercel` + `vercel login` |
| ChatGPT com geração de imagem | as imagens de cada seção | Na prática, sim | 9 — geração das mídias | Plano pago do ChatGPT, ou equivalente |
| Google Flow (ou Runway/Kling/Luma) | os clipes animados | Não — a página funciona com imagem parada | 9 — geração das mídias | [labs.google/flow](https://labs.google/flow) |
| Acesso ao registrador do domínio | apontar o domínio próprio do cliente | Não — a Vercel dá um `.vercel.app` | 12 — só o domínio | Login do Registro.br / GoDaddy, com o cliente |

Uma landing page padrão **não usa nenhuma chave de API**. Se o projeto pedir uma (formulário que
envia e-mail, mapa pago, analytics de terceiro), leia a seção "Segredo que precisa viver no
projeto" antes de qualquer coisa.

---

## A regra de ouro sobre token

**Nenhum token, senha ou chave é digitado no chat, colado em arquivo do projeto, ou enviado para
alguém.**

As ferramentas oficiais — `gh auth login`, `vercel login`, a autenticação do MCP — abrem o
navegador, você aprova lá, e a credencial fica guardada no cofre do sistema operacional (no
Windows, o Gerenciador de Credenciais; no macOS, o Keychain). Ninguém precisa ver o token, nem
você. É por isso que o `gh auth status` mostra `gho_************************************`: nem a
própria ferramenta exibe o valor.

Se alguém — pessoa, site ou ferramenta — pedir para você colar um token numa conversa, **a
resposta é não**. Não existe fluxo legítimo que precise disso. Um token colado num chat vira parte
do histórico da conversa, e histórico se copia, se exporta e se compartilha.

---

## GitHub

**Conta:** [github.com/signup](https://github.com/signup). Grátis.

**Instalar o GitHub CLI:**

| Sistema | Comando |
|---|---|
| Windows | `winget install GitHub.cli` |
| macOS | `brew install gh` |
| Linux | [instruções oficiais](https://github.com/cli/cli#installation) |

**Conectar:**

```bash
gh auth login
```

Responda: `GitHub.com` → `HTTPS` → `Y` para autenticar o Git → `Login with a web browser`. Ele
mostra um código de 8 caracteres, abre o navegador, você cola o código e aprova.

**Confirmar:** `gh auth status` precisa mostrar `Logged in to github.com account <seu-usuario>` e,
nos escopos, `repo`.

### O repositório é criado sozinho

Na Fase 12 o repositório nasce com o nome `lp-<slug-do-cliente>`. A regra do slug e os exemplos
resolvidos ficam num lugar só: [publicar-lp](../publicar-lp/SKILL.md) §2. Em resumo — minúsculas ASCII, acento
decomposto, hífen entre todas as palavras do nome, sufixo societário removido.

| Cliente | Repositório |
|---|---|
| Beleza Completa Barreiro | `lp-beleza-completa-barreiro` |
| Clínica Santa Lúcia | `lp-clinica-santa-lucia` |
| Auto Elétrica São Jorge ME | `lp-auto-eletrica-sao-jorge` |

```bash
git init -b main && git add -A
git commit -m "Landing page <cliente>"
gh repo create lp-<slug> --private --source . --remote origin --push
```

**Privado é o padrão.** Público só depois que a varredura mais abaixo passar — e mesmo assim, só
se você quiser o projeto no portfólio. Repositório privado funciona igual na Vercel.

**Se o navegador não abrir** (rede corporativa, proxy, máquina sem interface gráfica): `gh auth
login --with-token` lendo de um arquivo local que você apaga em seguida. Nunca cole o token no
chat — o arquivo temporário existe justamente para evitar isso.

---

## Vercel — caminho A: o servidor MCP (preferido)

O MCP da Vercel é uma conexão direta entre esta sessão e a sua conta na Vercel. Com ele, **nada é
instalado na sua máquina** e o deploy inteiro acontece de dentro da conversa.

### Como saber se está conectado

```bash
claude mcp list
```

| Linha que aparece | Significa | O que fazer |
|---|---|---|
| `vercel: https://mcp.vercel.com (HTTP) - ✔ Connected` | pronto, autenticado | nada |
| `... - ! Needs authentication` | o servidor responde, mas falta você aprovar | rode `/mcp` numa sessão interativa e autentique |
| não aparece nenhuma linha `vercel` | o servidor não está adicionado | adicione (abaixo) |

### Como conectar

```bash
claude mcp add --transport http vercel https://mcp.vercel.com
```

Depois, numa sessão **interativa**, rode `/mcp`, escolha `vercel`, escolha `Authenticate`. O
navegador abre, você entra na Vercel e aprova. O token fica na sessão, nunca no projeto.

**Conecte antes de começar o pipeline.** A janela de aprovação do navegador não abre em sessão não
interativa — se o MCP estiver pendente de autenticação na Fase 12, a publicação para e espera você.

### O que o MCP faz por nós

| Ferramenta | Para quê |
|---|---|
| `list_teams` | descobre o `teamId`. **Roda sempre primeiro** — o endpoint não tem time implícito, e sem o `teamId` o projeto vai parar na conta pessoal errada |
| `create_git_project` | cria o projeto na Vercel ligado ao repositório `lp-<slug>` e já dispara um deploy de preview |
| `deploy_to_vercel` | publica uma árvore de arquivos direto, sem repositório e sem CLI. `target: "preview"` ou `"production"` |
| `get_deployment` / `list_deployments` | estado da publicação e a URL |
| `get_deployment_build_logs` | por que o build quebrou, com `errorsOnly: true` para ver só as linhas de erro |
| `get_runtime_errors` / `get_runtime_logs` | erro que só aparece com o site no ar |
| `check_domain_availability_and_price`, `buy_domain`, `get_domain_order` | domínio: se está livre, quanto custa, comprar, acompanhar |
| `get_web_analytics` | visitas, depois que o site estiver no ar |

Os nomes reais têm o prefixo do plugin (`mcp__plugin_vercel_vercel__list_teams`). Você não digita
nenhum deles — estão aqui para você entender o que está acontecendo quando eu os uso.

### `create_git_project` ou `deploy_to_vercel`?

| | `create_git_project` | `deploy_to_vercel` |
|---|---|---|
| Precisa de repositório | sim | não |
| Como os arquivos chegam | a Vercel clona do GitHub | são enviados dentro da chamada |
| Assets pesados (AVIF, WebP, MP4) | sobem no `git push`, sem limite prático | teriam que ser codificados em base64 na chamada — inviável |
| A cada correção depois | um `git push` republica | reenviar a árvore inteira |
| Histórico e volta atrás | sim, é git | não |

**Para esta linha de produção, é sempre `create_git_project`.** Uma landing page carrega dezenas
de megabytes de imagem e vídeo; mandar isso por dentro de uma chamada de ferramenta não é opção.
`deploy_to_vercel` serve para protótipo pequeno de arquivo de texto.

### A ordem real da Fase 12

1. `gh repo create lp-<slug> --private --source . --remote origin --push`
2. `list_teams` → pega o `teamId`
3. `create_git_project` com `repo: "<owner>/lp-<slug>"` e o `teamId` → sai a URL de preview
4. Abrir a URL de preview e conferir o portão da fase
5. `git push` na branch `main` → a Vercel constrói e publica em produção sozinha, a cada push

Se o build quebrar, `get_deployment_build_logs` traz o erro para dentro da conversa e a correção
sai no mesmo turno — sem você copiar mensagem de erro de um painel para o chat. É a principal
razão de o MCP ser o caminho preferido.

---

## Vercel — caminho B: a CLI

Use quando não há MCP conectado, quando a rede bloqueia o servidor, ou quando você quer publicar
sem uma sessão de agente aberta.

```bash
npm i -g vercel
vercel login          # escolha "Continue with GitHub"
vercel whoami         # confirma
```

Na hora de publicar:

```bash
vercel link           # associa a pasta a um projeto; aceite os padrões
vercel --prod         # publica em produção
vercel logs <url>     # log de uma publicação
```

O que se perde: o log de build não volta para a conversa, então uma falha vira um ciclo de copiar
e colar. Funciona, só é mais lento.

---

## ChatGPT — geração de imagem

Você gera as imagens manualmente, na Fase 9, colando os prompts do `design/image-prompts.md`.

| Ferramenta | Observação |
|---|---|
| ChatGPT (GPT Image) | Mais prático: aceita **anexar as imagens de referência** junto do prompt, que é o que amarra a identidade da marca. Precisa de plano pago para volume |
| Midjourney | Melhor controle estético, curva de aprendizado maior, usa Discord |
| Sora / outros | Funcionam; a sintaxe do prompt muda pouco |

Cada bloco do `image-prompts.md` diz **quais arquivos anexar** (o logo, um ou dois posts, e a foto
real do local quando a seção falar dele) e **onde salvar** o resultado. Anexar não é opcional: sem
o logo anexado, a cor da marca sai aproximada e a diferença aparece quando a imagem fica ao lado
do logo de verdade no cabeçalho.

**Uma conversa nova por seção.** Numa thread longa o modelo passa a usar as imagens que ele mesmo
gerou como referência, e a seção 6 vira a seção 5 com outro objeto dentro.

Salve na maior resolução que a ferramenta oferecer. O pipeline gera as versões leves depois — mas
não inventa resolução que não existe.

---

## Google Flow — animação

Transforma cada imagem gerada num clipe curto, ainda na Fase 9.

**Acesso:** [labs.google/flow](https://labs.google/flow). Disponibilidade e plano variam por
região — confirme antes de prometer prazo ao cliente.

**Uso:** suba a imagem da seção, cole o prompt de movimento do `design/motion-prompts.md`, gere de
4 a 6 segundos, baixe o MP4.

**Alternativas:** Runway Gen-3, Kling, Luma Dream Machine. Os prompts funcionam nas três com
ajuste mínimo, porque descrevem movimento e câmera — não sintaxe de ferramenta.

**Sem acesso a nenhuma delas, o projeto continua.** A página usa as imagens paradas com parallax e
ken burns em CSS. Perde-se um grau de sofisticação, não a entrega.

---

## Domínio

Sem domínio próprio, a Vercel dá `lp-<slug>.vercel.app` de graça, e o site funciona.

Com domínio próprio, você precisa do **login do registrador** (Registro.br, GoDaddy, Hostgator) —
que é do cliente, não seu. No painel da Vercel: projeto → Settings → Domains → Add. A Vercel mostra
os registros DNS exatos.

**Copie os valores da tela da Vercel, nunca de um tutorial.** Os endereços de DNS dela já mudaram;
um IP velho copiado de um post de blog aponta para lugar nenhum e o erro leva horas para aparecer.
A propagação leva de minutos a algumas horas.

**Decida o domínio antes da Fase 10.** O endereço definitivo entra no `metadataBase`, no canonical,
no sitemap e no `og:url`. Trocar depois obriga a reconstruir o site e a esperar o Google reindexar.

---

## Segredo que precisa viver no projeto

Quando uma chave de terceiro é mesmo necessária:

1. Vai em `.env.local`, que já está no `.gitignore` e nunca sobe para o GitHub.
2. Na Vercel, a mesma chave é cadastrada por `vercel env add <NOME>` (ou pelo painel, em Settings
   → Environment Variables). Ela fica lá, não no repositório.
3. Nunca em `.env` sem sufixo, nunca em `next.config`, nunca hardcoded num componente.

Chave que começa com prefixo público (`NEXT_PUBLIC_`) vai para o navegador do visitante e é
legível por qualquer um. Só use esse prefixo para valor que pode mesmo ser público.

---

## Antes do primeiro commit: a varredura

Rode isto **antes** de `git add -A`, e leia a saída inteira:

```bash
git status --porcelain           # tudo que vai entrar no commit
git check-ignore -v .env.local   # tem que responder que está ignorado
```

Confira, arquivo por arquivo, que não entrou:

- [ ] contrato, proposta comercial ou orçamento
- [ ] CPF, CNH, RG ou dado pessoal de qualquer pessoa
- [ ] tabela de preço interna, margem, custo
- [ ] print de WhatsApp com conversa de cliente
- [ ] planilha de clientes, lista de telefones
- [ ] foto com rosto de terceiro sem autorização de uso de imagem
- [ ] qualquer `.env` que não seja `.env.example`

**Repositório público é público para sempre.** Apagar o arquivo num commit seguinte não resolve:
o git guarda o histórico, e o conteúdo continua acessível para quem clonar. A única correção real
é reescrever o histórico ou destruir o repositório e começar outro — e, se o dado já foi indexado,
nem isso resolve.

Por isso o padrão é `--private`. Quando em dúvida, fica privado.

---

## Quando falta cada credencial

| Falta | O que acontece | Contorno | Quando resolver |
|---|---|---|---|
| Node ≥20 | nada é construído | nenhum | antes da Fase 10 |
| GitHub | o código fica só na sua máquina | nenhum — a Vercel precisa do repositório | antes da Fase 12 |
| Vercel (MCP e CLI) | não há URL no ar | nenhum | antes da Fase 12 |
| Só o MCP (a CLI existe) | perde o log de build na conversa | use a CLI | opcional |
| ChatGPT / gerador | seções sem imagem | as seções carregam em tipografia grande e cor sólida, e as fotos reais do cliente cobrem o resto. Uma LP inteira sem imagem só funciona se houver foto real de sobra | antes da Fase 9 |
| Google Flow | sem clipes | stills com parallax e ken burns em CSS | opcional |
| Registrador do domínio | fica no `.vercel.app` | nenhum, e trocar depois custa rebuild + reindexação | antes da Fase 10 |

---

## Ferramentas locais

Já vêm resolvidas pelo `package.json` do projeto — você não instala nada globalmente:

| Pacote | Para quê |
|---|---|
| `ffmpeg-static`, `ffprobe-static` | transcodar vídeo e extrair frames |
| `sharp` | gerar AVIF/WebP responsivos e os placeholders borrados |

**Nunca instale ffmpeg globalmente nem escreva um caminho de binário fixo em script.** O caminho da
sua máquina não existe na máquina do próximo — é a origem clássica do pipeline que "funciona aqui".

---

## Dados do cliente

Não são credencial, mas travam igual. A lista completa, campo por campo e com o motivo de cada um,
está no [briefing-template.md](../briefing-cliente/briefing-template.md) — é o formulário que você
preenche na Fase 0. Não duplicamos aqui para não existirem duas versões da mesma lista.

O que mais aparece faltando, em ordem: o **WhatsApp com DDD** (quase sempre é um celular diferente
do telefone fixo), o **CEP**, o **horário de feriado**, e a autorização de uso de imagem para
depoimento com nome e foto.
