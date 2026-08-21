# CTA e FAQ — implementação

Complemento de [SKILL.md](SKILL.md). A decisão de *quantos* CTAs e *onde* está lá; aqui está o
código que a executa.

## Barra sticky de WhatsApp

Aparece a partir do share definido pelo arquétipo e some no capítulo de convite.

```tsx
// src/hooks/useStickyCta.ts
import { useLayoutEffect, useState } from 'react'

import { gsap, ScrollTrigger } from '../lib/gsap'

/**
 * `gsap.context()` porque o Strict Mode do React 19 monta duas vezes — sem o
 * revert, o ScrollTrigger fica registrado em dobro e o toggle pisca.
 */
export function useStickyCta(revealAt = 0.3, finaleId = 'agendar'): boolean {
  const [visible, setVisible] = useState(false)

  useLayoutEffect(() => {
    // Resolvido uma vez, não a cada frame de scroll: os filhos já estão no DOM
    // quando o layout effect do pai roda.
    const finale = document.getElementById(finaleId)

    const ctx = gsap.context(() => {
      ScrollTrigger.create({
        trigger: document.documentElement,
        start: 'top top',
        end: 'max',
        onUpdate: (self) => {
          const finaleOnScreen = finale
            ? finale.getBoundingClientRect().top < window.innerHeight * 0.8
            : false
          // Chamar setVisible com o mesmo booleano não re-renderiza: o React
          // compara por Object.is e desiste. Não precisa de throttle.
          setVisible(self.progress >= revealAt && !finaleOnScreen)
        },
      })
    })

    return () => ctx.revert()
  }, [revealAt, finaleId])

  return visible
}
```

```tsx
// src/components/ui/StickyCta.tsx
import { whatsappHref } from '../../data/cta'
import { useStickyCta } from '../../hooks/useStickyCta'
import { WhatsAppGlyph } from './Glyphs'

export function StickyCta({ revealAt = 0.3 }: { revealAt?: number }) {
  const visible = useStickyCta(revealAt)

  return (
    <div
      /*
         `inert` (booleano nativo no React 19) tira a barra invisível do Tab e da
         árvore de acessibilidade. `opacity: 0` sozinho deixa um botão focável
         que ninguém vê — o visitante de teclado pousa no nada.
       */
      inert={!visible}
      className={[
        'fixed inset-x-0 bottom-0 z-40 px-4 lg:hidden',
        // A barra tem que ficar acima da home bar do iPhone, senão o toque cai nela.
        'pb-[max(1rem,env(safe-area-inset-bottom))]',
        'transition-[opacity,transform] duration-300 ease-out-expo motion-reduce:transition-none',
        visible ? 'translate-y-0 opacity-100' : 'pointer-events-none translate-y-4 opacity-0',
      ].join(' ')}
    >
      <a
        href={whatsappHref('sticky')}
        target="_blank"
        rel="noopener noreferrer"
        aria-label="Falar com a clínica pelo WhatsApp"
        className="bg-ink text-white shadow-lift flex min-h-14 items-center justify-center gap-2 rounded-pill text-[0.9375rem] font-medium"
      >
        {/* Sem `aria-hidden` aqui: `GlyphProps` só aceita `className` e o glifo já
            põe o atributo dentro do próprio SVG. Atributo com hífen escapa da
            checagem de excesso do TypeScript — escrevê-lo compila e some em
            silêncio, que é pior do que quebrar. */}
        <WhatsAppGlyph className="size-4.5" />
        Falar no WhatsApp
      </a>
    </div>
  )
}
```

`lg:hidden` não é estilo, é a regra: em ≥ 1024px o header já está sempre visível e uma segunda
barra pede a mesma coisa duas vezes.

## Pareamento de variantes

`src/components/ui/Action.tsx` já tem as três variantes. A escolha depende do fundo, não do
gosto:

| Fundo do capítulo | Primário | Secundário |
|---|---|---|
| Claro (`canvas`, `canvas-soft`) | `solid` — campo de ink, tipo branca | `outline` |
| Escuro (`ink`) ou sobre filme | `inverse` — campo branco, tipo ink | `outline` |
| Sobre foto clara | `solid` + `Scrim` atrás | `outline` só com scrim |

Nunca dois `solid` na mesma tela. A variante é o que faz a escolha acontecer sem pensar.

## FAQ

