---
name: briefing-cliente
description: Use when a landing page project is starting and you must check credentials and collect the client briefing before any asset or design work. Fase 0 do pipeline: checa node, git, gh, vercel e gerador de imagem; entrega as tres listas do que pedir ao cliente (bloqueia, piora, enriquece) com o porque de cada campo; template de briefing em linguagem de leigo. Produz design/briefing.json.
argument-hint: [nome-do-cliente]
allowed-tools: Read, Write, Edit, Glob, AskUserQuestion, Bash(mkdir *), Bash(node --version), Bash(git --version), Bash(gh auth status), Bash(vercel whoami), Bash(claude mcp list)
---

# Briefing do cliente — Fase 0

| | |
|---|---|
| **ENTRADA** | Nada. É o início do pipeline: só a pasta do projeto e o dev |
| **SAÍDA** | `design/briefing.json` com `meta`, `credentials`, `business`, `services`, `doesNotOffer`, `differentiators`, `socialProof`, `pricing` — e a pasta `assets-source/` com o material do cliente dentro |
| **ANTES** | Nenhuma |
| **DEPOIS** | `estudo-assets` (Fase 1), que lê `assets-source/` |

O dev é leigo. O trabalho dele é trazer o material, dizer "gere", colar prompts no ChatGPT e no
Google Flow, e aprovar login quando pedido. Ele não sabe o que a página vai precisar — então esta
skill não entrega um formulário e espera. Ela **explica o porquê de cada campo, pede tudo de uma
vez, e aceita campo em branco.** Campo vazio aqui não é problema: é exatamente o que
`auditoria-dados` existe para caçar, depois que os assets já responderam metade.

Antes de qualquer coisa, fixe a pasta. O projeto inteiro vive num diretório só, e é o diretório em
que a sessão já está:

```bash
mkdir -p design/renders assets-source scripts
```

`assets-source/` é a **única** pasta de assets do projeto. Esse nome não é sugestão: é o que o
pipeline de mídia da Fase 10 lê e o que o `.gitignore` mantém fora do repositório.

## Checagem de credenciais

Roda antes do trabalho, não antes da publicação. Descobrir na Fase 12 que faltava um acesso custa o
projeto inteiro em espera, e o dev leigo não tem como saber que faltava.

```bash
node --version      # v20+
git --version
gh auth status      # precisa listar o escopo 'repo'
vercel whoami
claude mcp list     # procure a linha do servidor vercel
```

| Falha | O que trava | Bloqueia agora? | Ação |
|---|---|---|---|
| `node` ausente ou <v20 | build, pipeline de imagem, transcode de vídeo | Não — trava a Fase 10 | Instalar antes da implementação começar |
| `git` ausente | versionamento e deploy | Não — trava a Fase 12 | Instalar até a publicação |
| `gh auth status` falha | repositório e deploy | Não — trava a Fase 12 | O dev roda `gh auth login` quando pedirmos |
| `vercel whoami` falha | só o `--prod` | Não — trava a Fase 12 | `vercel login` na publicação |
| Sem gerador de imagem | as imagens de seção, ou seja, a página | **Sim, na prática** | Confirmar agora que ele tem ChatGPT ou equivalente |
| Sem Google Flow | as animações | Não | A LP funciona com stills; os clipes viram melhoria |

Passo a passo de instalação, o que fazer quando o navegador não abre e o que cada credencial
destrava por fase: [credentials.md da factory](../landing-page-factory/credentials.md). Não repita
aqui — duas versões da mesma lista divergem na primeira correção.

**Regra que não se quebra:** nunca peça, receba ou escreva um token no chat ou em arquivo do
projeto. `gh auth login` e `vercel login` abrem o navegador e guardam a credencial no cofre do
sistema. Se alguém pedir para colar um token numa conversa, a resposta é não — não existe fluxo
legítimo assim.

Registre o resultado em `briefing.json.credentials` com `checkedAt`. Uma checagem de três semanas
atrás não vale nada: o token do `gh` expira e ninguém avisa.

