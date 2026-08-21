# Fontes — o que buscar, o que a fonte devolve de verdade, e o fallback

Substitua os placeholders: `<NOME>` razão social ou nome fantasia, `<CIDADE>`, `<BAIRRO>`,
`<SERVICO>` o serviço principal em português coloquial ("banho e tosa", "harmonização facial",
"advogado trabalhista"), `<ENDERECO>` a rua do cliente.

Toda leitura entra em `design/pesquisa.md` com a **data**. Sem data, a tabela expira em silêncio.

## Regra de gasto

| Passada | Buscas + fetches | Concorrentes | Onde cortar primeiro |
|---|---|---|---|
| Completa | 25 a 40 | 3 a 5 | páginas internas de concorrente |
| Curta | ≤ 8 | 2 | frentes 1, 2 e 5 |

Se uma busca devolve o mesmo conjunto de resultados da anterior, pare de variar a query: o
mercado local tem o tamanho que tem, e cidade pequena costuma esgotar em 3 concorrentes reais.

---

## 1. Site atual do cliente

```
WebFetch <url>                          # home
WebFetch <url>/servicos  ou /precos     # o que ele vende e por quanto
WebFetch <url>/contato                  # telefone, endereço, horário, canais
```

Se o site for SPA e o fetch voltar quase vazio, o conteúdo está no JS. Não insista: peça ao dev
um print da home e da página de serviços, ou trabalhe com o cache do buscador:

```
WebSearch site:<dominio> <SERVICO>
WebSearch "<NOME>" <CIDADE> preço
```

O que sai daqui: fatos, canais de conversão, sinais de desatualização, e as frases que também
aparecem nos concorrentes (essas viram banidas).

---

## 2. Instagram e Facebook

`WebFetch` num perfil do Instagram normalmente devolve a parede de login ou um HTML sem os
posts. Trate isso como o caso normal, não como erro.

O que **tenta** funcionar:

```
WebSearch site:instagram.com "<NOME>" <CIDADE>        # acha o @ correto
WebFetch https://www.instagram.com/<handle>/          # às vezes traz bio e contagem
WebSearch "<NOME>" <CIDADE> instagram <SERVICO>       # posts indexados por terceiros
```

O que **nunca** funciona sem login, e por isso é pergunta para o dev (item 3 do bloco único):
alcance, impressões, salvamentos, compartilhamentos, taxa de engajamento, dados de audiência,
mensagens no direct. Tudo isso vive no painel profissional do dono da conta.

Quando o dev mandar os prints, extraia nesta ordem: serviço mais anunciado (contagem em 30
posts) → formato do post campeão → tiques de voz com exemplo literal → perguntas dos
comentários.

### Biblioteca de anúncios

Pública e fetchável, e diz o que o concorrente está pagando para dizer **agora** — que é mais
honesto que a home dele, feita há três anos:

```
WebFetch https://www.facebook.com/ads/library/?active_status=all&ad_type=all&country=BR&q=<NOME-DO-CONCORRENTE>
```

Se o resultado vier vazio, o concorrente não anuncia — o que também é informação: um mercado
local sem ninguém anunciando é um mercado onde o primeiro anúncio vale mais.

---

## 3. Avaliações no Google

```
WebSearch "<NOME>" <CIDADE> avaliações google
WebSearch "<NOME>" <CIDADE> reclamação        # a negativa raramente está no topo
WebFetch <link-do-Maps-que-o-dev-mandou>
```

O que o Maps devolve por fetch: nome, endereço, telefone, horário, nota e total de avaliações
quase sempre; alguns trechos de avaliação às vezes; a lista completa praticamente nunca — ela é
montada por JavaScript sob rolagem.

Fallback, na ordem: (a) peça ao dev as 10 mais recentes coladas como texto — no app é
selecionar e copiar; (b) procure o negócio em agregadores que publicam avaliação em HTML
estático (busque `"<NOME>" <CIDADE> avaliações` sem filtrar por site); (c) se o cliente tem
menos de 10 avaliações, diga isso no arquivo e trate a amostra como anedótica — três
avaliações não sustentam uma seção da página.

Registre sempre: nota, total, quantas de 1–2 estrelas, e o período coberto pela leitura.

