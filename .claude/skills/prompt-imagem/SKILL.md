---
name: "prompt-imagem"
description: "Use when writing the AI image prompt for each landing page section — freezing the Style Anchor, deciding which sections earn a generated still, and enforcing the zero-text and concrete-subject gates. Fase 8a, produz design/image-prompts.md. Prompt de imagem para ChatGPT/GPT Image, papel visual por secao, proporcao, quais arquivos anexar, SUBJECT COMPOSITION DETAIL FRAMING EXCLUDE, teste de aceitacao. Rejeita letra, rosto e assunto abstrato."
argument-hint: "[secao] [papel-visual]"
allowed-tools: Read, Glob, Write
---

# Prompt de imagem — Fase 8a

| | |
|---|---|
| **ENTRADA** | `design/design-system.json` (cores, materiais, voz) · `design/creative-direction.json` da Fase 7 (`wow[]` e `chapters[]` — qual seção carrega peso visual) · o story map e a copy da Fase 6 em `design/landing-blueprint.md` (seções, headline, onde o texto cai) · `design/inventario.json` (quais arquivos reais existem e quais servem de referência para anexar) |
| **SAÍDA** | `design/image-prompts.md` — o Style Anchor uma vez no topo, depois um bloco por seção |
| **ANTES** | `creative-direction-expert`, passe 2 (Fase 7), que grava `design/creative-direction.json`. Sem a copy da Fase 6 não existe linha `COMPOSITION` |
| **DEPOIS** | `prompt-animacao` (Fase 8b) lê os stills daqui e escreve `design/motion-prompts.md`. Na Fase 9 o dev gera cada bloco no ChatGPT e salva em `design/renders/NN-secao.png` — é essa pasta que a Fase 10 copia para `assets-source/` e consome |

O trabalho não é escrever um prompt bonito. É fazer oito imagens geradas em oito chamadas
independentes parecerem o mesmo ensaio, do mesmo fotógrafo, no mesmo dia.

**Quem executa é leigo.** A saída é um arquivo que a pessoa abre, copia um bloco, anexa os
arquivos que o bloco manda anexar, cola no ChatGPT e salva o PNG. Se o bloco não diz
literalmente quais arquivos anexar e onde salvar o resultado, o bloco está incompleto.

Arquivos de apoio: [bloco-exemplo.md](bloco-exemplo.md) (o formato de handoff preenchido) e
[rejeicoes.md](rejeicoes.md) (a causa de cada portão e o que fazer quando a geração volta errada).

## Os três portões

Nenhum prompt sai daqui sem passar nos três. Nenhuma imagem entra na página sem passar nos três.

| Portão | Regra | Falhou? |
|---|---|---|
| **Assunto concreto** | Toda imagem é sobre pessoa, lugar, animal, objeto ou gesto | Sem assunto concreto a seção não recebe imagem — recebe tipografia |
| **Texto zero** | Nenhuma letra, número, palavra, logo desenhado, placa escrita, etiqueta ou mostrador legível | Regera. Nunca conserta no CSS |
| **Referência anexada** | Todo bloco declara quais arquivos o dev anexa e com qual frase | Sem anexo a marca sai aproximada e destoa do logo real no header |

## O Style Anchor

Modelo generativo não tem memória entre chamadas. Oito prompts escritos de forma independente
produzem oito estúdios diferentes. A correção é um **style anchor**: um parágrafo, escrito uma
vez, prefixado **literalmente** em todo prompt de imagem do projeto. Ele nunca varia; só o bloco
de assunto depois dele muda. Ele mora no topo de `design/image-prompts.md` e é a fonte da
verdade — `prompt-animacao` cita este arquivo em vez de reescrever o parágrafo.

```
STYLE ANCHOR (idêntico em todo prompt, nunca editado no meio do projeto):
Professional editorial photography for a veterinary clinic brand. Soft diffused daylight from
camera left, gentle falloff, no hard shadows. Shallow depth of field at f/2.0. Warm neutral
palette built on cream #F7F4EF with deep violet #4E2A96 and amber #FCB400 appearing only as
real objects in frame — uniforms, equipment, powder-coated metal, lamp glow. Clean uncluttered
composition, generous negative space. Shot on 50mm. People appear only as hands, forearms,
backs or out-of-focus silhouettes — never a recognizable face. Natural skin and fur tones, no
color grading, no filter look.
```

1. **Descreva luz, lente e paleta.** Esses três carregam consistência entre chamadas
   independentes muito mais longe que qualquer descrição de assunto. Assunto não carrega.
2. **Amarre a cor da marca a objeto físico.** `violet uniforms and powder-coated metal`
   sobrevive à geração. `violet color scheme` produz uma lavagem roxa sobre a imagem inteira e
   denuncia IA em meio segundo.
