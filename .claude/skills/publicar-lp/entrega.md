# Entrega ao cliente e README do repositório

Última parte da Fase 12, depois da seção 7 de [SKILL.md](SKILL.md) fechada. Entra: a URL de
produção verificada. Sai: o cliente com o endereço na mão e o repositório documentado.

## A mensagem

Mande, nesta ordem: a URL, uma frase do que a página faz, e como pedir alteração.

> "O site está no ar em <URL>. O botão principal abre o WhatsApp de vocês com a mensagem já
> escrita, e o telefone disca direto no celular. Para mudar qualquer texto, preço ou horário, me
> manda um print da parte e o texto novo exatamente como deve aparecer — a alteração entra no ar
> em algumas horas. Guarda esse endereço: ele vale para o Instagram, para o Google e para o
> WhatsApp Business de vocês."

**Peça três coisas no mesmo recado** — é daí que vem o tráfego, e o cliente é a única pessoa com
acesso a esses três lugares: link na bio do Instagram, campo "site" no perfil do Google Empresas,
campo do site no WhatsApp Business.

| O cliente muda sozinho | O cliente **não** muda sozinho |
|---|---|
| Mensagem automática do WhatsApp dele | Qualquer texto, preço, foto ou seção da página |
| Perfil do Google Empresas e as fotos de lá | O número de telefone e o WhatsApp do site |
| Instagram, e o link na bio | O domínio e os endereços de e-mail |

Diga isso de forma direta e sem desculpa: a página é código, e mudança de código passa por você.
Prometer um painel de edição que não existe cria uma cobrança que você não vai conseguir honrar.

**Nunca mande:** o link do repositório como se fosse o site — o cliente clica, vê código e acha
que está quebrado —, senha de painel, nem nada com token.

## README do repositório

Ele existe para o você de daqui a seis meses, que não vai lembrar de nada.

```markdown
# lp-<slug> — <Cliente>

Landing page de <Cliente>, <cidade>. Produção: <URL>

## Rodar local
npm ci
npm run dev            # http://localhost:3000

## Regenerar imagens e vídeos
npm run assets         # lê assets-source/, escreve public/media/ e public/frames/
Os originais NÃO estão no repositório: assets-source/ (fotos do cliente) e
design/renders/ (mídia gerada na Fase 9) são pesados e ficam de fora.
Ficam em: <onde estão guardados>

## Onde mexer
src/data/site.ts       # todo o texto, telefone, endereço, horário e serviços
design/                # design system, blueprint, prompts de imagem e de motion

## Deploy
Push em `main` publica em produção sozinho (Vercel, projeto <nome-do-projeto>).

## Dados de contato
Confirmados por <nome do contato no cliente> em <data>.
Mudou algum? Atualize src/data/site.ts e confira o JSON-LD em app/page.tsx.
```

Sem token, sem CPF, sem contrato, sem tabela de preço interna — o README é público junto com o
resto, e continua público se o repositório virar portfólio depois.
