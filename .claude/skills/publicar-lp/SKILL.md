---
name: "publicar-lp"
description: "Use when a finished landing page must go live — secret and gitignore preflight, lp-<slug> repo on GitHub, Vercel deploy by MCP, immutable cache headers, domain DNS, post-deploy checks. Publicar o projeto, subir para o GitHub, colocar no ar, deploy na Vercel, dominio, DNS, cache, build que passa local e quebra na Vercel, entrega da URL ao cliente. Fase 12 do pipeline."
argument-hint: "[nome-do-cliente]"
---

# Publicar a landing page — Fase 12

| Contrato | |
|---|---|
| **ENTRADA** | os três laudos da Fase 11 com VEREDITO aprovado e nenhum BLOQUEIO aberto — `design/laudo-responsivo.md`, `design/laudo-acessibilidade.md`, `design/laudo-performance.md`; `design/lacunas.md` sem item em `gaps.blocking` e sem `conflicts[].resolved: false`; projeto rodando em `localhost:3000`; `design/briefing.json` para conferir os dados de contato |
| **SAÍDA** | repositório `lp-<slug>` no GitHub, URL de produção na Vercel e a mensagem de entrega enviada ao cliente |
| **ANTES** | `audit-responsivo`, `audit-acessibilidade` e `audit-performance` (Fase 11 — os três portões, os três sem BLOQUEIO) |
| **DEPOIS** | nada: é a última fase. Correção posterior reentra pela Fase 10 e volta a passar por aqui |

O dev decide **três** coisas aqui: o nome do cliente (de onde sai o slug), se o repositório é
público ou privado, e se autoriza publicar. O resto é comando ou tabela deste arquivo.

**Portão de entrada:** o parecer da Fase 11 aprovado item por item. Reprovado, esta fase não
começa — acessibilidade vira dívida que ninguém paga depois do deploy, porque o cliente já está
feliz e não há orçamento para consertar o que "já está funcionando".

---

## 1. Pré-voo — antes do primeiro commit

O histórico do Git guarda o arquivo **mesmo depois de você apagá-lo**. Um `.env.local` commitado
e removido no commit seguinte continua legível em `git log -p`, para sempre. E existem robôs que
clonam repositórios novos do GitHub minutos depois da criação, justamente à procura disso. Por
isso a conferência é **antes** do `git commit`, não depois do `git push`.

| # | Checagem | Comando | Passa quando |
|---|---|---|---|
| 1 | `.gitignore` cobre o que não pode subir | `git check-ignore -v .env .env.local node_modules .vercel assets-source` | cada caminho imprime a regra que o ignora |
| 2 | Os derivados **precisam** subir | `git check-ignore -v public/media public/frames assets/fonts` | **não imprime nada** (exit 1) |
| 3 | Nenhum segredo no que vai ser commitado | bloco "varredura de segredos" abaixo | saída vazia |
| 4 | Nenhum dado pessoal do cliente na árvore | bloco "dados pessoais" abaixo | saída vazia |
| 5 | O build passa do zero | `npm ci && npm run build` | exit 0, sem warning novo |
| 6 | Portão da Fase 11 | leia os laudos de `audit-responsivo`, `audit-acessibilidade` e `audit-performance` | os 3 sem BLOQUEIO, item por item |

A checagem 2 é invertida — passa quando **não** imprime nada — e é a que mais salva projeto desta
fábrica. `npm run assets` roda na **sua** máquina, com `ffmpeg` e `sharp`; a Vercel não roda esse
pipeline. Se `public/media/` ou `public/frames/` estiverem no `.gitignore`, o site sobe sem uma
imagem e sem um frame, e o build **passa**, porque na sua máquina os arquivos existem. Derivado
tem hash no nome, é determinístico e pesa poucos MB: vai commitado. Fora fica `assets-source/`,
com os originais pesados e as fotos cruas do cliente.

### `.gitignore` mínimo

