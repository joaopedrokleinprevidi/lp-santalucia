---
name: landing-page-factory
description: Use to run an entire landing page project end to end, from a folder of client assets to a live URL — extracting the brand, writing the structure and copy, generating one AI image per section, animating each into a short clip, building the scroll-driven Next.js site and deploying to GitHub and Vercel. Pipeline completo de landing page premium a partir dos assets do cliente até o site no ar. Orquestra todas as outras skills na ordem correta.
argument-hint: [pasta-de-assets] [nome-do-cliente]
---

# Landing Page Factory

Uma pasta com fotos e um telefone entram. Um site no ar sai.

Esta skill não faz o trabalho — ela **coordena** as outras doze na ordem certa, com um portão
de verificação entre cada etapa. É a diferença entre um time e um monte de especialistas
falando ao mesmo tempo.

> **Se você é leigo:** leia [para-leigos.md](para-leigos.md) primeiro. Explica o que cada
> etapa faz, o que você precisa ter em mãos e o que vai te ser pedido, sem jargão.

## Antes de começar: o que precisa existir

Rode esta verificação **antes da Fase 1**. Descobrir na Fase 6 que falta uma credencial custa
o projeto inteiro em retrabalho.

```bash
gh auth status          # GitHub conectado?
vercel whoami           # Vercel conectado?
node --version          # v20+
```

