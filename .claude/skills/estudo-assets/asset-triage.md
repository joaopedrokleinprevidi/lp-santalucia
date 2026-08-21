# Triagem de assets

Apoio de `estudo-assets` (Fase 1). O resumo operacional e o esquema de `design/inventario.json`
estão no [SKILL.md](SKILL.md); aqui está o script de inventário, os limiares numéricos e as
decisões que não cabem numa tabela.

## Script de inventário

Salve em `scripts/asset-inventory.mjs`. Sem dependências de propósito: na Fase 1 o projeto npm
ainda não existe, então `sharp` não está instalado. Lê largura e altura direto do cabeçalho de
PNG, JPEG e WebP.

```js
// Uso: node scripts/asset-inventory.mjs assets-source
import { readdirSync, readFileSync, statSync } from "node:fs";
import { join, extname } from "node:path";

const dir = process.argv[2] ?? "assets-source";
const IMG = new Set([".png", ".jpg", ".jpeg", ".webp"]);

function dimensions(buf) {
  // PNG: assinatura + IHDR
  if (buf.length > 24 && buf.readUInt32BE(0) === 0x89504e47) {
    return { w: buf.readUInt32BE(16), h: buf.readUInt32BE(20) };
  }
  // JPEG: varre marcadores até um SOFn
  if (buf[0] === 0xff && buf[1] === 0xd8) {
    let i = 2;
    while (i < buf.length - 9) {
      if (buf[i] !== 0xff) { i++; continue; }
      const m = buf[i + 1];
      if (m === 0xff) { i++; continue; }                                 // byte de preenchimento
      if (m === 0x01 || (m >= 0xd0 && m <= 0xd9)) { i += 2; continue; }  // marcador sem payload
      if (m >= 0xc0 && m <= 0xcf && m !== 0xc4 && m !== 0xc8 && m !== 0xcc) {
        return { h: buf.readUInt16BE(i + 5), w: buf.readUInt16BE(i + 7) };
      }
      i += 2 + buf.readUInt16BE(i + 2);
    }
  }
  // WebP: RIFF/WEBP + variante do chunk
  if (buf.length > 30 && buf.toString("ascii", 0, 4) === "RIFF"
      && buf.toString("ascii", 8, 12) === "WEBP") {
    const fmt = buf.toString("ascii", 12, 16);
    if (fmt === "VP8X") return { w: buf.readUIntLE(24, 3) + 1, h: buf.readUIntLE(27, 3) + 1 };
    if (fmt === "VP8 ") return { w: buf.readUInt16LE(26) & 0x3fff, h: buf.readUInt16LE(28) & 0x3fff };
    if (fmt === "VP8L") {
      const b = buf.readUInt32LE(21);
      return { w: (b & 0x3fff) + 1, h: ((b >> 14) & 0x3fff) + 1 };
    }
  }
  return null;
}

for (const name of readdirSync(dir)) {
  const file = join(dir, name);
  const kb = Math.round(statSync(file).size / 1024);
  if (!IMG.has(extname(name).toLowerCase())) {
    console.log(`${name}\t${kb} KB\t(não é imagem — abrir à mão)`);
    continue;
  }
  const d = dimensions(readFileSync(file));
  const px = d ? `${d.w}x${d.h}` : "ILEGÍVEL";
  const long = d ? Math.max(d.w, d.h) : 0;
  const verdict = long >= 1600 ? "full-bleed" : long >= 1000 ? "meia-largura"
    : long >= 600 ? "thumb/prova" : "só evidência";
  console.log(`${name}\t${px}\t${kb} KB\t${verdict}`);
}
```

`ILEGÍVEL` significa formato fora dos três (HEIC de iPhone, AVIF, TIFF, PSD) ou arquivo corrompido.
HEIC é o caso comum: o cliente mandou direto do iPhone. Peça reexportação como JPEG ou converta —
não tente adivinhar as dimensões.

SVG não tem dimensão em pixel e não precisa: é vetorial, escala sem perda, e é o formato ideal de
logo. O script o marca como "não é imagem"; abra e confirme que é vetor de verdade e não um
`<image>` bitmap embrulhado em SVG, que é frequente em arquivo vindo de designer apressado.

## Limiares de renderização

O maior lado do arquivo decide a maior largura em que ele pode ser renderizado sem borrar. A conta
é `largura CSS × 2` para telas retina.

| Maior lado | Serve para | Não serve para |
|---|---|---|
| ≥2400px | Hero full-bleed em desktop grande | — |
| 1600–2400px | Full-bleed até 1200px CSS | Hero em monitor 2560 |
| 1000–1600px | Coluna de meia largura, card grande | Full-bleed |
| 600–1000px | Thumbnail, prova, avatar, card pequeno | Qualquer coisa acima de 500px CSS |
| <600px | Só evidência para leitura e para amostragem de cor | Renderizar em qualquer lugar |

