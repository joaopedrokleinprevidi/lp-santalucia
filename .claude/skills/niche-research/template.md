# Modelo de `design/pesquisa.md`

Copie o esqueleto abaixo. Os seis títulos `##` são **fixos**: `estrutura-secoes` e
`copy-conversao` fazem `Grep` por eles e `scripts/check-banned-copy.mjs` corta o arquivo por
`## O que todo mundo diz`. Renomear um título quebra o portão sem erro visível.

As linhas preenchidas são **exemplo ilustrativo** — nomes de concorrente entre `<>` são
placeholders, não empresas reais. Apague todas antes de entregar.

---

````markdown
# Pesquisa — <Cliente>, <Cidade>

<!--
Passada: completa | curta
Período de leitura: <data inicial> a <data final>
Fontes lidas: <n> buscas, <n> fetches
Lacunas declaradas: <o que não abriu e como foi contornado>
-->

## Concorrentes

| Concorrente | Promessa principal (H1 literal) | Preço declarado | Faz melhor | Não faz | Lido em |
|---|---|---|---|---|---|
| <Nome A> | "Sua autoestima em boas mãos" | não publica | agenda online no site | não fala de preço, sem WhatsApp visível | 2026-08-20 |
| <Nome B> | "Odontologia estética de alto padrão" | "a partir de R$ 3.900" | mostra caso com foto | não diz quem atende, sem registro no rodapé | 2026-08-20 |
| <Nome C> (só Instagram) | bio: "Transformando sorrisos há 10 anos" | por direct | responde comentário em minutos | não tem site, nenhum endereço público | 2026-08-20 |

**Baseline visual do setor:** todas abrem com foto de banco de imagem de sorriso em close;
duas usam azul-clínico e uma usa dourado sobre preto; nenhuma mostra o espaço real nem o
processo. Aberturas em carrossel em 2 de 3.

<!-- 3 a 5 linhas. Data obrigatória em cada uma. A promessa principal é o H1 + o rótulo do
primeiro CTA; sem site, a bio. Ela é evidência: não pode aparecer em nenhum outro lugar
do projeto. -->

## O que todo mundo diz

- "autoestima" como promessa central — 3/3 concorrentes
- "alto padrão" / "excelência" — 2/3
- "atendimento humanizado" — 2/3
- "transformando sorrisos" — 2/3
- "profissionais altamente qualificados" — 3/3

<!-- Lista de PROIBIÇÃO. Nada daqui entra na copy, nem parafraseado.
Só promessa: horário, endereço e forma de pagamento são fato e ficam fora — banir fato
quebra a página e enche o script de falso positivo.
Corte de entrada: aparece em ≥3 de 5 (ou ≥2 de 3, na passada curta).
Se esta lista voltar vazia, você leu os concorrentes procurando inspiração. Refaça. -->

## A brecha

**Ninguém no mercado local diz o que acontece na primeira consulta.** Os três vendem o
resultado; nenhum descreve o processo, o tempo ou o que a pessoa decide em cada etapa.

- Teste 1 — ninguém diz: confirmado nas 3 homes e nas 3 bios (lidas em 2026-08-20).
- Teste 2 — o cliente prova: `facts.process` traz as cinco etapas nomeadas, e `facts.claims`
  traz "avaliação sem custo desde 2016".
- Teste 3 — derruba a objeção: **#2, "vão me empurrar um pacote"** — fatal.

**Tipo:** de método. **Forma sugerida:** a etapa nomeada, não o adjetivo.

<!-- Uma brecha, não três. Se você tem três candidatas, escolha a que derruba a objeção mais
cara e guarde as outras como corpo de seção. Adjetivo nunca é brecha: qualquer concorrente
publica "atendimento humanizado" na terça sem mudar nada no negócio. -->

## Palavras do cliente

- "ela explicou tudo antes, eu não me senti empurrada" — 5★, jun/2026 · uso: headline
- "achei que fosse doer e não doeu nada" — 5★, mai/2026 · uso: FAQ
- "o preço não é o mais barato mas eu sabia de tudo antes" — 4★, abr/2026 · uso: corpo (preço)
- "eles avisam quando atrasa" — 5★, abr/2026 · uso: corpo (experiência)
- "demorei pra ser atendida, quase uma hora depois do horário" — 2★, mar/2026 · uso: objeção #4
- "não consegui remarcar pelo WhatsApp, só ligando" — 2★, fev/2026 · uso: objeção #5
- "voltei porque me trataram bem quando deu problema" — 5★, fev/2026 · uso: corpo (garantia)
- "explicaram o valor certinho, sem surpresa no fim" — 5★, jan/2026 · uso: FAQ (preço)

