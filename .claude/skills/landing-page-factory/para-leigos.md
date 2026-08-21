# O processo, sem jargão

Você tem uma pasta com as fotos de um cliente e o telefone dele. No fim, tem um site no ar.

Este documento explica o que acontece no meio, o que vão te pedir, e — principalmente — o que
**não** é seu trabalho.

## O seu trabalho inteiro, em quatro linhas

1. Trazer os dados da empresa e as imagens que o cliente tiver.
2. Dizer "gere".
3. Depois, colar prompts no ChatGPT para gerar as imagens e no Google Flow para animá-las.
4. Aprovar o login no navegador quando uma credencial for pedida.

Só isso. Não existe passo 5.

## Como começar, literalmente

Crie uma pasta para o projeto, jogue o material do cliente dentro dela numa subpasta chamada
`assets-source/`, abra o Claude Code **nessa pasta** e escreva:

```
/landing-page-factory assets-source "Nome do Cliente"
```

A pasta em que você abriu o Claude Code é a pasta do projeto: tudo — o briefing, os prompts, as
imagens que você vai gerar e o site — nasce dentro dela. Não abra numa pasta e mande gerar em
outra; os caminhos dos arquivos param de bater.

Se preferir, escreva em português mesmo: "roda a landing page factory na pasta `assets-source`
para o cliente Fulano". O efeito é o mesmo.

| O que **VOCÊ** faz | O que **EU** faço |
|---|---|
| Junta a pasta de assets e responde o briefing | Leio cada arquivo, meço as cores, monto o design system |
| Confirma dados de contato com o cliente | Pesquiso o nicho, os concorrentes e as avaliações reais |
| Diz "gere" | Escrevo toda a copy, do zero, e decido a ordem das seções |
| Cola prompt no ChatGPT e salva o PNG | Escrevo os prompts, decido quais arquivos você anexa e onde salvar |
| Cola prompt no Google Flow e salva o MP4 | Escolho o movimento que o scroll vai conseguir controlar |
| Olha o site e diz se gostou | Escrevo o código, o layout, as animações, o SEO e a acessibilidade |
| Aprova o login no navegador | Crio o repositório, ligo na Vercel e publico |
| Decide se quer domínio próprio | Aponto o DNS e conto o que colar no registrador |

Você não precisa saber programar. Precisa saber copiar e colar, julgar se uma imagem ficou boa, e
confirmar dado com o cliente.

## O que ter em mãos antes de dizer "gere"

**Obrigatório:**

1. **A pasta `assets-source/` com o material do cliente.** Logo, fotos da loja, fotos do trabalho
   feito, posts do Instagram, material impresso. Não precisa estar organizado, e não precisa estar
   bonito — foto feia de lugar real vale mais que render bonito de lugar que não existe. O
   tratamento é comigo. O nome da pasta importa: é dela que o site puxa as imagens, e é ela que
   fica de fora do GitHub para o material cru do cliente não virar público.

2. **Os dados de contato, confirmados pelo cliente.** Telefone, **WhatsApp com DDD** (quase sempre
   é um número diferente do fixo), endereço com CEP, horário incluindo feriado. "Confirmado" quer
   dizer que o cliente olhou e disse que está certo — não que você achou no Google. Um dígito
   errado no telefone e o site inteiro não serve.

3. **Conta no GitHub e na Vercel.** As duas grátis. O passo a passo está em
   [credentials.md](credentials.md).

**Para as imagens e animações:**

4. **Acesso ao ChatGPT** (ou outro gerador de imagem), na Fase 9.
5. **Acesso ao Google Flow** (ou Runway, Kling, Luma), também na Fase 9. Este é opcional — sem
   ele a página usa as imagens paradas e continua boa.

---

## As 13 fases

Os números abaixo são os mesmos do [SKILL.md](SKILL.md) — uma fase, um número, nos dois arquivos.