```ts
// src/data/faq.ts
export interface FaqItem {
  readonly q: string  // ≤ 70 caracteres
  readonly a: string  // ≤ 400 caracteres, primeira frase responde
}

export const faq: readonly FaqItem[] = [
  {
    q: 'Quanto custa a avaliação?',
    a: 'A avaliação inicial é sem custo. Nela conversamos sobre o que você quer, analisamos suas características e apresentamos o protocolo com o valor fechado antes de qualquer procedimento.',
  },
  // …objeção mais cara primeiro, no máximo 7 itens.
]
```

```tsx
// src/components/ui/Faq.tsx
import { Plus } from 'lucide-react'

import type { FaqItem } from '../../data/faq'

/**
 * `<details>` nativo: o teclado já funciona e a resposta **continua no DOM com o
 * item fechado** — Ctrl+F acha, o leitor de tela acha, o Googlebot acha ao
 * renderizar. Atrás de `{open && …}` ela não existe enquanto está fechada.
 *
 * `name="faq"` faz o grupo virar acordeão exclusivo sem uma linha de script. Em
 * navegador antigo os itens simplesmente abrem de forma independente, que é uma
 * degradação inofensiva.
 */
export function Faq({ items }: { items: readonly FaqItem[] }) {
  return (
    <div className="mx-auto w-full max-w-2xl">
      {items.map((item, i) => (
        <details
          key={item.q}
          name="faq"
          open={i === 0}
          className="border-line group border-b"
        >
          <summary className="flex min-h-14 cursor-pointer list-none items-center justify-between gap-6 py-5 text-left [&::-webkit-details-marker]:hidden">
            <h3 className="font-sans text-[1.0625rem] leading-snug font-medium">{item.q}</h3>
            <Plus
              className="size-4 shrink-0 transition-transform duration-200 group-open:rotate-45 motion-reduce:transition-none"
              aria-hidden="true"
            />
          </summary>
          <p className="text-body-soft pb-6 text-[0.9375rem] leading-relaxed">{item.a}</p>
        </details>
      ))}
    </div>
  )
}
```

O `<h3>` dentro do `<summary>` mantém o FAQ na navegação por cabeçalhos — é assim que uma
landing longa é lida sem ver a tela. O capítulo que contém o FAQ já usa `<h2>`.

## JSON-LD

Só marque `FAQPage` se as respostas estiverem na página, palavra por palavra. Marcação que não
corresponde ao conteúdo visível é penalizada.

```tsx
// Dentro do capítulo que renderiza o FAQ.
import { faq } from '../../data/faq'

const faqJsonLd = {
  '@context': 'https://schema.org',
  '@type': 'FAQPage',
  mainEntity: faq.map(({ q, a }) => ({
    '@type': 'Question',
    name: q,
    acceptedAnswer: { '@type': 'Answer', text: a },
  })),
}

<script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(faqJsonLd) }} />
```

Para serviço local o `LocalBusiness` pesa mais que o `FAQPage`: ele é o que faz o telefone, o
endereço e o horário aparecerem no resultado de busca. No projeto de referência ele **já existe
e não fica em componente nenhum** — é um bloco `application/ld+json` escrito à mão no `<head>`
de `index.html`, com `@type: "BeautySalon"` (subtipo de `LocalBusiness`, mais específico) e
`openingHoursSpecification`. Ao mudar horário ou endereço, edite os dois lados: `index.html` e
`openingHours` de `src/data/site.ts`, que carrega os mesmos tokens (`Monday`, `opens`,
`closes`) para a UI. Divergência entre os dois é o Google anunciando um horário que a página
desmente.

## Origem da mensagem — taxonomia completa

Uma origem por posição de CTA, nunca uma genérica reaproveitada. O valor está em o atendente
saber qual objeção já caiu.

| Origem | Onde | O que o atendente aprende |
|---|---|---|
| `header` | Barra fixa | Decidiu cedo; ainda não leu nada. Precisa de contexto |
| `hero` | Primeira tela | Veio pela promessa. Confirmar que o serviço é aquele |
| `method` | Fim do capítulo de processo | Já sabe como funciona. Ir direto para agenda |
| `proof` | Depois do antes/depois | Está comparando com o próprio caso. Perguntar o caso |
| `faq` | Fim do FAQ | Tem uma dúvida que a página não cobriu. Ouvir primeiro |
| `finale` | Capítulo de convite | Leu tudo. Menor atrito, maior taxa de fechamento |
| `sticky` | Barra mobile | Pode estar em qualquer ponto. Perguntar o que precisa |

Se uma origem nunca chega em três meses, o CTA daquela posição não está sendo visto — mova ou
remova. Sem formulário, esse é o único sinal de funil disponível.