```gitignore
node_modules/
.next/
out/
.vercel
.env
.env*.local
*.log
.DS_Store
assets-source/
design/renders/
design/briefing.json
design/lacunas.md
```

`.vercel` é lixo de máquina. `design/briefing.json` e `design/lacunas.md` saem porque carregam o
que o cliente falou em reunião, os dados não confirmados e as contradições dele. `design/renders/`
sai por peso — são os masters da Fase 9, e a página serve os derivados com hash de `public/media/`.
O resto de `design/` fica: é o que explica a página para quem abrir o repositório em seis meses.

### Varredura de segredos

```bash
git add -A
git diff --cached --name-only          # leia a lista inteira, arquivo por arquivo
git diff --cached -U0 | grep -nEi \
  'api[_-]?key|secret|senha|password|bearer |ghp_[A-Za-z0-9]{20,}|sk-[A-Za-z0-9]{20,}|-----BEGIN [A-Z ]*PRIVATE KEY-----'
```

Saída vazia: pode commitar. Qualquer linha impressa: remova a origem, não a linha.

**Segredo commitado e ainda sem push:** apague o histórico inteiro, é mais barato que reescrevê-lo
— `rm -rf .git && git init -b main`, e recomece o pré-voo. **Já com push em repositório público:**
o segredo está vazado. Rotacione a credencial (gere outra e invalide a antiga) antes de qualquer
coisa; apagar o repositório não basta, porque forks e caches já existem.

### Dados pessoais do cliente

```bash
grep -rEil 'cpf|cnpj|rg [0-9]|contrato|honorári|tabela de pre|custo interno|margem' \
  --exclude-dir=node_modules --exclude-dir=.git --exclude-dir=.next .
```

Contrato, CPF, tabela de preço interna, print de conversa, planilha de custo, foto de documento:
nada disso entra no repositório, nem no privado — ele será clonado, compartilhado e vai sobreviver
ao projeto. O que a landing precisa mostrar já está em `design/`, como conteúdo público.

### Público ou privado

| Situação | Decisão |
|---|---|
| Padrão da fábrica | **privado** (`--private`) — funciona igual na Vercel Hobby e não expõe o que ninguém revisou ainda |
| Depois da entrega, para o portfólio | público, e só com as checagens 3 e 4 devolvendo saída vazia |
| Cliente pediu privado, ou nicho sensível (saúde, jurídico, financeiro) | privado, e não vira público depois |
| "Tem um dado que não devia estar lá, mas privado resolve" | **não resolve.** Remova o dado |

Privado é o padrão porque a decisão é assimétrica: tornar público um repositório limpo é um
clique, e tornar privado um que já vazou não desfaz nada — o conteúdo já foi clonado. Para abrir
depois: `gh repo edit <owner>/lp-<slug> --visibility public --accept-visibility-change-consequences`

---

## 2. Nome do repositório

Convenção: **`lp-<slug-do-cliente>`**, um prefixo só e sempre o mesmo, para que os projetos da
fábrica fiquem agrupados na lista do GitHub. Regra de slug: minúsculas ASCII, acento removido por
decomposição (não trocado por outra letra à mão), `&` vira `e`, qualquer outro caractere vira
hífen, hífens colapsam, sem hífen na ponta, no máximo 40 caracteres. Sufixo societário (Ltda, ME,
S.A., EIRELI) sai antes.

```bash
node -e "const s=process.argv[1];console.log('lp-'+s.normalize('NFD').replace(/\p{Diacritic}/gu,'').toLowerCase().replace(/['’]/g,'').replace(/&/g,' e ').replace(/[^a-z0-9]+/g,'-').replace(/^-+|-+$/g,'').slice(0,37).replace(/-+$/,''))" "Clínica São Lucas"
```