| # | Fase | O que sai | Você faz algo? |
|---|---|---|---|
| 0 | Briefing e credenciais | `briefing.json` começa | **Sim** — preenche o briefing, aprova os logins |
| 1 | Leitura dos assets | inventário de cada arquivo | Não |
| 2 | DNA da marca | `design-system.json` | Não |
| 3 | Auditoria de lacunas | `lacunas.md` | **Sim** — responde uma rodada única de perguntas |
| 4 | Pesquisa de nicho | `pesquisa.md` | Não |
| 5 | Direção criativa | quanto de "uau" a página ganha e onde | Não |
| 6 | Estrutura e copy | `landing-blueprint.md` | **Sim** — revisa o texto. É o momento barato de mudar |
| 7 | Ratificação criativa | o plano de animação fechado | Não |
| 8 | Prompts de imagem e animação | `image-prompts.md` + `motion-prompts.md` | Não |
| 9 | Você gera as imagens e os clipes | os arquivos em `design/renders/` | **Sim** — ChatGPT e Google Flow |
| 10 | Design, implementação e motion | o site rodando na sua máquina | Não |
| 11 | Auditoria | aprovação ou reprovação | Não — mas pode voltar para a 10 |
| 12 | Publicar | repositório + site no ar | **Sim** — confirma que quer publicar |

Nove das treze fases você não faz nada. As quatro onde você aparece — 0, 3, 6 e 9, mais a
confirmação da 12 — estão detalhadas abaixo.

### Fase 0 — Briefing e credenciais

Você recebe um formulário em português de leigo, campo por campo, com exemplo ao lado de cada um.
Preenche o que souber e **deixa em branco o que não souber** — campo vazio nesta fase não é
problema, é exatamente o que a Fase 3 existe para caçar.

Ao mesmo tempo, é checado se o GitHub, a Vercel e o Node estão prontos. Descobrir na hora de
publicar que faltava um acesso custa o projeto inteiro em retrabalho.

### Fase 1 — Leitura dos assets

Cada imagem da pasta é aberta e olhada de verdade, uma por uma. Nada é classificado pelo nome do
arquivo — `fachada-clinica-vertical.png` num projeto real tinha 345 pixels e era um recorte, não
uma fachada.

Sai um inventário que diz, de cada arquivo: o que aparece nele, se dá para usar no site e em que
tamanho, e se serve de referência para anexar aos prompts do ChatGPT. Os dois últimos são
independentes: um logo pequeno demais para o cabeçalho ainda é o melhor anexo possível.

*Você não faz nada.*

### Fase 2 — DNA da marca

O computador mede as cores pixel a pixel. Não é "achar que o roxo é mais ou menos esse" — é medir
e descobrir que o roxo é exatamente `#603084` e ocupa 87% do logo. Também identifica a classe de
tipografia, as formas que se repetem e o tom de voz.

E separa **fato** (está num arquivo) de **suposição** (foi deduzido) de **não verificado** (trava
a publicação). Essa separação é o que impede o site de prometer coisa que a empresa não faz.

*Você não faz nada.* Sai o `design-system.json`.

### Fase 3 — Auditoria de lacunas

Só agora, depois de ler tudo, é que aparecem as perguntas. **Uma rodada só.**

Você recebe três blocos: o que **bloqueia** (sem isso não publica), o que é **importante** (a
página piora sem), e o que é **opcional**. Cada pergunta vem com a resposta padrão que será adotada
se você não souber — "não sei" é resposta válida e sem custo.

Aqui também aparecem as **contradições**. Exemplo real: o post do Instagram anuncia atendimento
24 horas para pets exóticos, e o FAQ do site diz "foco em cães e gatos, consulte-nos". As duas
frases são do próprio cliente e não podem ser publicadas juntas. Contradição é a única coisa que
eu não decido sozinho — porque qualquer escolha publica uma afirmação que outra fonte dele nega.

*Você faz:* leva o `lacunas.md` para a conversa com o cliente. Ele não tem jargão nenhum, é para
ser lido em voz alta.