3. **Congele.** Se o anchor mudar na seção 5, as seções 1–4 deixam de casar. Regere todas ou não
   mexa em nenhuma.

O anchor também é onde mora a regra de rosto — ali ela não é esquecida seção a seção.

## Passo 1 — Papel visual por seção

Nem toda seção recebe imagem gerada. Decida o papel primeiro:

| Papel | Gera imagem? | Por quê |
|---|---|---|
| Hero | Sim — é onde vale investir demais | Fixa a referência que todas as outras vão ter que casar |
| Prova / credencial | **Não** — foto real | Fachada gerada é afirmação falsa sobre um lugar que existe |
| Equipe / pessoas | **Não** | Fabricar equipe identificável deturpa o negócio |
| Serviço / processo | Sim | Genérico o bastante para gerar, específico o bastante para importar |
| Ambiente / transição | Sim | Material e luz, risco baixo — desde que tenha assunto concreto |
| Números / estatística | Não | Tipo e movimento carregam; imagem compete com o número |
| Depoimento | Não | Nome real merece contexto real, ou nenhum |
| CTA | Opcional | Frequentemente mais forte como cor chapada da marca |

**Linha dura:** nunca gere imagem que afirme um fato sobre o negócio real — o prédio, a equipe,
o equipamento que o cliente pode não ter. Para qualquer coisa que faça uma afirmação, use a foto
real do cliente. Imagem gerada serve papel ilustrativo e atmosférico.

## Passo 2 — Portão do assunto concreto

Toda imagem é sobre **pessoa, lugar, animal, objeto ou gesto**. Forma abstrata, gradiente
decorativo, textura genérica, bokeh, malha 3D flutuante e "luz volumétrica" estão proibidos —
as quatro causas estão em [rejeicoes.md](rejeicoes.md).

Se você não consegue nomear o assunto num substantivo concreto — *mãos*, *corredor*, *manta*,
*azulejo molhado* — a seção não recebe imagem. Recebe tipografia grande e cor sólida, e fica
melhor por isso.

| A seção quer dizer | Assunto concreto que diz isso | Nunca |
|---|---|---|
| Cuidado, confiança | Duas mãos de jaleco apoiando o peito de um cão deitado | Gradiente roxo com brilho suave |
| Estrutura, equipamento | O aparelho na sala, painel apagado, cabo enrolado no gancho | Forma 3D abstrata flutuando |
| Urgência, plantão | Corredor vazio à noite com uma arandela acesa no fim | Linhas de velocidade, riscos de luz |
| Higiene, limpeza | Azulejo molhado recém-lavado com reflexo da luminária | Textura de mármore de banco de imagem |
| Acolhimento | Manta dobrada sobre a cama do canil, luz baixa | Bokeh dourado desfocado |
| Transformação, antes/depois | Pelo molhado escorrendo virando pelo seco e volumoso | Partículas, swoosh, "energia" |
| Número, prova social | Nada. Tipografia. | Qualquer textura "para dar fundo" |

**Como isso convive com a regra de rosto.** Pessoa aparece como **mãos, antebraços, costas,
nuca, ombro no canto do quadro ou silhueta fora de foco**. Duas razões: não temos autorização de
uso de imagem de ninguém, e rosto é onde o modelo mais erra (olhos assimétricos, dente a mais,
orelha derretida) num tamanho em que o erro é visível. Isso **não** contradiz o portão do
assunto concreto: uma foto de duas mãos secando a orelha de um cachorro é uma imagem sobre uma
pessoa — o assunto é o gesto humano. Um bokeh dourado quente não é sobre ninguém.

**Teste:** troque o assunto por um gradiente na sua cabeça. Se a seção continua dizendo
exatamente a mesma coisa, a imagem não estava dizendo nada — corte a imagem.

## Passo 3 — Imagens de referência anexadas

O ChatGPT aceita imagem junto do prompt. Anexo amarra identidade visual muito melhor que
descrição textual: escrever `warm cream #F7F4EF` aproxima; anexar o logo entrega o hex.

| Papel da seção | Anexos | Observação |
|---|---|---|
| Hero | logo + 1 post | **Nunca a foto real do local.** O hero é o quadro mais reconhecível da página; um hero quase-igual ao lugar real é pior que um hero declaradamente ilustrativo |
| Serviço / processo | logo + 1 post | O post ensina tratamento de foto, não assunto |
| Ambiente do cliente | logo + 1 post + 1 foto real do local | Só quando a seção fala do ambiente. Referência de material e luz, nunca de enquadramento |
| Atmosfera / transição | logo apenas | Menos referência, mais liberdade — essa seção só precisa amarrar cor |
| Prova, fachada, equipe | nenhum — não gera | Foto real do cliente, direto |

