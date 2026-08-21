# Anti-plágio — o script do portão

O gate roda duas vezes com o mesmo arquivo: na **Fase 4**, sem copy no disco, ele confere que a
lista de banidas existe e tem tamanho útil; na **Fase 6**, com o caminho da copy como argumento,
ele reprova a copy que repetir uma frase do setor.

Grave-o em `scripts/check-banned-copy.mjs` do projeto — não deixe o código só nesta skill, porque
o portão da Fase 6 é rodado por `copy-conversao` e por `landing-page-factory`, e um script que só
existe na cabeça de quem pesquisou não roda.

```js
// scripts/check-banned-copy.mjs
// Fase 4, sem argumento e sem copy no disco: modo linha de base — confere que a lista existe.
// Fase 6: node scripts/check-banned-copy.mjs src/data/site.ts — o portão contra a copy final.
import { existsSync, readFileSync } from 'node:fs'

const norm = (s) =>
  s.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, '')
    .replace(/[^a-z0-9\s]/g, ' ').replace(/\s+/g, ' ').trim()

const section = (md, title) => md.split(`## ${title}`)[1]?.split('\n## ')[0] ?? ''

// A frase é o que está entre aspas; sem aspas, o que vem antes do travessão da contagem.
// Sem isso, o "— 3/3" entra na frase e nenhuma comparação casa.
const phraseOf = (line) => {
  const quoted = line.match(/["“”'](.+?)["“”']/)
  return norm(quoted ? quoted[1] : line.replace(/^-\s*/, '').split(/\s[—–-]\s/)[0])
}
const bullets = (block) =>
  block.split('\n').filter((l) => l.startsWith('- ')).map(phraseOf).filter(Boolean)

const grams = (s, n) => {
  const w = s.split(' ')
  return new Set(w.slice(0, Math.max(0, w.length - n + 1)).map((_, i) => w.slice(i, i + n).join(' ')))
}

const banned = bullets(section(readFileSync('design/pesquisa.md', 'utf8'), 'O que todo mundo diz'))
const copyPath = process.argv[2] ?? 'src/data/site.ts'

if (!existsSync(copyPath)) {
  // Ainda não há copy: valide a própria lista. Menos de 3 frases significa que os
  // concorrentes foram lidos procurando inspiração, não repetição.
  console.log(`baseline: ${banned.length} frases banidas\n${banned.map((b) => `  · ${b}`).join('\n')}`)
  process.exit(banned.length >= 3 ? 0 : 1)
}

const copy = norm(readFileSync(copyPath, 'utf8'))
const copy4 = grams(copy, 4)

const hits = banned.flatMap((phrase) => {
  const words = phrase.split(' ')
  if (words.length < 4) return copy.includes(phrase) ? [phrase] : []
  return [...grams(phrase, 4)].filter((g) => copy4.has(g))
})

console.log(hits.length ? `PROIBIDO: ${[...new Set(hits)].join(' | ')}` : 'ok — nenhuma frase do setor na copy')
process.exit(hits.length ? 1 : 0)
```

## Por que 4 palavras

| n-grama | O que acontece |
|---|---|
| 3 | Dispara em construção comum do português — "para você e", "o melhor do" — e o portão vira ruído que ninguém roda |
| **4** | Pega a paráfrase preguiçosa sem pegar a gramática |
| 5 | Deixa passar a frase do concorrente com uma palavra trocada, que é exatamente o caso que o portão existe para pegar |

Duas decisões acompanham o limiar:

- **A normalização remove acento e pontuação** antes de comparar. Sem isso, "atendimento
  humanizado," e "atendimento humanizado" são strings diferentes e o portão passa.
- **Só promessa entra na lista de banidas.** Horário, endereço e forma de pagamento
  compartilhados com o concorrente são fato, não plágio; incluí-los inflaria o script com falso
  positivo até alguém desligá-lo.

## Quando o portão reprova

A saída é o n-grama encontrado, não a linha da copy. Procure-o com `Grep` no arquivo apontado.

Reprovou não significa reescrever a frase inteira: significa que o **ângulo** é o do setor. Troque
a afirmação pela brecha (`## A brecha` do `design/pesquisa.md`), que por construção nenhum
concorrente diz. Trocar sinônimo mantém o ângulo e o portão volta a reprovar na próxima rodada
com outro n-grama.

Se a lista de banidas tiver menos de 3 frases na Fase 4, o problema não é o script: a frente 4 da
pesquisa foi feita procurando inspiração em vez de repetição. Refaça a contagem de afirmações
descrita em [fontes.md](fontes.md#contagem-de-afirmações).