### Fase 4 — Pesquisa de nicho

O site atual do cliente, o Instagram, as avaliações do Google e três a cinco concorrentes da mesma
cidade são lidos. Sai uma lista do que **todo mundo do setor já diz** — e essas frases ficam
proibidas na nossa copy. "Atendimento humanizado" e "profissionais qualificados" estão em todos os
concorrentes; repetir isso é entrar num site indistinguível dos outros.

Também sai a brecha (o que ninguém naquela cidade está reivindicando), as palavras literais que os
clientes usam nas avaliações, e as objeções em ordem de custo.

*Você não faz nada.*

### Fase 5 — Direção criativa

Antes de escrever uma linha, se decide **quanto** de experiência a página ganha: qual é o
momento que o visitante vai lembrar, quantos momentos menores existem, e onde a página fica
propositalmente quieta — porque uma página animada do começo ao fim cansa e nada se destaca.

*Você não faz nada.*

### Fase 6 — Estrutura e copy

Aqui se define a ordem das seções e se escreve **todo o texto do site, do zero**.

Não é o texto do site antigo com outras palavras. Se o site antigo convertesse, o cliente não
estaria pedindo um novo.

Para dar certo, três versões diferentes da página são escritas em paralelo, com abordagens opostas.
Depois, três avaliadores julgam as três — um olha conversão, outro olha se o português está bom,
outro olha se a experiência é memorável. A melhor vence e rouba as boas ideias das outras duas.

**Revise este arquivo com atenção.** É o único momento barato de mudar de ideia, e a razão está na
fase seguinte.

*Você faz:* lê o `landing-blueprint.md` e aprova ou pede ajuste.

### Fase 7 — Ratificação criativa

O plano da Fase 5 é conferido contra a estrutura que a Fase 6 produziu: qual seção fica com o
momento marcante, quanto de rolagem cada uma recebe, e qual entra em silêncio.

*Você não faz nada.*

### Fase 8 — Os prompts são escritos

Saem o `design/image-prompts.md` e o `design/motion-prompts.md`, com um bloco por seção. Cada
bloco diz: quais arquivos anexar, a frase de anexo, o prompt completo, onde salvar, e o que faz
a imagem voltar para trás.

*Você não faz nada — ainda. Na fase seguinte você usa esses dois arquivos.*

### Fase 9 — Você gera as imagens e os clipes

**Esta é a sua fase mais longa.** O passo a passo completo, com as listas de rejeição e o
checklist final, está em [handoff-imagens.md](handoff-imagens.md) — abra esse arquivo e trabalhe
por ele. O que segue aqui é só o resumo.

No topo do `design/image-prompts.md` tem um parágrafo em inglês chamado **Style Anchor**. Ele é o
segredo da consistência: você cola esse
mesmo parágrafo junto de todos os prompts. É o que faz oito imagens geradas separadamente parecerem
tiradas pelo mesmo fotógrafo, no mesmo dia. **Não edite o Style Anchor no meio do caminho** — se
editar na quinta imagem, as quatro primeiras deixam de combinar e você refaz tudo.

**O que fazer, por seção:**
1. Abra uma conversa **nova** no ChatGPT (uma por seção — em thread longa o modelo começa a copiar
   a si mesmo e as seções ficam iguais).
2. Anexe os arquivos que o bloco mandar anexar. Normalmente o logo, um ou dois posts, e a foto real
   do local quando a seção falar dele. Nunca mais de três.
3. Num envio só, **nesta ordem**: os anexos, o Style Anchor colado inteiro, o prompt da seção e,
   por último, a frase de anexo. A frase vem **depois** do prompt — é ela que impede o modelo de
   copiar o enquadramento da referência.
4. Salve a imagem no caminho que o bloco indica, na maior resolução disponível.

**As cinco regras das imagens, e por que elas existem:**