## O pedido ao dev: um pedido só

Entregue o [briefing-template.md](briefing-template.md). Ele é escrito em linguagem de leigo, campo
por campo, com exemplo preenchido ao lado de cada um. O dev preenche o que souber e deixa em branco
o que não souber.

O que segue é o *porquê* de cada campo, para você usar ao explicar. Um dev que entende para que
serve o CEP traz o CEP; um dev que só vê um formulário pula o campo.

### Nível 1 — BLOQUEIA (sem isso não começa)

| Dado | Por que a LP precisa | Onde aparece no código |
|---|---|---|
| Nome como aparece na marca | Duas grafias ("Clínica Santa Lúcia" / "Santa Lúcia Clínica Veterinária") quebram o casamento com o perfil do Google | `<title>`, JSON-LD `name`, `alt` do logo, rodapé |
| O que faz, em uma frase | É a meta description e a subheadline do hero. Sem ela, a página abre sem dizer o que é | `<meta description>`, JSON-LD `description` |
| Cidade e região atendida | A busca real é "veterinário em Caxias do Sul", não "veterinário". Sem cidade não há busca local | JSON-LD `areaServed`, `addressLocality`, headline |
| Telefone | Vira link discável no celular | `tel:`, JSON-LD `telephone` |
| **WhatsApp com DDD** | É o CTA principal da página. Quase sempre é um celular **diferente** do fixo — perguntar os dois separadamente não é redundância | `https://wa.me/55DDDNUMERO?text=…` |
| Endereço completo com CEP | O CEP é campo obrigatório do `PostalAddress` e é o que impede o mapa de errar a quadra | JSON-LD `PostalAddress`, mapa, rodapé |
| Horário, incluindo feriados | É a pergunta número um de quem chega pelo celular. "Fecha domingo?" precisa estar respondida antes de ser feita | `openingHoursSpecification` |
| Lista de serviços | Viram as seções. Sem elas o storytelling não tem o que ordenar e a estrutura é inventada | seções, JSON-LD `hasOfferCatalog` |
| Logo na maior resolução | Um logo de 360 px em JPEG chapado não tem transparência e não sobe em cabeçalho retina | `<header>`, favicon, OG image |
| Fotos do local | É a única prova de que o lugar existe. LP 100 % de imagem gerada é percebida como catálogo | seção de prova/estrutura |
| Fotos do trabalho sendo feito | Imagem gerada não pode afirmar fato sobre o negócio; foto real pode | seções de serviço |

**Sobre os assets mínimos:** sem logo e sem fotos, a paleta da Fase 2 vira chute e a página inteira
vira ilustração. O dev leigo costuma achar que "as fotos são feias demais". Traga assim mesmo — foto
feia de lugar real vale mais que render bonito de lugar inexistente, e o tratamento é nosso. O piso
praticável é 1 logo, 2 fotos do local, 3 do trabalho feito e 2 posts com a redação do cliente;
abaixo disso, a pergunta certa é "o cliente consegue tirar dez fotos de celular esta semana?".

### Nível 2 — PIORA A LP SE FALTAR

