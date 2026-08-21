---
name: "coleta-dados"
description: "Use when the project starts from almost nothing — a business name and a city — and the material still has to be harvested from Google Business, the current site, Instagram and reviews. Fase 0b do pipeline: divide o garimpo entre o que eu busco sozinho na web e o que so o dev consegue baixar, com o roteiro passo a passo de cada fonte e as extensoes de Chrome. Produz assets-source/ populado e design/coleta.json."
argument-hint: "[nome-da-empresa] [cidade]"
allowed-tools: WebSearch, WebFetch, Read, Write, Edit, Glob, AskUserQuestion, Bash(mkdir *), Bash(ls *)
---

# Coleta de dados — Fase 0b

| | |
|---|---|
| **ENTRADA** | O nome da empresa e a cidade. É o mínimo que o dev sabe de cor |
| **SAÍDA** | `assets-source/` populado e `design/coleta.json` com tudo o que veio da web, marcado como não confirmado |
| **ANTES** | `briefing-cliente` (Fase 0), que já fixou a pasta e checou credenciais |
| **DEPOIS** | `estudo-assets` (Fase 1), que abre e lê cada arquivo baixado |

O dev abre um repositório vazio e pergunta "o que eu preciso?". A resposta não é um formulário —
é um **roteiro de garimpo**, dividido entre o que eu busco sozinho e o que só ele consegue.

Essa divisão é o ponto da skill. Metade do briefing está em fonte pública e eu alcanço com
`WebSearch` e `WebFetch`. Mandar o dev copiar à mão o que eu leio em dez segundos é queimar a
paciência dele no primeiro dia.

## A divisão do trabalho

| Eu busco sozinho | Só você consegue | Por quê |
|---|---|---|
| Google Business: endereço, telefone, horário, nota, nº de avaliações, categoria | **Logo em vetor ou alta resolução** | Está no e-mail do designer, não na web. O do site é 200 px e não sobe em retina |
| Todo o texto do site atual: serviços, FAQ, sobre | **Fotos originais do Instagram** | A web serve versão comprimida; a original vem pela extensão |
| Avaliações públicas do Google, na íntegra | **Fotos que só existem no celular do cliente** | Interior, equipe, trabalho feito. É a prova de que o lugar existe |
| Handles, bio e links das redes públicas | **WhatsApp comercial confirmado** | Nunca está publicado, e é o CTA principal da página |
| 3 a 5 concorrentes locais e o que eles prometem | **CNPJ e registro profissional** | Documento interno |
| Menções, notícias, diretórios do setor | **Autorização de uso de imagem e depoimento** | Exige o cliente dizendo sim, por escrito |
| | **Faixa de preço** | Raramente publicada |
| | **O que a empresa NÃO faz** | Ninguém publica o que não vende, e é o campo que mais evita estrago |

**Regra de procedência:** tudo que eu busco entra como `web:` — parece fato e não é. Contato,
horário e preço vindos da web viram pergunta de confirmação na Fase 3, sem exceção. O motivo é
banal e caro: é exatamente o campo que o cliente esqueceu de atualizar em 2019.

---

## Rota 1 — Google Business (eu faço)

A fonte mais densa que existe: endereço, telefone, horário, categoria, nota e volume de
avaliações num lugar só.

```
WebSearch: "<nome da empresa>" <cidade> <segmento>
WebSearch: "<nome>" <cidade> telefone endereço horário
```

Depois `WebFetch` no site oficial e nos diretórios que aparecerem. Diretório de nicho (Petlove,
Doctoralia, associação de classe) costuma trazer o WhatsApp que o site esconde — foi assim que o
número comercial da Santa Lúcia apareceu, depois de ele constar como pendência bloqueante.

O que colher, cada um com a URL de origem: nome exato, endereço com CEP, telefone fixo, celular
ou WhatsApp, horário, categoria, nota, nº de avaliações, ano de fundação e o link do perfil.

**Não é confiável:** horário em feriado, preço, e "aberto agora". Marque para confirmação.

## Rota 2 — Site atual (o dev baixa, eu leio)

O site atual é fonte de **fato** e de **conflito**. Nunca de copy — se o site convertesse, não
haveria projeto.

Eu leio com `WebFetch` página a página. Quando o site é grande, ou tem blog, o dev baixa de uma vez:

