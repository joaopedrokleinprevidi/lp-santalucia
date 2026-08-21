# Arquétipos — detalhamento por seção

Complemento de [SKILL.md](SKILL.md), que carrega a tabela de escolha e as sequências. Aqui está o
briefing de cada seção: o que ela tem que entregar, qual prova a sustenta, o que quebra quando ela
é montada errada, e o que muda em mobile.

Os tetos de caractere e a estratégia de CTA não estão aqui — são de
[copy-conversao](../copy-conversao/SKILL.md).

---

## Urgência — clínica veterinária 24h, plantão, emergência

O visitante está com o animal no colo ou dirigindo. Ele não vai ler a página; vai varrer. Toda
seção tem que ser compreensível em uma passada de olho.

### 01 · Estado agora — beat 7, 12%

Afirma disponibilidade no presente: "Atendimento veterinário 24 horas, agora, no Barreiro". O
eyebrow carrega estado, não rótulo — "Aberto agora", "Plantão 24h". Este é o único arquétipo em
que o eyebrow informa.

Primário e secundário visíveis **sem rolar** em 375px, o que aperta o teto da headline aqui.

Quebra quando: a headline vende reputação ("A clínica de referência há 12 anos"). O visitante em
emergência não está escolhendo a melhor clínica, está procurando uma aberta.

### 02 · Como chegar — beat 6, 8%

Endereço em texto grande, ponto de referência que um motorista reconhece, link de mapa, tempo de
deslocamento do bairro vizinho. Prova de disponibilidade — "3 veterinários de plantão" — pertence
aqui, e não é prova social: é informação operacional.

Quebra quando: o mapa é um embed pesado. Um `<a>` para o Google Maps com o endereço em texto
converte melhor e não custa 900 KB de iframe.

### 03 · Quando é emergência — beat 9, 12%

O pico da página. Lista de sinais concretos e observáveis: "não consegue urinar há 12 horas",
"gengiva pálida", "convulsão". Nada de categorias abstratas ("problemas gastrointestinais") — o
visitante não sabe traduzir o que está vendo para o vocabulário clínico.

Quebra quando: a lista tem mais de 8 itens. Além disso ela vira leitura, e o visitante que estava
com pressa perde a seção inteira.

### 04 · O que acontece ao chegar — beat 5, 16%

A queda de quatro pontos é deliberada. Tensão sem alívio fecha a aba. Protocolo em passos
temporais: triagem em X minutos, exame, estabilização, contato com o tutor. Números onde
existirem.

Esta é a seção mais longa do arquétipo porque é a que mais reduz medo — segue a regra geral de
pacing.

### 05 · Estrutura — beat 4, 12%

Equipamento nomeado, não adjetivado: "raio-x digital", "ultrassom", "internação com oxigênio".
Cada item ganha a consequência ("resultado na mesma consulta"), senão é lista de compras.

### 06 · Quem atende — beat 5, 10%

Nome e CRMV. Rosto real. Um stock photo de "equipe sorrindo" destrói mais confiança do que nenhuma
foto — em emergência o visitante procura sinais de que o lugar é real.

### 07 · Depoimentos de emergência — beat 7, 10%

Depoimento com desfecho, não com elogio. "Chegamos às 3h com o Thor convulsionando e ele voltou
para casa no dia seguinte" funciona. "Excelente atendimento, recomendo" não prova nada.

### 08 · FAQ — beat 3, 10%

A primeira é sempre "vocês estão abertos agora", mesmo já respondida no hero: quem chegou por
busca pode ter caído no meio da página.

### 09 · Contato completo — beat 7, 10%

Endereço, telefone clicável, WhatsApp, horário, mapa. Tudo em texto no DOM. É a informação mais
copiada da página inteira.

### Mobile

Share total ~76% do desktop. As seções 05 e 06 podem fundir-se numa só em telas < 480px: com
pressa, "temos estrutura e temos plantonista" é uma informação, não duas.

---

## Transformação — estética, harmonização, resultado no corpo

O visitante decide em dias ou semanas e volta à página mais de uma vez. O medo não é de perder
tempo, é de errar o próprio rosto.

### 01 · Promessa — beat 5, 16%

Fala de pessoa, não de procedimento. A copy real do projeto: "Sua beleza merece um cuidado pensado
exclusivamente para você." Nenhum pedido de agendamento aqui; um botão de navegação interna
("Conhecer a clínica") é aceitável porque pede leitura, não compromisso.

Quebra quando: lista procedimentos na primeira tela. "Botox, preenchimento, bioestimuladores" no
hero converte a página numa tabela de preços e mata o arco inteiro.

### 02 · A experiência — beat 6, 12%

Ambiente, acolhimento, o que se sente ao entrar. Quatro pilares no máximo, uma frase cada. Esta
seção existe para o visitante se ver no lugar antes de pensar em agulha.

### 03 · O processo em etapas — beat 5, 20%

A seção mais longa da página. Cinco etapas numeradas — agendamento, avaliação, planejamento,
procedimento, acompanhamento. A contagem *é* o alívio, então ela pode aparecer na headline: "Seu
atendimento acontece em cinco etapas".