| Nome do cliente | Slug | Por quê |
|---|---|---|
| Beleza Completa Barreiro | `lp-beleza-completa-barreiro` | nome composto: hífen entre todas as palavras |
| Clínica São Lucas | `lp-clinica-sao-lucas` | `í`→`i`, `ã`→`a` — decomposto, não traduzido |
| Açaí & Cia | `lp-acai-e-cia` | `&` vira `e`; nunca `%26` nem `and` |
| PetShop Dr. Márcio | `lp-petshop-dr-marcio` | o ponto de "Dr." some, não vira hífen solto |
| D'Angelo Odontologia | `lp-dangelo-odontologia` | apóstrofo some sem deixar hífen |
| Ótica Visão ME | `lp-otica-visao` | sufixo societário removido antes de gerar |

Esse slug não é só nome de pasta: por padrão ele vira o nome do projeto na Vercel e, com isso, o
endereço `lp-<slug>.vercel.app` que o cliente vai ler e às vezes ditar ao telefone.

---

## 3. Criar e subir

```bash
git init -b main                        # repositório local já na branch main
git add -A                              # tudo que o .gitignore não barrou
git commit -m "Landing page <Cliente>"  # primeiro e único commit da entrega
gh repo create <owner>/lp-<slug> --private --source . --remote origin --push
```

O último comando cria o repositório, define a visibilidade, registra o diretório atual como origem
(`--source .`), cadastra o remote `origin` e empurra a branch (`--push`). Sem `--push` o
repositório nasce vazio e o deploy da seção 4 não acha nada para construir. `<owner>` é seu
usuário ou a organização; confira com `gh api user --jq .login`.

| Erro | O que significa | Correção |
|---|---|---|
| `Name already exists on this account` | já existe repo seu com esse nome | reaproveite (bloco abaixo) ou qualifique o slug com a cidade: `lp-otica-visao-contagem` |
| `fatal: remote origin already exists` | `git init` numa pasta que já tinha remote | `git remote set-url origin https://github.com/<owner>/lp-<slug>.git` |
| `HTTP 403` / `must have admin rights` | token do `gh` sem escopo `repo`, ou a org não deixa criar | `gh auth refresh -s repo` |
| `--source: not a git repository` | rodou antes do `git init` | volte dois comandos |
| `fatal: no commits yet` / push vazio | rodou antes do `git commit` | commite primeiro |
| `remote contains work that you do not have` | o repo remoto nasceu com README pelo site | `git pull --rebase origin main` e empurre de novo |

Nome de repositório é único **por dono**, não no GitHub inteiro. Se o conflito for com projeto seu
antigo do mesmo cliente, reaproveite em vez de criar um segundo — dois repositórios do mesmo
cliente é como se perde qual está no ar. Para reaproveitar:
`git remote add origin https://github.com/<owner>/lp-<slug>.git && git push -u origin main`

---

## 4. Deploy na Vercel

### 4a. MCP — o caminho real

**Estado verificado nesta máquina:** o MCP da Vercel está conectado e autenticado, as ferramentas
têm prefixo `mcp__plugin_vercel_vercel__`, e `list_teams` (sem parâmetro) devolve o time
**`joaopedrokleinprevidis-projects`**, id **`team_MoO27VgQeIU975A8aZ7UklLn`**. A CLI da Vercel
**não** está instalada aqui: o MCP não é plano B, é o caminho.

**`teamId` é obrigatório** em `create_git_project` e em `list_projects`. Sem ele o projeto nasce
na conta pessoal errada, e o dev descobre isso quando a URL não aparece no painel do time.

1. `list_teams` — confirme o id antes de tudo; ele muda se a conta mudar.
2. `create_git_project` com `repo: "<owner>/lp-<slug>"` e `teamId`. Cria o projeto ligado ao
   repositório (ou reaproveita o que já estiver ligado), detecta o Next sozinha e dispara um
   **deploy de preview** com Vercel Authentication desligada, ou seja, a URL abre para qualquer
   um. `projectName` só se quiser nome diferente do repositório; `rootDirectory` só em monorepo.
3. **Produção.** O passo 2 entrega preview. Com o repositório ligado, todo push para `main`
   publica em produção sozinho: confira a preview, commite o que faltou, empurre.
