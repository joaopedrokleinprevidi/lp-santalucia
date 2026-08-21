---
name: landing-page-factory
description: Use to run a whole landing page project end to end, from a folder of client assets to a live URL. Algoritmo completo da fábrica de landing page: briefing, estudo dos assets, DNA da marca, lacunas, pesquisa de nicho, direção criativa, estrutura, copy, prompts de imagem e animação, build Next.js com scroll, portões de auditoria e deploy no GitHub e Vercel. Orquestra as outras 23 skills.
argument-hint: [pasta-de-assets] [nome-do-cliente]
---

# Landing Page Factory

| | |
|---|---|
| **ENTRADA** | a pasta de assets do cliente e os dados que o dev trouxe. Nada mais: todo artefato do pipeline é escrito por uma das 23 skills chamadas daqui |
| **SAÍDA** | nenhum arquivo próprio. A saída é a sequência executada e os três portões respeitados; o último artefato é o de `publicar-lp` — repo `lp-<slug>` e URL de produção |
| **ANTES** | nenhuma. É o ponto de entrada do projeto |
| **DEPOIS** | `briefing-cliente` (Fase 0), e daí a ordem do algoritmo abaixo |

Esta skill é o único lugar onde a numeração das 13 fases é canônica. Cada uma das outras 23
declara o próprio contrato ENTRADA/SAÍDA/ANTES/DEPOIS no topo do SKILL.md dela, derivado desta
ordem. Se um contrato divergir daqui, o contrato está errado.

Uma pasta com fotos e um telefone entram. Um site no ar sai.

Esta skill não faz o trabalho — ela é o **algoritmo** que chama as outras 23 na ordem certa,
com um portão entre cada passo.

## O contrato com o dev

O dev é leigo. O trabalho dele inteiro são quatro linhas:

1. Trazer os dados da empresa e as imagens que o cliente tiver.
2. Dizer "gere".
3. Colar os prompts que eu entrego no ChatGPT (imagens) e no Google Flow (clipes), salvando nos
   caminhos que cada bloco indica.
4. Aprovar o login no navegador quando uma credencial for pedida.

Todo o resto é meu. Não existe passo 5.

Consequência operacional: **pergunte numa rodada consolidada e declare suposição para tudo que
não for bloqueante.** Pergunta pingada é um telefonema do dev para o cliente; na terceira, o
cliente para de atender.

> Versão sem jargão, para o dev ler: [para-leigos.md](para-leigos.md).

---

## O algoritmo

