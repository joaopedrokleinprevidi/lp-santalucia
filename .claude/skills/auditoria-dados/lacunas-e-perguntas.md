# Perguntas para o cliente

Apoio de `auditoria-dados` (Fase 3): a redação literal de cada pergunta, campo por campo, e o modelo
do bloco de conflito. O procedimento, o agrupamento em `AskUserQuestion`, o modelo do
`design/lacunas.md` e a regra da sessão não interativa estão no [SKILL.md](SKILL.md) e não se
repetem aqui.

## Como escrever uma pergunta para leigo

| Não escreva | Escreva |
|---|---|
| "Confirmar o endpoint de conversão" | "Qual o número do WhatsApp que a empresa atende, com DDD?" |
| "Validar o `openingHoursSpecification`" | "Abre no feriado? Em qual feriado fecha?" |
| "Definir a estratégia de prova social" | "Quantas avaliações a empresa tem no Google e qual a nota?" |
| "O CNPJ está pendente" | "Precisamos do CNPJ para colocar no rodapé — é o que mostra que a empresa é registrada" |
| "Assets em resolução insuficiente" | "O logo que você mandou está pequeno. Existe o arquivo original, do designer? Termina em `.svg`, `.ai` ou `.pdf`" |

Regras: uma pergunta por frase, sem sigla não explicada, e sempre com o motivo em uma linha. O dev
vai repassar a pergunta ao cliente — se ele não entender, ele não pergunta.

## Banco de perguntas

Copie a coluna "pergunta" literalmente. A coluna "se não responder" é o que vai para
`briefing.json.assumptions` e precisa aparecer na própria pergunta.

### Bloco BLOQUEIA

| Campo | Pergunta | Se não responder |
|---|---|---|
| Nome oficial | "O nome da empresa se escreve exatamente como no logo? Escreva do jeito certo, com acento." | Nada. Uso o que li no logo e marco para conferência |
| WhatsApp | "Qual o número do WhatsApp que a empresa atende, com DDD? É o mesmo do telefone fixo ou é um celular separado?" | **Adia a publicação.** Sem ele não existe o botão principal |
| Telefone | "O telefone `(XX) XXXX-XXXX` que achei na placa/no Google está certo e atende hoje?" | **Adia a publicação** |
| Endereço + CEP | "Confirma o endereço completo com CEP? Se tiver complemento (sala, loja, andar), me diga também." | **Adia a publicação** |
| Horário | "Qual o horário de cada dia da semana? Abre sábado, domingo e feriado? Fecha para o almoço?" | **Adia a publicação** |
| Serviços | "Esta é a lista de serviços que eu montei a partir do material. Está faltando algum? Tem algum que a empresa não faz mais?" | Uso a lista dos assets e marco cada item como não confirmado |
| Cidade/região | "Além de <cidade>, a empresa atende quais outras cidades ou bairros?" | Uso só a cidade do endereço |
| Logo original | "O logo que recebi está em resolução baixa e sem fundo transparente. Existe o arquivo original do designer — `.svg`, `.ai`, `.eps` ou `.pdf`?" | Uso o que tenho e o logo aparece menor do que deveria |
| Fotos do local | "Tem foto da fachada e de dentro? Pode ser do celular. Se não tiver, o cliente consegue tirar cinco fotos esta semana?" | A página fica sem prova visual de que o lugar existe |
| Conflito detectado | Ver a seção "Conflitos", abaixo | **Adia a publicação** |

### Bloco IMPORTANTE

| Campo | Pergunta | Se não responder |
|---|---|---|
| Domínio | "A empresa já tem um endereço de site comprado? Se não tem, pretende comprar? Enquanto isso o site vai para um endereço da Vercel, que funciona mas é mais difícil de divulgar." | Assumo `.vercel.app` e registro que a troca depois custa reindexação |
| E-mail | "Tem e-mail de contato para colocar no rodapé?" | Rodapé sem e-mail; só WhatsApp e telefone |
| Redes sociais | "Quais são os @ das redes sociais? Instagram, Facebook, YouTube, TikTok — o que existir." | Rodapé sem links sociais; a página perde a ligação com o perfil no Google |
| Google Business | "Qual o link do perfil no Google, a nota e quantas avaliações tem?" | A página fica sem o número de prova social mais forte que existe |
| Diferenciais | "Se um cliente perguntar por que escolher esta empresa e não a do lado, quais são as três respostas?" | Deduzo dos assets e marco como não confirmado |
| **O que não faz** | "Tem algum serviço que as pessoas pedem e a empresa não faz? Preciso saber para a página não prometer errado." | Sigo sem, mas cada serviço da página fica limitado ao que aparece em asset |
| Depoimentos | "Podemos publicar avaliações com o nome de quem escreveu? O cliente pediu autorização a essas pessoas?" | Uso o texto da avaliação pública, sem foto e sem sobrenome |
| Preço | "Quer mostrar preço no site? Se não, o botão diz 'falar no WhatsApp'." | Assumo sem preço, CTA de conversa |
| Fundação | "Em que ano a empresa abriu?" | Página sem "desde ####" |
| CNPJ | "Qual o CNPJ? Vai no rodapé, é o que mostra que a empresa é registrada." | Rodapé sem CNPJ |
| Registro do conselho | "A profissão tem conselho (CRMV, CRO, CREA, OAB…)? Qual o número de registro e o nome do responsável técnico? Exibir é obrigatório quando o conselho exige." | **Adia a publicação se o nicho exigir.** Publicar sem é infração do cliente |

### Bloco OPCIONAL

| Campo | Pergunta | Se não responder |
|---|---|---|
| Vídeos | "Tem algum vídeo, mesmo curto, mesmo do celular? Vídeo real do lugar vale mais que oito imagens geradas." | Página só com imagem |
| Impresso | "Tem cartão de visita, folder ou cardápio? Serve para pegar cores e ícones que não aparecem nos posts." | Paleta vem só dos posts |
| Site atual | "Existe site atual? Mande o endereço — serve para conferir dados, não para copiar o texto." | Menos fontes para cruzar |
| Referências | "Tem algum site que o cliente ache bonito? Pode ser de qualquer setor." | Direção criativa decide sozinha |
| Vetos | "Tem alguma cor, imagem ou estilo que o cliente detesta?" | Descubro na primeira rejeição de imagem, que custa uma rodada |

## Conflitos

Um conflito nunca vira suposição. A pergunta apresenta as duas fontes e pede o desempate:

```
Achei duas informações que não batem e preciso que o cliente desempate:

  • O post de Instagram diz: "Especialistas em pets não convencionais — atendimento 24h"
  • O FAQ do site atual diz: "Nosso foco principal são cães e gatos; consulte-nos para
    outras espécies"

A empresa atende coelho, ave, roedor e réptil hoje, 24 horas, ou não?

Por que preciso saber: se atende, é o diferencial mais forte da página. Se não atende e a
página disser que sim, um tutor de coelho vai dirigir até lá de madrugada e não ser atendido.

Não vou escolher por você. Enquanto não houver resposta, a página não menciona pets não
convencionais.
```

A estrutura tem quatro partes e nenhuma é dispensável: as duas fontes citadas literalmente, a
pergunta direta, o custo concreto de errar, e o que acontece enquanto não há resposta. Sem a
terceira parte, o dev trata como detalhe e não repassa.
