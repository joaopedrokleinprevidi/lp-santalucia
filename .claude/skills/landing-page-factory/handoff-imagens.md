# Handoff de imagens e animações

Seu trabalho nesta etapa: pegar prompts prontos, colar no ChatGPT para gerar uma imagem por
seção do site, e depois subir cada imagem no Google Flow para virar um clipe de 5 segundos.

**Quanto tempo leva:** é a etapa mais lenta do projeto inteiro, e a culpa não é sua. Você vai
rejeitar imagem e gerar de novo várias vezes. Para um site de 8 seções, conte de **15 a 30
gerações de imagem** até fechar as 8 boas, mais uma rodada parecida nos clipes. Reserve algumas
horas, não vinte minutos. Rejeitar rápido é mais barato que aceitar algo ruim.

Você não precisa entender de IA. Precisa copiar, colar, olhar com atenção e dizer "não" quando
estiver errado. O julgamento é o trabalho.

---

## Antes de começar — checagem de dois minutos

| Confira | Onde |
|---|---|
| O arquivo com os prompts de imagem existe | `design/image-prompts.md` |
| O arquivo com os prompts de animação existe | `design/motion-prompts.md` |
| A pasta de saída existe (crie se não existir) | `design/renders/` |
| Os arquivos que os prompts mandam anexar existem | pasta de assets do cliente |
| Você consegue entrar no ChatGPT e no Google Flow | — |

Se faltar qualquer um dos dois primeiros arquivos, pare. Eles são gerados na etapa anterior e
não é você quem escreve.

---

# PARTE A — gerar as imagens no ChatGPT

## O Style Anchor: leia isto antes do passo 1

No topo do `design/image-prompts.md` tem um parágrafo em inglês chamado **STYLE ANCHOR**.

O ChatGPT não lembra da imagem anterior. Cada geração é um estranho começando do zero. Se você
pedir oito imagens sem esse parágrafo, recebe oito fotos de oito fotógrafos diferentes, e a
página vira colagem.

O Style Anchor descreve a luz, a lente e as cores da marca. Colado junto de todo prompt, ele é o
que faz as oito imagens parecerem tiradas pelo mesmo fotógrafo, no mesmo dia, no mesmo lugar.

Três regras sobre ele, e não tem exceção:

1. Vai **junto de todo prompt**, sempre, inclusive no décimo quinto que você gerar.
2. Vai **sem editar uma vírgula**. Nem para "melhorar", nem para encurtar.
3. Se ele mudar no meio do caminho, as imagens já feitas param de combinar com as novas. Aí você
   refaz **todas**, não só as novas.

Você não precisa entender o que ele diz em inglês. Precisa não mexer nele.

## Passo a passo

1. **Abra o `design/image-prompts.md`.** Cada seção do site tem um bloco. O bloco traz o papel
   da imagem, a proporção, o que anexar, o prompt completo e o teste de aceitação.

2. **Comece pela seção 01 e siga a ordem.** A primeira imagem costuma ser a do hero, e é ela que
   as outras vão ter que combinar. Fechar o hero primeiro economiza retrabalho.

3. **Abra uma conversa NOVA no ChatGPT para cada seção.** Não continue na mesma conversa da
   seção anterior. Numa conversa longa o modelo passa a copiar as imagens que ele mesmo gerou
   antes, e a seção 6 volta parecida com a seção 5 com outro objeto dentro.

4. **Anexe os arquivos que o bloco mandar, na ordem em que ele lista.** Cada bloco tem uma
   tabela "ANEXAR" dizendo qual arquivo e para que ele serve. Normalmente é o logo, um ou dois
   posts do Instagram da marca, e a foto real do local quando a seção fala daquele ambiente.
   Nunca anexe mais de três arquivos: com quatro o modelo faz uma média das referências e
   devolve uma colagem.

5. **Monte o envio nesta ordem, num único envio:** os anexos, depois o Style Anchor colado
   inteiro, depois o prompt da seção, e por último a frase de anexo abaixo.

6. **Cole esta frase, exatamente assim, depois do prompt.** Ela é o que impede o modelo de
   copiar o enquadramento da referência em vez do estilo dela:

   ```
   Use the attached images as reference for color, light and material treatment only.
   Do not copy their framing, composition, crop or subject placement.
   Do not reproduce any logo, mark, lettering or symbol seen in them.
   ```

   Em português, ela diz: use os anexos como referência de cor, luz e material — não copie o
   enquadramento nem o assunto, e não redesenhe nenhum logo ou letra que aparecer neles. Sem
   essa frase, o modelo tende a devolver o mesmo recorte do post que você anexou, e a copiar as
   letras que existem dentro dele.

