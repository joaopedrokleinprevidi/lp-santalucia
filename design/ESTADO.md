# Estado do projeto — Santa Lúcia

Última atualização: 2026-08-21.

## Onde o projeto está

**Fim da Fase 8.** As decisões estão todas tomadas e escritas. Não existe código ainda.
O próximo passo é a Fase 9, que é manual e roda fora deste repositório: gerar as imagens
no ChatGPT e os clipes no Google Flow.

Este projeto foi montado antes da reestruturação do pipeline. Ele tem os artefatos de
saída das fases 2, 6 e 8, mas nunca teve os intermediários (`briefing.json`,
`inventario.json`, `pesquisa.md`, `creative-direction.json`). Isso é histórico, não
pendência: as decisões que esses arquivos sustentariam já foram tomadas e estão dentro
dos artefatos finais. Não regere nem refaça nenhum deles.

## O que já existe

| Artefato | Quem produziu |
|---|---|
| `assets-source/` (9 arquivos) | Material do cliente. Base medida do design system. |
| `design-system.json` | `brand-dna-extractor`, amostrando pixel a pixel de `assets-source/` |
| `landing-blueprint.md` | 11 seções com a copy definitiva — três arquétipos julgados por três lentes |
| `image-prompts.md` | 8 prompts com o Style Anchor congelado |
| `motion-prompts.md` | 7 prompts de animação, com a técnica de destino de cada seção |

`design-system.json` é a fonte da verdade de cor, voz e fatos. Os outros leem dele.

**Nada disso pode ser reescrito.** A copy custou um painel de julgamento e as cores foram
medidas, não estimadas. Se algo parecer estranho, leia `design/README.md` antes de mudar.

## O que falta

1. **Fase 9 (manual, fora do repo)** — gerar os stills no ChatGPT e animar no Google Flow.
2. **Fase 10** — extrair frames, montar o Next.js, amarrar ao scroll.
3. **Fases 11 e 12** — portões de auditoria e deploy.

## Próximo passo concreto

Abra `design/image-prompts.md`. Copie o **Style Anchor** e cole junto com cada prompt de
seção no ChatGPT. O anchor é idêntico em todos os prompts e não pode ser editado no meio
do caminho — é ele que faz oito imagens geradas em oito chamadas parecerem o mesmo ensaio.

Salve tudo em `design/renders/`, com o nome `NN-id-secao.png`. Os clipes do Flow saem como
`NN-id-secao.mp4`, na mesma pasta. A pasta existe e está fora do versionamento: os arquivos
são pesados e regeráveis a partir dos prompts.

Cada bloco de `image-prompts.md` termina com uma linha **Salvar como:** que dá o nome exato do
arquivo daquela seção. Use o nome literal dessa linha, sem inventar variação — os comandos
`ffmpeg` da Fase 10 já estão escritos com esses caminhos e falham com *No such file* se o nome
não bater.

Seções 01 e 11 não se geram — usam foto real de `assets-source/`. Elas ainda assim rendem
clipe: a foto real sobe no Flow como primeiro frame e o clipe que volta é salvo em
`design/renders/`, como nas outras.

## Da Fase 9 para a Fase 10 — o passo que falha calado

**Salvar em `design/renders/` não coloca nada na página.** Essa pasta é matéria-prima, não é
servida. Ela está no `.gitignore`, então os arquivos não vão para o GitHub nem para a Vercel:
se alguém parar aqui achando que "as mídias já estão no projeto", o build sobe com as seções
vazias e ninguém recebe erro nenhum. É o ponto em que este projeto tem mais chance de quebrar
em silêncio.

O que converte uma coisa na outra é a Fase 10, e ela roda dentro deste repositório:

| De | Para | Quem faz |
|---|---|---|
| `design/renders/NN-<id>.mp4` | `public/frames/<id>/frame_%04d.webp` | comandos `ffmpeg` já escritos em `motion-prompts.md` (seções 04 e 06) |
| `design/renders/NN-<id>.mp4` | `public/video/<id>-1440.mp4` e `-720.mp4` | receita única de encode no fim de `motion-prompts.md` |
| `design/renders/NN-<id>.png` | still servido + poster de LCP | Fase 10, junto com a montagem do Next.js |

Só o que cai em `public/` é versionado e chega à Vercel.

**Nada disso é automático.** Não existe watcher olhando `design/renders/`. Quando os arquivos
estiverem lá, abra o Claude nesta pasta e diga literalmente: *"as mídias da Fase 9 estão em
`design/renders/`, siga para a Fase 10"*. Aí sim ele lê `motion-prompts.md`, roda a extração
seção por seção e monta o projeto.

Antes de dizer isso, três conferências de trinta segundos:

- [ ] `ls design/renders/` — os nomes batem exatamente com as linhas **Salvar como:** do
      `image-prompts.md`? Um `04-primeiros minutos.png` com espaço já quebra o `ffmpeg`.
- [ ] Cada clipe tem **4 a 6 segundos**? Abaixo de 150 frames o scroll estroba, e isso não tem
      conserto depois da extração — só regerando no Flow.
- [ ] Depois de extrair: `ls public/frames/<id> | wc -l` devolve 150 em cada sequência.

## Pendências que bloqueiam publicação

Estão em `design-system.json`, campo `unverified`. As duas que travam:

- **WhatsApp — existe candidato, falta confirmar.** Nenhum asset do cliente traz o número;
  o `(54) 3025-2223` é fixo. A busca pública achou **(54) 99967-4276**, citado por diretório
  de nicho e agregadores locais. Está como `confirmed: false` e é ali que fica: procedência
  web não vira fato. Enquanto o cliente não confirmar, esse número não entra em botão, link
  `wa.me` nem JSON-LD.
- **Pets não convencionais — conflito entre duas fontes do próprio cliente.** O post de
  `assets-source/` anuncia atendimento 24h para não convencionais; o FAQ do site atual diz
  que o foco é cão e gato. Um dos dois está desatualizado. A copy foi escrita a favor do
  post, defensivamente. Se o cliente disser que o FAQ é que vale, saem a faixa da seção 07
  e a pergunta 7 do FAQ — está previsto no blueprint, não quebra a página.

## Pendência barata de asset

**Foto noturna real da fachada**, da calçada oposta, com a vitrine acesa. Um celular, 22h,
dois minutos. A foto real que temos é de dia com céu encoberto, e a graduação noturna
resolve — mas é a única concessão visual do documento. Essa foto melhora o hero e o fecho
de uma vez. Pedir junto com as confirmações acima.