**Teto: 3 anexos.** Com 4 ou mais o modelo passa a fazer média das composições e devolve uma
colagem. Sintoma medido: a imagem gerada reproduz o crop da referência com outro assunto dentro.
Quando o anexo começa a virar cópia, os três sintomas e as correções estão em
[rejeicoes.md](rejeicoes.md).

**Como escolher o post.** Pelo tratamento fotográfico, não pelo assunto. Um post que é 80%
tipografia sobre fundo chapado não ensina nada sobre fotografia. Se todos os posts da marca
forem arte com texto, anexe só o logo e declare a limitação.

**A frase de anexo.** Cole depois do prompt, no mesmo envio, sempre com estas palavras:

```
Use the attached images as reference for color, light and material treatment only.
Do not copy their framing, composition, crop or subject placement.
Do not reproduce any logo, mark, lettering or symbol seen in them.
```

"Referência de cor e tratamento, não de enquadramento" é a regra inteira. O enquadramento é
definido pela linha `COMPOSITION`, porque é ela que reserva o espaço vazio onde a copy do DOM
cai — uma referência que sequestra o enquadramento destrói esse espaço.

**Uma conversa nova por seção.** Thread longa faz o modelo usar as imagens que ele mesmo gerou
como referência implícita, e a seção 6 vira a seção 5 com outro objeto dentro. A consistência
vem do anchor congelado e dos mesmos 2–3 anexos deliberados, nunca da memória da conversa.

## Passo 4 — Escrever o prompt

Estrutura, nesta ordem. A ordem importa: o modelo pesa mais os tokens do começo.

```
[STYLE ANCHOR literal]

SUBJECT: <uma frase, concreta, presente do indicativo, um substantivo nomeável>
COMPOSITION: <onde o assunto fica, onde está o espaço vazio para a copy>
DETAIL: <duas ou três especificidades que tornam a cena real>
FRAMING: <tamanho do plano e ângulo>
EXCLUDE: <bloco EXCLUDE base + as superfícies de texto específicas desta cena>
```

`COMPOSITION` é instrução de layout, não gosto. A imagem tem que segurar copy do DOM em cima:

| Layout da seção | Linha COMPOSITION |
|---|---|
| Copy à esquerda | "Subject occupies the right third; left two-thirds is soft out-of-focus background" |
| Copy à direita | espelho do anterior |
| Copy embaixo | "Subject in the upper half, lower third is a clean uninterrupted surface" |
| Full-bleed com scrim | "Subject centered, even tonal field, no bright hotspots near the edges" |

Errar isso leva a jogar um scrim de 70% de opacidade sobre uma foto boa para salvar
legibilidade, o que desperdiça a foto.

| Papel | Proporção | Nota |
|---|---|---|
| Hero, full-bleed | 16:9 | Gere mais largo que o necessário; o pipeline corta, nunca amplia |
| Split de seção | 4:3 ou 1:1 | Casa com meia coluna sem crop torto |
| Retrato / mobile-first | 9:16 | Só quando a seção é liderada pelo mobile |
| Textura / transição | 21:9 | Lê como faixa, não como quadro |

Duas restrições de enquadramento valem para todos: o assunto cabe nos **60% centrais
horizontais** (é o crop 9:16 do mobile) e os **7% externos de cada borda** são padding do canvas
(`IMAGE_SCALE` 0.85 em `video-to-website`). Nada que importa mora nessas faixas.

Gere na maior resolução que a ferramenta oferecer. `prepare-assets.mjs` emite AVIF e WebP a
partir do original — ele reduz, nunca inventa resolução.

## Passo 5 — Portão de texto zero

Qualquer letra, número, palavra, logo desenhado, placa, etiqueta, legenda, marca d'água ou
mostrador legível no quadro **rejeita a imagem**. Regere. Sem exceção. As seis causas, a lista de
"consertos" que não funcionam e a tabela de superfícies que convidam o modelo a escrever estão em
[rejeicoes.md](rejeicoes.md) — a correção nunca é insistir no `EXCLUDE`, é tirar a superfície do
`SUBJECT`.

O **EXCLUDE base**, presente em todo prompt, concatenado com as superfícies da cena:

```
EXCLUDE: text, letters, words, numbers, digits, logos, watermarks, signage copy, labels,
packaging labels, UI overlays, screens with content, clock faces, subtitles, extra limbs,
distorted paws, malformed eyes, recognizable human faces, color grading, filter look, bloom,
lens flare, oversaturation.
```

**O logo nunca entra no quadro** — nem pedido, nem herdado da referência. Ele entra como SVG no
DOM, por cima: nítido em qualquer densidade, escala com o layout, muda de cor por token e
continua sendo o arquivo oficial. Logo desenhado dentro da foto é falha de texto zero e falha de
marca ao mesmo tempo.

## Passo 6 — Emitir `design/image-prompts.md`