Quebra quando: as etapas descrevem a técnica em vez do que acontece com o visitante. Ele quer
saber o que vai sentir e quanto tempo leva, não o nome da cânula.

### 04 · A consulta — beat 6, 14%

Responde "vão me ouvir ou me empurrar um pacote?". Tempo dedicado, escuta antes de indicação,
orientação completa antes de decidir.

### 05 · O princípio — beat 8, 16%

O pico. Responde o medo central: "vou ficar artificial?". A resposta é o compromisso com
naturalidade, sustentado por antes/depois real ou por um manifesto curto. Se houver antes/depois,
ele vive aqui e em nenhum outro lugar.

Quebra quando: o antes/depois aparece na seção 02. Sem o processo explicado antes, a foto lê como
propaganda e ativa ceticismo em vez de desejo.

### 06 · A clínica — beat 4, 12%

Fotos reais do espaço, privacidade, conforto. O FAQ mora dentro deste capítulo, não como capítulo
próprio: um capítulo de FAQ entre a prova e o convite esfria o arco exatamente onde ele deveria
fechar.

### 07 · Convite — beat 7, 10%

Sem imagem. Depois de seis capítulos de fotografia, a página termina na cor da marca e nada mais,
então a única coisa que sobra para olhar é o pedido. Contato completo e assinatura com bairro e
cidade.

### Mobile

Share total ~76% do desktop. A seção 04 é a primeira candidata a encolher se o total passar do
orçamento — ela é a que menos perde ao ficar curta.

---

## Credencial — saúde especializada, jurídico, consultoria

Ticket alto, decisão em dias, e o visitante já foi mal atendido antes em algum lugar. A prova é
uma pessoa com nome e registro.

### 01 · Afirmação com número — beat 5, 14%

Número duro na headline, mas só se ele for a promessa. "Desde 2016" bate "12 anos de experiência"
— a data não envelhece o site. Aqui, ao contrário de Transformação, o visitante já está procurando
um profissional específico, então o pedido pode vir cedo.

### 02 · O risco de errar — beat 8, 12%

O pico vem cedo. Consequência concreta de escolher mal, dita sem alarmismo e sem atacar concorrente
nominalmente. Um risco descrito com precisão é credencial; um ataque é ruído.

### 03 · O método — beat 5, 18%

Como o trabalho acontece, em etapas nomeadas. A seção mais longa, pela mesma razão de sempre.

### 04 · Quem é a pessoa — beat 6, 14%

Rosto, nome, registro profissional, formação. Esta seção é a prova; se ela for genérica, o
arquétipo inteiro falha.

### 05 · Casos — beat 7, 12%

Caso com desfecho e, quando o sigilo permitir, com número. Sem número e sem desfecho é depoimento
decorativo.

### 06 · FAQ — beat 3, 12%

Aqui o FAQ **é** capítulo próprio, ao contrário de Transformação: honorários, prazo e sigilo são
objeções pesadas demais para viver dentro de outra seção.

### 07 · Contato — beat 6, 10%

Canal e horário de atendimento declarados. "Respondemos em até 1 dia útil" vale mais que "entre em
contato".

### Rodapé com registro — beat 1, 8%

Registro profissional, endereço, CNPJ. Em serviço regulado isso não é rodapé decorativo, é parte
da prova.

---

## Produto — SaaS, self-serve

O único arquétipo em que prova social vem na seção 02, porque o visitante compara contra quarenta
concorrentes idênticos e a pergunta "alguém sério confia nisso?" já está feita antes de ele
chegar. Em serviço local ela não está.

### 01 · O que é, em uma frase — beat 5, 14%

Uma frase que diz o que faz e para quem, com signup e demo disponíveis acima da dobra.

### 02 · Quem já usa — beat 4, 6%

Logos ou número de contas. A menor seção da página: é uma confirmação, não um argumento.

### 03 · O custo do problema — beat 8, 12%

O número do desperdício atual, não a dor genérica. "Seu time gasta 6 horas por semana reconciliando
planilhas" bate "processos manuais são ineficientes".

### 04 · Como funciona em 3 passos — beat 5, 16%

Responde "vai dar trabalho?". Tempo de setup declarado em minutos.

### 05 · Features por resultado — beat 4, 18%

Agrupadas por resultado, nunca por módulo. Cada uma são três coisas: feature → por que importa →
resultado. Duas delas sozinhas não movem ninguém.

### 06 · Comparação — beat 6, 10%

Tabela honesta, incluindo onde a alternativa é melhor. Uma tabela em que a própria coluna vence em
todas as linhas é lida como propaganda e derruba a credibilidade das linhas verdadeiras.

### 07 · Depoimento com métrica — beat 7, 8%

Um depoimento, com número, de alguém identificável.

### 08 · Preço + FAQ — beat 3, 10%

Tabela de preço visível. "Sob consulta" em produto self-serve elimina o self-serve.

### 09 · CTA final — beat 6, 6%

Curto. A decisão já foi tomada na seção 08 ou não foi.
