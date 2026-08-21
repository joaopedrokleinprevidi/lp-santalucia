# Comece por aqui

Você tem um repositório vazio, esta pasta de skills, e um cliente. Isto é o que fazer.

## O comando

Crie a pasta do projeto, abra o Claude Code **dentro dela** e escreva:

```
/landing-page-factory assets-source "Nome do Cliente"
```

Ou, em português mesmo: *"roda a landing page factory para o cliente Fulano, em Barreiro"*.
O efeito é o mesmo. A pasta em que você abriu o Claude Code é a pasta do projeto — tudo nasce
dentro dela.

**A pasta de assets pode estar vazia.** A Fase 0b existe para enchê-la.

## O que ter antes de dizer "gere"

| # | O quê | Trava na fase | Como |
|---|---|---|---|
| 1 | **Node ≥20 e git** | 10 | [nodejs.org](https://nodejs.org) (LTS), [git-scm.com](https://git-scm.com) |
| 2 | **Conta no GitHub** + `gh auth login` | 12 | Grátis |
| 3 | **Conta na Vercel** (MCP ou CLI) | 12 | Grátis |
| 4 | **AIX Downloader** e **MarkDownload**, extensões do Chrome | **0b** | Grátis, um minuto. Baixam o Instagram e o site do cliente |
| 5 | **ChatGPT** com geração de imagem | 9 | Sem ele a página fica sem as imagens de seção |
| 6 | **Google Flow** (opcional) | 9 | Sem ele a página usa imagens paradas e continua boa |
| 7 | **Nome da empresa e a cidade** | 0 | É o mínimo. O resto eu garimpo ou peço |

O passo a passo de cada uma, e o que fazer quando o navegador não abre:
[landing-page-factory/credentials.md](landing-page-factory/credentials.md).

## O seu trabalho inteiro

1. Baixar o que só você alcança, pelo roteiro da Fase 0b — o que está na internet aberta eu busco.
2. Dizer "gere".
3. Colar prompts no ChatGPT (imagens) e no Google Flow (clipes), salvando onde cada bloco indica.
4. Aprovar o login no navegador quando uma credencial for pedida.

Não existe passo 5. Você não precisa saber programar.

**Leia isto antes de começar:** [landing-page-factory/para-leigos.md](landing-page-factory/para-leigos.md)
— o processo inteiro sem jargão, fase por fase, com o que esperam de você em cada uma.

## Como as 25 skills se encaixam

`landing-page-factory` é a dona da numeração das 13 fases e chama as outras 24 na ordem. Você
invoca **só ela**; as outras entram sozinhas quando chega a vez.

```
0 briefing-cliente → 0b coleta-dados → 1 estudo-assets → 2 brand-dna-extractor
→ 3 auditoria-dados → 4 niche-research → 5 creative-direction-expert
→ 6a estrutura-secoes → 6b copy-conversao → 7 creative-direction-expert
→ 8a prompt-imagem → 8b prompt-animacao → 9 você gera as mídias
→ 10 product-design-expert → frontend-design → landing-motion-expert
→ 11 audit-responsivo → audit-acessibilidade → audit-performance → 12 publicar-lp
```

Quatro paradas esperam por você: **0b** baixar, **3** responder, **6b** revisar e **9** gerar,
mais a autorização da **12**. As outras rodam sem você.

A numeração canônica vive em [landing-page-factory/SKILL.md](landing-page-factory/SKILL.md).
Se algum arquivo divergir dela, o arquivo está errado.
