# Exigências por nicho — o que a página tem que ter e o que não pode prometer

Duas coisas diferentes moram aqui:

- **Regulação** — conselho profissional, lei ou norma sanitária. Descumprir gera sanção ao
  cliente e a landing é a peça publicitária que registra a infração.
- **Expectativa** — ninguém obriga, mas o visitante do nicho procura e vai embora se não achar.
  Marque como `expectativa` no arquivo. Confundir as duas faz a página exibir capricho como
  obrigação e omitir obrigação como capricho.

**Norma muda.** Cada linha traz a busca de confirmação. Confirme antes de escrever e registre a
data. Onde a busca não fechar, a saída é `unverified` no `design-system.json`, nunca uma
redação confiante.

## Vale para todo nicho

| Regra | Consequência na página | Confirmar com |
|---|---|---|
| Publicidade obriga quem anuncia (CDC, arts. 30 e 35) | Preço, prazo e brinde anunciados viram obrigação. Só publique o que o cliente cumpre em todo atendimento | `WebSearch CDC artigo 30 oferta publicidade vincula` |
| Superlativo exige prova | "O melhor da cidade", "o mais moderno" sem substanciação é peça derrubável pelo CONAR. Troque por número verificável ou corte | `WebSearch CONAR código autorregulamentação princípio veracidade` |
| Antes/depois é território minado | Vários conselhos restringem ou proíbem. Antes de usar, confirme o conselho **daquele** profissional | busca da linha do nicho, abaixo |
| Dado pessoal coletado exige base legal (LGPD) | Formulário pede finalidade declarada e política de privacidade linkada. Clique em `wa.me` não coleta nada — é mais um motivo para o WhatsApp ser o canal | `WebSearch LGPD formulário site consentimento finalidade` |
| CNPJ e endereço no rodapé | `expectativa`, mas custa uma linha e é o que separa página de negócio de página de golpe | — |

## Por nicho

| Nicho | Conselho / norma | A página **tem que** | **Não pode** prometer | Confirmar com |
|---|---|---|---|---|
| Medicina, clínica médica | CFM | Nome e CRM do responsável, especialidade registrada | Garantia de resultado, antes/depois, sensacionalismo, concorrência de preço | `WebSearch CFM resolução publicidade médica vigente` |
| Odontologia | CFO | Nome e CRO do responsável técnico, CRO da clínica | Resultado garantido; imagem de antes/depois sob restrição | `WebSearch CFO publicidade odontológica resolução vigente` |
| Estética não médica, beleza | Vigilância sanitária municipal; sem conselho próprio para esteticista | Escopo claro do que é procedimento estético, não médico | Efeito terapêutico, "emagrecimento", "tratamento" de doença, resultado permanente | `WebSearch procedimento estético não invasivo regulamentação ANVISA` |
| Veterinária, pet com clínica | CRMV | Registro do estabelecimento no CRMV e o responsável técnico com CRMV | Cura, desfecho clínico, diagnóstico à distância | `WebSearch CRMV responsável técnico estabelecimento veterinário registro` |
| Pet shop sem ato veterinário (banho, tosa, venda) | Vigilância sanitária; CRMV só se houver ato veterinário | Escopo: dizer que não é clínica, se não for | Vacinação, medicação ou procedimento sem veterinário responsável | `WebSearch banho e tosa exige responsável técnico CRMV` |
| Advocacia | OAB | Nome, número de inscrição e seccional | Resultado, valores de honorário como oferta, captação de clientela, "advogado especialista em ganhar" | `WebSearch OAB provimento publicidade advocacia vigente` |
| Nutrição | CFN | CRN do profissional | Antes/depois sob restrição, promessa de perda de peso em prazo | `WebSearch CFN código de ética publicidade nutricionista` |
| Psicologia | CFP | CRP do profissional | Garantia de cura, técnica sem reconhecimento, exposição de caso identificável | `WebSearch CFP resolução publicidade psicólogo` |
| Educação física, academia, personal | CREF | CREF do responsável | Resultado corporal em prazo, "seca X kg em Y dias" | `WebSearch CREF publicidade academia registro responsável técnico` |
| Alimentação, restaurante, delivery | Vigilância sanitária, ANVISA | Alergênicos declarados no cardápio; alvará quando o visitante espera vê-lo | "Natural", "saudável", "detox" como alegação de saúde sem respaldo | `WebSearch rotulagem alergênicos ANVISA obrigatoriedade cardápio` |
| Imobiliária, corretor | CRECI / COFECI | Número CRECI no material publicitário — é obrigatório, não decorativo | Rentabilidade garantida, valorização prometida | `WebSearch CRECI obrigatoriedade número anúncio imobiliário` |
| Crédito, consórcio, financeiro | Banco Central, CDC art. 52 | Custo Efetivo Total quando anunciar condição de crédito; quem é a instituição por trás | Aprovação garantida, "sem consulta ao SPC" como isca, taxa sem CET | `WebSearch anúncio de crédito CET obrigatório Banco Central` |
| Plano de saúde, odontológico | ANS | Número de registro do produto na ANS | Cobertura além do rol, carência inexistente | `WebSearch ANS número de registro do plano divulgação obrigatória` |
| Farmácia, manipulação | ANVISA, CRF | Farmacêutico responsável | Indicação terapêutica de manipulado, cura, promoção de medicamento sujeito a receita | `WebSearch ANVISA publicidade medicamento farmácia manipulação` |
| Educação, curso livre | MEC só quando o curso é regulado | Carga horária, o que o certificado vale de fato | Reconhecimento pelo MEC quando não há, empregabilidade garantida | `WebSearch curso livre certificado reconhecimento MEC publicidade` |

## Expectativas que não são lei e derrubam a página igual

Estas não geram sanção. Geram abandono, que sai mais caro por visitante.

| Nicho | O visitante procura e não acha |
|---|---|
| Serviço local em geral | Faixa de preço, horário de hoje, se atende no fim de semana |
| Saúde e estética | Quem é a pessoa que vai atender — rosto, nome, registro |
| Veterinária | Se atende emergência, se atende a espécie dele, se tem plantão de verdade |
| Alimentação | Cardápio com preço, tempo de entrega, raio de entrega |
| Qualquer um com atendimento presencial | Estacionamento, acesso, ponto de referência |
| Ticket alto | O que acontece se der errado — política de retorno, reagendamento, garantia |

Ausência aqui é objeção **adiadora** — e adiada é perdida. Cada linha não respondida na página
vira uma pergunta no WhatsApp que o atendente responde uma vez por visitante, ou não responde.

## Como isto entra no arquivo

```markdown
## Exigências do nicho

| Exige | Fonte | Onde entra na página | Confirmado em |
|---|---|---|---|
| CRO do responsável técnico | CFO — regulação | rodapé + seção de credencial | 2026-08-20 |
| Faixa de preço visível | expectativa | FAQ, pergunta 1 | — |

### Não pode prometer
- Resultado garantido em procedimento estético — CFO/CFM vedam garantia de desfecho.
- "A melhor clínica da região" — superlativo sem substanciação (CONAR).
```

Toda linha de "Não pode prometer" vira restrição de copy que `copy-conversao`
respeita e o portão da Fase 6b confere. Toda dúvida que sobrar vira `unverified` no
`design-system.json` — e `unverified` pendente bloqueia publicação.
