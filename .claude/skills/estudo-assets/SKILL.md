---
name: estudo-assets
description: Use when a folder of client assets must be inventoried and every single image actually opened and read before any design decision. Fase 1 do pipeline: le cada imagem uma por uma, nunca classifica pelo nome do arquivo, classifica por tipo e fato que carrega, e da dois vereditos por arquivo — serve na pagina e em que largura, e serve como referencia para anexar no ChatGPT. Triagem de asset inutil. Produz design/inventario.json.
argument-hint: [pasta-de-assets]
allowed-tools: Read, Glob, Write, Edit, Bash(node *), Bash(ls *), Bash(mkdir *)
---

# Estudo dos assets — Fase 1

| | |
|---|---|
| **ENTRADA** | A pasta `assets-source/`, e `design/briefing.json` para saber o nicho e o nome oficial |
| **SAÍDA** | `design/inventario.json` — uma entrada por arquivo, com dois vereditos independentes |
| **ANTES** | `briefing-cliente` (Fase 0) |
| **DEPOIS** | `brand-dna-extractor` (Fase 2), que amostra cor e forma dos arquivos aprovados aqui |

O inventário é feito **uma vez, aqui**. `brand-dna-extractor` lê `inventario.json` em vez de
re-listar a pasta, e confia no veredito de qualidade que já foi dado. Este arquivo não toca em
`briefing.json`: ele mora ao lado.

A fronteira em uma linha: aqui a pergunta é "o que esta imagem mostra, é verdade e podemos
publicar?". Em `brand-dna-extractor` a pergunta é "de que cor e forma a marca é feita?".

## Regra dura: leia cada imagem

**Nunca classifique, descreva ou amostre cor de um arquivo que você não abriu.** O nome do arquivo é
a intenção de quem salvou, não o conteúdo, e mente com frequência: `fachada-clinica-vertical.png` na
Santa Lúcia tem 345×451 e é um recorte, não uma fachada. `fachada-clinica-thumb.jpg` tem 260×175 e
não é utilizável. Só `fachada-clinica-rua.webp`, com 1360×1020, é uma fachada de verdade.

Abra com `Read`, uma por uma, e anote cinco coisas por arquivo:

1. **O que está em quadro**, em uma frase concreta: "duas mãos com luva azul segurando um gato cinza
   sobre mesa de inox" — não "atendimento veterinário".
2. **Que fato ela carrega**: telefone na placa, horário no post, equipamento visível, número de
   pessoas na equipe, selo de "24 h" no logo.
3. **Onde é o espaço vazio**: área lisa suficiente para copy cair por cima faz a imagem virar fundo
   de seção. Cheia até as bordas, ela é card ou nada.
4. **Problema de direito de uso**: marca d'água, rosto identificável, marca de terceiro em quadro.
5. **Se contradiz outra fonte**: anote e leve para `auditoria-dados`. Você não resolve o conflito
   aqui, mas é aqui que ele aparece.

Um asset sem `shows` preenchido é a prova de que o arquivo não foi aberto. É o item de portão mais
fácil de verificar e o mais fácil de burlar por descuido.

## Inventário automático