<!-- Mínimo 8, sendo ≥2 negativas. Literal, com o erro de digitação e o regionalismo: o valor
da citação é ser a palavra que o visitante já usa.
Sem nome, sem foto, sem link do perfil — depoimento identificado exige autorização.
Nota do cliente: <n,n>★ · <total> avaliações · <n> de 1–2 estrelas · período lido: <datas> -->

## Objeções

1. **"Quanto custa?"** — fatal · fonte: nenhum concorrente publica preço; 2 avaliações citam
   preço · derrubar em: FAQ posição 1 e no corpo antes de 40% do scroll
2. **"Vão me empurrar um pacote"** — fatal · fonte: avaliação 5★ jun/2026; ausência de processo
   descrito nos 3 concorrentes · derrubar em: capítulo de método
3. **"Vou ficar artificial"** — fatal · fonte: pergunta recorrente nos comentários (4×) ·
   derrubar em: capítulo de prova, com resultado real
4. **"Vou esperar muito?"** — adiadora · fonte: avaliação 2★ mar/2026 · derrubar em: FAQ 3
5. **"Como remarco?"** — adiadora · fonte: avaliação 2★ fev/2026 · derrubar em: FAQ 4
6. **"Tem estacionamento?"** — residual · fonte: expectativa do nicho · derrubar em: contato
7. **"Aceita cartão / parcela?"** — residual · fonte: expectativa do nicho · derrubar em: FAQ 5

<!-- 5 a 7, nesta ordem: fatal → adiadora → residual. Cada uma com fonte real.
Fatal = fecha a aba e não volta. Adiadora = adia, e adiado é perdido. Residual = só aparece
em quem já decidiu.
Esta ordem É a ordem de derrubada que o storytelling usa para sequenciar seção e FAQ. -->

## Exigências do nicho

| Exige | Fonte | Onde entra na página | Confirmado em |
|---|---|---|---|
| Nome e CRO do responsável técnico | CFO — regulação | rodapé + capítulo de credencial | 2026-08-20 |
| Escopo declarado (estético, não terapêutico) | CFO — regulação | corpo do capítulo de método | 2026-08-20 |
| Faixa de preço visível | expectativa | FAQ, pergunta 1 | — |
| Quem vai atender, com rosto | expectativa | capítulo de credencial | — |
| Estacionamento / acesso | expectativa | seção de contato | — |

### Não pode prometer

- Resultado garantido ou desfecho estético específico — o conselho veda garantia de resultado.
- Antes/depois sem confirmar a regra vigente do conselho — checar antes de qualquer foto.
- "A melhor clínica da região" — superlativo sem substanciação (CONAR).
- Prazo de recuperação fixo — varia por pessoa e vira obrigação ao ser anunciado (CDC art. 30).

<!-- Regulação e expectativa são coisas diferentes e a coluna Fonte diz qual é.
Toda dúvida que a busca não fechou vira `unverified` no design-system.json, e `unverified`
pendente bloqueia publicação. -->

## Devolvido ao design-system.json

<!-- Seção opcional, mas escreva-a: é o rastro de que a fonte da verdade foi atualizada. -->

- `facts.claims += "1.240 avaliações desde 2016"` — fonte: perfil do Google, lido em 2026-08-20
- `unverified += "atende convênio X"` — o site antigo promete, nenhum asset confirma
- `facts.process` conferido contra os posts — bate
````

---

## Erros que este modelo existe para evitar

| Erro | O que acontece depois |
|---|---|
| Renomear um título `##` | O `Grep` da skill seguinte não acha a seção e a copy é escrita sem a lista de banidas |
| Tabela de concorrente sem a coluna de data | Seis meses depois ninguém sabe se o preço ainda vale, e a decisão foi tomada em cima dele |
| Citação parafraseada | A pesquisa devolve o nosso vocabulário para nós mesmos e não trouxe nada |
| Objeção sem classe | O storytelling não sabe o que vai antes de 40% do scroll e ordena por gosto |
| Brecha sem o fato que a prova | Vira headline, e uma promessa não sustentada é cancelamento na primeira semana |
| Lista de banidas vazia | Sinal de que a leitura foi por inspiração; a copy vai repetir o setor sem perceber |
| Nome do avaliador na citação | Depoimento identificado sem autorização — problema jurídico, não estético |
