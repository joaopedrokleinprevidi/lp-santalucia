# Esquema do `design/briefing.json`

O artefato que o pipeline inteiro lê. A regra que sustenta tudo: **nenhum valor existe sem
procedência.** Um campo sem `source` não é fato, é lembrança.

Três skills escrevem neste arquivo, cada uma nos seus blocos. Nenhuma reescreve o bloco da outra:

| Bloco | Quem escreve | Fase |
|---|---|---|
| `meta`, `credentials`, `business`, `services`, `doesNotOffer`, `differentiators`, `socialProof`, `pricing` | `briefing-cliente` | 0 |
| `gaps`, `conflicts`, `assumptions`, `unverified` | `auditoria-dados` | 3 |
| o inventário de arquivos | `estudo-assets`, em `design/inventario.json` — **não** aqui | 1 |

## Prefixos de procedência

| Prefixo | Significa | Exemplo |
|---|---|---|
| `asset:` | Está visível numa imagem que foi aberta e lida | `asset:fachada-clinica-rua.webp` |
| `doc:` | Está num documento que o cliente entregou | `doc:dados-google-da-empresa.txt` |
| `dev:` | O dev respondeu, com a data da resposta | `dev:2026-08-20` |
| `web:` | Veio de fonte pública (Google Business, site atual) e ainda não foi confirmado | `web:google-business` |
| `assumed` | Ninguém disse; foi adotado por padrão | exige entrada em `assumptions` |

`web:` é o mais perigoso dos cinco: parece fato, e é o que o cliente esqueceu de atualizar em 2019.
Todo campo `web:` de contato, horário ou preço vira pergunta de confirmação.

## Estrutura

```jsonc
{
  "meta": {
    "client": "string",              // nome curto, usado em caminho e nome de projeto
    "intakeDate": "YYYY-MM-DD",
    "assetsDir": "string",
    "interactive": true,             // false = perguntas escritas em lacunas.md, não feitas
    "niche": "string"                // define qual conselho profissional se aplica
  },

  "credentials": {
    "node": "v20.11.0 | missing",
    "git": "ok | missing",
    "gh": "ok | missing",
    "vercel": "ok | missing",
    "imageGenerator": "chatgpt | midjourney | none",
    "videoGenerator": "google-flow | none",
    "checkedAt": "YYYY-MM-DD"
  },

  "business": {
    // todo campo segue { value, source } e opcionalmente { assumed, confirmed, note }
    "name":        { "value": "", "source": "" },
    "shortName":   { "value": "", "source": "" },
    "oneLiner":    { "value": "", "source": "" },
    "city":        { "value": "", "source": "" },
    "state":       { "value": "", "source": "" },
    "areaServed":  { "value": [], "source": "" },
    "phone":       { "value": "", "source": "", "confirmed": false },
    "whatsapp":    { "value": null, "source": null },
    "email":       { "value": null, "source": null },
    "address":     { "value": "", "cep": "", "source": "" },
    "hours":       { "value": "", "holidays": "", "source": "" },
    "founded":     { "value": null, "source": "" },
    "cnpj":        { "value": null, "source": null },
    "license":     { "type": "CRMV|CRO|CREA|OAB|null", "value": null, "holder": null, "source": null },
    "domain":      { "value": null, "status": "owned|to-buy|none", "source": "" }
  },

  "services":     [ { "name": "", "group": "", "source": "", "confirmed": false } ],
  "doesNotOffer": [ { "item": "", "source": "" } ],
  "differentiators": [ { "claim": "", "source": "", "verifiable": true } ],
  "socialProof": {
    "googleUrl": null,
    "rating":  { "value": null, "reviews": null, "source": null },
    "numbers": [ { "claim": "", "source": "" } ],
    "testimonials": [ { "text": "", "author": "", "authorized": false, "source": "" } ]
  },
  "pricing": { "publish": false, "range": null, "ctaFallback": "", "source": "" },

  // O inventário de arquivos NÃO mora aqui. Ele é o artefato de `estudo-assets`, em
  // design/inventario.json, com uma entrada por arquivo e os dois vereditos
  // (maxRenderWidth e reference). Referencie; não copie.

  // Daqui para baixo, quem escreve é `auditoria-dados` na Fase 3.
  "conflicts": [
    { "subject": "", "sourceA": "", "claimA": "", "sourceB": "", "claimB": "",
      "hypothesis": "", "cost": "", "resolved": false }
  ],

  "gaps": { "blocking": [], "important": [], "optional": [] },
  "assumptions": [ { "field": "", "assumed": "", "reason": "", "reversible": true } ],
  "unverified": [ { "item": "", "impact": "", "action": "" } ]
}
```

