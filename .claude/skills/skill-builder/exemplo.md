# Exemplo completo de skill

Skill mínima e completa, calibrada para este repositório. Ela cabe em menos de 150 linhas porque
delega toda regra a outras skills — é o piso do que ainda conta como skill, não o alvo.

**Arquivo:** `.claude/skills/chapter-audit/SKILL.md`

````markdown
---
name: chapter-audit
description: Use when auditing a chapter component against the project's motion and accessibility rules. Audita um capitulo de src/components/chapters contra as regras de motion: matchMedia, revert no cleanup, so transform e opacity, entrada diferente da anterior, altura minima com pin, copy no DOM, contraste 4.5:1. Palavras-chave: auditar capitulo, conferir timeline, capitulo quebrado.
argument-hint: [nome-do-capitulo]
allowed-tools: Read, Grep, Glob, Bash(npm run typecheck)
---

# Chapter Audit

| | |
|---|---|
| **ENTRADA** | `src/components/chapters/Chapter$0.tsx` e todo hook que ele importa de `src/hooks/` |
| **SAÍDA** | Relatório em texto: reprovações com arquivo e linha. Não edita nada |
| **ANTES** | `landing-motion-expert` e os especialistas que ele roteia (Fase 10) |
| **DEPOIS** | `audit-responsivo` (Fase 11a) — este relatório é insumo, não substituto |

Verifica um capítulo contra as regras de motion do projeto e devolve uma lista de correções com
arquivo e linha. Não conserta: editar um capítulo é trabalho de `gsap-scrolltrigger-expert`.

Sem argumento, audite todos os arquivos de `src/components/chapters/`.

## Passos

1. Leia o componente e todo hook importado de `src/hooks/`.
2. Verifique cada regra abaixo. Cite arquivo e linha em cada reprovação.

| # | Regra | Reprova quando |
|---|---|---|
| 1 | Timeline dentro de `gsap.matchMedia` com escopo | efeito sem guard de reduced motion |
| 2 | Cleanup chama `mm.revert()` ou `ctx.revert()` | cleanup ausente — Strict Mode registra em dobro |
| 3 | Só `transform` e `opacity` animados | `margin`, `top`, `width` na timeline — dispara layout na main thread |
| 4 | Entrada diferente da do capítulo anterior | mesma direção duas vezes seguidas |
| 5 | Altura do capítulo ≥ 300vh se tem pin | pin com scroll curto — a cena passa antes de ser lida |
| 6 | Copy é texto no DOM, em ordem de leitura | texto dentro de canvas ou imagem |
| 7 | Contraste do corpo ≥ 4.5:1 sobre o fundo | abaixo de 4.5:1 — o número é de `audit-acessibilidade` |

3. Rode `npm run typecheck`. Erro de tipo entra como reprovação bloqueante.

## Saída

## Auditoria: Chapter$0

**Reprovações**
- [regra N] `caminho:linha` — <o que está errado>
  Conserto: <mudança concreta>

**Aprovadas:** N/7  •  Typecheck: <ok | N erros>

## Notas

- Não conserte nada.
- Capítulo sem timeline não reprova nas regras 1–5; anote "estático, fora de escopo".
- Não invente número de linha. Sem localizar, escreva "arquivo inteiro".
````

## O que este exemplo demonstra

| Elemento | Por que está aí |
|---|---|
| Contrato no topo com `$0` no caminho de ENTRADA | O argumento aparece onde ele muda o comportamento, não como enfeite |
| `allowed-tools` só de leitura + um comando | Coerente com uma skill que declara não editar. Frontmatter que contradiz o corpo é a primeira coisa que alguém ignora |
| Tabela com **critério de reprovação**, não lista de boas práticas | "Reprova quando" é executável; "prefira" não é |
| Regra 7 nomeando a dona do número | O 4.5:1 é de `audit-acessibilidade`. Escrevê-lo aqui sem dono cria a segunda verdade |
| Template de saída literal | Duas execuções produzem o mesmo formato, então o relatório é comparável entre semanas |
| Nota que impede invadir escopo | Sem ela, a skill começa a consertar e vira uma segunda `gsap-scrolltrigger-expert` |
| `DEPOIS` apontando um portão | Deixa claro que este relatório não substitui a Fase 11 |

## O que ele deliberadamente não tem

- **`context: fork`** — o relatório precisa voltar para a conversa que pediu a auditoria.
- **`disable-model-invocation`** — a skill não gasta nada e não destrói nada.
- **Seção de acessibilidade no rodapé** — a regra de contraste está na tabela, no passo em que é
  verificada.
- **Números próprios** — todo threshold citado tem dona nomeada em outra skill.
