# lp-santalucia

Landing page da **Santa Lúcia — Clínica Veterinária & Pet Center**, clínica com atendimento 24 horas em Caxias do Sul (RS).

Este repositório reúne, por enquanto, os materiais de base do projeto: identidade visual, conteúdo textual, dados da empresa e o workflow de especialistas que conduz a implementação.

## Estrutura

```
.claude/skills/     14 especialistas: marca, storytelling, design, motion, vídeo, auditoria
assets/             Imagens da marca, fachada e posts sociais
design/             Design system, blueprint da LP, prompts de imagem e animação
docs/               Conteúdo textual do site antigo e dados da empresa
CLAUDE.md           Workflow obrigatório e critérios de qualidade
```

## Estado do projeto

As quatro primeiras fases do pipeline estão prontas em [design/](design/):

| Artefato | O que é |
| --- | --- |
| [design-system.json](design/design-system.json) | Cores medidas pixel a pixel dos assets, tipografia, motifs, voz e os fatos verificáveis do negócio |
| [landing-blueprint.md](design/landing-blueprint.md) | 11 seções com a copy definitiva, escrita do zero |
| [image-prompts.md](design/image-prompts.md) | 8 prompts para gerar as imagens no GPT, com o Style Anchor congelado |
| [motion-prompts.md](design/motion-prompts.md) | 7 prompts de animação para o Google Flow, com a técnica de destino de cada seção |

Não existe `coleta.json` aqui: o projeto foi montado antes da Fase 0b (skill `coleta-dados`)
existir, e ela nunca rodou completa para a Santa Lúcia. Uma execução nova começaria pelo garimpo,
antes do briefing — a rota 1 rodada como teste já devolveu um candidato a WhatsApp e os canais
digitais, registrados em [design/README.md](design/README.md).

O que falta: gerar as imagens, animá-las, e construir o site. O passo a passo está em
[design/README.md](design/README.md), e o pipeline completo na skill `landing-page-factory`.

## Assets

| Arquivo | Uso |
| --- | --- |
| `logo-santa-lucia-fundo-roxo.jpg` | Logo em fundo roxo (formato story) |
| `hero-veterinaria-com-gato.jpg` | Foto ampla para hero |
| `fachada-clinica-rua.webp` | Fachada vista da rua |
| `fachada-clinica-vertical.png` | Fachada em enquadramento vertical |
| `fachada-clinica-thumb.jpg` | Fachada em miniatura |
| `post-atendimento-24h-pets-nao-convencionais.jpg` | Post: pets não convencionais |
| `post-banho-e-tosa.jpg` | Post: banho e tosa |
| `post-exclusividade-felina.jpg` | Post: exclusividade felina |
| `post-farmacia-completa-24h.jpg` | Post: farmácia 24h |

## Identidade

Roxo e amarelo são as cores da marca, presentes na fachada, no logo e em todo o material social.

## Workflow

A implementação segue a ordem definida em [CLAUDE.md](CLAUDE.md): direção criativa, storytelling, design de produto, motion, scroll e, por fim, responsividade e acessibilidade. Cada etapa tem um especialista dedicado em [.claude/skills/](.claude/skills/).

## Negócio

- Endereço: R. Jacob Luchesi, 3230 — Santa Lúcia, Caxias do Sul (RS), CEP 95032-000
- Telefone: (54) 3025-2223
- Atendimento: 24 horas, todos os dias
- Google: 4,8/5 com mais de 1.100 avaliações