O projeto npm ainda não existe, então o inventário não pode depender de `sharp`. O script sem
dependências lê largura e altura direto do cabeçalho de PNG, JPEG e WebP. Código completo em
[asset-triage.md](asset-triage.md#script-de-inventário); salve em `scripts/asset-inventory.mjs` e
rode:

```bash
mkdir -p scripts design
node scripts/asset-inventory.mjs assets-source
```

Saída real da Santa Lúcia:

```
hero-veterinaria-com-gato.jpg     1672x941    182 KB   full-bleed
fachada-clinica-rua.webp          1360x1020   195 KB   meia-largura
post-banho-e-tosa.jpg             1020x1020   303 KB   meia-largura
logo-santa-lucia-fundo-roxo.jpg    360x640     11 KB   thumb/prova
fachada-clinica-thumb.jpg          260x175     23 KB   só evidência
```

Três decisões saem daí antes de qualquer leitura visual: o hero existe, a fachada só serve a meia
largura, e **o logo é um JPEG de 11 KB sobre fundo chapado** — sem canal alpha, com artefato de
compressão em volta das letras. Isso vira pedido de original, e vira pedido na Fase 3, não na hora
do build.

`ILEGÍVEL` na saída significa formato fora dos três (HEIC de iPhone, AVIF, TIFF, PSD) ou arquivo
corrompido. HEIC é o caso comum — o cliente mandou direto do iPhone. Peça reexportação ou converta;
não adivinhe as dimensões. SVG não tem dimensão em pixel e não precisa: abra e confirme que é vetor
de verdade, não um `<image>` bitmap embrulhado em SVG, o que é frequente em arquivo de designer
apressado.

## Os dois vereditos, que são independentes

Cada arquivo recebe duas decisões que não se implicam:

| Veredito | Campo | A pergunta |
|---|---|---|
| Renderiza na página? | `maxRenderWidth` | Em que largura CSS este arquivo aparece sem borrar? `0` significa "só evidência" |
| Serve de referência? | `reference` | O dev anexa este arquivo ao prompt no ChatGPT para segurar a marca? |

O logo JPEG de 360 px da Santa Lúcia é `maxRenderWidth: 0` e `reference: true`: não entra no
cabeçalho e é o **melhor anexo possível** para manter as imagens geradas dentro da paleta. Tratar os
dois como um veredito só é o erro que faz o dev gerar oito imagens sem referência de marca porque "o
logo não presta".

`maxRenderWidth` sai do maior lado do arquivo, com a conta `largura CSS × 2` para retina:

| Maior lado | `maxRenderWidth` | Serve para |
|---|---|---|
| ≥2400 px | 1600+ | Hero full-bleed em desktop grande |
| 1600–2400 px | 1200 | Full-bleed até 1200 px CSS |
| 1000–1600 px | 700 | Coluna de meia largura, card grande |
| 600–1000 px | 400 | Thumbnail, prova, avatar, card pequeno |
| <600 px | 0 | Nada. Só evidência para leitura e amostragem de cor |

**Nunca faça upscale.** Upscale comum borra; upscale de IA inventa textura e deixa a foto plástica
ao lado das outras. Asset pequeno demais é pedido de original, não problema de pipeline. Os limiares
do logo são outros — SVG ideal, PNG com alpha ≥512 px serve, JPEG nunca serve para cabeçalho — e
estão em [asset-triage.md](asset-triage.md#logo-os-limiares-são-outros).

## Classificação por tipo

Cada tipo entrega um tipo diferente de evidência **factual**. Cor e forma são assunto de
`brand-dna-extractor`, não daqui.

| Tipo | Fato que costuma carregar | Renderiza? | Cuidado |
|---|---|---|---|
| `logo` | Nome oficial, selo ("24 h", "desde 2015"), tagline | Sim, se tiver alpha e ≥512 px no maior lado | JPEG de logo não tem transparência |
| `post` | Serviços com a redação do próprio cliente, promessas, horário, telefone | Raramente — traz texto queimado e proporção de feed | O texto do post é a fonte de copy do cliente, não a nossa |
| `fachada` | Endereço na placa, telefone na placa, horário, se há estacionamento | Sim — é a prova de que o lugar existe | Nunca gerar por IA. Se está feia, é a fachada dele |
| `interior` | Equipamento que o cliente realmente tem | Sim | Não afirme equipamento que não aparece em nenhuma foto |
| `equipe` | Quantas pessoas, uniforme, crachá com registro | Só com autorização de imagem por escrito | Sem autorização: enquadramento sem rosto, ou nenhuma |
| `produto` | O que o serviço entrega na prática | Sim — é o ativo mais subestimado da pasta | Confirme que a foto é do trabalho dele |
| `impresso` | Tabela de preço, lista completa de serviços, CNPJ, registro profissional | Não | Preço impresso costuma estar desatualizado — confirme antes de publicar |
| `print` | Depoimento, horário, confirmação de dado | Não, nunca | Contém dado pessoal de terceiro; é evidência e não sobe ao repositório (`repo: false`) |
| `video` | O local em movimento, o trabalho sendo feito | Sim — candidato a timeline de scroll | Áudio com voz de terceiro precisa da mesma autorização que rosto |

## Triagem: o que não serve

| Sintoma | Por que descarta | Destino |
|---|---|---|
| Maior lado <600 px | Upscale de IA fica plástico; upscale normal fica borrado | `maxRenderWidth: 0`. Peça o original |
| Marca d'água de terceiro (fotógrafo, Freepik, Shutterstock) | Publicar é violação de licença do cliente, não opinião nossa | `usable: false`. Peça o arquivo licenciado ou substitua |
| Foto de banco de imagem que o cliente não comprou | Mesma coisa, e o visitante reconhece stock | `usable: false` |
| Print de Instagram com barra de curtidas | O cromo da interface aparece na página | Evidência. Peça o arquivo original do post |
| Duplicata em tamanhos diferentes | Duas verdades para o mesmo asset | Fica o maior; registre o descarte com o motivo |
| Marca de concorrente em quadro | Publicidade gratuita para o outro | Fora, salvo recorte que elimine a marca |
| Rosto de cliente ou terceiro identificável | Direito de imagem | Só com autorização escrita em `rights`; senão, recorte sem rosto |

Como cada marca d'água aparece na prática, e o que fazer com cada descarte:
[asset-triage.md](asset-triage.md#detecção-de-problema-de-direito-de-uso).

**Descartar em silêncio é pior que não descartar.** Toda exclusão entra no inventário com
`usable: false` e `reason` escrito, porque a pergunta "cadê aquela foto que eu mandei?" sempre
aparece, e sem o registro a resposta é um encolher de ombros.

## O artefato: `design/inventario.json`

```json
{
  "meta": { "assetsDir": "assets-source", "readAt": "2026-08-20", "count": 9 },
  "assets": [
    {
      "file": "hero-veterinaria-com-gato.jpg",
      "type": "produto",
      "dimensions": "1672x941",
      "sizeKb": 182,
      "usable": true,
      "maxRenderWidth": 1200,
      "reason": "",
      "shows": "profissional de jaleco segurando um gato cinza, mesa clara ao fundo",
      "carries": ["uniforme da equipe", "ambiente de consultório"],
      "negativeSpace": "left",
      "rights": "cliente",
      "reference": true,
      "repo": true
    },
    {
      "file": "logo-santa-lucia-fundo-roxo.jpg",
      "type": "logo",
      "dimensions": "360x640",
      "sizeKb": 11,
      "usable": false,
      "maxRenderWidth": 0,
      "reason": "JPEG sobre fundo chapado: sem canal alpha e com artefato de compressão nas letras. Pedir SVG, AI, EPS ou PNG com transparência.",
      "shows": "marca sobre fundo roxo, com selo de 24h",
      "carries": ["nome oficial", "selo 24h"],
      "negativeSpace": "none",
      "rights": "cliente",
      "reference": true,
      "repo": true
    }
  ],
  "discarded": [
    { "file": "fachada-clinica-thumb.jpg", "reason": "260 px no maior lado; duplicata pequena de fachada-clinica-rua.webp" }
  ],
  "contradictions": [
    { "file": "post-atendimento-24h-pets-nao-convencionais.jpg",
      "claim": "Especialistas em pets não convencionais, atendimento 24h",
      "conflictsWith": "doc:conteudo-textual-site.md" }
  ]
}
```

Quatro regras estruturais:

1. **`usable: false` exige `reason`.** Descarte sem motivo registrado vira discussão depois.
2. **`shows` é obrigatório em todo asset.** É a única prova de que o arquivo foi aberto.
3. **`repo: false`** em todo print de conversa, contrato ou documento com dado pessoal. Esse arquivo
   não entra no commit, e o histórico do git guarda o que foi apagado.
4. **`contradictions` é uma lista de sinais, não de decisões.** Você aponta; `auditoria-dados`
   pergunta. Escolher a fonte "mais provável" aqui publica uma afirmação que outra fonte do próprio
   cliente nega.

Os prefixos de procedência (`asset:` / `doc:` / `dev:` / `web:`) são os mesmos de
[briefing-schema.md](../briefing-cliente/briefing-schema.md).

## Portão

- [ ] `node -e "JSON.parse(require('fs').readFileSync('design/inventario.json','utf8'))"` sai limpo
- [ ] O número de entradas em `assets` + `discarded` bate com `ls assets-source | wc -l`
- [ ] Toda entrada tem `shows` preenchido — nenhuma classificada pelo nome do arquivo
- [ ] Toda entrada tem `dimensions`, `usable`, `maxRenderWidth`, `reference`, `rights` e `repo`
- [ ] Todo `usable: false` tem `reason` escrito em português, não vazio
- [ ] Todo asset com `rights: "desconhecido"` ou `"proibido"` está `usable: false`
- [ ] Existe pelo menos um asset com `reference: true`, ou o pedido do logo original está anotado
- [ ] Nenhum print com dado pessoal de terceiro está com `repo: true`

Reprovou: reabra os arquivos. Aqui custa minutos. O mesmo erro descoberto na Fase 8 vira prompt
escrito em cima de um asset que não existe, e uma rodada inteira de geração jogada fora.

## Anti-patterns

- **Classificar imagem pelo nome do arquivo** — `fachada-clinica-vertical.png` na Santa Lúcia é um
  recorte de 345 px. O nome descreve a intenção de quem salvou, não o conteúdo.
- **Confundir os dois vereditos** — um logo pequeno demais para o cabeçalho continua sendo o melhor
  anexo possível para o ChatGPT. Tratar `usable` e `reference` como a mesma coisa faz a Fase 8 gerar
  imagem sem referência de marca.
- **Fazer upscale de asset pequeno** — upscale de IA inventa textura, e a foto fica plástica ao lado
  das reais. O contraste entre elas é mais visível que a baixa resolução teria sido.
- **Descartar em silêncio** — sem `reason` registrado, "cadê aquela foto?" não tem resposta.
- **Amostrar cor aqui** — cor é `brand-dna-extractor`. Duas skills amostrando a mesma imagem é a
  origem de "três roxos" no design system.
- **Resolver contradição encontrada num asset** — anote em `contradictions` e siga. Decidir aqui é
  apostar em qual fonte do cliente está certa, sem perguntar a ele.
- **Deixar print de WhatsApp na pasta versionada** — dado pessoal de terceiro no repositório é
  problema de LGPD do cliente, e o git guarda mesmo depois do `rm`.

## Handoff

`design/inventario.json` é lido por `brand-dna-extractor` (amostra os pixels dos arquivos aprovados,
sem re-listar a pasta), `auditoria-dados` (cruza o que os assets responderam com o que o briefing
deixou em branco) e `prompt-imagem`, que depende de dois campos daqui:
`maxRenderWidth > 0` diz quais seções já têm foto real e portanto **não** recebem imagem gerada;
`reference: true` diz quais arquivos o dev anexa ao prompt no ChatGPT — no máximo três por bloco.
A Fase 10 lê `maxRenderWidth` de novo para dimensionar as renditions do pipeline de mídia.