**Nunca faça upscale.** Upscale comum borra; upscale de IA inventa textura e deixa a foto com
aparência plástica ao lado das outras. Um asset pequeno demais é um pedido de original, não um
problema de pipeline.

### Logo: os limiares são outros

| Situação | Veredito |
|---|---|
| `.svg` vetorial de verdade | Ideal. Escala para qualquer lugar, inclusive favicon |
| `.ai`, `.eps`, `.pdf` | Ótimo — o designer do cliente tem o original. Converta para SVG |
| PNG com transparência, maior lado ≥512px | Serve para cabeçalho e rodapé |
| PNG com transparência, 256–512px | Só cabeçalho pequeno. Registre a limitação |
| **JPEG, qualquer tamanho** | Não tem canal alpha: o logo vem com um retângulo de fundo colado. Só dá para usar sobre exatamente aquela cor, e ainda com artefato de compressão em volta das letras |
| Maior lado <256px | Insuficiente. Bloqueante |

O caso real da Santa Lúcia: `logo-santa-lucia-fundo-roxo.jpg`, 360×640, 11 KB. Roxo chapado, sem
alpha, JPEG. Serve para medir o roxo da marca e serve como prova de qual é o nome oficial. Não
serve para o cabeçalho do site. Vira pedido de original no bloco BLOQUEIA.

O script diz o tamanho; só o olho diz o conteúdo. As cinco coisas que se anota por arquivo depois de
abri-lo com `Read` estão no [SKILL.md](SKILL.md#regra-dura-leia-cada-imagem) e não se repetem aqui.

## Detecção de problema de direito de uso

| O que procurar | Como aparece |
|---|---|
| Marca d'água de banco de imagem | Texto repetido em diagonal, logotipo semitransparente no centro, grade de marcas |
| Assinatura de fotógrafo | Nome pequeno num canto, geralmente inferior direito |
| Print de rede social | Barra de status do celular no topo, ícone de curtida, nome de usuário, cantos arredondados da interface |
| Foto de acervo do Google | Costuma vir em resolução baixa e proporção estranha, e o mesmo arquivo aparece em outros negócios |
| Marca de concorrente | Placa, uniforme, embalagem ou fachada com outro nome ao fundo |
| Terceiro identificável | Rosto nítido de cliente, criança, paciente |

Nenhum desses é opinião estética. Publicar imagem sem direito expõe o cliente, e o dano cai nele.

## O que fazer com cada descarte

| Caso | Ação |
|---|---|
| Resolução baixa demais | Pergunta no bloco BLOQUEIA ou IMPORTANTE: "existe o arquivo original, maior?" |
| Marca d'água | Pergunta: "o cliente comprou essa imagem? Tem o arquivo sem marca?" Se não, substitua por imagem gerada ou por foto real |
| Rosto de terceiro | Recorte que elimine o rosto, ou descarte. Nunca desfoque e publique como se fosse escolha estética se a pessoa é reconhecível pelo contexto |
| Print de rede social | Guarde como evidência fora da pasta que vai para o repositório |
| Duplicata | Fica o maior. Registre o descarte com o motivo |
| Formato ilegível (HEIC) | Peça reexportação, ou converta e registre que a conversão foi feita |

Registre **todo** descarte em `inventario.json` — em `assets` com `usable: false` e `reason`, ou em
`discarded` quando o arquivo sai da lista de vez. A pergunta "cadê aquela foto que eu mandei?"
sempre aparece, e sem o registro não há resposta.

## Assets que vão para o repositório

Repositório público é público. Antes do primeiro commit:

- Print de conversa de WhatsApp, contrato, tabela de preço interna e documento com CPF ficam
  **fora** da pasta versionada. Servem de evidência durante o intake e param aí.
- Foto de terceiro identificável só entra com autorização escrita registrada em
  `inventario.json.assets[].rights`.
- O material impresso do cliente pode conter CNPJ, endereço residencial do sócio e dados de
  fornecedor. Leia antes de commitar.

## Quando os assets não bastam

Se a pasta tem menos do que isto, a página vai depender demais de imagem gerada e o visitante vai
sentir — imagem gerada não afirma fato sobre o negócio, então uma página feita só delas não prova
nada:

| Mínimo praticável | Quantidade |
|---|---|
| Logo utilizável | 1 |
| Foto do local (fachada ou interior) | 2 |
| Foto do trabalho sendo feito | 3 |
| Post ou material com a redação do cliente | 2 |

Abaixo disso, a pergunta certa não é "como compensamos?" e sim "o cliente consegue tirar dez fotos
com o celular esta semana?". Dez fotos ruins de celular resolvem mais que dez horas de prompt.