4. **Build falhou:** `get_deployment_build_logs` com `idOrUrl` (o hostname do deploy serve),
   `teamId` e `errorsOnly: true` — devolve o fim do log, que é onde o erro mora.
5. **No ar e quebrou:** `get_runtime_errors` com `projectId`, `teamId` e `since: "24h"` agrupa por
   tipo, com rota e contagem; leia antes de `get_runtime_logs`, que é linha a linha. Máximo 7 dias.
6. **A URL pede login da Vercel:** é proteção de deploy ligada.
   `get_project_deployment_protection` mostra o estado, `update_project_deployment_protection`
   desliga. Landing pública com proteção ligada é o cliente escrevendo "não abre".

O MCP não gerencia variável de ambiente: `NEXT_PUBLIC_SITE_URL` entra pelo painel (Project →
Settings → Environment Variables) e exige **redeploy** para valer.

### 4b. CLI — só depois de instalar

`npm i -g vercel`, e então `vercel link` (grava `.vercel/project.json`, já ignorado),
`vercel env add NEXT_PUBLIC_SITE_URL production`, `vercel --prod` e `vercel logs <url>`. As skills
`/vercel:status`, `/vercel:deploy prod` e `/vercel:env` embrulham essa CLI e só funcionam com ela
instalada.

### 4c. Passa local, falha na Vercel

| Linha no log | Causa | Correção |
|---|---|---|
| `Module not found: Can't resolve './components/Hero'` | **caixa do nome do arquivo.** Linux distingue `Hero.tsx` de `hero.tsx`; Windows e macOS não, então o import errado funciona na sua máquina | `git mv --force src/components/hero.tsx src/components/Hero.tsx`, commit, push |
| `Environment variable ... is not defined`, `Invalid URL`, `metadataBase` | a variável só existe no `.env.local`, que nunca subiu | cadastre em produção **e** em preview, e redeploy |
| `ENOENT: public/media/...` ou imagem 404 no ar | derivado gerado localmente e ignorado no git | tire `public/media` e `public/frames` do `.gitignore`, commit |
| `npm ERR! ... peer` ou versão diferente da local | `package-lock.json` não commitado | commite o lock; teste com `npm ci`, nunca `npm i` |
| `The engine "node" is incompatible` | major do Node diferente | fixe `"engines": { "node": "22.x" }` no `package.json` e alinhe o local |
| Deploy verde, site em branco | `outputDirectory` ou `framework` errados no `vercel.json` | remova os dois; a Vercel detecta o Next sozinha |

A caixa do nome merece prevenção porque só aparece na Vercel: no Windows o Git nasce com
`core.ignorecase=true` e não registra a renomeação de `Hero.tsx` para `hero.tsx`.

```bash
git ls-files | sort -f | uniq -Di      # nomes que só diferem em caixa: qualquer saída é bug
git ls-files 'src/**' | grep -i hero   # o nome que o Git guardou, não o que o Explorer mostra
```

---

## 5. Cache imutável e domínio

O `vercel.json` inteiro, os registros DNS, a propagação e o que dizer ao cliente enquanto ela
acontece: [cache-e-dominio.md](cache-e-dominio.md). Em uma linha: `Cache-Control: public,
max-age=31536000, immutable` em `/frames/(.*)` e `/media/(.*)\.(mp4|avif|webp)` — caminho com
hash, nunca HTML, senão a correção de amanhã não chega em quem já visitou. Sem domínio,
`lp-<slug>.vercel.app` já é a entrega; com domínio novo, trocar `NEXT_PUBLIC_SITE_URL` **exige
redeploy**, senão canonical e sitemap ficam no `.vercel.app` e o Google indexa o endereço errado.
Confirme com `curl -I https://<url>/media/<arquivo>.mp4 | grep -i cache-control`.

---

## 6. Verificação pós-deploy

Tudo aqui roda contra a **URL de produção**, não contra `localhost`. Metade destes itens só falha
no ar.