7. **Gere na maior resolução que a ferramenta oferecer.** Se houver escolha de tamanho ou
   qualidade, escolha sempre a maior.

8. **Nunca amplie a imagem depois.** Ampliar não recupera detalhe que não foi gerado — o
   programa inventa pixels, e o resultado fica com cara de plástico ou de borrão. O nosso
   processo só sabe **reduzir** a imagem para versões leves. Ele nunca inventa resolução. Uma
   imagem gerada pequena fica pequena para sempre.

9. **Olhe a imagem com a lista de rejeição na mão** (a seção seguinte). Se reprovar, gere de
   novo. Não tente consertar depois.

10. **Salve com o nome exato que o bloco pede**, dentro de `design/renders/`:

    ```
    design/renders/NN-id-secao.png
    ```

    Exemplo: `design/renders/04-estrutura.png`. O `NN` é o número da seção com dois dígitos
    (01, 02, ... 10). O nome não é decoração: o site procura o arquivo por esse nome. Um nome
    diferente e a imagem simplesmente não aparece na página.

11. **Repita para a próxima seção,** em conversa nova.

---

## A lista de rejeição das imagens

Olhe cada item. Qualquer um que bater, a imagem volta para geração.

| O que você vê | Por que derruba a página | O que fazer |
|---|---|---|
| Qualquer letra, palavra, número, placa escrita, etiqueta, rótulo, legenda ou relógio com mostrador | O modelo não escreve português de verdade: "Clínica" volta como "Clínlca", e o acento sai torto. Além disso, texto dentro da imagem não pode ser copiado, nem traduzido, nem lido pelo Google, nem lido por quem usa leitor de tela. Todo o texto do site é digitado no código, nunca desenhado na foto | Rejeita sempre, sem exceção |
| Um logo desenhado dentro da cena, mesmo que pareça o certo | O logo de verdade entra por cima, no código, nítido e na cor certa. Um logo desenhado pelo modelo é uma imitação errada | Rejeita |
| Rosto de pessoa que dá para reconhecer | Não temos autorização de imagem de ninguém. Um rosto identificável numa página comercial é problema jurídico, e não é nosso para resolver. Pessoa pode aparecer: mãos, antebraços, costas, ombro, silhueta fora de foco | Rejeita |
| Mão, dedo, pata, orelha ou olho deformado | É o erro clássico da IA: seis dedos, pata com articulação errada, olho torto. Passa despercebido na miniatura e salta aos olhos na tela cheia | Rejeita |
| O espaço vazio não está onde o prompt pediu | O prompt reserva uma área lisa e sem detalhe porque **é ali que o texto do site cai por cima**. Se essa área foi preenchida por um objeto, o texto fica ilegível e não tem conserto sem estragar a foto | Rejeita |
| Colocada ao lado da imagem do hero, parece de outra marca | Cor diferente, luz diferente, "clima" diferente. Uma página em que uma seção destoa parece feita às pressas | Rejeita e confira se o Style Anchor foi colado inteiro |
| A cor da marca virou um banho de cor sobre a imagem toda | A foto parece pintada de roxo. Isso denuncia IA em meio segundo. A cor da marca tem que estar em objetos reais da cena: parede pintada, uniforme, metal, luz de luminária | Rejeita |
| A imagem repete o mesmo recorte do post que você anexou | O modelo usou a referência como molde de enquadramento, não como estilo | Rejeita, anexe só o logo e gere de novo |

**Como olhar mão, pata e olho:** abra o arquivo salvo e aumente o zoom até que a mão (ou a pata,
ou o rosto do animal) ocupe boa parte da tela. Conte os dedos. Veja se as duas orelhas fazem
sentido. Miniatura esconde defeito; zoom mostra.

**A regra que resume todas:** uma seção 80% certa derruba a página mais do que uma seção sem
imagem nenhuma. Sem imagem, a seção fica com texto grande sobre cor sólida e parece uma escolha
de design. Com imagem quase certa, parece descuido — e descuido contamina a confiança no resto
da página, inclusive no telefone e no endereço.

Nunca aceite pensando "depois a gente arruma no site". Não se arruma. Recortar em cima da placa
destrói o espaço reservado para o texto. Borrar vira uma mancha visível. Escurecer por cima
precisa de tanta opacidade que mata a foto. E se a imagem virar clipe, o texto ainda se mexe.

---

# PARTE B — animar no Google Flow

## Por que os prompts pedem movimento lento e um movimento só

Leia isto antes de gerar o primeiro clipe. Sem entender esta parte, você vai aceitar clipes
bonitos e inúteis.

