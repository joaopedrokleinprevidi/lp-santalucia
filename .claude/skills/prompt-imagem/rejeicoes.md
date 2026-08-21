# Rejeições — por que o portão existe e o que fazer quando o modelo insiste

Referência de consulta do `prompt-imagem`. O SKILL.md tem as regras; este arquivo tem a causa de
cada uma e a correção quando a geração volta errada.

## Por que texto zero é absoluto

1. **Garatuja.** O modelo aproxima forma de letra; diacrítico português (`ç`, `ã`, `õ`, `ú`) sai
   deformado. `Clínica` volta `Clínlca`.
2. **Não é selecionável.** Ninguém copia o telefone que está dentro do pixel.
3. **Não é traduzível.** O Chrome traduz DOM, não imagem.
4. **Não é indexável.** O Google lê DOM. Texto em pixel não vira resultado de busca.
5. **É invisível para leitor de tela** — a menos que você duplique tudo no `alt`, e aí a mesma
   frase existe em dois lugares que divergem na primeira correção.
6. **Quebra em outro idioma e em outro tamanho.** Não reflui, não respeita `clamp()` e não
   sobrevive ao zoom de 200% que a WCAG exige.

## Nunca conserte no CSS

| "Conserto" | Por que falha |
|---|---|
| Crop em cima da placa | Destrói o enquadramento que a `COMPOSITION` pediu, e a copy perde o espaço vazio |
| Blur local | Vira mancha visível em retina e em qualquer zoom |
| Overlay / gradiente por cima | Precisa de opacidade alta para cobrir letra, e aí mata a foto inteira |
| Patch em editor de imagem | E no clipe do Flow o texto se move: você mascararia frame a frame |

## Quando o modelo insiste em escrever

Não brigue no `EXCLUDE` — ele já está lá e já foi ignorado. Negação em prompt tem peso fraco; o
substantivo positivo do `SUBJECT` pesa mais. A correção é **remover do `SUBJECT` a superfície que
convida à escrita**.

| Superfície no SUBJECT | O que o modelo escreve | Troque por |
|---|---|---|
| Placa, letreiro, fachada | Nome falso da clínica | Vista de dentro para fora; a placa fica fora do quadro |
| Monitor, tela, tablet, display | UI inventada com rótulos | Tela desligada e escura, ou monitor fora do quadro |
| Crachá, jaleco bordado, uniforme com logo | Logo borrado e errado | Jaleco liso, sem bolso frontal visível |
| Embalagem, frasco, caixa, saco de ração | Rótulo em garatuja | Frasco âmbar liso sem rótulo, ou fora do quadro |
| Prancheta, ficha, receituário, livro | Linhas de escrita falsa | Prancheta virada de costas, ou objeto sem superfície plana clara |
| Relógio, balança, termômetro | Mostrador com números tortos | Tirar do quadro — mostrador é número por definição |
| Cartaz, adesivo de porta, quadro na parede | Pôster falso | Parede lisa pintada |

Regra derivada: **toda superfície plana, clara e retangular no quadro é convite para o modelo
escrever.** Se a cena precisa de uma, coloque-a de perfil, fora de foco, ou fora do quadro.

## Por que assunto abstrato é proibido

1. **Lê como template.** Gradiente e textura genérica é o que toda landing comprada tem, e a
   página inteira existe para comprar credibilidade — o primeiro elemento que parece stock
   devolve o visitante ao ponto de partida.
2. **Não tem `alt` honesto.** `alt="forma abstrata roxa"` é ruído para leitor de tela; a saída
   correta vira `alt=""` + `aria-hidden`, e você pagou 400 KB de banda por um elemento que a
   norma classifica como decorativo — coisa que `background: linear-gradient()` faz em 0 KB.
3. **Não sobrevive ao Flow.** Forma abstrata não tem geometria estável: cada frame reinventa o
   objeto e, sob scrub, o resultado é morph psicodélico, não movimento.
4. **Peso sem informação.** Se a intenção é mesmo um gradiente, escreva CSS — mais nítido, mais
   leve, escala em qualquer densidade e muda de cor por token.

## Quando o anexo vira cópia

Anexo em GPT puxa para **composição**, não só para estilo.

| Falha | Sintoma | Correção |
|---|---|---|
| Referência pesa demais | Três seções voltam com o mesmo crop, mesmo objeto no mesmo canto do post anexado | Corte para o logo apenas e alongue `SUBJECT`/`COMPOSITION` — descrição específica tira peso do anexo |
| Post tem texto (quase todos têm) | O modelo copia as letras para a cena gerada | A frase de anexo precisa proibir explicitamente reproduzir marca ou letra |
| Foto real do local anexada | Volta algo perto demais do prédio real | Só anexe foto de **interior**, e mande inventar outra sala |