| Regra | Por quê |
|---|---|
| **Nenhuma letra, número, palavra, logo ou placa escrita.** Nunca | O modelo escreve português acentuado como garatuja — "Clínica" volta "Clínlca". E texto dentro da imagem não é selecionável, nem traduzível, nem indexável pelo Google. Todo texto do site é texto de verdade, por cima |
| A imagem é sobre **algo concreto** — pessoa, lugar, objeto, animal, gesto | Gradiente e textura genérica é o que toda landing page comprada tem. O primeiro elemento que parece banco de imagem devolve o visitante ao ponto de partida |
| Pessoa aparece **sem rosto reconhecível** — mãos, antebraços, costas, silhueta desfocada | Não temos autorização de uso de imagem de ninguém. E rosto é onde o modelo mais erra, num tamanho em que o erro é visível. Foto de mãos trabalhando continua sendo sobre uma pessoa |
| **Anexe as referências** que o bloco pedir | Escrever "roxo #603084" aproxima. Anexar o logo entrega o hex exato. Sem anexo, a cor destoa do logo de verdade no cabeçalho |
| **Nunca gere a fachada real, a equipe real ou equipamento** que o cliente pode não ter | Seria inventar como é um lugar que existe. Para isso usamos a foto real dele |

**Quando rejeitar e gerar de novo:** apareceu qualquer letra ou número; o assunto virou uma textura
bonita sem nada dentro; mão, pata ou orelha saiu deformada; a imagem copiou o enquadramento do post
que você anexou; o roxo virou uma lavagem sobre a cena inteira em vez de estar em objetos; ou o
espaço vazio não está onde o prompt pediu — esse espaço é onde o texto do site vai cair por cima.

**Depois das imagens, os clipes.** O `motion-prompts.md` tem um bloco por seção, dizendo qual
imagem animar e qual movimento pedir.

**O que fazer:** abra o Google Flow, suba a imagem da seção, cole o prompt de movimento, gere de 4
a 6 segundos, salve o MP4 no caminho indicado — mesmo nome do PNG, extensão `.mp4`. Só anime
seções cuja imagem você já aprovou.

**As regras estranhas têm motivo.** Os prompts pedem um movimento só, lento, sem corte e sem
tremor. Não é preciosismo: o site vai controlar esse vídeo pelo **scroll** do visitante. Se tiver
um corte no meio, rolar a página parece um defeito. Se o movimento for rápido, pisca. Se tiver
dois movimentos, dá um tranco na emenda.

**Quando rejeitar:** qualquer corte, qualquer tremor de câmera, qualquer mudança de velocidade. E
olhe o **último quadro** do vídeo — ele é o que fica mais tempo na tela e é o que aparece para quem
desligou animação no celular.

Sem acesso ao Flow, pule os clipes. A página usa as imagens paradas com movimento de CSS.

**Quando terminar**, marque o checklist final do [handoff-imagens.md](handoff-imagens.md) item
por item e avise. É o portão desta fase.

### Fase 10 — Design, implementação e motion

Três coisas, nesta ordem, e nenhuma é sua:

**Design.** Escala tipográfica, espaçamento, grade, hierarquia, contraste medido, e a escolha da
família de fonte — testada com `ção`, `não` e `ú`, porque fonte bonita que não tem acento em
português é inútil aqui.

**Implementação.** As seções, o carregamento das imagens em formatos leves, e todo o SEO — o que
faz o Google entender o site e o que faz aparecer a capinha bonita quando alguém manda o link no
WhatsApp.

**Motion.** O que acontece quando o visitante rola a página, os vídeos controlados por scroll, e
as reações de botão, card e menu. Animação aqui não é enfeite — cada uma existe para guiar a
atenção, explicar algo ou reforçar a hierarquia. Se a resposta for "porque fica legal", ela é
removida.

*Você não faz nada.* No fim, o site roda na sua máquina para você olhar.

### Fase 11 — Auditoria

