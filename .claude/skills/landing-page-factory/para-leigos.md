# O processo, sem jargão

Você tem uma pasta com as fotos de um cliente e o telefone dele. No fim, tem um site no ar.
Este documento explica o que acontece no meio, o que você precisa ter em mãos, e em quais
momentos vão te pedir alguma coisa.

## O que você precisa antes de começar

**Obrigatório:**

1. **Uma pasta com o material do cliente.** Logo, fotos da loja, fotos da equipe, posts que ele
   já publicou no Instagram, material impresso. Quanto mais, melhor — é daí que sai a
   identidade visual. Não precisa estar organizado.

2. **Os dados de contato, confirmados pelo cliente.** Telefone, WhatsApp, endereço completo,
   horário de funcionamento. "Confirmado" quer dizer que o cliente olhou e disse que está certo
   — não que você achou no Google. Um dígito errado no telefone e o site inteiro não serve.

3. **Uma conta no GitHub** e outra **na Vercel**. As duas são grátis. Veja
   [credentials.md](credentials.md) para criar e conectar.

**Para as imagens e animações:**

4. **Acesso ao ChatGPT** (ou outro gerador de imagem). Você vai colar prompts prontos lá e
   salvar as imagens que saírem.

5. **Acesso ao Google Flow.** Você vai subir cada imagem gerada e colar um prompt de animação
   para transformá-la num vídeo de 5 segundos.

## O que acontece, fase por fase

### Fase 1 — Ler a marca

O computador abre cada imagem do cliente e mede as cores pixel a pixel. Não é "achar que o roxo
é mais ou menos esse" — é medir e descobrir que o roxo da marca é exatamente `#603084` e que ele
ocupa 87% do logo.

Também identifica o tipo de letra, as formas que se repetem (corações, patinhas, cantos
arredondados) e separa o que é **fato** do que é **suposição**.

*Você não faz nada nesta fase.* Sai um arquivo chamado `design-system.json`.

### Fase 2 — Decidir o que a página diz

Aqui se define a ordem das seções e se escreve todo o texto do site, do zero.

Não é o texto do site antigo com outras palavras — é texto novo. Se o site antigo convertesse
bem, o cliente não estaria pedindo um novo.

Para dar certo, três versões diferentes da página são escritas em paralelo, por três "cabeças"
com abordagens opostas. Depois, três avaliadores diferentes julgam as três (um olha conversão,
outro olha se o português está bom, outro olha se a experiência é memorável). No fim, a melhor
vence e rouba as boas ideias das outras duas.

*Você não faz nada.* Sai o `landing-blueprint.md` com o texto pronto para você revisar.

**Revise este arquivo.** É o único momento barato de mudar de ideia.

### Fase 3 — Você gera as imagens

Sai um arquivo `image-prompts.md` com um prompt por seção do site.

No topo dele tem um parágrafo chamado **Style Anchor**. Ele é o segredo da consistência: você
cola esse mesmo parágrafo junto com todos os prompts. É o que faz oito imagens geradas
separadamente parecerem tiradas pelo mesmo fotógrafo, no mesmo dia.

**Não edite o Style Anchor no meio do caminho.** Se editar na quinta imagem, as quatro
primeiras deixam de combinar e você refaz tudo.

**O que fazer:**
1. Abra o ChatGPT.
2. Cole: o Style Anchor + o prompt da seção 1.
3. Salve a imagem em `assets/generated/`, com o nome da seção.
4. Repita para cada seção.

**Quando rejeitar e gerar de novo:** se apareceu texto ou letra na imagem (o modelo escreve
português acentuado como garatuja), se a patinha ou a mão saiu deformada, se ficou com cara de
outra marca, ou se o espaço vazio não está onde o prompt pediu — esse espaço é onde o texto do
site vai cair por cima.

### Fase 4 — Você anima as imagens

Sai um arquivo `motion-prompts.md`. Cada bloco diz qual imagem animar e qual movimento pedir.

**O que fazer:**
1. Abra o Google Flow.
2. Suba a imagem da seção.
3. Cole o prompt de movimento correspondente.
4. Gere um clipe de 5 segundos.
5. Salve em `assets/generated/`.

**As regras estranhas têm motivo.** Os prompts pedem movimento lento, um só movimento, sem
corte e sem tremor. Isso não é preciosismo: o site vai controlar esse vídeo pelo *scroll* do
mouse. Se tiver um corte no meio, quando o visitante rolar a página, vai parecer um defeito.
Se o movimento for rápido, vai piscar.

**Quando rejeitar:** qualquer corte, qualquer tremor de câmera, qualquer mudança de velocidade.
E olhe o último quadro do vídeo — ele é o que fica mais tempo na tela.

### Fase 5 — Construir o site

O código é escrito: layout, cores, tipografia, as animações, o carregamento das imagens em
formatos leves, e todo o SEO (o que faz o Google entender o site e o que faz aparecer a
capinha bonita quando alguém manda o link no WhatsApp).

*Você não faz nada.* No fim, o site roda na sua máquina para você olhar.

### Fase 6 — Auditoria

Um especialista revisa: funciona no celular? Dá para navegar só com o teclado? Uma pessoa cega
usando leitor de tela entende? O contraste das cores é suficiente? Carrega rápido no 4G ruim?

**Esta fase pode reprovar e mandar voltar.** É de propósito. Acessibilidade que fica para
"depois" nunca é feita.

### Fase 7 — Publicar

O código vai para o GitHub (onde fica guardado e versionado) e o site vai para o ar na Vercel
(que hospeda de graça e dá um endereço). Se o cliente tiver domínio próprio, aponta para lá.

*Você confirma que quer publicar.* Depois disso, o site está no ar.

## Perguntas que sempre aparecem

**Quanto tempo leva?**
As fases 1, 2 e 5 são as demoradas do lado do computador. As fases 3 e 4 dependem de você
gerar as imagens e os vídeos — normalmente é o trecho mais lento, porque envolve rejeitar e
refazer. Um projeto de 8 seções costuma pedir de 15 a 30 gerações de imagem até fechar.

**Posso pular a geração de imagem e usar só fotos reais?**
Pode, e às vezes deve. Foto real do cliente é sempre melhor quando a seção fala do negócio
dele — a fachada, a equipe, a estrutura. Imagem gerada serve para seções ilustrativas e de
atmosfera. Aliás, é proibido gerar a fachada ou a equipe: seria inventar como é um lugar real
e pessoas reais.

**E se o cliente não gostar do texto?**
Revise na Fase 2, antes das imagens existirem. Mudar o texto depois que as imagens foram
geradas é caro, porque a composição de cada imagem foi desenhada em volta daquele texto.

**Preciso saber programar?**
Não para rodar o processo. Você precisa saber copiar e colar prompts, julgar se uma imagem
ficou boa, e confirmar dados com o cliente.

**Quanto custa?**
A hospedagem é grátis no plano gratuito da Vercel. O GitHub é grátis. O custo é o de gerar
imagem e vídeo, que depende do plano que você tem no ChatGPT e no Google Flow.

**E se faltar uma credencial?**
Você descobre logo na primeira verificação, antes de qualquer trabalho. Está tudo listado em
[credentials.md](credentials.md).