| Item | Obrigatório? | Como conseguir |
|---|---|---|
| Pasta com os assets do cliente | **Sim** | Logo, fotos, posts de rede social, material impresso |
| Dados de contato reais | **Sim** | Telefone, WhatsApp, endereço, horário. Confirmados pelo cliente, não deduzidos |
| GitHub autenticado (`gh auth login`) | **Sim** | [Credenciais](credentials.md#github) |
| Vercel autenticado (`vercel login`) | Só para publicar | [Credenciais](credentials.md#vercel) |
| Acesso a um gerador de imagem (ChatGPT/GPT Image, Midjourney, Sora) | Para as imagens | Você gera manualmente, colando os prompts |
| Acesso ao Google Flow / Veo | Para as animações | Você gera manualmente, colando os prompts |
| Domínio | Não | Vercel dá um `.vercel.app` de graça |

Detalhe de cada uma, incluindo o que fazer quando falta: [credentials.md](credentials.md).

**Regra de ouro sobre credenciais:** nunca peça, receba ou escreva um token no chat ou em
arquivo do projeto. `gh auth login` e `vercel login` abrem o navegador e guardam a credencial
no sistema. Chave que precise viver no projeto vai em `.env.local`, que já está no `.gitignore`.

## Decisões que travam a Fase 1

Se o usuário não disse, decida e **declare a decisão**, não pergunte:

| Decisão | Padrão | Quando mudar |
|---|---|---|
| Framework | **Next.js App Router** | Vite + React se não houver necessidade de SEO (app interno, página de evento fechada) |
| Experience Score | ★★★★☆ | ★★★★★ só se houver orçamento de imagem/vídeo e o cliente pedir |
| Idioma | pt-BR | — |
| Conversão | WhatsApp | Formulário só se o cliente não usa WhatsApp comercial |
| Hospedagem | Vercel | — |

Next.js é o padrão porque landing page é conteúdo público: precisa de `metadata`, JSON-LD,
sitemap, Open Graph e renderização no servidor. Vite não entrega isso sem trabalho manual.

---

## As sete fases

Cada fase tem um **artefato** e um **portão**. Não avance com o portão vermelho — o custo de
consertar dobra a cada fase que passa.

### Fase 1 — DNA da marca

**Skill:** `brand-dna-extractor`
**Artefato:** `design/design-system.json`

Amostra os pixels dos assets e produz cores medidas, classe tipográfica, formas, motifs, voz e
os fatos verificáveis do negócio. Separa `facts` (está num asset) de `inferences` (foi deduzido)
de `unverified` (bloqueia publicação).

**Portão:** o JSON parseia; toda cor tem o arquivo de origem ao lado; contraste calculado;
`typography.observed` traz a classe tipográfica e `typography.chosen` continua `null` — quem
escolhe a família é `frontend-design`, na Fase 5, e é lá que o teste de `ção`/`não`/`ú` roda.

### Fase 2 — Estrutura e copy

**Skills:** `creative-direction-expert` (passada 1) → `landing-storytelling-director` →
`creative-direction-expert` (passada 2)
**Artefato:** `design/landing-blueprint.md` + `design/creative-direction.json`

A ordem é a do CLAUDE.md: direção criativa primeiro. Na passada 1 ela fixa o Experience Score, o
`depth` total da página e em que posição da curva o pico cai — e nada por capítulo, porque os
capítulos ainda não existem. Então o storytelling escolhe o arquétipo, ordena as seções, escreve
a copy definitiva e reparte o `share`, convertendo-o em `scroll` / `scrollMobile` com
`scrollProp()`. Na passada 2 a direção criativa lê o story map pronto e ratifica: `band` por
capítulo, pontos, entradas, silêncio, e onde ficam o WOW major, os dois medium e os small.

Para projeto que vale o custo, gere **três arquétipos concorrentes** e julgue por três lentes
independentes (conversão, qualidade de copy em pt-BR, experiência), depois sintetize a
vencedora enxertando o melhor das outras. Uma estrutura escolhida às cegas é a origem da
maioria das LPs medianas.

**Portão:** copy 100% nova (nada reaproveitado do site antigo do cliente); todo número citado
existe no `design-system.json`; nenhuma promessa que caia em `unverified`; headline do hero
≤62 caracteres e de capítulo ≤56 (tetos de `landing-storytelling-director`, medidos contra os
tokens de tipo — não os aperte aqui); existe 1 WOW major.

### Fase 3 — Prompts de imagem

**Skill:** `ai-visual-prompt-director`
**Artefato:** `design/image-prompts.md`

Escreve o **Style Anchor** — um parágrafo em inglês, idêntico em todos os prompts, que carrega
luz, lente e paleta. É ele que faz oito imagens geradas separadamente parecerem o mesmo ensaio.
Depois, um bloco por seção.

**Portão:** o anchor está congelado; todo prompt tem `EXCLUDE: text, letters, logos,
watermarks`; a linha `COMPOSITION` diz onde fica o espaço vazio para a copy cair por cima;
nenhuma seção pede geração da fachada real ou da equipe real.

> **Você gera as imagens aqui, manualmente.** Cole cada prompt no ChatGPT (ou no gerador que
> preferir). Salve em `assets/generated/<secao>.png`, na maior resolução possível.

### Fase 4 — Prompts de animação

**Skill:** `ai-visual-prompt-director` + `video-to-website`
**Artefato:** `design/motion-prompts.md`

Cada still vira um clipe de 4–6 segundos. As restrições vêm da etapa seguinte, não do gosto:
um movimento contínuo, sem corte, sem tremor, sem loop, lento.

**Portão:** no máximo **duas** seções recebem sequência de frames em canvas; a tabela-resumo
fecha abaixo de 12 MB desktop / 5 MB mobile; cada bloco diz qual técnica é o destino.

> **Você gera os clipes aqui, manualmente,** no Google Flow. Salve em
> `assets/generated/<secao>.mp4`.

### Fase 5 — Design e implementação

**Skills, nesta ordem:** `product-design-expert` → `frontend-design` → `landing-motion-expert`
(que roteia para `gsap-scrolltrigger-expert`, `scroll-video-director`, `video-to-website` e
`motion-ui-expert`)

**Artefato:** o projeto Next.js funcionando local.

Scaffold, tokens, pipeline de assets, SEO, seções, motion. O detalhe técnico está em
[stack.md](stack.md): estrutura de pastas, `metadata`, JSON-LD `VeterinaryCare`/`LocalBusiness`,
sitemap, `next/font`, pipeline `sharp` + `ffmpeg`, e o wiring de GSAP + Lenis no App Router
(que exige `'use client'` e cuidado com Strict Mode).

**Portão:** `npm run build` passa; nenhuma seção repete a entrada da anterior; o Lighthouse
local não regride abaixo de 90 em Performance e 100 em SEO.

### Fase 6 — Auditoria

**Skill:** `responsive-e-acessibility` — **este portão bloqueia o deploy.**

Roda o protocolo de auditoria: breakpoints, alvos de toque de 44px, ordem de foco, semântica,
contraste medido, alt text útil, `prefers-reduced-motion`, peso em conexão lenta.

**Portão:** aprovação explícita, item por item. Reprovou, volta para a Fase 5. Não existe
"depois a gente arruma".

### Fase 7 — Publicar

**Artefato:** repositório no GitHub + URL de produção na Vercel.

```bash
git init -b main && git add -A
git commit -m "Landing page <cliente>"
gh repo create <owner>/<repo> --public --source . --remote origin --push
vercel link && vercel --prod
```

Antes de rodar: confirme que `.env*` está no `.gitignore` e que nenhum dado pessoal do cliente
(contrato, CPF, tabela de preços interna) entrou no commit. Repositório público é público.

**Portão:** a URL abre; o WhatsApp abre com a mensagem certa; o telefone disca no mobile; os
dados de contato batem com o que o cliente confirmou.

---

## Coordenação: como as doze skills se encaixam

```
Fase 1   brand-dna-extractor ──────────────► design-system.json
                                                      │
Fase 2   creative-direction-expert (passada 1) ◄──────┤
         landing-storytelling-director ◄──────────────┤  (todas leem
         creative-direction-expert (passada 2)        │   deste arquivo)
                    │                                 │
Fase 3   ai-visual-prompt-director ◄──────────────────┤
Fase 4   ai-visual-prompt-director + video-to-website ┤
                    │                                 │
Fase 5   product-design-expert ◄──────────────────────┤
         frontend-design ◄────────────────────────────┤
         landing-motion-expert ──┬── gsap-scrolltrigger-expert
                                 ├── scroll-video-director ── video-to-website
                                 └── motion-ui-expert
                    │
Fase 6   responsive-e-acessibility  ◄── PORTÃO BLOQUEANTE
                    │
Fase 7   git + gh + vercel
```

Duas regras de coordenação que evitam o retrabalho mais caro:

1. **Uma fonte da verdade.** `design-system.json` não é copiado para dentro dos outros
   artefatos — é referenciado. Uma cor corrigida na Fase 1 propaga sozinha; uma cor colada em
   cinco arquivos vira cinco versões da marca.
2. **Nunca pule a ordem.** Animar antes de definir a estrutura produz timeline linda para
   seção que vai ser cortada. Gerar imagem antes da copy produz imagem sem lugar para o texto.

## Anti-patterns do pipeline

- **Gerar imagem antes da copy existir** — a composição precisa saber onde o texto cai.
- **Reescrever o Style Anchor no meio** — as seções param de casar entre si e a página vira
  colagem de bancos de imagem diferentes.
- **Frame sequence em toda seção** — vira download de 40 MB. Duas, no máximo.
- **Publicar com pendência de `unverified`** — promessa falsa sobre negócio real custa mais
  caro que atraso.
- **Deploy antes do portão da Fase 6** — acessibilidade vira dívida que ninguém paga.
- **Copiar a copy do site antigo do cliente** — se o site antigo convertesse, não haveria projeto.
- **Repositório público com dados internos** — confira o `.gitignore` antes do primeiro commit.

## Verificação final

- [ ] `npm run build` passa sem warning novo
- [ ] Lighthouse: Performance ≥90, Acessibilidade 100, SEO 100
- [ ] `prefers-reduced-motion` ligado: página inteira legível, nada quebrado
- [ ] Teclado: navega tudo, foco sempre visível
- [ ] Mobile 375px: sem scroll horizontal, alvos ≥44px
- [ ] WhatsApp abre com mensagem pré-preenchida; telefone disca
- [ ] Dados de contato conferidos com o cliente, não deduzidos
- [ ] JSON-LD valida no Rich Results Test do Google
- [ ] OG image aparece ao colar o link no WhatsApp
- [ ] Nenhum `unverified` pendente virou promessa na página
