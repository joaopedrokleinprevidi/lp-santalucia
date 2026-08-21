# Cache imutável e domínio

Detalhe da seção 5 de [SKILL.md](SKILL.md). Entra: o projeto já no ar numa URL da Vercel. Sai: a
segunda visita sem request de mídia, e o domínio do cliente resolvendo com HTTPS.

## `vercel.json`

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": "nextjs",
  "headers": [
    {
      "source": "/frames/(.*)",
      "headers": [{ "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }]
    },
    {
      "source": "/media/(.*)\\.(mp4|avif|webp)",
      "headers": [{ "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }]
    }
  ]
}
```

O arquivo completo — com os headers de segurança e a regra curta para `/media/*.jpg` — está em
[stack.md](../landing-page-factory/stack.md) §8. Declare headers **num lugar só**: aqui ou em
`headers()` no `next.config.ts`. A mesma `source` nos dois é a origem de "mudei o cache e nada
mudou".

**Por que isso pesa na segunda visita.** `/frames/` e `/media/` são versionados por nome: o
pipeline põe o hash do conteúdo no caminho, então aquele arquivo naquele endereço nunca muda. Sem
`immutable`, o navegador guarda o arquivo e ainda pergunta "mudou?" a cada visita — um
`304 Not Modified` por arquivo. Numa sequência de canvas são 150 a 300 arquivos: 300 idas e voltas
antes do primeiro frame, com o 4G cobrando latência em cada uma. Com `immutable`, a segunda visita
não faz **nenhum** desses requests. Os bytes são os mesmos; o que muda é o tempo.

`immutable` só é seguro em caminho com hash. Nunca em `/`, em `/sitemap.xml`, em `/robots.txt` nem
em HTML — você congelaria a página na máquina do visitante e a correção de amanhã não chegaria
nela. Contrapartida da regra: para trocar uma imagem, troque o nome do arquivo; o pipeline já faz
isso ao recalcular o hash.

```bash
curl -I https://<url>/media/<arquivo>.mp4 | grep -i cache-control   # espera immutable
curl -I https://<url> | grep -i cache-control                       # NÃO pode ter immutable
```

## Domínio

**Cliente sem domínio:** `lp-<slug>.vercel.app` já é o site final — gratuito, com HTTPS, indexa no
Google e ganha um domínio depois **sem refazer nada**, porque o domínio é só mais uma entrada
apontando para o mesmo projeto. Não segure a entrega por isso.

**Cliente com domínio:** painel da Vercel → projeto → Settings → Domains → Add. A Vercel imprime os
registros exatos; copie de lá e **não decore IP** — o endereço recomendado de apex já mudou mais de
uma vez, e um IP escrito de memória gera o domínio que "não propaga nunca".

| Entrada | Tipo | Valor |
|---|---|---|
| `cliente.com.br` (apex) | `A` | o IP que o painel imprimir |
| `www.cliente.com.br` | `CNAME` | o alvo que o painel imprimir (`cname.vercel-dns.com` é o clássico) |

Onde configurar: no registrador do cliente — Registro.br, GoDaddy, Hostgator, Locaweb. Precisa do
login dele, e essa costuma ser a senha que ninguém acha: peça no **início** da Fase 12, não no fim.
Escolha **um** endereço principal (com `www` ou sem) e deixe a Vercel redirecionar o outro;
`NEXT_PUBLIC_SITE_URL`, canonical, sitemap e `og:url` apontam todos para ele.

```bash
nslookup cliente.com.br 8.8.8.8        # resolve pelo DNS do Google
nslookup www.cliente.com.br 8.8.8.8
```

O que dizer ao cliente enquanto propaga: o site já está no ar no `.vercel.app`, o domínio começa a
funcionar em algumas horas e pode levar até um dia porque cada provedor de internet atualiza no seu
tempo, e se abrir o site antigo é a memória do computador dele — teste pelo 4G do celular. Enquanto
os dois endereços não resolverem em duas redes diferentes, ninguém imprime o domínio em panfleto
nem coloca em anúncio pago.

No dia em que o domínio entrar: cadastre `NEXT_PUBLIC_SITE_URL` com ele e **faça redeploy**. Sem o
redeploy, sitemap e canonical seguem apontando para o `.vercel.app` e o Google trata o domínio novo
como cópia do próprio site.
