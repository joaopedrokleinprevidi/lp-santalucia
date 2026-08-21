# Design — Santa Lúcia

Este diretório é a ponte entre os assets que o cliente já tem e a landing page que ainda não
existe. Nada aqui é código: são as decisões que o código vai executar.

## Os arquivos

| Arquivo | O que é | Quem escreveu |
|---|---|---|
| `design-system.json` | O DNA da marca em tokens: cores medidas, tipografia, formas, motifs, voz, fatos reais | `brand-dna-extractor`, a partir de `/assets` |
| `landing-blueprint.md` | A estrutura da página seção a seção, com a copy definitiva | `landing-storytelling-director` + painel de julgamento |
| `image-prompts.md` | Um prompt de imagem por seção, para gerar no GPT | `ai-visual-prompt-director` |
| `motion-prompts.md` | Um prompt de animação por seção, para o Google Flow | `ai-visual-prompt-director` |

`design-system.json` é a **fonte da verdade**. Os outros três leem dele. Se uma cor mudar lá,
os demais não são reescritos — eles referenciam.

## O pipeline

```
/assets (fotos reais + posts)
   │
   ├─ brand-dna-extractor ──────────→ design-system.json
   │                                       │
   ├─ landing-storytelling-director ───────┼──→ landing-blueprint.md
   │                                       │         │
   ├─ ai-visual-prompt-director ───────────┴─────────┼──→ image-prompts.md
   │                                                 │        │
   │                                                 │        ▼
   │                                                 │   [ GPT gera o still ]
   │                                                 │        │
   ├─ ai-visual-prompt-director ─────────────────────┴──→ motion-prompts.md
   │                                                          │
   │                                                          ▼
   │                                                 [ Google Flow anima ]
   │                                                          │
   └─ video-to-website ←──────── clipes .mp4 ─────────────────┘
             │
             ▼
      frames .webp + canvas + GSAP  →  a landing page
```

## Como rodar cada etapa

**1. Extrair o DNA** (já feito) — `brand-dna-extractor` amostrou os pixels dos assets. As cores
não foram estimadas a olho: passe A pega os tons dominantes, passe B isola os cromáticos por
janela de matiz. É por isso que `#603084` está lá com "87,3% dos pixels do logo" ao lado.

**2. Estruturar e escrever** (já feito) — três arquétipos narrativos concorrentes foram
gerados, julgados por três lentes independentes (conversão, qualidade de copy em português,
experiência) e sintetizados. O `landing-blueprint.md` diz qual venceu e o que foi enxertado
das outras.

**3. Gerar as imagens** — abra `image-prompts.md`. Copie o **Style Anchor** e cole junto com
cada prompt de seção no GPT. O anchor é idêntico em todos os prompts e não pode ser editado no
meio do caminho: é ele que faz oito imagens geradas em oito chamadas parecerem o mesmo
ensaio fotográfico.

Salve os resultados em `assets/generated/<secao>.png`, na maior resolução disponível.

**4. Animar no Google Flow** — abra `motion-prompts.md`. Cada bloco diz qual still animar e
qual movimento pedir. As restrições ali não são estéticas, vêm da etapa seguinte: clipe de
4–6s, um movimento contínuo, sem corte, sem tremor, sem loop. Um corte no meio do clipe vira
um glitch visível quando o scroll passa por cima dele.

Salve em `assets/generated/<secao>.mp4`.

**5. Virar site** — `video-to-website` extrai os frames, monta o canvas e amarra ao scroll.
A tabela no fim de `motion-prompts.md` já diz quais seções recebem sequência de frames
(no máximo duas) e quais ficam como loop ou still.

## Regras que valem para a página inteira

**O amarelo não é cor de texto sobre claro.** Medido: `#FCB400` sobre o creme dá 1,66:1, e o
mínimo legível é 4,5:1. Nos posts do Instagram ele aparece como texto sobre fundo claro — isso
não sobrevive numa página que precisa passar em acessibilidade. Amarelo é texto **sobre roxo**
(6,21:1) ou é forma, contorno e acento. Quando precisar mesmo de texto âmbar sobre creme, use
`amber-800` (`#8B6609`, 4,78:1).

**Fatos e inferências são coisas diferentes.** No `design-system.json`, `facts` só contém o que
está legível num asset ou documento do cliente. `inferences` contém o que foi deduzido, com a
base declarada. `unverified` contém o que bloqueia publicação. Nada de `inferences` vira copy
sem alguém confirmar.

**Não gerar imagem que afirme um fato sobre o negócio real.** A fachada e a equipe existem em
`/assets` como fotos reais e entram na página como estão. Gerar um prédio que não é o prédio
deles, ou um veterinário que não trabalha lá, é afirmar algo falso sobre um negócio real.
Imagem gerada serve para papel ilustrativo e atmosférico.

**Nenhum texto dentro das imagens.** Toda copy é DOM. Modelo generativo escreve português
acentuado como garatuja, e texto em imagem não é selecionável, não é traduzível e não é
indexável.

## Pendências antes de publicar

Estão listadas em `design-system.json` no campo `unverified`. As que bloqueiam:

- **Número do WhatsApp.** É o canal de conversão principal desta página e não existe número
  confirmado para ele em nenhum asset. O `(54) 3025-2223` é telefone fixo.
- **Pets não convencionais.** O post anuncia atendimento 24h para exóticos; o FAQ do site atual
  diz que o foco é cão e gato. Um dos dois está desatualizado. É um diferencial forte se
  confirmado e uma promessa falsa se não.
- **Estacionamento.** As fotos mostram vagas na rua, não estacionamento próprio. A copy não
  pode prometer o que não existe.
- **Handles das redes sociais** para o rodapé.