Esses vídeos **não tocam sozinhos no site**. Quem toca é a rolagem do visitante. Ele rola para
baixo, o vídeo anda para a frente. Para de rolar, o vídeo para. Rola para cima, o vídeo volta.
É como se o dedo dele fosse o botão de avanço e retrocesso.

Isso muda tudo:

- **Um corte no meio do clipe** (a cena muda de lugar) vira um salto de imagem no meio da
  rolagem. O visitante não pensa "que edição interessante". Ele pensa que a página bugou.
- **Movimento rápido** pisca. Vídeo gerado por IA não tem borrão de movimento como uma câmera de
  verdade — cada quadro é nítido e independente. Quando a rolagem passa depressa por quadros
  nítidos e muito diferentes entre si, a tela cintila.
- **Câmera que vai e volta** faz a imagem ir e voltar enquanto o visitante rola só para a
  frente. Parece que a página travou.
- **Tremor de câmera** (aquele balanço de câmera na mão) fica dez vezes pior quando o vídeo é
  controlado pelo dedo. O que passaria despercebido tocando normal vira vibração.

Por isso todo prompt pede: um movimento contínuo, uma direção só, devagar, sem corte. Não é
preciosismo. É o que sobrevive ao jeito que o site usa o vídeo.

## Passo a passo

1. **Abra o `design/motion-prompts.md`.** Ele tem um bloco por seção, na mesma ordem do arquivo
   de imagens. Cada bloco diz qual imagem animar.

2. **Só anime seções que já têm a imagem aprovada.** Animar um still que você ainda vai rejeitar
   é jogar tempo fora duas vezes.

