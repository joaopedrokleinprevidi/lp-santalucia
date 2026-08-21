# Credenciais e acessos

Verifique tudo **antes da Fase 1**. Descobrir na hora de publicar que falta um acesso custa o
projeto inteiro em retrabalho.

## Verificação rápida

```bash
gh auth status     # GitHub
vercel whoami      # Vercel
node --version     # precisa ser v20 ou superior
git --version
```

Os dois primeiros falhando é normal na primeira vez. Siga as seções abaixo.

---

## Regra que não se quebra

**Nenhum token, senha ou chave de API é digitado no chat, colado num arquivo do projeto, ou
enviado para alguém.**

As ferramentas oficiais (`gh auth login`, `vercel login`) abrem o navegador, você aprova lá, e
a credencial fica guardada no cofre do seu sistema operacional. Ninguém precisa ver o token —
nem você.

Quando uma chave precisa mesmo viver no projeto (uma API de terceiro, por exemplo), ela vai em
`.env.local`, que já está no `.gitignore` e nunca sobe para o GitHub. Na Vercel, a mesma chave
é cadastrada por `vercel env add`, não commitada.

Se alguém — pessoa ou ferramenta — pedir para você colar um token numa conversa, a resposta é
não. Não existe fluxo legítimo que precise disso.

---

## GitHub

Onde o código fica guardado, versionado e público (ou privado, se preferir).

**Criar conta:** [github.com/signup](https://github.com/signup). Grátis.

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

Responda: `GitHub.com` → `HTTPS` → `Y` para autenticar o Git → `Login with a web browser`.
Ele mostra um código de 8 caracteres, abre o navegador, você cola o código e aprova.

**Confirmar:**

```bash
gh auth status
```

Precisa mostrar `Logged in to github.com account <seu-usuario>` e, nos escopos, `repo`.

**Se der errado:** `gh auth logout` e repita. Em rede corporativa com proxy, o navegador pode
não abrir — use `gh auth login --with-token` lendo de um arquivo local que você apaga depois.

---

## Vercel

Onde o site fica no ar. O plano gratuito (Hobby) atende landing page com folga.

**Criar conta:** [vercel.com/signup](https://vercel.com/signup). Entre **com o GitHub** — isso
já conecta os dois e economiza um passo.

**Instalar a CLI:**

```bash
npm i -g vercel
```

**Conectar:**

```bash
vercel login
```

Escolha `Continue with GitHub`. Abre o navegador, você aprova, pronto.

**Confirmar:**

```bash
vercel whoami
```

**Publicar, quando chegar a Fase 7:**

```bash
vercel link      # associa a pasta a um projeto na Vercel
vercel --prod    # publica em produção
```

O primeiro `vercel link` pergunta o escopo (sua conta ou um time) e se quer criar projeto novo.
Aceite os padrões e confirme o nome.

**Domínio próprio:** no painel da Vercel, projeto → Settings → Domains → Add. A Vercel mostra
os registros DNS para configurar no registrador do cliente (Registro.br, GoDaddy, etc). A
propagação leva de minutos a algumas horas.

---

## Gerador de imagem

Você gera as imagens manualmente, colando os prompts. Qualquer um serve:

| Ferramenta | Observação |
|---|---|
| ChatGPT (GPT Image) | Mais prático — cola o prompt e baixa. Precisa de plano pago para volume |
| Midjourney | Melhor controle estético, curva de aprendizado maior, usa Discord |
| Sora / outros | Funcionam; ajuste a sintaxe do prompt à ferramenta |

Não há integração automática. O arquivo `image-prompts.md` existe justamente para você copiar
e colar.

**Onde salvar:** `assets/generated/<nome-da-secao>.png`, na maior resolução que a ferramenta
oferecer. O pipeline de assets gera as versões leves depois — mas ele não consegue inventar
resolução que não existe.

---

## Google Flow (animação)

Transforma cada imagem gerada num clipe curto.

**Acesso:** [labs.google/flow](https://labs.google/flow). Disponibilidade e plano variam por
região — confirme antes de prometer prazo ao cliente.

**Uso:** suba a imagem da seção, cole o prompt de movimento correspondente do
`motion-prompts.md`, gere 5 segundos, baixe o MP4.

**Onde salvar:** `assets/generated/<nome-da-secao>.mp4`.

**Alternativas** se o Flow não estiver disponível: Runway Gen-3, Kling, Luma Dream Machine. Os
prompts do `motion-prompts.md` funcionam nas três com ajuste mínimo, porque descrevem
movimento e câmera, não sintaxe de ferramenta.

---

## Ferramentas locais

Já vêm resolvidas pelo `package.json` do projeto — você não instala nada globalmente:

| Pacote | Para quê |
|---|---|
| `ffmpeg-static`, `ffprobe-static` | Transcodar vídeo e extrair os frames |
| `sharp` | Gerar AVIF/WebP responsivos e os placeholders borrados |

**Nunca instale ffmpeg globalmente nem escreva um caminho de binário fixo em script.** O
caminho da sua máquina não existe na máquina do próximo — é a origem clássica de um pipeline
que "funciona aqui".

---

## Dados do cliente

Não é credencial, mas bloqueia igual. Antes da Fase 2, tenha **confirmado pelo cliente**:

- [ ] Telefone fixo e **número do WhatsApp** (costumam ser diferentes)
- [ ] Endereço completo com CEP
- [ ] Horário de funcionamento, incluindo feriados
- [ ] Lista de serviços que ele realmente oferece hoje
- [ ] O que ele **não** oferece (para não prometer por engano)
- [ ] Handles das redes sociais
- [ ] Se pode usar fotos de clientes/pets reais, e depoimentos com nome

O último item importa juridicamente: depoimento com nome real e foto de terceiro precisa de
autorização. Sem ela, use só o texto da avaliação pública.