## Regras estruturais

1. **`facts` e `assumptions` nunca se misturam.** Um campo com `assumed: true` exige uma entrada
   correspondente em `assumptions`. O downstream pode renderizar valor com `source`; não pode
   renderizar valor `assumed` sem checagem humana.
2. **`whatsapp` e `phone` são campos separados e ambos obrigatórios.** Preencher um com o valor do
   outro é o erro mais comum e o mais caro.
3. **`conflicts[].resolved: false` trava a publicação**, não o avanço de fase. O pipeline continua;
   o `vercel --prod` não.
4. **Nada de arquivo aqui.** Dimensão, veredito de renderização e direito de uso moram em
   `design/inventario.json`, escrito por `estudo-assets`. Duplicar a lista de assets nos dois
   arquivos produz duas verdades sobre o mesmo logo.
5. **Campo não informado é `null`, nunca string vazia.** `""` parece resposta e passa despercebido
   no portão; `null` obriga uma decisão na Fase 3.

## Exemplo preenchido — Santa Lúcia

Recortado nos campos que ensinam alguma coisa. Note que `whatsapp` continua `null` mesmo com nove
assets lidos e dois documentos: ele não está em lugar nenhum, e é o CTA principal.

```json
{
  "meta": {
    "client": "santa-lucia",
    "intakeDate": "2026-08-20",
    "assetsDir": "assets-source",
    "interactive": false,
    "niche": "veterinaria"
  },
  "credentials": {
    "node": "v20.11.0", "git": "ok", "gh": "ok", "vercel": "missing",
    "imageGenerator": "chatgpt", "videoGenerator": "google-flow",
    "checkedAt": "2026-08-20"
  },
  "business": {
    "name": { "value": "Santa Lúcia Clínica Veterinária", "source": "doc:dados-google-da-empresa.txt" },
    "shortName": { "value": "Santa Lúcia", "source": "asset:logo-santa-lucia-fundo-roxo.jpg" },
    "oneLiner": { "value": "Clínica veterinária 24h com pet shop e estética animal", "source": "doc:dados-google-da-empresa.txt" },
    "city": { "value": "Caxias do Sul", "source": "doc:dados-google-da-empresa.txt" },
    "state": { "value": "RS", "source": "doc:dados-google-da-empresa.txt" },
    "areaServed": { "value": ["Caxias do Sul"], "source": "doc:dados-google-da-empresa.txt" },
    "phone": { "value": "(54) 3025-2223", "source": "doc:dados-google-da-empresa.txt + asset:fachada-clinica-rua.webp", "confirmed": false },
    "whatsapp": { "value": null, "source": null },
    "email": { "value": null, "source": null },
    "address": { "value": "R. Jacob Luchesi, 3230 — Santa Lúcia, Caxias do Sul/RS", "cep": "95032-000", "source": "doc:dados-google-da-empresa.txt" },
    "hours": { "value": "24 horas, todos os dias", "holidays": "inclusive feriados", "source": "doc:dados-google-da-empresa.txt + asset:logo-santa-lucia-fundo-roxo.jpg" },
    "founded": { "value": 2015, "source": "doc:dados-google-da-empresa.txt" },
    "cnpj": { "value": null, "source": null },
    "license": { "type": "CRMV", "value": null, "holder": null, "source": null },
    "domain": { "value": null, "status": "none", "source": "assumed" }
  },
  "services": [
    { "name": "pronto-socorro 24h", "group": "clinical", "source": "doc:conteudo-textual-site.md", "confirmed": false },
    { "name": "internação com alas separadas para cães e gatos", "group": "clinical", "source": "doc:dados-google-da-empresa.txt", "confirmed": false },
    { "name": "laboratório interno próprio", "group": "diagnostic", "source": "doc:dados-google-da-empresa.txt", "confirmed": false },
    { "name": "banho com sedação", "group": "grooming", "source": "asset:post-banho-e-tosa.jpg", "confirmed": false },
    { "name": "farmácia veterinária 24h", "group": "retail", "source": "asset:post-farmacia-completa-24h.jpg", "confirmed": false }
  ],
  "doesNotOffer": [],
  "differentiators": [
    { "claim": "Plantão 24h com equipe no local, não sobreaviso", "source": "doc:conteudo-textual-site.md", "verifiable": false },
    { "claim": "Ala felina separada: recepção, consultório e internação", "source": "asset:post-exclusividade-felina.jpg", "verifiable": true }
  ],
  "socialProof": {
    "googleUrl": null,
    "rating": { "value": "4,8", "reviews": "mais de 1.100", "source": "doc:dados-google-da-empresa.txt" },
    "numbers": [ { "claim": "mais de 70.000 pets atendidos", "source": "doc:conteudo-textual-site.md" } ],
    "testimonials": []
  },
  "pricing": { "publish": false, "range": null, "ctaFallback": "Falar no WhatsApp", "source": "assumed" },
  "conflicts": [
    {
      "subject": "atendimento a pets não convencionais",
      "sourceA": "asset:post-atendimento-24h-pets-nao-convencionais.jpg",
      "claimA": "Especialistas em pets não convencionais, atendimento 24h",
      "sourceB": "doc:conteudo-textual-site.md",
      "claimB": "Foco principal em cães e gatos; consulte-nos para outras espécies",
      "hypothesis": "O post é mais recente e mais específico; a hipótese é que o FAQ do site está desatualizado.",
      "cost": "Se a página afirmar e for falso, um tutor dirige até lá de madrugada e não é atendido.",
      "resolved": false
    },
    {
      "subject": "tempo de mercado",
      "sourceA": "doc:conteudo-textual-site.md", "claimA": "10+ anos",
      "sourceB": "doc:dados-google-da-empresa.txt", "claimB": "fundação em 2015, o que dá 11 anos em 2026",
      "hypothesis": "O texto do site não foi atualizado. Usar '10+' ou o número exato conferido com o cliente.",
      "cost": "Baixo, mas é um número na página e números errados corroem a confiança nos outros.",
      "resolved": false
    }
  ],
  "gaps": {
    "blocking": ["whatsapp", "logo-original", "conflito:pets-nao-convencionais"],
    "important": ["dominio", "email", "redes-sociais", "cnpj", "crmv", "doesNotOffer"],
    "optional": ["videos", "material-impresso", "vetos-esteticos"]
  },
  "assumptions": [
    { "field": "domain", "assumed": "santa-lucia.vercel.app", "reason": "Nenhum domínio informado.", "reversible": true },
    { "field": "pricing.publish", "assumed": false, "reason": "Nenhum preço em asset ou documento.", "reversible": true }
  ],
  "unverified": [
    { "item": "Estacionamento: o site diz 'vagas próximas'; a foto da fachada mostra vagas na calçada pública.", "impact": "médio", "action": "Ajustar a copy para não prometer estacionamento privativo." },
    { "item": "Nomes e CRMV da equipe clínica.", "impact": "alto se houver seção de equipe", "action": "Sem isso, não gerar rostos — usar foto real ou nenhuma." }
  ]
}
```

## Verificação

```bash
node -e "JSON.parse(require('fs').readFileSync('design/briefing.json','utf8')); console.log('ok')"
```

- [ ] Parseia
- [ ] Todo campo de `business` tem `source`, ou `assumed` com entrada em `assumptions`
- [ ] Campo não informado está `null`, não `""`
- [ ] `whatsapp` e `phone` têm valores diferentes, ou o dev confirmou por escrito que são o mesmo
- [ ] `credentials.checkedAt` é de hoje
- [ ] Nenhum item de `gaps.blocking` continua sem pergunta correspondente em `lacunas.md`
- [ ] Nenhum bloco de arquivo foi copiado de `inventario.json` para cá