Um especialista revisa: funciona no celular de 375 pixels? Dá para navegar só com o teclado? Uma
pessoa cega usando leitor de tela entende? O contraste das cores passa? Carrega rápido no 4G ruim?
Quem desligou animação no sistema consegue ler tudo?

**Esta fase pode reprovar e mandar voltar para a Fase 10.** É de propósito. Acessibilidade que
fica para "depois" nunca é feita.

*Você não faz nada.*

### Fase 12 — Publicar

Antes de qualquer coisa, uma varredura: nenhum contrato, CPF, tabela de preço interna ou print de
WhatsApp pode ter entrado no commit. Repositório é para sempre.

Depois, automático: o repositório `lp-<nome-do-cliente>` é criado no GitHub **privado**, o projeto
é ligado na Vercel, o site sobe, e a URL volta. Se o cliente tiver domínio próprio, o DNS é
apontado e você recebe exatamente o que colar no painel do registrador.

*Você faz:* confirma que quer publicar, e aprova o login no navegador se ainda não tiver aprovado.

---

## Perguntas que sempre aparecem

**Quanto tempo leva?**
Do lado do computador, as fases 6 e 10 são as demoradas. Do seu lado, a Fase 9 é o trecho mais
lento do projeto inteiro, porque envolve rejeitar e refazer. Um projeto de 8 seções costuma pedir
de 15 a 30 gerações de imagem até fechar. Reserve mais tempo para isso do que para tudo o resto
somado.

**Quanto custa?**
Hospedagem na Vercel: grátis no plano Hobby. GitHub: grátis. O custo real é o plano do ChatGPT e o
do Google Flow. Domínio próprio, se o cliente quiser, é a única despesa fixa — e ela é dele.

**Posso pular a geração de imagem e usar só foto real?**
Pode, e às vezes deve. Foto real é sempre melhor quando a seção fala do negócio dele: a fachada, a
equipe, a estrutura, o trabalho feito. Aliás, é **proibido** gerar essas — seria inventar como é um
lugar real e pessoas reais. Imagem gerada serve para seção ilustrativa e de atmosfera. Uma página
100% de imagem gerada é percebida como catálogo; uma página com foto real do lugar tem prova.

**E se o cliente não gostar do texto?**
Revise na **Fase 6**, antes de qualquer imagem existir. Depois fica caro — e não é caro por
teimosia: a composição de cada imagem é desenhada **em volta daquele texto**. A linha
`COMPOSITION` de cada prompt reserva o canto vazio exato onde a headline vai cair. Trocar uma
headline de três palavras por uma de nove muda onde o espaço vazio precisa estar, e a imagem
inteira volta para a fila de geração. Mudar o texto na Fase 6 custa cinco minutos; mudar na Fase
10 custa uma rodada nova de imagens e de clipes.

**Preciso saber programar?**
Não. Copiar e colar, julgar imagem, e confirmar dado com o cliente.

**E se faltar uma credencial?**
Você descobre na Fase 0, antes de qualquer trabalho. Cada uma tem a fase exata em que trava e o que
fazer sem ela, em [credentials.md](credentials.md).

**Por que não posso pedir para a imagem trazer o nome da empresa escrito?**
Porque o modelo desenha letra, não escreve texto. Palavra com acento sai deformada, e mesmo se
saísse perfeita, ninguém consegue selecionar, o tradutor do navegador não vê, o Google não indexa e
o leitor de tela não lê. O nome da empresa entra como texto de verdade por cima da imagem, e o logo
entra como arquivo vetorial — sempre nítido, em qualquer tela.

**O site fica no ar de graça para sempre?**
No plano Hobby da Vercel, sim, para uma landing page com este volume. O que custa é o domínio
próprio, e ele é do cliente.

**Dá para mudar alguma coisa depois de publicado?**
Dá. Cada alteração vira um `git push` e a Vercel republica sozinha em segundos. Mudar texto e cor é
barato a qualquer momento. Mudar a estrutura de uma seção com imagem já gerada é o caro — pela
mesma razão da pergunta sobre o texto.