| Dado | O que se perde sem ele |
|---|---|
| Domínio (comprado ou a comprar) | `metadataBase`, canonical, sitemap e `og:url` precisam de domínio absoluto. Sem, sai `.vercel.app` — e trocar depois exige rebuild e reindexação no Google |
| E-mail | Canal para quem não usa WhatsApp; `email` no JSON-LD |
| Redes sociais com o @ | `sameAs` é o que liga a LP ao perfil no Knowledge Panel do Google. É o campo de SEO mais barato que existe |
| Google Business: link, nota e nº de avaliações | "4,8 de 5 em mais de 1.100 avaliações" é número verificável. "Excelência no atendimento" não é nada |
| Diferenciais reais | São as seções do meio da página. Sem eles a LP vira lista de serviços e não se distingue do concorrente |
| **O que a empresa NÃO faz** | Ninguém oferece isso espontaneamente e é o campo que mais evita estrago. Uma LP que sugere serviço inexistente gera ligação irritada, não cliente |
| Depoimentos **com autorização de uso** | Nome e foto de terceiro sem autorização é risco de direito de imagem e LGPD. Sem autorização: use o texto da avaliação pública, sem foto e sem sobrenome |
| Faixa de preço | Decide se existe seção de preço ou se o CTA é "consultar valor". Sem o dado, o CTA é obrigado a ser vago |
| Ano de fundação | "Desde 2015" é o sinal de confiança mais barato do repertório |
| CNPJ | Rodapé. É o que separa negócio real de página de golpe aos olhos de quem desconfia |
| Registro profissional | Quando o conselho exige, exibir é obrigação do cliente: **CRMV** (veterinária), **CRO** (odontologia), **CREA/CAU** (engenharia e arquitetura), **OAB** (advocacia), **CRM** (medicina), **CRN**, **CREFITO**. Publicar sem é infração dele — mas o erro de entregar assim é nosso |

### Nível 3 — ENRIQUECE

| Dado | O que destrava |
|---|---|
| Vídeos existentes | Pelo CLAUDE.md, vídeo é candidato a virar timeline controlada por scroll. Um vídeo real do local vale mais que oito imagens geradas |
| Material impresso (cartão, folder, cardápio) | Paleta secundária e iconografia que os posts não mostram |
| Site atual | Fonte de **fatos** e de **conflitos** — nunca de copy. Se o site atual convertesse, não haveria projeto |
| Concorrentes que ele admira | Calibra a direção criativa em cinco minutos em vez de duas rodadas |
| Vetos estéticos ("odeio azul", "chega de foto de pet fofo") | Economiza uma rodada inteira de geração de imagem, que é a parte cara e lenta |

## O artefato: `design/briefing.json`

**Nenhum valor existe sem procedência.** Um campo sem `source` não é fato, é lembrança.

| Prefixo | Significa |
|---|---|
| `asset:` | Está visível numa imagem que foi aberta e lida — `asset:fachada-clinica-rua.webp` |
| `doc:` | Está num documento que o cliente entregou — `doc:dados-google-da-empresa.txt` |
| `dev:` | O dev respondeu, com a data — `dev:2026-08-20` |
| `web:` | Veio de fonte pública e ainda não foi confirmado — `web:google-business` |
| `assumed` | Ninguém disse; foi adotado por padrão, e exige entrada em `assumptions` |

`web:` é o mais perigoso dos cinco: parece fato e é o que o cliente esqueceu de atualizar em 2019.
Todo campo `web:` de contato, horário ou preço vira pergunta de confirmação na Fase 3.

```json
{
  "meta": { "client": "santa-lucia", "intakeDate": "2026-08-20", "assetsDir": "assets-source",
            "interactive": false, "niche": "veterinaria" },
  "credentials": { "node": "v20.11.0", "git": "ok", "gh": "ok", "vercel": "missing",
                   "imageGenerator": "chatgpt", "videoGenerator": "google-flow",
                   "checkedAt": "2026-08-20" },
  "business": {
    "name":     { "value": "Santa Lúcia Clínica Veterinária", "source": "doc:dados-google-da-empresa.txt" },
    "phone":    { "value": "(54) 3025-2223", "source": "doc:dados-google-da-empresa.txt", "confirmed": false },
    "whatsapp": { "value": null, "source": null },
    "hours":    { "value": "24 horas, todos os dias", "holidays": "inclusive feriados", "source": "doc:dados-google-da-empresa.txt" },
    "license":  { "type": "CRMV", "value": null, "holder": null, "source": null }
  },
  "services": [ { "name": "pronto-socorro 24h", "group": "clinical", "source": "doc:conteudo-textual-site.md", "confirmed": false } ],
  "doesNotOffer": []
}
```