```
ALGORITMO landing_page(pasta_assets, dados_do_cliente)

  ── ENTRADA ──────────────────────────────────────────────────────────────────

  0.  briefing-cliente(dados_do_cliente)                    → design/briefing.json
      SE falta node≥20 | git | gh            ENTÃO PARE · peça ao dev (credentials.md)
      SE falta campo do bloco BLOQUEIA       ENTÃO PARE · peça numa rodada só
      SENÃO                                        registre a suposição e siga

  1.  PARA CADA arquivo EM pasta_assets:
        estudo-assets(arquivo)                              → design/inventario.json
        · abra e OLHE a imagem — nunca classifique pelo nome do arquivo
        · dois vereditos: serve na página (e em que largura) · serve como referência
      SE nenhum logo utilizável              ENTÃO PARE · peça o logo em vetor ou alta

  2.  brand-dna-extractor(inventario)                       → design/design-system.json
      SE cor da marca reprova contraste      ENTÃO registre o tom que passa · SIGA
        (nunca troque a cor da marca em silêncio)

  3.  auditoria-dados(briefing, inventario, design-system)  → design/lacunas.md
      SE existe item BLOQUEIA                ENTÃO ESPERE resposta_do_dev  ⟵ parada 1
      SE existe CONFLITO entre fontes        ENTÃO ESPERE sempre
        (duas fontes do próprio cliente se negando não é suposição minha)

  4.  niche-research()                                      → design/pesquisa.md
                                                            + scripts/check-banned-copy.mjs
      SE o diferencial já está documentado E há ≥2 números verificáveis
        ENTÃO passada CURTA (2 concorrentes, ≤8 buscas)
        SENÃO passada COMPLETA (5 frentes, 3 a 5 concorrentes)
      NUNCA zero: as seis seções de pesquisa.md são ENTRADA da 6a e da 6b, e o
      script é o portão de saída da 6b — sem os dois arquivos a Fase 6 trava

  ── DIREÇÃO ──────────────────────────────────────────────────────────────────

  5.  creative-direction-expert(passe 1)                    → Score, budget, profundidade
      · fixa Experience Score, MB de mídia e onde cai o pico da curva
      · ainda NÃO decide por capítulo — os capítulos não existem
      scaffold Next.js na própria pasta (stack.md §2)       → src/, app/, package.json
      · aqui, não na Fase 10: a 6a grava src/data/story.ts e a 6b grava src/data/site.ts
      · create-next-app aborta numa pasta que já tem src/ — depois da 6a é tarde

  6a. estrutura-secoes(briefing, pesquisa, budget)          → ordem + scroll por seção
  6b. copy-conversao(estrutura, pesquisa)                   → design/landing-blueprint.md
      SE a página vale o custo
        ENTÃO gere 3 arquétipos concorrentes
              julgue por 3 lentes (conversão · copy pt-BR · experiência)
              sintetize a vencedora enxertando o melhor das outras
      ESPERE revisão_do_dev                                            ⟵ parada 2
        (último momento barato de mudar: depois disso a composição de cada
         imagem já foi desenhada em volta deste texto)

  7.  creative-direction-expert(passe 2)                    → design/creative-direction.json
      · ratifica banda, pontos e entrada de cada capítulo
      SE algum capítulo estourou a banda    ENTÃO devolva para 6a

  ── MÍDIA ────────────────────────────────────────────────────────────────────

  8a. PARA CADA seção EM blueprint:
        SE a seção afirma fato sobre o negócio real
          ENTÃO marque USAR FOTO REAL <arquivo>   (fachada e equipe nunca se geram)
        SENÃO SE a seção tem assunto concreto (pessoa|lugar|animal|objeto|gesto)
          ENTÃO prompt-imagem(seção)                        → bloco em image-prompts.md
        SENÃO marque SEM IMAGEM · a seção é tipografia

  8b. PARA CADA still EM image-prompts:
        prompt-animacao(still)                              → bloco em motion-prompts.md
      SE soma das frame sequences > budget.mediaDesktopMB
        ENTÃO rebaixe a mais barata para loop · REPITA a soma
      SE nº de frame sequences > 1
        ENTÃO mantenha só a mais forte, salvo se estiverem distantes na página

  9.  PARA CADA bloco EM image-prompts:                                ⟵ parada 3
        REPITA
          dev cola no ChatGPT (Style Anchor + prompt + anexos)
        ATÉ passar nos três portões:
          sem letra/número/placa · sem rosto reconhecível · assunto concreto
      PARA CADA clipe EM motion-prompts:
        REPITA
          dev anima no Google Flow
        ATÉ sem corte · sem tremor · sem mudança de velocidade
      SE 3 tentativas sem resultado         ENTÃO reescreva o prompt · não aceite a 4ª
      → design/renders/NN-secao.png e .mp4     (guia: handoff-imagens.md)

  ── CONSTRUÇÃO ───────────────────────────────────────────────────────────────

  10. copie design/renders/ → assets-source/    (senão o build passa sem as mídias)
      product-design-expert()                   → tokens, escala, grade
                                                → src/components/chapters/*.tsx, um por seção
      frontend-design()                         → par tipográfico, textura, caráter
      landing-motion-expert()                   → roteia, por seção:
        SE scroll controla algo                 → gsap-scrolltrigger-expert
        SE existe vídeo                         → video-decisao
             SE a mudança no tempo É a batida   → video-to-website (canvas frames)
             SENÃO                              → video-encode (loop | once)
        SE é botão|card|menu|form|modal         → motion-ui-expert
      SE a entrada da seção repete a anterior   ENTÃO troque a entrada

  ── PORTÕES ──────────────────────────────────────────────────────────────────

  11. audit-responsivo()      → BLOQUEIO | RESSALVA
      audit-acessibilidade()  → BLOQUEIO | RESSALVA
      audit-performance()     → BLOQUEIO | RESSALVA
      SE qualquer BLOQUEIO                     ENTÃO volte para 10 · REPITA
      (não existe "depois a gente arruma": acessibilidade adiada nunca é feita)

  ── PUBLICAÇÃO ───────────────────────────────────────────────────────────────

  12. SE ainda existe item BLOQUEIA em lacunas.md
        ENTÃO PARE · publicar promessa não confirmada custa mais que atrasar
      publicar-lp()                            → repo lp-<slug> + URL de produção

FIM
```

---

## Quem faz o que

| # | Fase | Skill dona | Artefato | DEV atua? |
|---|---|---|---|---|
| 0 | Briefing e credenciais | `briefing-cliente` | `design/briefing.json` | **Sim** — traz dados e imagens |
| 1 | Estudo dos assets | `estudo-assets` | `design/inventario.json` | Não |
| 2 | DNA visual | `brand-dna-extractor` | `design/design-system.json` | Não |
| 3 | Auditoria de lacunas | `auditoria-dados` | `design/lacunas.md` | **Sim** — 1 rodada única |
| 4 | Pesquisa de nicho | `niche-research` | `design/pesquisa.md` | Não |
| 5 | Direção criativa I + scaffold | `creative-direction-expert` | Score + orçamento · projeto Next.js criado | Não |
| 6a | Estrutura | `estrutura-secoes` | ordem e scroll por seção | Não |
| 6b | Copy | `copy-conversao` | `design/landing-blueprint.md` | **Sim** — revisa o texto |
| 7 | Direção criativa II | `creative-direction-expert` | `design/creative-direction.json` | Não |
| 8a | Prompts de imagem | `prompt-imagem` | `design/image-prompts.md` | Não |
| 8b | Prompts de animação | `prompt-animacao` | `design/motion-prompts.md` | Não |
| 9 | Geração das mídias | manual | `design/renders/` | **Sim** — ChatGPT + Flow |
| 10 | Design e implementação | `product-design-expert` (tokens + componentes de capítulo) → `frontend-design` → `landing-motion-expert` | Next.js rodando local | Não |
| 11 | Portões | `audit-responsivo` · `audit-acessibilidade` · `audit-performance` | laudo aprovado | Não |
| 12 | Publicar | `publicar-lp` | repo + URL | **Sim** — autoriza |