**Anonimize na hora de escrever.** A citação entra com estrelas e mês/ano, sem nome e sem link
para o perfil do avaliador.

---

## 4. Concorrentes

Buscas, nesta ordem, porque cada uma revela um conjunto diferente:

```
WebSearch <SERVICO> <BAIRRO> <CIDADE>              # quem disputa o mesmo raio
WebSearch melhores <SERVICO> em <CIDADE>           # quem investe em SEO/listas
WebSearch <SERVICO> <CIDADE> preço                 # quem publica preço — a minoria
WebSearch <SERVICO> perto de <ENDERECO>            # vizinhança imediata
WebSearch site:instagram.com <SERVICO> <CIDADE>    # o concorrente que só existe no Instagram
```

O último importa mais do que parece: em serviço local, boa parte da concorrência real não tem
site nenhum e a promessa dela mora na bio.

Por concorrente escolhido, no máximo três leituras:

```
WebFetch <url-da-home>                 # o H1 e o rótulo do primeiro CTA = a promessa
WebFetch <url-de-um-servico>           # preço, escopo, o que ele detalha
WebFetch https://www.instagram.com/<handle>/   # bio, quando não houver site
```

### Contagem de afirmações

O produto desta frente não é a tabela, é a contagem. Procedimento:

1. Copie **toda** afirmação de valor de cada concorrente numa lista única — H1, subtítulo,
   rótulos de seção, bullets de "por que nos escolher".
2. Normalize para o núcleo semântico: "atendimento humanizado", "atendimento acolhedor" e
   "cuidado humanizado" são a mesma afirmação.
3. Conte quantos concorrentes fazem cada uma.
4. **≥3 de 5 (ou ≥2 de 3, na passada curta) → lista de banidas**, com a contagem ao lado.
5. Afirmação que **nenhum** faz e o cliente pode provar → candidata a brecha.

Fato operacional (horário, endereço, forma de pagamento, aceita convênio) fica fora da
contagem. Banir "aberto de segunda a sábado" quebraria a página e entupiria o script de
falso positivo.

### Baseline visual

Três linhas, olhando as home lado a lado: foto de banco ou foto real? qual cor domina o setor
naquela cidade? qual o padrão de abertura (carrossel, vídeo, foto com faixa)? É a média da qual
`creative-direction-expert` vai se desviar de propósito.

---

## 5. Exigências do nicho

Comece pela tabela de [nichos.md](nichos.md) e confirme a norma vigente — resolução de conselho
muda e a versão antiga circula em blog:

```
WebSearch <CONSELHO> publicidade resolução <ano-corrente>
WebSearch <CONSELHO> código de ética publicidade profissional
WebSearch CONAR <SERVICO> publicidade decisão
```

Registre por exigência: o que exige, a fonte (conselho, lei, ou "expectativa do visitante" —
que não é norma e deve ser marcada como tal), onde entra na página, e a data em que você
confirmou.

Quando a busca não fechar a dúvida, a saída é `unverified` no `design-system.json` e uma linha
em "Não pode prometer". Nunca uma aposta redigida com segurança.

---

## Quando a fonte não abre

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| Fetch devolve HTML sem conteúdo | SPA renderizada por JS | Buscar pelo cache do buscador; pedir print ao dev |
| Fetch devolve página de login | Instagram, Facebook, alguns diretórios | Pedir ao dev; nunca criar conta nem contornar |
| Maps traz nota mas nenhuma avaliação | Lista carregada sob rolagem | Pedir as 10 mais recentes como texto |
| Concorrente sem site | Comum em serviço local | Usar a bio do Instagram como promessa principal |
| Cidade com menos de 3 concorrentes | Mercado pequeno | Ampliar o raio e declarar o raio usado no arquivo |
| Cliente com <10 avaliações | Negócio novo ou sem pedido de avaliação | Declarar amostra anedótica; virar recomendação ao cliente |

Fonte que não abriu vira linha declarada no arquivo ("Instagram não acessível sem login —
amostra veio de 5 prints enviados em <data>"). Silêncio sobre a lacuna faz a próxima skill
tratar a pesquisa como completa.