O anchor uma vez no topo, depois um bloco por seção na ordem de colagem. Os oito campos
obrigatórios de todo bloco:

1. **Papel visual** — o que a imagem faz pela seção, em uma frase.
2. **Proporção** — da tabela do Passo 4.
3. **Caminho de salvamento** — `design/renders/NN-id-secao.png`, nome exato.
4. **Tabela ANEXAR** — arquivo, para que serve de referência, para que **não** serve.
5. **Frase de anexo** — literal, colada depois do prompt.
6. **Prompt completo** — anchor + `SUBJECT`/`COMPOSITION`/`DETAIL`/`FRAMING`/`EXCLUDE`.
7. **Onde cai a copy** — qual texto do DOM ocupa qual faixa da imagem.
8. **Teste de aceitação** — as condições concretas de regeração desta cena.

Faltando qualquer um, o dev improvisa na Fase 9 — e improviso é onde a consistência morre. O
formato preenchido, e o bloco de seção sem imagem (que se declara, não se omite), estão em
[bloco-exemplo.md](bloco-exemplo.md).

## Passo 7 — Revisar o que voltou

Rejeite e regere em vez de aceitar algo que você vai consertar no CSS.

- [ ] Alguma letra, número, rótulo, mostrador ou logo em qualquer lugar do quadro? → regera
- [ ] O assunto é nomeável num substantivo concreto, ou voltou uma textura bonita? → regera
- [ ] Fica ao lado da imagem do hero sem parecer outra marca?
- [ ] O espaço negativo está onde a linha `COMPOSITION` pediu?
- [ ] A composição repete o crop da referência? → corte os anexos para o logo e regere
- [ ] Cor da marca presente como objeto, não como lavagem sobre tudo?
- [ ] Rosto reconhecível de alguém? → regera, não temos autorização de imagem
- [ ] Mãos, patas, orelhas e olhos anatomicamente sãos a 100% de zoom?
- [ ] O assunto cabe nos 60% centrais horizontais e fora dos 7% de borda?
- [ ] Todo bloco tem os oito campos, e toda seção sem imagem está declarada?

## O que perguntar (e onde)

Estas perguntas entram na rodada única da Fase 3 (`auditoria-dados`, `design/lacunas.md`), nunca
numa rodada nova aqui — cada rodada extra é um telefonema do dev para o cliente. Para o que não
vier, declare a suposição e siga.

| Pergunta | Por que muda o output | Suposição se não vier |
|---|---|---|
| Logo em PNG ou SVG, fundo transparente | É o único anexo que carrega o hex exato | Só o anchor; a cor sai aproximada e a diferença aparece ao lado do logo real no header |
| 2 posts com foto de verdade, não arte com texto | Ensina tratamento fotográfico | Anexa só o logo e declara a perda de fidelidade |
| Tem foto do interior do local? Pode usar? | Decide quais seções são reais e quais são geradas | Assume que não; nenhuma seção afirma o ambiente real |
| Tem alguém autorizado a aparecer de rosto? | Decide o tratamento de pessoa | Assume que não; mãos, costas e silhueta apenas |

## Anti-patterns

- **Reescrever o anchor por seção** — causa única de página que parece montada com bancos de
  imagem diferentes. Um anchor, congelado.
- **Slot de formato sem assunto** — "esta seção precisa de uma faixa 21:9" não é motivo para
  gerar imagem. Sem assunto concreto, a faixa não existe.
- **Gradiente, textura ou bokeh para preencher** — lê como template, não tem `alt` honesto, vira
  morph sob scrub, e o CSS faz o mesmo em 0 KB.
- **Consertar texto gerado no CSS** — crop destrói a composição, blur vira mancha, overlay mata a
  foto, e no clipe o texto se move.
- **Insistir no `EXCLUDE` quando o modelo escreve** — negação pesa pouco. Remova a superfície do
  `SUBJECT`.
- **Quatro ou mais anexos** — o modelo faz média das composições e devolve colagem.
- **Anexar post sem proibir reproduzir letra** — quase todo post de marca tem tipografia, e ela
  vai parar dentro da cena gerada.
- **Anexar a foto da fachada real** — volta perto demais do prédio real e colide com a proibição
  de gerar o local do cliente.
- **Pedir o logo dentro da imagem** — o logo é SVG no DOM. Desenhado, sai errado e ainda queima o
  portão de texto zero.
- **Thread única para todas as seções** — o modelo se auto-referencia e as seções convergem.
- **"Cinematic, 8k, hyperrealistic, award-winning"** — sopa de palavra de qualidade que empurra
  para render genérico. Descreva luz, lente e material.
- **Cor da marca como esquema de cor** — produz lavagem monocromática. Amarre a objeto.
- **Aceitar um quase-lá** — uma seção 80% na marca derruba a página inteira mais do que uma seção
  sem imagem nenhuma.