Três paradas esperam o dev: **3**, **6b** e **9**, mais a autorização da **12**. As outras rodam
sem ele.

> Esta numeração é canônica e vale em todos os arquivos da fábrica. Se um arquivo divergir, o
> arquivo está errado — conserte o arquivo, não a tabela.

---

## Decisões que eu tomo sozinho e declaro

Perguntar isso a um dev leigo devolve "não sei" e gasta a rodada da Fase 3.

| Decisão | Padrão | Por quê | Quando muda |
|---|---|---|---|
| Framework | **Next.js App Router** | `metadata`, JSON-LD, sitemap e OG servidos pelo framework; no Vite os quatro são trabalho manual que ninguém mantém | Vite se não houver SEO em jogo |
| Experience Score | **★★★★☆** | Default do CLAUDE.md: GSAP, storytelling pinado, motion em camadas, sem exigir produção de vídeo | ★★★★★ com orçamento real de mídia **e** pedido do cliente |
| Idioma | **pt-BR** | `lang="pt-BR"`, `latin-ext` nas fontes, telefone e moeda locais | Nunca, nesta fábrica |
| Conversão | **WhatsApp** | Negócio local brasileiro responde no WhatsApp; formulário adiciona um canal que ninguém checa | Se o cliente não tem WhatsApp comercial |
| Hospedagem | **Vercel** | Detecta o Next sozinha, Hobby é grátis, e o MCP traz o log de build para dentro da conversa | Nunca, nesta fábrica |
| Repositório | **`lp-<slug>`, privado** | Privado funciona igual na Vercel e não expõe o que ninguém revisou | Público após a varredura da Fase 12 |

## Fontes da verdade

```
design/briefing.json ──┐
design/inventario.json ─┼──→ lidos por todas as fases seguintes
design/design-system.json ─┘   NUNCA copiados para dentro de outro artefato
```

Um hex corrigido na Fase 2 propaga sozinho. Um hex colado em cinco arquivos vira cinco versões
da marca.

## Anti-patterns

- **Gerar imagem antes da copy existir** — a composição precisa saber onde o texto cai.
- **Reescrever o Style Anchor no meio** — as seções param de casar e a página vira colagem.
- **Frame sequence em toda seção** — vira download. Uma por página; duas só se distantes.
- **Publicar com item BLOQUEIA aberto** — promessa falsa sobre negócio real custa mais que atraso.
- **Decidir um CONFLITO sozinho** — duas fontes do cliente se negando exige o cliente.
- **Pular o portão da Fase 11** — acessibilidade adiada vira dívida que ninguém paga.
- **Repositório público com dado interno** — o histórico do git guarda mesmo depois de apagar.
- **Aceitar imagem 80% certa** — arrasta a página mais para baixo que uma seção sem imagem.
- **Perguntar em conta-gotas** — queima a paciência do cliente do dev.
- **`create-next-app <nome>`** — aninha o projeto e deixa `design/` fora do repositório. Use `.`.

## Verificação final

```bash
npm run build                      # passa sem warning novo
npm start                          # build de produção no ar, é onde a Fase 11c mede
git check-ignore -v .env.local design/renders design/lacunas.md   # os três ignorados
gh repo view --json visibility     # privado até a varredura passar
```

- [ ] Lighthouse: Performance ≥90 · Acessibilidade 100 · SEO 100
- [ ] `prefers-reduced-motion` ligado: página inteira legível, nada quebrado
- [ ] Teclado: navega tudo, foco sempre visível
- [ ] 375px: sem scroll horizontal, alvos ≥44px
- [ ] WhatsApp abre com a mensagem pronta; telefone disca no celular
- [ ] Nenhuma imagem com letra, número ou rosto reconhecível
- [ ] JSON-LD valida no Rich Results Test
- [ ] OG image aparece ao colar o link no WhatsApp
- [ ] Dados de contato conferidos com o cliente, não deduzidos
- [ ] Nenhum item `unverified` virou promessa na página

## Arquivos da fábrica

[para-leigos.md](para-leigos.md) · [credentials.md](credentials.md) ·
[handoff-imagens.md](handoff-imagens.md) · [stack.md](stack.md) · skill [`publicar-lp`](../publicar-lp/SKILL.md)