3. **Abra o Google Flow** em [labs.google/flow](https://labs.google/flow).

4. **Suba a imagem da seção** (`design/renders/NN-id-secao.png`). Ela é o primeiro quadro do
   clipe — o vídeo começa exatamente nela.

5. **Cole o prompt de movimento** do bloco correspondente, sem editar.

6. **Gere um clipe de 5 segundos.** Se a ferramenta deixar escolher, fique entre 4 e 6 segundos.
   Menos que isso não dá quadros suficientes e o movimento fica engasgado. Mais que isso pesa
   demais para carregar no celular.

7. **Assista com a lista de rejeição na mão** (seção seguinte).

8. **Baixe e salve com o mesmo nome da imagem, trocando a extensão:**

   ```
   design/renders/NN-id-secao.mp4
   ```

   Exemplo: a imagem `04-estrutura.png` vira `04-estrutura.mp4`, na mesma pasta. Mesmo número,
   mesmo nome. É assim que o site sabe que os dois são da mesma seção.

9. **Repita para cada seção que tiver bloco de movimento.** Nem toda seção tem — algumas ficam
   só com a imagem parada, e isso está declarado no arquivo.

**Se o Google Flow não estiver disponível para você** (região, plano ou créditos acabados): não
improvise em outra ferramenta sem avisar. Me diga, e a página passa a usar as imagens paradas
com um movimento suave feito por código. Fica bom, e é melhor do que oito clipes de qualidade
irregular.

---

## A lista de rejeição dos clipes

Assista ao clipe inteiro pelo menos duas vezes. Depois assista devagar, arrastando a barrinha do
player com o mouse — é assim que o visitante vai ver.

| O que você vê | Por que derruba | O que fazer |
|---|---|---|
| Um corte, uma troca de cena, uma "virada" para outro ângulo | Vira salto de imagem no meio da rolagem. Lido como bug | Rejeita |
| Tremor de câmera, balanço tipo câmera na mão | O controle por rolagem amplifica o tremor em vibração | Rejeita |
| O movimento acelera ou desacelera no meio | A rolagem é constante; a velocidade do vídeo tem que ser também. Se mudar, parece que a página engasgou | Rejeita |
| A câmera avança e depois recua (ou vira para um lado e volta) | Rolando só para a frente, a imagem vai e volta sozinha. Parece travamento | Rejeita |
| Dois movimentos ao mesmo tempo ou em sequência | Na emenda entre eles dá um tranco visível | Rejeita |
| Alguma letra, número ou placa apareceu durante o movimento (mesmo que não estivesse na imagem) | Mesma regra da imagem, e pior: aqui o texto errado ainda se mexe | Rejeita |
| Mão, pata ou orelha que deforma enquanto se move | Erro de IA que só aparece em movimento. Olhe as extremidades, não o centro | Rejeita |

**O teste do último quadro — não pule este.** Pause o vídeo no finalzinho e olhe só aquele
quadro parado. Ele precisa ser uma boa fotografia sozinho, tão boa quanto a imagem inicial.

Motivo: o clipe não fica em loop. Ele para no último quadro e descansa lá. Então esse quadro é o
que fica **mais tempo na tela** de quem terminou de rolar a seção. Ele também é a imagem que
aparece para quem ligou "reduzir animações" no celular, e é o que sai num print da página.

Se o último quadro ficou com o assunto cortado pela metade, desfocado ou apontando para o nada,
o clipe está reprovado, mesmo que os primeiros quatro segundos sejam lindos.

---

## Quando voltar para mim

**A regra das três tentativas.** Gerou três vezes o mesmo prompt e nenhuma passou na lista de
rejeição? Pare. Não gere a quarta.

Três resultados ruins seguidos não são azar. São sintoma de que **o prompt está pedindo a coisa
errada** — normalmente ele descreve algo que o modelo não consegue desenhar sem errar, ou uma
superfície que convida o modelo a escrever. A quarta tentativa tem a mesma chance das três
primeiras. A correção é reescrever o prompt, e quem reescreve sou eu.

**Me mande, numa mensagem só:**

1. O número e o nome da seção (ex.: "04 estrutura").
2. Em uma linha, o que voltou errado nas três tentativas.
3. Um print ou o arquivo de uma das tentativas, se puder.

Com isso eu já sei o que mudar. Alguns exemplos do que costuma acontecer e do que eu faço:

| O que você me conta | O que normalmente é a causa | O que eu mudo |
|---|---|---|
| "Sempre aparece uma placa escrita / um rótulo / uma tela com escrita" | A cena tem uma superfície plana, clara e retangular. Isso é um convite para o modelo escrever | Tiro a superfície da cena ou coloco ela de lado, fora de foco |
| "As mãos sempre saem tortas" | O prompt está pedindo um gesto complicado | Troco por um gesto mais simples, ou tiro a mão do quadro |
| "Volta sempre com o mesmo recorte do post que anexei" | A referência anexada está pesando demais | Corto os anexos para só o logo e detalho mais a descrição da cena |
| "Fica tudo roxo, parece pintado" | A cor da marca está sendo pedida como clima, não como objeto | Amarro a cor a objetos reais: parede, uniforme, metal, luminária |
| "Não sobra espaço para o texto" | A descrição de composição está fraca | Reescrevo dizendo exatamente qual faixa da imagem fica lisa |
| "O clipe sempre tem corte" | O movimento pedido é complexo demais para 5 segundos | Simplifico para um único movimento de câmera bem lento |

Se uma seção continuar impossível depois do prompt reescrito, ela fica **sem imagem**, com texto
grande sobre cor sólida. É uma saída legítima e às vezes melhor. Não é derrota.

---

## Checklist final — antes de dizer "terminei"

Abra a pasta `design/renders/` e confira item por item. Todos precisam estar marcados.

**Arquivos**

- [ ] Existe um `.png` para cada seção que o `image-prompts.md` pediu imagem
- [ ] Existe um `.mp4` para cada seção que o `motion-prompts.md` pediu clipe
- [ ] Todos os nomes seguem `NN-id-secao` exatamente como está escrito no bloco do prompt
- [ ] O `.mp4` de cada seção tem o mesmo nome do `.png` dela
- [ ] Está tudo dentro de `design/renders/`, sem subpasta e sem cópia solta em outro lugar
- [ ] Nenhum arquivo se chama "final", "final2", "copia" ou "novo"

**Imagens**

- [ ] Nenhuma tem letra, palavra, número, placa, etiqueta, rótulo, mostrador ou logo desenhado
- [ ] Nenhuma tem rosto de pessoa que dê para reconhecer
- [ ] Olhei mãos, patas, orelhas e olhos com zoom, não na miniatura
- [ ] Colocadas lado a lado, as oito parecem do mesmo ensaio
- [ ] O espaço liso reservado para o texto está onde o prompt pediu, em todas
- [ ] Nenhuma foi ampliada depois de gerada

**Clipes**

- [ ] Nenhum tem corte ou troca de cena
- [ ] Nenhum tem tremor de câmera
- [ ] Nenhum muda de velocidade no meio
- [ ] Nenhuma câmera avança e recua dentro do mesmo clipe
- [ ] O último quadro de cada um funciona sozinho como fotografia

**Controle** — preencha conforme fecha cada seção:

| # | Seção | PNG salvo | MP4 salvo | Passou na lista |
|---|---|---|---|---|
| 01 | | | | |
| 02 | | | | |
| 03 | | | | |
| 04 | | | | |
| 05 | | | | |
| 06 | | | | |
| 07 | | | | |
| 08 | | | | |

Com tudo marcado, me avise. A próxima etapa é minha: o site é construído em volta desses
arquivos, e a partir daí você só volta a aparecer para conferir os dados de contato e autorizar
a publicação.