> **MarkDownload** — extensão do Chrome que salva a página inteira como `.md`, texto limpo,
> sem menu e sem rodapé.
> [chromewebstore.google.com/detail/markdownload](https://chromewebstore.google.com/detail/markdownload/pklblaefkkcmofjcobdmcflmdphecpne)
>
> Instale, abra cada página do site, clique no ícone, salve em `assets-source/site/`.
> Vale para: home, serviços, sobre, FAQ, contato e os posts do blog que ainda fazem sentido.

Colher: lista de serviços com a redação do cliente, FAQ (viram perguntas reais da página nova),
números que ele reivindica, e **toda contradição com o Google Business** — divergência entre
duas fontes do próprio cliente é item bloqueante, não suposição minha.

## Rota 3 — Instagram e Facebook (o dev baixa)

É a fonte mais rica de identidade visual: a marca aplicada de verdade, com a tipografia, os
motifs e a voz que o cliente usa.

> **AIX Downloader** — extensão do Chrome que baixa todas as imagens e vídeos de uma página de
> uma vez. Versão 9.0.58, ~200 mil usuários.
> [chromewebstore.google.com/detail/aix-downloader](https://chromewebstore.google.com/detail/aix-downloaderimagem-v%C3%ADdeo/ibdfeimkglcmdejppabkaidpippniiob)
>
> Instale, abra o perfil, **role até o fim do feed** (o Instagram só carrega o que está na tela),
> clique no ícone, selecione tudo e baixe em `assets-source/instagram/`.

Peça os **50 posts mais recentes**, ou o ano inteiro se postarem pouco. Menos que 20 não dá
amostra de identidade; mais que 100 é peso morto na Fase 1, que abre uma por uma.

Junto, anote do perfil: handle exato, bio, link, nº de seguidores, e quais 3 posts tiveram mais
engajamento — esses dizem o que o público dele responde, e viram ângulo de copy na Fase 6.

**Não sirvo para isso:** o Instagram exige login e bloqueia leitura automatizada. É a extensão
ou é a mão.

## Rota 4 — Avaliações (eu faço)

As avaliações são o material de copy mais barato que existe, porque estão nas palavras do cliente
final, não nas nossas.

Colher literalmente: o que elogiam (vira copy e ordem de seção), o que reclamam (vira objeção a
derrubar), e as palavras que se repetem. Guarde o texto integral das 10 melhores em
`design/coleta.json`.

**Direito de imagem:** avaliação pública pode ser citada. Nome completo e foto **não**, sem
autorização por escrito. Sem ela: texto da avaliação, primeiro nome, sem foto.

## Rota 5 — O que só o cliente tem

Depois das quatro rotas, o que sobrar vai numa lista curta para o dev levar. Sempre sobra isto:

- Logo em vetor (`.ai`, `.svg`, `.pdf`) ou PNG com transparência acima de 1000 px
- WhatsApp comercial, confirmado, com DDD
- Fotos do interior, da equipe e do trabalho sendo feito — celular serve
- CNPJ e registro profissional do conselho, quando o nicho exige
- Faixa de preço, e o que a empresa não faz
- Autorização de depoimento, se for usar nome

Uma frase pronta para o dev mandar ao cliente está em [pedido-ao-cliente.md](pedido-ao-cliente.md).

---

## O artefato

`design/coleta.json`. Todo campo carrega `source` com a URL e `confirmed: false`.

```json
{
  "coletadoEm": "2026-08-21",
  "fontes": [
    { "tipo": "google-business", "url": "...", "colhidoPor": "websearch" },
    { "tipo": "site", "url": "...", "colhidoPor": "webfetch" },
    { "tipo": "instagram", "url": "...", "colhidoPor": "dev+aix-downloader", "arquivos": 47 }
  ],
  "business": {
    "whatsapp": { "value": "(54) 99967-4276", "source": "web:diretorio-petlove", "confirmed": false }
  },
  "avaliacoes": [ { "texto": "...", "nota": 5, "fonte": "google" } ],
  "conflitos": [
    { "campo": "espécies atendidas",
      "fonteA": "post do Instagram: atende pets não convencionais 24h",
      "fonteB": "FAQ do site: foco em cães e gatos",
      "acao": "confirmar com o cliente — bloqueante" }
  ],
  "naoEncontrado": ["CNPJ", "faixa de preço", "autorização de depoimento"]
}
```

`briefing-cliente` mescla isto em `briefing.json`. O que veio como `web:` continua `web:` até o
cliente confirmar — a mesclagem não promove procedência.

## Portão

- [ ] `design/coleta.json` parseia e todo campo tem `source`
- [ ] `assets-source/` tem logo, e ao menos 20 imagens entre posts e fotos
- [ ] Nenhum campo `web:` de contato, horário ou preço marcado como `confirmed: true`
- [ ] Os conflitos entre fontes estão listados, não resolvidos por mim
- [ ] A lista do que só o cliente tem foi entregue ao dev

Reprovou por falta de logo ou de fotos: bloqueia. Sem asset real a Fase 2 vira chute e a página
inteira vira ilustração.

## Anti-patterns

- **Mandar o dev copiar o que eu leio** — Google Business e site inteiro eu busco em segundos.
  Cada tarefa manual desnecessária é atrito no serviço dele.
- **Tratar dado da web como confirmado** — é o campo desatualizado que vira ligação perdida.
- **Print de tela de post** — perde resolução e ganha barra de status. Arquivo original, sempre.
- **Baixar o Instagram sem rolar até o fim** — a extensão só vê o que carregou; vêm 12 posts e
  a amostra de identidade fica enviesada nos mais recentes.
- **Usar o logo do site** — quase sempre 200 px, otimizado para carregar rápido, não para imprimir.
- **Resolver conflito entre fontes sozinho** — duas fontes do próprio cliente se negando exige o
  cliente, sempre.
- **Citar depoimento com nome completo e foto sem autorização** — direito de imagem e LGPD.
- **Copiar a copy do site atual** — é fonte de fato, nunca de redação.