```bash
curl -sI https://<url> | head -1                                     # espera HTTP/2 200
curl -s https://<url> | grep -oE '(wa\.me|api\.whatsapp\.com)[^"]*'  # links de WhatsApp
curl -s https://<url> | grep -oE 'tel:\+?[0-9]+'                     # links de telefone
curl -s https://<url> | grep -oE '<meta property="og:[^>]+>'         # cartão do WhatsApp
curl -s https://<url>/sitemap.xml | head -5                          # URL de produção lá dentro?
```

- [ ] **WhatsApp abre com a mensagem certa**, testado no celular e não no desktop. O número em
      `wa.me` vai sem `+`, parêntese, hífen ou espaço: `55` + DDD + número, 13 dígitos no celular
      (`5531988887777`). A mensagem vai percent-encoded — `?text=Ol%C3%A1%21%20Vim%20pelo%20site`
      — e precisa chegar acentuada, não como `Ol%E1`.
- [ ] **Telefone disca:** `tel:+553133334444`, com `+55`, testado num celular de verdade.
- [ ] **A capinha aparece no WhatsApp**, colando a URL numa conversa com você mesmo. O `og:image`
      precisa ser absoluto; relativo não renderiza. Cartão errado se corrige com `?v=2` no fim da
      URL — o WhatsApp guarda o cartão por dias e não tem botão de limpar.
- [ ] **JSON-LD valida** no Rich Results Test, zero erro; `aggregateRating` só com review real.
- [ ] **Contato conferido campo a campo contra `design/briefing.json`** — telefone, WhatsApp,
      endereço, horário, serviços. Um dígito errado invalida a página inteira.
- [ ] `curl -I https://<url>/media/<arquivo>.mp4` devolve `immutable`.
- [ ] `/robots.txt` e `/sitemap.xml` respondem, com a URL de produção dentro.
- [ ] A URL não pede login da Vercel (proteção de deploy desligada).
- [ ] Celular real em 4G, não emulador: a página abre, rola e os vídeos tocam.
- [ ] Sem erro de runtime nas primeiras 24 h — `get_runtime_errors` pelo MCP.

---

## 7. Entrega ao cliente

A mensagem pronta, o que o cliente muda sozinho, o README do repositório e o que **nunca** se
manda: [entrega.md](entrega.md). Mande a URL só com a seção 6 fechada — o cliente testa o telefone
antes de olhar o design.

## Anti-patterns desta fase

- **Conferir o `.gitignore` depois do push** — o histórico já guardou o arquivo; a única correção
  real é rotacionar a credencial.
- **`npm i` em vez de `npm ci`** — o `npm i` conserta o lockfile em silêncio na sua máquina; a
  Vercel usa o lock como está e quebra.
- **`create_git_project` sem `teamId`** — o projeto nasce na conta pessoal errada.
- **Domínio apontado e sem redeploy** — o Google indexa o `.vercel.app` por semanas antes de
  alguém notar.
- **Publicar sem o portão da Fase 11** — nenhuma auditoria de acessibilidade acontece depois que
  o site está no ar.
- **Testar o WhatsApp e o `tel:` no desktop** — os dois só se comportam de verdade no celular.

## Checklist final

- [ ] Os seis itens do pré-voo verdes **antes** do primeiro commit, com a checagem 2 sem saída
- [ ] `public/media` e `public/frames` commitados; `assets-source/` fora
- [ ] Repositório `lp-<slug>` criado, com o push da `main` concluído
- [ ] Projeto criado com o `teamId` do time certo; deploy de produção verde
- [ ] A URL abre sem pedir login da Vercel
- [ ] `vercel.json` com `immutable` em `/frames/` e `/media/`, confirmado por `curl -I`
- [ ] Os cinco `curl` da seção 6 rodados contra a URL de produção
- [ ] `NEXT_PUBLIC_SITE_URL` no endereço definitivo, com redeploy feito
- [ ] README escrito, sem segredo e sem dado pessoal
- [ ] Mensagem de entrega enviada, com o pedido dos três links (Instagram, Google, WhatsApp)