Campo completo por campo, com o exemplo inteiro da Santa Lúcia e as sete regras estruturais:
[briefing-schema.md](briefing-schema.md). Duas delas valem repetir porque são o erro caro desta
fase: **`whatsapp` e `phone` são campos separados e ambos obrigatórios** — preencher um com o valor
do outro é o defeito mais comum do intake; e **`facts` e `assumptions` nunca se misturam**, porque
o downstream renderiza valor com `source` sem checar e não pode renderizar suposição.

Os blocos `assets` (escrito por `estudo-assets` em `inventario.json`) e `conflicts`, `gaps`,
`assumptions`, `unverified` (escritos por `auditoria-dados`) não são preenchidos aqui. Crie-os
vazios e siga.

## Fronteira com as skills vizinhas

| Pergunta | Skill dona |
|---|---|
| "O cliente tem acesso e o que ele diz que é?" | **`briefing-cliente`** (aqui) |
| "O que os arquivos mostram e quais servem?" | `estudo-assets`, Fase 1 |
| "De que cor e forma a marca é feita?" | `brand-dna-extractor`, Fase 2 |
| "O que falta e o que se contradiz?" | `auditoria-dados`, Fase 3 |

Não pergunte aqui nada que um asset possa responder. A ordem existe por isso: perguntar antes de
ler queima a única rodada de perguntas com dados que estavam na placa da fachada.

## Portão

- [ ] `node -e "JSON.parse(require('fs').readFileSync('design/briefing.json','utf8'))"` sai limpo
- [ ] `credentials` preenchido com `checkedAt` de hoje
- [ ] O dev confirmou que tem acesso a um gerador de imagem
- [ ] `assets-source/` existe e tem pelo menos um logo dentro
- [ ] Todo campo de `business` preenchido tem `source`; o que ninguém informou está `null`, não chutado
- [ ] `whatsapp` e `phone` têm valores diferentes, ou o dev confirmou por escrito que são o mesmo
- [ ] Nenhum token foi digitado no chat ou escrito em arquivo do projeto

Reprovou por falta de `gh` ou `vercel`: não bloqueia agora — anote a fase de trava e siga. Reprovou
por falta de assets: bloqueia. Sem asset, a Fase 2 vira chute.

## Anti-patterns

- **Pedir dado em conta-gotas** — cada rodada é um telefonema do dev para o cliente. Na terceira, o
  cliente para de atender. Um pedido, o template inteiro.
- **Insistir num campo em branco** — o dev leigo frequentemente não sabe mesmo. Deixe vazio; a Fase
  3 pergunta uma vez só, já sabendo o que os assets responderam.
- **Preencher `whatsapp` com o telefone fixo** — o botão principal da página abre uma conversa que
  ninguém lê, e ninguém descobre por semanas.
- **Copiar copy do site atual para o briefing** — o site atual é fonte de fato e de conflito. A copy
  é escrita do zero na Fase 6, e reaproveitar texto é como o projeto herda o problema que veio
  resolver.
- **Aceitar "depois eu mando o logo em alta"** — a resolução some do radar e reaparece no build,
  quando trocar custa uma rodada inteira de assets. Peça agora ou registre como bloqueante.
- **Adiar a checagem de credencial para a publicação** — `gh auth login` num dev que nunca usou
  GitHub é meia hora. Meia hora na Fase 0 é planejamento; na Fase 12 é o projeto parado.
- **Aceitar token colado no chat** — o cofre do sistema existe. Token em transcrição vaza junto com
  a transcrição, e a única correção real é rotacionar a credencial.

## Handoff

`design/briefing.json` é lido, em ordem, por `estudo-assets` (que escreve o inventário ao lado, sem
tocar neste arquivo), `brand-dna-extractor`, `auditoria-dados`, `niche-research`, `estrutura-secoes`
e `copy-conversao` (serviços, diferenciais e `doesNotOffer`, que define o que a copy não pode
prometer) e a fase de implementação, onde contato, horário e endereço viram JSON-LD.

Ele é **referenciado, nunca copiado**. Um telefone corrigido na Fase 3 propaga sozinho; um telefone
colado em cinco arquivos vira cinco versões do negócio, e a página publica a errada.
